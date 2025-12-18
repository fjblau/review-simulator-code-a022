# Technical Specification: Kessler Simulation React Game

## Task Difficulty Assessment

**Complexity Level**: **HARD**

**Reasoning**:
- Complex game state with multiple interdependent systems
- Real-time collision detection with O(n²) complexity
- Multiple game mechanics (budget, insurance, missions, events)
- Turn-based logic with cascading effects
- Requires sophisticated state management
- Performance optimization needed for 100+ objects
- Rich UI/UX requirements

---

## Technical Context

### Source Material
- **File**: `kessler.ipynb` (Jupyter notebook, 997 lines)
- **Language**: Python (numpy, matplotlib, ipywidgets)
- **Primary Game Cell**: #4a (lines 498-802)
- **Game Type**: Turn-based strategy simulation

### Target Technology Stack

#### Core
- **Framework**: React 18.x
- **Language**: TypeScript 5.x
- **State Management**: Redux Toolkit or Zustand
- **Build Tool**: Vite

#### Visualization
- **2D Graphics**: HTML5 Canvas API or Konva.js
- **Charts**: Recharts or Victory
- **Animations**: Framer Motion

#### Supporting Libraries
- **Random Number Generation**: seedrandom (for deterministic gameplay)
- **Testing**: Vitest + React Testing Library
- **Linting**: ESLint + TypeScript ESLint
- **Styling**: Tailwind CSS

### Dependencies
```json
{
  "react": "^18.3.0",
  "react-dom": "^18.3.0",
  "typescript": "^5.6.0",
  "@reduxjs/toolkit": "^2.5.0",
  "react-redux": "^9.2.0",
  "framer-motion": "^11.15.0",
  "recharts": "^2.15.0",
  "seedrandom": "^3.0.5"
}
```

---

## Implementation Approach

### Architecture Pattern

**Model-View-Controller (MVC) with Unidirectional Data Flow**

```
User Action → Dispatcher → Reducers → State Update → Component Re-render → UI Update
```

### State Management Strategy

Use Redux Toolkit with the following slice structure:

1. **gameSlice** - Core game state (satellites, debris, step counter)
2. **economySlice** - Budget, costs, insurance tracking
3. **missionsSlice** - Mission definitions, progress, completion
4. **metricsSlice** - Historical data, charts, statistics
5. **uiSlice** - UI state (selected orbit, view mode, modals)

### Core Game Loop

**Turn Execution Flow**:
```
1. Player makes launch decision
2. Validate budget availability
3. Create satellite if affordable
4. Age all satellites (increment age counter)
5. Remove expired LEO satellites
6. Detect collisions (O(n²) within each layer)
7. Resolve collisions (create debris, destroy satellites)
8. Process insurance payouts
9. Trigger random events (solar storms)
10. Update metrics and risk level
11. Check mission completion
12. Check end game conditions
13. Advance step counter
```

### Collision Detection Algorithm

**Optimization**: Spatial partitioning by orbital layer

```typescript
function detectCollisions(satellites: Satellite[]): Collision[] {
  const collisions: Collision[] = [];
  const byLayer = groupByLayer(satellites);
  
  for (const layer of ['LEO', 'MEO', 'GEO']) {
    const sats = byLayer[layer] || [];
    const threshold = COLLISION_THRESHOLDS[layer];
    
    for (let i = 0; i < sats.length; i++) {
      for (let j = i + 1; j < sats.length; j++) {
        const distance = calculateDistance(sats[i], sats[j]);
        if (distance < threshold) {
          collisions.push({ sat1: sats[i], sat2: sats[j], layer });
        }
      }
    }
  }
  
  return collisions;
}
```

**Complexity**: O(n²) worst case, O(n²/3) average case with layer partitioning

---

## Source Code Structure

### New Files to Create

