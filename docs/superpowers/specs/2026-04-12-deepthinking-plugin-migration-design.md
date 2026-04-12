# deepthinking-plugin Migration Design

**Date**: 2026-04-12
**Status**: Approved (user confirmed design in brainstorming session)
**Scope**: Convert deepthinking-mcp (v9.1.3, TypeScript MCP server with 237 files / ~102K LOC / 13 tools / 34 reasoning modes) into a Claude Code plugin composed of prompt-based skills. Claude internalizes the reasoning methods and applies them natively; no Node.js server runs at invocation time.

## Problem Statement

deepthinking-mcp works but suffers from context pollution: all 34 reasoning modes are always visible to Claude even when only 2-3 are relevant for a given task. The TypeScript runtime, validation, and session infrastructure are not adding value proportional to their weight — the reasoning knowledge encoded in handler logic and mode docs is the actual product. The goal is for Claude Code to *learn* the deepthinking methods and produce structured outputs in the mode-specific formats, not to call an external reasoning server.

## Goals

1. Claude Code applies the 34 reasoning methods natively as first-class skills.
2. Only the relevant category loads into context per invocation (solving context pollution).
3. Preserve the structured per-mode output formats from the original codebase.
4. Preserve visual export capability (Mermaid, DOT, ASCII, SVG, and other formats).
5. Ship as a distributable Claude Code plugin via `.claude-plugin/plugin.json`.
6. Support both manual mode selection (`/think bayesian ...`) and auto-recommendation (`/think ...`).

## Non-Goals

- Runtime type validation (moves from Zod to prompt-based output templates + self-check checklists).
- Persistent session state (drop the 5,148-test session tracking subsystem; each invocation is self-contained).
- Multi-instance file-locked storage (no longer needed without a server).
- Backwards compatibility with the MCP's tool names — the plugin is a new product.

## Architecture

### Plugin Layout

```
deepthinking-plugin/
├── .claude-plugin/
│   └── plugin.json                 # Manifest (name, version, author, description)
├── skills/
│   ├── think/
│   │   ├── SKILL.md                # Router + auto-recommend + output format spec
│   │   └── mode-index.md           # Decision tree: which mode for which problem
│   ├── think-core/SKILL.md         # Inductive, Deductive, Abductive
│   ├── think-standard/SKILL.md     # Sequential, Shannon, Hybrid
│   ├── think-mathematics/SKILL.md  # Mathematics, Physics, Computability
│   ├── think-temporal/SKILL.md     # Temporal, Historical
│   ├── think-probabilistic/SKILL.md # Bayesian, Evidential
│   ├── think-causal/SKILL.md       # Causal, Counterfactual
│   ├── think-strategic/SKILL.md    # GameTheory, Optimization, Constraint
│   ├── think-analytical/SKILL.md   # Analogical, FirstPrinciples, MetaReasoning, Cryptanalytic
│   ├── think-scientific/SKILL.md   # ScientificMethod, SystemsThinking, FormalLogic
│   ├── think-engineering/SKILL.md  # Engineering, Algorithmic
│   ├── think-academic/SKILL.md     # Synthesis, Argumentation, Critique, Analysis
│   └── think-advanced/SKILL.md     # Recursive, Modal, Stochastic
├── agents/
│   └── visual-exporter.md          # Subagent: converts structured thought → diagram
├── reference/
│   ├── output-formats/             # One .md per mode with JSON schema + worked example
│   │   ├── bayesian.md
│   │   └── … (34 files)
│   ├── taxonomy.md                 # "Which mode should I use?" decision tree
│   └── visual-grammar.md           # Per-mode rules for diagram structure
├── scripts/
│   └── render-diagram.py           # DOT→SVG and Mermaid→SVG format conversion
├── test/
│   ├── smoke/                      # Per-skill smoke tests (one per mode)
│   ├── router/                     # 20 problem descriptions → expected mode selection
│   └── visual/                     # Diagram syntax validation tests
├── README.md
└── CHANGELOG.md
```

### Component Boundaries

