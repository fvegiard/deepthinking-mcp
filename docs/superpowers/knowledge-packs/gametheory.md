# Knowledge Pack: gametheory

**Mode**: `gametheory`
**Category skill**: `think-strategic`
**Source doc**: `docs/modes/GAMETHEORY.md`

## Overview

Game Theory mode provides **strategic analysis** using payoff matrices, Nash equilibria, dominant strategies, and game trees. Phase 11 extends this with **von Neumann's cooperative game theory** including minimax theorem, Shapley values, and coalition analysis.

This mode captures the structure of game-theoretic reasoning - from game definition through strategy analysis to equilibrium finding.

## When to Use

Use game theory mode when you need to:

- **Model strategic interactions** - Multiple agents with conflicting interests
- **Find equilibria** - Nash equilibria, dominant strategy equilibria
- **Analyze coalitions** - Cooperative games, fair division
- **Compute fair allocations** - Shapley value, nucleolus
- **Analyze zero-sum games** - Minimax strategies

## Core Concepts

(no 'core concepts' section in source doc)

## Thought Type (TypeScript Interface)

```typescript
interface GameTheoryThought extends BaseThought {
mode: ThinkingMode.GAMETHEORY;
  thoughtType:
    | 'game_definition'
    | 'strategy_analysis'
    | 'equilibrium_finding'
    | 'payoff_computation'
    | 'dominance_analysis'
    // Phase 11: Von Neumann extensions
    | 'minimax_analysis'        // Von Neumann's minimax theorem
    | 'cooperative_analysis'    // Cooperative game theory
    | 'coalition_formation'     // Coalition analysis
    | 'shapley_value'          // Fair allocation computation
    | 'core_analysis';         // Core stability analysis

  game?: Game;
  players?: Player[];
  strategies?: Strategy[];
  payoffMatrix?: PayoffMatrix;
  nashEquilibria?: NashEquilibrium[];
  dominantStrategies?: DominantStrategy[];
  gameTree?: GameTree;

  // Phase 11: Von Neumann extensions
  minimaxAnalysis?: MinimaxAnalysis;
  cooperativeGame?: CooperativeGame;
  coalitionAnalysis?: CoalitionAnalysis;
}
```

## Supporting Interfaces

```typescript
interface Game {
id: string;
  name: string;
  description: string;
  type: 'normal_form' | 'extensive_form' | 'cooperative' | 'non_cooperative';
  numPlayers: number;
  isZeroSum: boolean;
  isPerfectInformation: boolean;
}

interface Player {
id: string;
  name: string;
  role?: string;
  isRational: boolean;
  availableStrategies: string[]; // Strategy IDs
}

interface Strategy {
id: string;
  playerId: string;
  name: string;
  description: string;
  isPure: boolean; // true for pure strategy, false for mixed
  probability?: number; // for mixed strategies (0-1)
}

interface PayoffMatrix {
players: string[]; // Player IDs in order
  dimensions: number[]; // Number of strategies per player
  payoffs: PayoffEntry[];
  latex?: string; // LaTeX representation of the payoff matrix
}

interface PayoffEntry {
strategyProfile: string[]; // Strategy IDs for each player
  payoffs: number[]; // Payoff for each player
}

interface NashEquilibrium {
id: string;
  strategyProfile: string[]; // Strategy IDs for each player
  payoffs: number[]; // Resulting payoffs
  type: 'pure' | 'mixed';
  isStrict: boolean; // No player wants to deviate
  stability: number; // 0-1, how stable is this equilibrium
  formula?: string; // LaTeX formula showing equilibrium conditions
}

interface DominantStrategy {
playerId: string;
  strategyId: string;
  type: 'strictly_dominant' | 'weakly_dominant';
  dominatesStrategies: string[]; // Other strategy IDs this dominates
  justification: string;
  formula?: string; // LaTeX formula showing dominance relationship
}

interface GameTree {
rootNode: string; // Node ID
  nodes: GameNode[];
  informationSets?: InformationSet[];
}

```

## Worked Example (from source doc)

```typescript
// Game definition
const game = await deepthinking_strategic({
  mode: 'gametheory',
  thought: 'Define the Prisoner\'s Dilemma game',
  thoughtNumber: 1,
  totalThoughts: 5,
  nextThoughtNeeded: true,

  thoughtType: 'game_definition',
  game: {
    id: 'prisoners_dilemma',
    name: 'Prisoner\'s Dilemma',
    description: 'Classic game of cooperation vs defection',
    type: 'normal_form',
    numPlayers: 2,
    isZeroSum: false,
    isPerfectInformation: true
  },
  players: [
    { id: 'p1', name: 'Player 1', isRational: true, availableStrategies: ['cooperate', 'defect'] },
    { id: 'p2', name: 'Player 2', isRational: true, availableStrategies: ['cooperate', 'defect'] }
  ],
  strategies: [
    { id: 'cooperate', playerId: 'p1', name: 'Cooperate', description: 'Stay silent', isPure: true },
    { id: 'defect', playerId: 'p1', name: 'Defect', description: 'Betray partner', isPure: true }
  ]
});

// Payoff matrix
const payoffs = await deepthinking_strategic({
  mode: 'gametheory',
  thought: 'Define payoff matrix',
  thoughtNumber: 2,
  totalThoughts: 5,
  nextThoughtNeeded: true,

  thoughtType: 'payoff_computation',
  payoffMatrix: {
    players: ['p1', 'p2'],
    dimensions: [2, 2],
    payoffs: [
      { strategyProfile: ['cooperate', 'cooperate'], payoffs: [-1, -1] },
      { strategyProfile: ['cooperate', 'defect'], payoffs: [-3, 0] },
      { strategyProfile: ['defect', 'cooperate'], payoffs: [0, -3] },
      { strategyProfile: ['defect', 'defect'], payoffs: [-2, -2] }
    ],
    latex: '\\begin{pmatrix} (-1,-1) & (-3,0) \\\\ (0,-3) & (-2,-2) \\end{pmatrix}'
  }
});

// Dominance analysis
const dominance = await deepthinking_strategic({
  mode: 'gametheory',
  thought: 'Identify dominant strategies',
  thoughtNumber: 3,
  totalThoughts: 5,
  nextThoughtNeeded: true,

  thoughtType: 'dominance_analysis',
  dominantStrategies: [
    {
      playerId: 'p1',
      strategyId: 'defect',
      type: 'strictly_dominant',
      dominatesStrategies: ['cooperate'],
      justification: 'Defect yields higher payoff regardless of opponent\'s choice',
      formula: 'u_1(D, s_2) > u_1(C, s_2) \\forall s_2'
    },
    {
      playerId: 'p2',
      strategyId: 'defect',
      type: 'strictly_dominant',
      dominatesStrategies: ['cooperate'],
      justification: 'Symmetric game - defect dominates for P2 as well'
    }
  ]
});

// Nash equilibrium
const equilibrium = await deepthinking_strategic({
  mode: 'gametheory',
  thought: 'Find Nash equi

---

*Generated by RLM extraction pass on 2026-04-12*
