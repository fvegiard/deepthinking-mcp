# Knowledge Pack: temporal

**Mode**: `temporal`
**Category skill**: `think-temporal`
**Source doc**: `docs/modes/TEMPORAL.md`

## Overview

Temporal mode provides **time-dependent reasoning** using events, intervals, constraints, and causal relations over time. It supports Allen's interval algebra for temporal relationships and timeline construction.

This mode captures the structure of temporal reasoning - from event definition through interval analysis to causality timelines.

## When to Use

Use temporal mode when you need to:

- **Reason about time** - Events, durations, sequences
- **Analyze temporal relationships** - Before, after, during, overlaps
- **Build timelines** - Construct event sequences
- **Apply temporal constraints** - Scheduling, planning
- **Trace causality over time** - Cause and effect with timing

## Core Concepts

(no 'core concepts' section in source doc)

## Thought Type (TypeScript Interface)

```typescript
interface TemporalThought extends BaseThought {
mode: ThinkingMode.TEMPORAL;
  thoughtType:
    | 'event_definition'
    | 'interval_analysis'
    | 'temporal_constraint'
    | 'sequence_construction'
    | 'causality_timeline';

  timeline?: Timeline;
  events?: TemporalEvent[];
  intervals?: TimeInterval[];
  constraints?: TemporalConstraint[];
  relations?: TemporalRelation[];
}
```

## Supporting Interfaces

```typescript
interface Timeline {
id: string;
  name: string;
  timeUnit: 'milliseconds' | 'seconds' | 'minutes' | 'hours' | 'days' | 'months' | 'years';
  startTime?: number;
  endTime?: number;
  events: string[]; // Event IDs
}

interface TemporalEvent {
id: string;
  name: string;
  description: string;
  timestamp: number;
  duration?: number; // For interval events
  type: 'instant' | 'interval';
  properties: Record<string, any>;
}

interface TimeInterval {
id: string;
  name: string;
  start: number;
  end: number;
  overlaps?: string[]; // IDs of overlapping intervals
  contains?: string[]; // IDs of contained intervals
}

interface TemporalConstraint {
id: string;
  type: 'before' | 'after' | 'during' | 'overlaps' | 'meets' | 'starts' | 'finishes' | 'equals';
  subject: string; // Event/Interval ID
  object: string; // Event/Interval ID
  confidence: number; // 0-1
  formula?: string; // LaTeX formula for temporal logic constraint
}

interface TemporalRelation {
id: string;
  from: string; // Event ID
  to: string; // Event ID
  relationType: 'causes' | 'enables' | 'prevents' | 'precedes' | 'follows';
  strength: number; // 0-1
  delay?: number; // Time delay between events
  formula?: string; // LaTeX formula for relationship dynamics
}

```

## Worked Example (from source doc)

```typescript
// Define timeline
const timeline = await deepthinking_temporal({
  mode: 'temporal',
  thought: 'Define project timeline for software release',
  thoughtNumber: 1,
  totalThoughts: 5,
  nextThoughtNeeded: true,

  thoughtType: 'event_definition',
  timeline: {
    id: 'project_timeline',
    name: 'Release v2.0 Timeline',
    timeUnit: 'days',
    startTime: 0,
    endTime: 90,
    events: ['design', 'development', 'testing', 'release']
  },
  events: [
    {
      id: 'design',
      name: 'Design Phase',
      description: 'Complete system design',
      timestamp: 0,
      duration: 14,
      type: 'interval',
      properties: { team: 'architecture', deliverables: ['design doc'] }
    },
    {
      id: 'development',
      name: 'Development Phase',
      description: 'Implement features',
      timestamp: 10,
      duration: 45,
      type: 'interval',
      properties: { team: 'engineering' }
    },
    {
      id: 'testing',
      name: 'Testing Phase',
      description: 'QA and testing',
      timestamp: 40,
      duration: 30,
      type: 'interval',
      properties: { team: 'qa' }
    },
    {
      id: 'release',
      name: 'Release Event',
      description: 'Production release',
      timestamp: 90,
      type: 'instant',
      properties: { milestone: true }
    }
  ]
});

// Define intervals
const intervals = await deepthinking_temporal({
  mode: 'temporal',
  thought: 'Analyze interval relationships',
  thoughtNumber: 2,
  totalThoughts: 5,
  nextThoughtNeeded: true,

  thoughtType: 'interval_analysis',
  intervals: [
    {
      id: 'design_interval',
      name: 'Design Phase',
      start: 0,
      end: 14
    },
    {
      id: 'dev_interval',
      name: 'Development Phase',
      start: 10,
      end: 55,
      overlaps: ['design_interval', 'test_interval']
    },
    {
      id: 'test_interval',
      name: 'Testing Phase',
      start: 40,
      end: 70,
      overlaps: ['dev_interval']
    }
  ]
});

// Define temporal constraints
const constraints = await deepthinking_temporal({
  mode: 'temporal',
  thought: 'Define temporal constraints between phases',
  thoughtNumber: 3,
  totalThoughts: 5,
  nextThoughtNeeded: true,

  thoughtType: 'temporal_constraint',
  constraints: [
    {
      id: 'tc_1',
      type: 'overlaps',
      subject: 'design_interval',
      object: 'dev_interval',
      confidence: 1.0,
      formula: 'design.end > dev.start ∧ design.end < dev.end'
    },
    {
      id: 'tc_2',
      type: 

---

*Generated by RLM extraction pass on 2026-04-12*