Plugin contains **13 skills total**: 1 router (`think`) + 12 category skills.

| Component | Responsibility | Interface |
|---|---|---|
| `/think` router skill (1) | Route to category skill, auto-recommend from problem description, own structured-output contract | Input: `/think <mode?> "<problem>"`. Output: loads the right category skill and output template |
| Category skills (12) | Teach Claude 2-4 reasoning methods each, with worked examples and structured-field definitions | Claude reads the method, applies it, produces thought matching the mode's schema |
| `visual-exporter` agent | Convert a structured thought into diagram code (Mermaid/DOT/SVG/etc.) | Input: thought object + format. Output: diagram string or file path |
| `reference/output-formats/` | Authoritative schema for each mode's thought structure, as markdown templates | Loaded on demand by category skills when generating output |
| `scripts/render-diagram.py` | Mechanical format conversion (DOT→SVG via `dot`, Mermaid→PNG via `mmdc`) | CLI script invoked by visual-exporter agent |

Each component is independently understandable. The router does not know how Bayesian works. The Bayesian section does not know about visual export. The visual exporter does not care which mode produced the thought.

### Data Flow

**Happy path — user triggers a specific mode:**

```
User: /think bayesian "Is this flaky test caused by the recent DB migration?"
  ↓
think router loads (reads input, sees "bayesian" specified)
  ↓
think-probabilistic skill loads (contains Bayesian method: hypothesis, prior, likelihood, posterior)
  ↓
Claude reasons natively following the Bayesian framework
  ↓
Claude outputs structured thought matching bayesian.md template:
  { mode: "bayesian", hypothesis: {...}, prior: {...}, likelihood: {...}, posterior: {...} }
  ↓
(Optional) User: "render this as Mermaid"
  ↓
visual-exporter agent loads, reads structured thought, generates Mermaid Bayesian network
```

**Auto-recommend path:**

```
User: /think "What caused the production outage last Tuesday?"
  ↓
think router loads (no mode specified — reads mode-index.md decision tree)
  ↓
Claude picks: "counterfactual" based on problem structure ("what caused")
  ↓
Explains choice, loads think-causal, proceeds as above
```

**Multi-mode analysis path (replaces `deepthinking_analyze`):**

```
User: /think analyze "Should we migrate from Postgres to DynamoDB?"
  ↓
think router recognizes analyze intent
  ↓
Loads 3 complementary category skills: think-strategic, think-probabilistic, think-engineering
  ↓
Claude runs three reasoning passes, synthesizes the results
```

### Mode-to-Skill Mapping

34 modes distributed across 12 category skills, 2-4 modes per skill. Each category skill stays under ~12KB to keep context load bounded:

| Skill | Modes | Est. size |
|---|---|---|
| think-core | Inductive, Deductive, Abductive | ~8KB |
| think-standard | Sequential, Shannon, Hybrid | ~8KB |
| think-mathematics | Mathematics, Physics, Computability | ~10KB |
| think-temporal | Temporal, Historical | ~7KB |
| think-probabilistic | Bayesian, Evidential | ~9KB |
| think-causal | Causal, Counterfactual | ~9KB |
| think-strategic | GameTheory, Optimization, Constraint | ~11KB |
| think-analytical | Analogical, FirstPrinciples, MetaReasoning, Cryptanalytic | ~12KB |
| think-scientific | ScientificMethod, SystemsThinking, FormalLogic | ~11KB |
| think-engineering | Engineering, Algorithmic | ~9KB |
| think-academic | Synthesis, Argumentation, Critique, Analysis | ~12KB |
| think-advanced | Recursive, Modal, Stochastic | ~9KB |

Router skill (`think/SKILL.md`) stays ~5KB — decision tree, mode list, pointer to category skills, and top-level output format contract.

### Output Format Strategy

The current MCP validates with Zod at runtime. Skills cannot run code, so format discipline comes from:

1. **The method description** adapted from `docs/modes/<MODE>.md`
2. **A JSON template** extracted from `src/types/modes/<mode>.ts` showing the exact thought structure
3. **A worked example** — complete input → reasoning → output trace
4. **A pre-output verification checklist** — "before emitting, verify: all probabilities in [0, 1], evidence has both P(E|H) and P(E|¬H), alternatives are mutually exclusive, etc."