```
src/
├── main.tsx                          # App entry point
├── App.tsx                           # Root component
├── store/
│   ├── index.ts                      # Store configuration
│   ├── slices/
│   │   ├── gameSlice.ts              # Core game state
│   │   ├── economySlice.ts           # Budget & costs
│   │   ├── missionsSlice.ts          # Missions system
│   │   ├── metricsSlice.ts           # Historical data
│   │   └── uiSlice.ts                # UI state
│   └── hooks.ts                      # Typed hooks
├── components/
│   ├── GameBoard/
│   │   ├── GameBoard.tsx             # Main game visualization
│   │   ├── OrbitVisualization.tsx    # Canvas rendering
│   │   ├── SatelliteSprite.tsx       # Satellite rendering
│   │   └── DebrisParticle.tsx        # Debris rendering
│   ├── ControlPanel/
│   │   ├── ControlPanel.tsx          # Launch controls
│   │   ├── OrbitSelector.tsx         # LEO/MEO/GEO buttons
│   │   ├── InsuranceToggle.tsx       # Insurance checkbox
│   │   └── StatusDisplay.tsx         # Metrics display
│   ├── MissionPanel/
│   │   ├── MissionPanel.tsx          # Mission list
│   │   ├── MissionCard.tsx           # Individual mission
│   │   └── MissionProgress.tsx       # Progress indicator
│   ├── Charts/
│   │   ├── DebrisChart.tsx           # Debris over time
│   │   └── SatelliteChart.tsx        # Satellites over time
│   └── EventLog/
│       ├── EventLog.tsx              # Game events display
│       └── EventItem.tsx             # Single event entry
├── game/
│   ├── engine/
│   │   ├── collision.ts              # Collision detection
│   │   ├── simulation.ts             # Turn processing
│   │   ├── events.ts                 # Random events
│   │   └── missions.ts               # Mission logic
│   ├── constants.ts                  # Game configuration
│   ├── types.ts                      # TypeScript types
│   └── utils.ts                      # Helper functions
├── hooks/
│   ├── useGameLoop.ts                # Game loop hook
│   ├── useCollisionDetection.ts      # Collision hook
│   └── useMissionCheck.ts            # Mission checking
└── assets/
    ├── sounds/
    │   ├── launch.mp3
    │   ├── collision.mp3
    │   └── alert.mp3
    └── images/
        ├── satellite-gps.svg
        ├── satellite-comms.svg
        └── satellite-weather.svg
```

### Configuration Files

```
.
├── package.json                      # Dependencies
├── tsconfig.json                     # TypeScript config
├── vite.config.ts                    # Vite config
├── tailwind.config.js                # Tailwind CSS
├── .eslintrc.json                    # ESLint rules
└── vitest.config.ts                  # Test config
```

---

## Data Model / API / Interface Changes

### Core TypeScript Types

```typescript
// Game constants
type OrbitLayer = 'LEO' | 'MEO' | 'GEO';
type SatelliteType = 'Weather' | 'Comms' | 'GPS';
type RiskLevel = 'LOW' | 'MEDIUM' | 'CRITICAL';
type MissionId = string;

// Core entities
interface Satellite {
  id: string;
  x: number;
  y: number;
  layer: OrbitLayer;
  purpose: SatelliteType;
  age: number;
  insured: boolean;
}

interface Debris {
  id: string;
  x: number;
  y: number;
  layer: OrbitLayer;
}

interface Mission {
  id: MissionId;
  name: string;
  description: string;
  check: (state: GameState) => boolean;
  completed: boolean;
}

// Game state
interface GameState {
  // Simulation
  step: number;
  maxSteps: number;
  gameOver: boolean;
  
  // Assets
  satellites: Satellite[];
  debris: Debris[];
  
  // Economy
  budget: number;
  
  // Missions
  activeMissions: Mission[];
  completedMissions: MissionId[];
  
  // Metrics
  debrisHistory: number[];
  satelliteHistory: number[];
  
  // Trackers
  gpsLaunched: number;
  riskStreak: number;
  insurancePayouts: number;
  expiredLeoSats: number;
  cascadeTriggered: boolean;
  launchHistory: (OrbitLayer | 'none')[];
  launchedLayers: Set<OrbitLayer>;
  purposeCounts: Record<SatelliteType, number>;
}

// Actions
interface LaunchAction {
  orbit: OrbitLayer;
  insured: boolean;
}

interface CollisionResult {
  destroyedSatellites: string[];
  newDebris: Debris[];
  cascadeTriggered: boolean;
}

// Configuration
interface GameConfig {
  collisionThresholds: Record<OrbitLayer, number>;
  layerBounds: Record<OrbitLayer, [number, number]>;
  launchCosts: Record<OrbitLayer, number>;
  insuranceCost: number;
  insuranceRefund: number;
  startingBudget: number;
  debrisPerCollision: number;
  maxDebrisLimit: number;
  maxSteps: number;
  leoLifetime: number;
  solarActivityChance: number;
  debrisDecayRate: number;
  cascadeThreshold: number;
}
```

