# Knowledge Pack: mathematics

**Mode**: `mathematics`
**Category skill**: `think-mathematics`
**Source doc**: `docs/modes/MATHEMATICS.md`

## Overview

Mathematics mode provides **formal mathematical reasoning** with support for proofs, theorems, symbolic computation, and rigorous logical deduction. It's designed for problems requiring mathematical precision, including **proof decomposition** and **inconsistency detection** (Phase 8 extensions).

This mode captures the structure of mathematical reasoning - from axiom definitions through proof construction to final conclusions.

## When to Use

Use mathematics mode when you need to:

- **Construct formal proofs** - Rigorous mathematical arguments
- **Define and verify theorems** - State and prove mathematical claims
- **Perform symbolic computation** - Algebraic manipulations, simplifications
- **Analyze proof structure** - Find gaps, check consistency
- **Work with mathematical models** - Create and manipulate models

## Core Concepts

(no 'core concepts' section in source doc)

## Thought Type (TypeScript Interface)

```typescript
interface MathematicsThought extends BaseThought {
mode: ThinkingMode.MATHEMATICS;
  thoughtType: MathematicsThoughtType;
  mathematicalModel?: MathematicalModel;
  proofStrategy?: ProofStrategy;
  theorems?: Theorem[];
  dependencies: string[];
  assumptions: string[];
  uncertainty: number; // 0-1
  logicalForm?: LogicalForm;
  references?: Reference[];

  // Phase 8: Proof Decomposition Fields (populated by MathematicsReasoningEngine)
  decomposition?: ProofDecomposition;
  consistencyReport?: ConsistencyReport;
  gapAnalysis?: GapAnalysis;
  assumptionAnalysis?: AssumptionAnalysis;
}
```

## Supporting Interfaces

```typescript
interface AtomicStatement {
id: string;
  statement: string;
  latex?: string;
  type: 'axiom' | 'definition' | 'hypothesis' | 'lemma' | 'derived' | 'conclusion';

  // Provenance
  justification?: string;
  derivedFrom?: string[]; // IDs of statements this depends on
  usedInferenceRule?: InferenceRule;

  // Metadata
  confidence: number; // 0-1, how certain is this statement
  isExplicit: boolean; // Was this explicitly stated or inferred?
  sourceLocation?: {
    stepNumber?: number;
    section?: string;
}

interface DependencyEdge {
from: string; // Source statement ID
  to: string; // Target statement ID (depends on source)
  type: 'logical' | 'definitional' | 'computational' | 'implicit';
  strength: number; // 0-1, how strong is the dependency
  inferenceRule?: InferenceRule;
}

interface DependencyGraph {
nodes: Map<string, AtomicStatement>;
  edges: DependencyEdge[];

  // Computed properties
  roots: string[]; // Axioms/hypotheses (no incoming edges)
  leaves: string[]; // Final conclusions (no outgoing edges)

  // Graph metrics
  depth: number; // Longest path from root to leaf
  width: number; // Maximum nodes at any level
  hasCycles: boolean; // Indicates circular reasoning

  // Analysis
  stronglyConnectedComponents?: string[][]; // For cycle detection
  topologicalOrder?: string[]; // Valid if acyclic
}

interface ProofGap {
id: string;
  type:
    | 'missing_step'
    | 'unjustified_leap'
    | 'implicit_assumption'
    | 'undefined_term'
    | 'scope_error';
  location: {
    from: string; // Statement ID before gap
    to: string; // Statement ID after gap
}

interface ImplicitAssumption {
id: string;
  statement: string;
  type:
    | 'domain_assumption'
    | 'existence_assumption'
    | 'uniqueness_assumption'
    | 'continuity_assumption'
    | 'finiteness_assumption'
    | 'well_ordering';
  usedInStep: string; // Where this assumption is needed
  shouldBeExplicit: boolean;
  suggestedFormulation: string;
}

interface AssumptionChain {
conclusion: string; // Statement ID
  assumptions: string[]; // Ordered list of assumption IDs
  path: string[]; // Full derivation path
  allAssumptionsExplicit: boolean;
  implicitAssumptions: ImplicitAssumption[];
}

interface ProofDecomposition {
id: string;
  originalProof: string;
  theorem?: string;

  // Decomposed elements
  atoms: AtomicStatement[];
  dependencies: DependencyGraph;

  // Analysis results
  assumptionChains: AssumptionChain[];
  gaps: ProofGap[];
  implicitAssumptions: ImplicitAssumption[];

  // Metrics
  completeness: number; // 0-1
  rigorLevel: 'informal' | 'textbook' | 'rigorous' | 'formal';
  atomCount: number;
  maxDependencyDepth: number;
}

interface Inconsistency {
id: string;
  type: InconsistencyType;
  involvedStatements: string[];
  explanation: string;
  derivationPath?: string[];
  severity: 'warning' | 'error' | 'critical';
  suggestedResolution?: string;
}

```