Example excerpt from `think-probabilistic/SKILL.md`:

```markdown
## Bayesian Reasoning Output Format

Produce output as a JSON object with this exact structure:

{
  "mode": "bayesian",
  "hypothesis": {
    "statement": "<the hypothesis>",
    "alternatives": ["<alt 1>", "<alt 2>"]
  },
  "prior": { "probability": 0.0-1.0, "justification": "..." },
  "likelihood": { "probability": 0.0-1.0, "description": "..." },
  "evidence": [
    { "description": "...", "likelihoodGivenHypothesis": 0.0-1.0, "likelihoodGivenNotHypothesis": 0.0-1.0 }
  ],
  "posterior": { "probability": 0.0-1.0, "calculation": "P(H|E) = ...", "confidence": 0.0-1.0 },
  "bayesFactor": <optional number>
}

Before outputting, verify:
- All probabilities are in [0, 1]
- posterior.probability matches calculation shown in posterior.calculation
- If alternatives provided, prior and alternatives' implicit priors are consistent
```

### Visual Export Strategy

The MCP has 42 visual exporters (24 mode-specific, 15 utils, 3 root) producing 11 formats. Handled in three layers:

**Layer 1 — Claude generates directly (~70% of cases):**
Claude is already strong at Mermaid, DOT, Markdown, JSON, and ASCII. The `visual-exporter` agent uses `reference/visual-grammar.md` which says things like: "For Bayesian thoughts, generate a Mermaid graph with hypothesis as the root, evidence nodes as children, arrows labeled with likelihood ratios."

**Layer 2 — Mechanical format conversion (~25% more):**
`scripts/render-diagram.py` wraps:
- DOT → SVG/PNG via `graphviz` (`dot` binary)
- Mermaid → SVG/PNG via `mmdc` (`@mermaid-js/mermaid-cli`)
- HTML via simple wrapper templates

These are documented as optional dependencies; the skill gracefully degrades to inline DOT/Mermaid source if the tools are not installed.

**Layer 3 — Complex formats (~5% rest):**
TikZ, GraphML, Modelica, UML — low-usage but supported. Encoded as generation rules in `reference/visual-grammar.md` for Claude to follow directly. The existing builder class code is preserved in `reference/builders/` as documentation only (not executed).

### Error Handling and Validation

- **Input clarity** — Each category skill opens with: "If the user's problem does not match this mode's shape, recommend switching modes rather than forcing the fit."
- **Output discipline** — The JSON output template includes a verification checklist; Claude self-checks before emitting.
- **Graceful degradation** — If Claude cannot fill all fields (no evidence available for Bayesian, no observed history for Historical), it emits the partial structure with explicit `null` or `"unknown"` markers instead of hallucinating.
- **Mode mismatch** — If the router picks a mode that fails when applied, it retries auto-recommendation once with a note explaining the switch.

### Testing Strategy

- **Skill smoke tests** — One test **per mode** (not per skill file). Load the relevant category skill, feed it one representative problem for the target mode, verify the output matches the JSON schema (shape, required fields, probability ranges where applicable). **34 tests total (one per mode).**
- **Router tests** — 20 problem descriptions → expected category. Multiple answers acceptable; a wrong category triggers investigation.
- **Visual export tests** — Per mode: generate Mermaid and DOT; verify both parse syntactically.
- **No unit tests for reasoning correctness** — That is Claude's job, not the skill's.

Tests live in `test/` inside the plugin. A harness script runs them via Claude CLI in headless mode.

## Migration Plan (Phased)

### Phase 1: Plugin scaffold and router
Create the plugin chassis before extracting any modes. Deliverables:
- Write `.claude-plugin/plugin.json` manifest
- Create directory layout (`skills/`, `agents/`, `reference/`, `scripts/`, `test/`)
- Write `skills/think/SKILL.md` with placeholder auto-recommend decision tree
- Write `skills/think/mode-index.md` stub (to be populated during Phase 3)
- Write `reference/taxonomy.md` with initial mode selection rules
- Install locally with `--plugin-dir` and verify the plugin loads