### Redux Actions

```typescript
// Game actions
const gameActions = {
  launchSatellite: (orbit: OrbitLayer, insured: boolean) => {},
  skipTurn: () => {},
  ageSatellites: () => {},
  processCollisions: () => {},
  triggerSolarEvent: () => {},
  updateMetrics: () => {},
  checkMissions: () => {},
  endGame: () => {},
  resetGame: () => {},
};

// UI actions
const uiActions = {
  selectOrbit: (orbit: OrbitLayer | null) => {},
  toggleInsurance: () => {},
  setViewMode: (mode: '2D' | '3D') => {},
  toggleEventLog: () => {},
  openMissionDetails: (missionId: MissionId) => {},
};
```

### API Contracts

No external API calls required. All game logic runs client-side.

---

## Verification Approach

### Testing Strategy

#### Unit Tests
- **Collision detection**: Test all edge cases (same layer, different layers, exact threshold)
- **Position generation**: Verify positions within layer bounds
- **Budget calculations**: Test launch costs, insurance, payouts
- **Mission logic**: Test each mission's completion condition
- **Satellite aging**: Test expiration timing

#### Integration Tests
- **Full turn simulation**: Execute complete turn and verify state changes
- **Multi-turn scenarios**: Test cascading effects over multiple turns
- **Mission completion paths**: Verify mission unlock and completion
- **Edge cases**: Zero budget, max debris, all satellites destroyed

#### E2E Tests
- **Complete game playthrough**: Start to finish simulation
- **UI interactions**: Button clicks, orbit selection, insurance toggle
- **Save/load**: Persist and restore game state

### Manual Verification Steps

1. **Launch satellites** into each orbit (LEO, MEO, GEO)
2. **Verify collision detection** by launching multiple satellites in same orbit
3. **Test insurance system** by insuring satellites and observing payouts
4. **Trigger solar event** and verify debris reduction
5. **Complete missions** and verify checkmark display
6. **Reach max debris** and verify game end
7. **Test budget constraints** by attempting to launch without funds
8. **Verify LEO expiration** after 20 turns
9. **Check charts update** with each turn
10. **Test responsive layout** on different screen sizes

### Lint and Type Check Commands

```bash
# Type checking
npm run typecheck

# Linting
npm run lint

# Tests
npm run test
npm run test:coverage

# Build verification
npm run build
npm run preview
```

### Success Criteria

- ✅ All unit tests pass
- ✅ No TypeScript errors
- ✅ No ESLint errors
- ✅ Code coverage >80%
- ✅ Build completes without errors
- ✅ Game playable start to finish
- ✅ All missions achievable
- ✅ UI responsive on mobile and desktop
- ✅ Performance: 60fps with 100+ objects

---

## Implementation Plan

Given the **HARD** complexity, breaking down into concrete milestones:

### Milestone 1: Core Game Engine (1-2 weeks)
**Files**: `game/engine/`, `store/slices/gameSlice.ts`, `game/types.ts`

**Tasks**:
- [ ] Set up TypeScript types and interfaces
- [ ] Implement position generation
- [ ] Implement collision detection algorithm
- [ ] Create game state reducer
- [ ] Write unit tests for collision detection
- [ ] Implement turn processing logic

**Verification**: Unit tests pass, can simulate turns programmatically

---

### Milestone 2: Economic System (3-4 days)
**Files**: `store/slices/economySlice.ts`, `game/engine/simulation.ts`

**Tasks**:
- [ ] Implement budget tracking
- [ ] Add launch cost deduction
- [ ] Implement insurance system (purchase + payout)
- [ ] Add budget validation
- [ ] Write tests for economic calculations