## Worked Example (from source doc)

```typescript
// Define a theorem
const theorem = await deepthinking_mathematics({
  mode: 'mathematics',
  thought: 'State the fundamental theorem of calculus',
  thoughtNumber: 1,
  totalThoughts: 5,
  nextThoughtNeeded: true,

  thoughtType: 'theorem_statement',
  theorems: [{
    name: 'Fundamental Theorem of Calculus (Part 1)',
    statement: 'If f is continuous on [a,b] and F(x) = ∫ₐˣ f(t)dt, then F is differentiable and F\'(x) = f(x)',
    hypotheses: ['f continuous on [a,b]', 'F(x) = ∫ₐˣ f(t)dt'],
    conclusion: 'F\'(x) = f(x)'
  }],
  dependencies: ['continuity definition', 'integral definition'],
  assumptions: ['f is integrable'],
  uncertainty: 0.1
});

// Construct a proof
const proof = await deepthinking_mathematics({
  mode: 'mathematics',
  thought: 'Prove using definition of derivative and properties of integrals',
  thoughtNumber: 2,
  totalThoughts: 5,
  nextThoughtNeeded: true,

  thoughtType: 'proof_construction',
  proofStrategy: {
    type: 'direct',
    steps: [
      'Consider (F(x+h) - F(x))/h',
      'Express as (1/h)∫ₓˣ⁺ʰ f(t)dt',
      'Apply mean value theorem for integrals',
      'Take limit as h → 0'
    ],
    completeness: 0.85
  },
  dependencies: ['theorem_statement'],
  assumptions: ['mean value theorem applies'],
  uncertainty: 0.15,

  logicalForm: {
    premises: ['f continuous', 'integral well-defined'],
    conclusion: 'F differentiable with F\' = f',
    inferenceRule: 'direct_implication',
    rules: ['limit definition', 'integral properties']
  }
});

// Analyze proof for gaps (Phase 8)
const analysis = await deepthinking_mathematics({
  mode: 'mathematics',
  thought: 'Check proof for gaps and implicit assumptions',
  thoughtNumber: 3,
  totalThoughts: 5,
  nextThoughtNeeded: true,

  thoughtType: 'gap_identification',
  gapAnalysis: {
    completeness: 0.85,
    gaps: [{
      id: 'gap_1',
      type: 'unjustified_leap',
      location: { from: 'step_2', to: 'step_3' },
      description: 'Mean value theorem application needs justification',
      severity: 'minor',
      suggestedFix: 'Cite integral MVT explicitly'
    }],
    implicitAssumptions: [{
      id: 'impl_1',
      statement: 'h can be either positive or negative',
      type: 'domain_assumption',
      usedInStep: 'step_1',
      shouldBeExplicit: true,
      suggestedFormulation: 'Consider limits from both sides'
    }],
    unjustifiedSteps: [],
    suggestions: ['Make two-sided limit explicit']
  },
  dependencies: ['proof_construction'],
  a

---

*Generated by RLM extraction pass on 2026-04-12*