**Exit criteria:** `/think` router responds and explains it has no modes available yet; plugin-validator passes on the empty scaffold.

### Phase 2: Prototype mode extraction (3 modes)
Extract the simplest modes first: Sequential, Inductive, Deductive. For each, read `src/modes/handlers/<X>Handler.ts`, `src/types/modes/<x>.ts`, and `docs/modes/<X>.md`. Distill into the first two category skills (`think-standard` for Sequential; `think-core` for Inductive + Deductive). Update the router and `mode-index.md` to know about these three modes.

**Exit criteria:** `/think inductive "<problem>"` produces correct structured output; `/think "<ambiguous-problem>"` auto-routes to one of the 3 prototype modes.

### Phase 3: Parallel mode extraction (remaining 30 modes)
All category skills except the prototypes authored concurrently by parallel agents. Each category skill gets:
- Its category's reasoning methods from the relevant `<X>Handler.ts` files
- Output format templates from `src/types/modes/<x>.ts`
- Worked examples adapted from `docs/modes/<X>.md`

**Exit criteria:** All 34 modes are represented across 12 category skills.

### Phase 4: Visual export
- Write `agents/visual-exporter.md`
- Write `reference/visual-grammar.md` (per-mode diagram rules)
- Write `scripts/render-diagram.py`
- Copy existing builder class code into `reference/builders/` as documentation

**Exit criteria:** Every mode can produce valid Mermaid and DOT output, and at least 3 modes can round-trip through `render-diagram.py` to produce SVG.

### Phase 5: Testing and polish
- Author smoke tests (34)
- Author router tests (20)
- Author visual tests (34 × 2 = 68)
- Run skill-reviewer against each category skill
- Run plugin-validator on the final package
- Write README and CHANGELOG
- Tag v1.0.0

**Exit criteria:** All tests pass, skill-reviewer and plugin-validator both approve, plugin installs cleanly from local directory.

### Phase 6 (optional): Deprecate deepthinking-mcp
- Archive the v9 repo or park a deprecation notice
- Final npm publish of deepthinking-mcp with a pointer to the plugin

## Team Dispatch

The following agents will be coordinated:

| Agent | Job |
|---|---|
| `feature-dev:code-explorer` (parallel, ×3 batches) | Trace execution paths through handler files, extract method knowledge into draft markdown |
| `plugin-dev:plugin-structure` | Scaffold `.claude-plugin/plugin.json` and directory layout |
| `plugin-dev:skill-development` | Author each SKILL.md from the extracted method drafts |
| `plugin-dev:agent-development` | Author `agents/visual-exporter.md` |
| `feature-dev:code-architect` | Design the router logic, mode-index decision tree, and output-format template system |
| `plugin-dev:skill-reviewer` | Review each skill against `writing-skills` criteria |
| `plugin-dev:plugin-validator` | Validate the final plugin structure before packaging |

Parallelization comes in Phase 3: once the router and templates are locked, the 11 remaining category skills can be authored concurrently.

## Open Questions

None blocking. The following will be decided during implementation:

1. Exact taxonomy phrasing in the router's decision tree (refine against router tests).
2. Whether `reference/builders/` ends up being useful documentation or dead weight — keep for v1, prune in v1.1 if unused.
3. Plugin distribution target (personal marketplace vs. official submission) — deferred to post-v1.

## Success Criteria

1. `/think <mode> "<problem>"` produces a structured thought matching the mode's schema for all 34 modes.
2. `/think "<problem>"` auto-recommends a sensible mode for ≥18 of 20 router test cases.
3. Every mode can produce valid Mermaid and DOT diagram output.
4. Total context load per invocation is ≤15KB (router skill + one category skill).
5. Plugin installs cleanly via `--plugin-dir` and passes `plugin-validator`.
6. The 34 smoke tests, 20 router tests, and 68 visual tests all pass.
