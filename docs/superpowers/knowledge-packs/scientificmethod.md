# Knowledge Pack: scientificmethod

**Mode**: `scientificmethod`
**Category skill**: `think-scientific`
**Source doc**: `docs/modes/SCIENTIFICMETHOD.md`

## Overview

Scientific Method mode provides **hypothesis-driven experimentation** and rigorous analysis. It supports the full scientific cycle from question formulation through experimental design, data collection, statistical analysis, to conclusions.

This mode captures the structure of scientific inquiry - from observation through hypothesis to verification.

## When to Use

Use scientific method mode when you need to:

- **Test hypotheses rigorously** - Formal hypothesis testing
- **Design experiments** - Controlled experimentation
- **Analyze data statistically** - Statistical inference
- **Draw valid conclusions** - Evidence-based conclusions
- **Document research** - Reproducible research

## Core Concepts

(no 'core concepts' section in source doc)

## Thought Type (TypeScript Interface)

```typescript
interface ScientificMethodThought extends BaseThought {
mode: ThinkingMode.SCIENTIFICMETHOD;
  thoughtType:
    | 'question_formulation'
    | 'hypothesis_generation'
    | 'experiment_design'
    | 'data_collection'
    | 'analysis'
    | 'conclusion';

  researchQuestion?: ResearchQuestion;
  scientificHypotheses?: Hypothesis[];
  experiment?: ExperimentDesign;
  data?: DataCollection;
  analysis?: StatisticalAnalysis;
  conclusion?: ScientificConclusion;
}
```

## Supporting Interfaces

```typescript
interface ResearchQuestion {
id: string;
  question: string;
  background: string;
  rationale: string;
  significance: string;
  variables: {
    independent: string[];
    dependent: string[];
    control: string[];
}

interface Hypothesis {
id: string;
  type: 'null' | 'alternative' | 'directional' | 'non_directional';
  statement: string;
  prediction: string;
  rationale: string;
  testable: boolean;
  falsifiable: boolean;
  latex?: string; // Statistical notation
}

interface ExperimentDesign {
id: string;
  type: 'experimental' | 'quasi_experimental' | 'observational' | 'correlational';
  design: string; // e.g., "randomized controlled trial", "within-subjects"
  independentVariables: Variable[];
  dependentVariables: Variable[];
  controlVariables: Variable[];
  sampleSize: number;
  sampleSizeJustification?: string;
  randomization: boolean;
  blinding?: 'none' | 'single' | 'double' | 'triple';
  controls: string[];
  procedure: string[];
  materials?: string[];
  duration?: string;
  ethicalConsiderations?: string[];
}

interface Variable {
id: string;
  name: string;
  type: 'independent' | 'dependent' | 'control' | 'confounding';
  description: string;
  measurementScale: 'nominal' | 'ordinal' | 'interval' | 'ratio';
  unit?: string;
  operationalDefinition: string;
  range?: [number, number];
  levels?: string[] | number[];
}

interface DataCollection {
id: string;
  method: string[];
  instruments: string[];
  observations: Observation[];
  measurements: Measurement[];
  sampleCharacteristics?: Record<string, any>;
  dataQuality: {
    completeness: number; // 0-1
    reliability: number; // 0-1
    validity: number; // 0-1
}

interface Observation {
id: string;
  timestamp?: string;
  condition: string;
  values: Record<string, any>;
  notes?: string;
}

interface Measurement {
variableId: string;
  values: number[];
  descriptiveStats?: {
    mean?: number;
    median?: number;
    mode?: number;
    stdDev?: number;
    variance?: number;
    min?: number;
    max?: number;
    n: number;
}

interface StatisticalAnalysis {
id: string;
  tests: StatisticalTest[];
  summary: string;
  assumptions: {
    assumption: string;
    met: boolean;
    evidence: string;
}

```

## Worked Example (from source doc)

```typescript
// Formulate research question
const question = await deepthinking_scientific({
  mode: 'scientificmethod',
  thought: 'Define the research question for cache performance',
  thoughtNumber: 1,
  totalThoughts: 6,
  nextThoughtNeeded: true,

  thoughtType: 'question_formulation',
  researchQuestion: {
    id: 'rq1',
    question: 'Does increasing cache size from 1GB to 4GB reduce API response latency?',
    background: 'Current API p99 latency is 200ms, which is above our SLA target of 150ms',
    rationale: 'Cache misses account for 40% of slow requests based on profiling data',
    significance: 'Meeting SLA could retain enterprise customers worth $500K ARR',
    variables: {
      independent: ['Cache size'],
      dependent: ['API p99 latency', 'Cache hit rate'],
      control: ['Traffic load', 'Query complexity', 'Server hardware']
    }
  }
});

// Generate hypotheses
const hypotheses = await deepthinking_scientific({
  mode: 'scientificmethod',
  thought: 'Generate testable hypotheses',
  thoughtNumber: 2,
  totalThoughts: 6,
  nextThoughtNeeded: true,

  thoughtType: 'hypothesis_generation',
  scientificHypotheses: [
    {
      id: 'h0',
      type: 'null',
      statement: 'Increasing cache size has no effect on p99 latency',
      prediction: 'p99_4GB = p99_1GB',
      rationale: 'Default assumption of no effect',
      testable: true,
      falsifiable: true,
      latex: 'H_0: \\mu_{4GB} = \\mu_{1GB}'
    },
    {
      id: 'h1',
      type: 'directional',
      statement: 'Increasing cache size reduces p99 latency by at least 25%',
      prediction: 'p99_4GB < 150ms (25% improvement from 200ms)',
      rationale: 'Based on cache hit rate analysis and expected improvement',
      testable: true,
      falsifiable: true,
      latex: 'H_1: \\mu_{4GB} < 0.75 \\times \\mu_{1GB}'
    }
  ]
});

// Design experiment
const experiment = await deepthinking_scientific({
  mode: 'scientificmethod',
  thought: 'Design the experiment',
  thoughtNumber: 3,
  totalThoughts: 6,
  nextThoughtNeeded: true,

  thoughtType: 'experiment_design',
  experiment: {
    id: 'exp1',
    type: 'experimental',
    design: 'A/B test with random traffic assignment',
    independentVariables: [{
      id: 'cache_size',
      name: 'Cache Size',
      type: 'independent',
      description: 'Redis cache memory allocation',
      measurementScale: 'ratio',
      unit: 'GB',
      operationalDefinition: 'maxmemory setting in Redis config',
      levels: [1, 4]
   

---

*Generated by RLM extraction pass on 2026-04-12*