**Verification**: Can launch satellites with budget constraints, insurance payouts work

---

### Milestone 3: Basic Visualization (1 week)
**Files**: `components/GameBoard/`, `components/ControlPanel/`

**Tasks**:
- [ ] Create Canvas-based orbital visualization
- [ ] Render concentric orbit circles
- [ ] Display satellites as sprites
- [ ] Display debris as particles
- [ ] Add orbit selector buttons
- [ ] Add turn advance button
- [ ] Display budget and metrics

**Verification**: Visual game playable with basic UI

---

### Milestone 4: Mission System (4-5 days)
**Files**: `store/slices/missionsSlice.ts`, `game/engine/missions.ts`, `components/MissionPanel/`

**Tasks**:
- [ ] Implement all 13 mission checks
- [ ] Create mission selection logic (random 3)
- [ ] Build mission display UI
- [ ] Add mission progress tracking
- [ ] Write tests for mission completion
- [ ] Display mission status in UI

**Verification**: Missions can be completed and tracked

---

### Milestone 5: Random Events & Polish (3-4 days)
**Files**: `game/engine/events.ts`, `components/EventLog/`

**Tasks**:
- [ ] Implement solar storm event
- [ ] Add cascade detection
- [ ] Create event log component
- [ ] Add collision animations
- [ ] Implement sound effects
- [ ] Add risk level indicators

**Verification**: Events trigger correctly, UI feedback clear

---

### Milestone 6: Data Visualization (3-4 days)
**Files**: `components/Charts/`, `store/slices/metricsSlice.ts`

**Tasks**:
- [ ] Implement debris time series chart
- [ ] Implement satellite count chart
- [ ] Add real-time chart updates
- [ ] Display risk trend indicator
- [ ] Show budget projection

**Verification**: Charts display correctly and update each turn

---

### Milestone 7: Game Flow & UX (4-5 days)
**Files**: `App.tsx`, `components/`, `hooks/`

**Tasks**:
- [ ] Add game start screen
- [ ] Implement game over screen with score
- [ ] Add tutorial/instructions modal
- [ ] Implement save/load state
- [ ] Add undo last turn
- [ ] Polish animations and transitions
- [ ] Add responsive layout

**Verification**: Complete game flow from start to end

---

### Milestone 8: Testing & Optimization (3-4 days)
**Files**: All test files, performance optimizations

**Tasks**:
- [ ] Write integration tests
- [ ] Add E2E tests
- [ ] Performance profiling
- [ ] Optimize collision detection
- [ ] Add Web Worker for physics
- [ ] Final bug fixes
- [ ] Cross-browser testing

**Verification**: All tests pass, performance benchmarks met

---

## Estimated Timeline

**Total**: 6-8 weeks for fully-featured game

**Breakdown**:
- Core Engine: 1-2 weeks
- Economic System: 3-4 days
- Basic Visualization: 1 week
- Mission System: 4-5 days
- Events & Polish: 3-4 days
- Data Visualization: 3-4 days
- Game Flow & UX: 4-5 days
- Testing & Optimization: 3-4 days

---

## Risk Assessment

### Technical Risks

1. **Performance with 100+ objects**
   - **Mitigation**: Spatial partitioning, Web Workers, RAF optimization

2. **Mobile responsiveness**
   - **Mitigation**: Touch-optimized controls, responsive breakpoints

3. **Game balance**
   - **Mitigation**: Playtesting, adjustable difficulty constants

4. **Browser compatibility**
   - **Mitigation**: Canvas API widely supported, polyfills if needed

### Project Risks

1. **Scope creep**
   - **Mitigation**: Stick to milestones, defer nice-to-haves

2. **Time estimation**
   - **Mitigation**: Buffer time in estimates, prioritize MVP

---

## Success Metrics

- ✅ Game is playable and engaging
- ✅ All 13 missions achievable
- ✅ Performance >60fps
- ✅ Mobile responsive
- ✅ Code coverage >80%
- ✅ Zero critical bugs
- ✅ Positive playtester feedback

---

**Specification Created**: December 18, 2025  
**Source**: kessler.ipynb (Cell #4a)  
**Recommended**: Proceed with implementation following milestone plan
