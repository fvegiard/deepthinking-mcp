# deepthinking-plugin Phase 3: Batch Mode Extraction Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extend `deepthinking-plugin` from 3 modes (v0.1.0) to all 34 modes (v0.2.0) by adding 11 category skill files, 31 new JSON schemas, 31 new reference output-format docs, and updating the router/mode-index/taxonomy/`/think` command to route to all modes. Knowledge packs at `docs/superpowers/knowledge-packs/` (in the source repo) contain pre-distilled content for each of the 31 new modes.

**Architecture:** Each category skill (`skills/think-<category>/SKILL.md`) aggregates 2-4 reasoning methods. The router reads `mode-index.md` to pick the right category skill for a given mode name. Each mode also gets its own JSON schema (for output validation) and a reference/output-formats doc (for readers wanting full detail). The `commands/think.md` slash command is updated to list all 34 modes.

**Tech Stack:** Markdown (SKILL.md files), JSON Schema (draft-07), Python (test harness), knowledge packs as distilled source.

**Source knowledge:**
- `C:/Users/danie/Dropbox/Github/deepthinking-mcp/docs/superpowers/knowledge-packs/<mode>.md` — **one pack per mode**, each containing overview, when-to-use, core concepts, TypeScript interface, and worked example. The RLM extraction pass already produced these; implementers do NOT need to re-read the original handler/type/doc files.
- If a knowledge pack is thin (for modes whose source docs were minimal), the implementer may supplement by reading the original TS type file at `C:/Users/danie/Dropbox/Github/deepthinking-mcp/src/types/modes/<mode>.ts` — but knowledge packs should be sufficient for most modes.

**Working directory:** `C:/Users/danie/Dropbox/Github/deepthinking-plugin/` (the plugin repo tagged v0.1.0).

## Mode Distribution

34 modes → 12 category skills (router + 11 categories). 3 modes already shipped in v0.1.0 (Sequential in `think-standard`, Inductive + Deductive in `think-core`). This plan adds 31 more modes across the following skills:

| Skill | v0.1.0 modes | New modes to add | Total |
|---|---|---|---|
| think-core | Inductive, Deductive | **Abductive** | 3 |
| think-standard | Sequential | **Shannon, Hybrid** | 3 |
| think-mathematics | — | **Mathematics, Physics, Computability** | 3 |
| think-temporal | — | **Temporal, Historical** | 2 |
| think-probabilistic | — | **Bayesian, Evidential** | 2 |
| think-causal | — | **Causal, Counterfactual** | 2 |
| think-strategic | — | **GameTheory, Optimization, Constraint** | 3 |
| think-analytical | — | **Analogical, FirstPrinciples, MetaReasoning, Cryptanalytic** | 4 |
| think-scientific | — | **ScientificMethod, SystemsThinking, FormalLogic** | 3 |
| think-engineering | — | **Engineering, Algorithmic** | 2 |
| think-academic | — | **Synthesis, Argumentation, Critique, Analysis** | 4 |
| think-advanced | — | **Recursive, Modal, Stochastic** | 3 |

## File Structure at End of Phase 3

New files (relative to plugin root):

```
skills/
├── think-core/SKILL.md                    # UPDATED: adds Abductive
├── think-standard/SKILL.md                # UPDATED: adds Shannon + Hybrid
├── think-mathematics/SKILL.md             # NEW
├── think-temporal/SKILL.md                # NEW
├── think-probabilistic/SKILL.md           # NEW
├── think-causal/SKILL.md                  # NEW
├── think-strategic/SKILL.md               # NEW
├── think-analytical/SKILL.md              # NEW
├── think-scientific/SKILL.md              # NEW
├── think-engineering/SKILL.md             # NEW
├── think-academic/SKILL.md                # NEW
├── think-advanced/SKILL.md                # NEW
└── think/
    ├── SKILL.md                           # UPDATED: all 34 modes in table
    └── mode-index.md                      # UPDATED: decision tree for all 34

reference/
├── output-formats/                        # 31 NEW files (+ 3 existing)
│   ├── abductive.md
│   ├── shannon.md
│   ├── hybrid.md
│   ├── mathematics.md
│   ├── physics.md
│   ├── computability.md
│   ├── temporal.md
│   ├── historical.md
│   ├── bayesian.md
│   ├── evidential.md
│   ├── causal.md
│   ├── counterfactual.md
│   ├── gametheory.md
│   ├── optimization.md
│   ├── constraint.md
│   ├── analogical.md
│   ├── firstprinciples.md
│   ├── metareasoning.md
│   ├── cryptanalytic.md
│   ├── scientificmethod.md
│   ├── systemsthinking.md
│   ├── formallogic.md
│   ├── engineering.md
│   ├── algorithmic.md
│   ├── synthesis.md
│   ├── argumentation.md
│   ├── critique.md
│   ├── analysis.md
│   ├── recursive.md
│   ├── modal.md
│   └── stochastic.md
└── taxonomy.md                            # UPDATED: all 34 modes graduated

commands/think.md                          # UPDATED: lists all 34 modes

test/
├── schemas/                               # 31 NEW JSON Schemas
│   └── <mode>.json × 31
├── samples/                               # 31 NEW valid sample thoughts
│   └── <mode>-valid.json × 31
└── harness.py                             # UPDATED: SAMPLES list expanded to 34
```

## Parallelization Strategy

The 11 category skill authoring tasks (Tasks 1-11) are **mostly independent** and can be dispatched as parallel subagents. Each task gets:
1. The knowledge packs for its modes
2. The existing v0.1.0 SKILL.md pattern (from `think-core` or `think-standard`) as a template
3. Instructions to write the SKILL.md, the schema files, and the reference/output-formats files for its modes

After all 11 tasks complete, Tasks 12-15 run sequentially: router/mode-index update, test harness expansion, smoke tests, tag v0.2.0.

---

## Task 1: Expand think-core with Abductive

**Files:**
- Modify: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/skills/think-core/SKILL.md`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/schemas/abductive.json`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/samples/abductive-valid.json`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/reference/output-formats/abductive.md`

**Knowledge pack:** `C:/Users/danie/Dropbox/Github/deepthinking-mcp/docs/superpowers/knowledge-packs/abductive.md`

- [ ] **Step 1: Read the knowledge pack**

Read the file at the knowledge-pack path above. It contains: overview, when-to-use, core concepts, the TypeScript interface (`AbductiveThought`), and supporting interfaces (`Observation`, `Hypothesis`, `Evidence`, `EvaluationCriteria`).

- [ ] **Step 2: Derive the JSON Schema**

Write `test/schemas/abductive.json`. Translate the TypeScript interface into a JSON Schema draft-07 with `additionalProperties: false`. The `mode` field must be const `"abductive"`. Required fields mirror the non-optional fields in the interface. Match the pattern used by `test/schemas/inductive.json` (already in the repo).

- [ ] **Step 3: Write a sample valid thought**

Write `test/samples/abductive-valid.json`. Use a realistic scenario (e.g., "what's the best explanation for users reporting 503 errors only on Tuesdays?"). Include at least 2 hypotheses, 1 observation, and an evaluation per criterion. Must validate against the schema you just wrote.

- [ ] **Step 4: Write the reference/output-formats entry**

Write `reference/output-formats/abductive.md` following the pattern of `reference/output-formats/inductive.md`:
- `# Abductive Thought — Output Format` heading
- `## JSON Schema` with a JSON code block showing the structure
- `## Required Fields` bullet list
- `## Worked Example` with the sample from Step 3
- `## Verification Checklist` with mode-specific pre-output checks

- [ ] **Step 5: Append the Abductive section to think-core/SKILL.md**

Read the current `skills/think-core/SKILL.md`. Find the "## Future Methods (not in v0.1.0)" section at the bottom. **Replace** that section with a new "## Abductive Reasoning" section (matching the pattern of the existing Inductive and Deductive sections) and remove the "Future Methods" notice since Abductive is now included.

The new section must have these subsections, following the pattern of the Inductive section:
- When to Use / When NOT to use
- How to Reason Abductively (numbered steps from the knowledge pack's Core Concepts)
- Output Format (reference pointer)
- Quick Template (JSON template with placeholder values)
- Verification Before Emitting
- Worked Example

Also update the frontmatter `description` field to drop the "In v0.1.0 this skill contains Inductive and Deductive; Abductive ships in a future version" language and reflect all three modes are present.

- [ ] **Step 6: Run harness to verify the new schema catches a broken sample**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/harness.py
```

The harness will NOT yet include `abductive-valid.json` (that's added in Task 14). For now, verify it still passes the 4 existing cases. Also independently validate your new sample:

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python -c "
import json, jsonschema
with open('test/schemas/abductive.json') as f: s = json.load(f)
with open('test/samples/abductive-valid.json') as f: t = json.load(f)
jsonschema.validate(t, s)
print('PASS: abductive sample validates against new schema')
"
```

Expected: `PASS: abductive sample validates against new schema`

- [ ] **Step 7: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat(think-core): add Abductive reasoning method"
```

---

## Task 2: Expand think-standard with Shannon and Hybrid

**Files:**
- Modify: `skills/think-standard/SKILL.md`
- Create: `test/schemas/shannon.json`, `test/schemas/hybrid.json`
- Create: `test/samples/shannon-valid.json`, `test/samples/hybrid-valid.json`
- Create: `reference/output-formats/shannon.md`, `reference/output-formats/hybrid.md`

**Knowledge packs:**
- `C:/Users/danie/Dropbox/Github/deepthinking-mcp/docs/superpowers/knowledge-packs/shannon.md`
- `C:/Users/danie/Dropbox/Github/deepthinking-mcp/docs/superpowers/knowledge-packs/hybrid.md`

- [ ] **Step 1: Read both knowledge packs**

- [ ] **Step 2: Write both JSON schemas** (`shannon.json`, `hybrid.json`)

Follow the pattern in `test/schemas/sequential.json`. Translate the TypeScript interfaces to draft-07 schemas with `additionalProperties: false` and const `mode` values.

- [ ] **Step 3: Write both valid samples** (`shannon-valid.json`, `hybrid-valid.json`)

Use realistic scenarios. Validate each against its schema independently before moving on.

- [ ] **Step 4: Write both reference/output-formats docs** (`shannon.md`, `hybrid.md`)

Pattern from `reference/output-formats/sequential.md`.

- [ ] **Step 5: Append Shannon and Hybrid sections to think-standard/SKILL.md**

Read current file. Remove the "## Future Methods (not in v0.1.0)" notice. Add two new top-level sections: "## Shannon-Style Decomposition" and "## Hybrid Reasoning", each with the standard subsections (When to Use, How to..., Output Format, Quick Template, Verification, Worked Example). Update frontmatter description to reflect all three modes present.

- [ ] **Step 6: Validate both schemas catch their samples**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python -c "
import json, jsonschema
for mode in ['shannon', 'hybrid']:
    with open(f'test/schemas/{mode}.json') as f: s = json.load(f)
    with open(f'test/samples/{mode}-valid.json') as f: t = json.load(f)
    jsonschema.validate(t, s)
    print(f'PASS: {mode} sample validates')
"
```

- [ ] **Step 7: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat(think-standard): add Shannon and Hybrid reasoning methods"
```

---

## Tasks 3-11: Create 9 new category skills

Each of the following tasks follows the **same pattern** as Task 2 but creates a *new* category skill directory instead of modifying an existing one. The pattern is:

1. Create `skills/think-<category>/SKILL.md` with YAML frontmatter (`name`, `description`, `argument-hint`) and a `## User Invocation` section using `$ARGUMENTS`.
2. Read the knowledge packs for that category's modes.
3. Write one method section per mode (following the Inductive/Deductive pattern).
4. Write JSON schemas (one per mode).
5. Write valid samples (one per mode).
6. Write reference/output-formats docs (one per mode).
7. Validate schemas catch their samples.
8. Commit with `feat(think-<category>): add <mode list>`.

The template for a new category SKILL.md:

````markdown
---
name: think-<category>
description: <one sentence describing what this category covers, mentioning each mode name and `/think <mode>` invocations so Claude Code can auto-invoke the skill>
argument-hint: "[<mode1>|<mode2>|...] <problem>"
---

# think-<category> — <Human-Readable Category Name>

## User Invocation

```
$ARGUMENTS
```

Parse these arguments. The first word should be one of: <mode1>, <mode2>, ... The rest is the problem to reason about.

---

## <Mode1> Reasoning

<paraphrased overview from knowledge pack>

### When to Use

<from knowledge pack>

### How to Reason With <Mode1>

<numbered steps derived from knowledge pack's Core Concepts>

### Output Format

See `reference/output-formats/<mode1>.md`.

### Quick Template

```json
<JSON template with placeholders>
```

### Verification Before Emitting

- <checks specific to this mode>

### Worked Example

<from knowledge pack's example>

---

## <Mode2> Reasoning

... (same structure)
````

### Task 3: think-mathematics (Mathematics, Physics, Computability)

**Knowledge packs:**
- `docs/superpowers/knowledge-packs/mathematics.md`
- `docs/superpowers/knowledge-packs/physics.md`
- `docs/superpowers/knowledge-packs/computability.md`

**Files to create:**
- `skills/think-mathematics/SKILL.md`
- `test/schemas/{mathematics,physics,computability}.json`
- `test/samples/{mathematics,physics,computability}-valid.json`
- `reference/output-formats/{mathematics,physics,computability}.md`

**Steps:** Follow the generic pattern (Steps 1-8 above). After implementation, commit with `feat(think-mathematics): add Mathematics, Physics, and Computability reasoning methods`.

### Task 4: think-temporal (Temporal, Historical)

**Knowledge packs:** `docs/superpowers/knowledge-packs/{temporal,historical}.md`
**Files:** `skills/think-temporal/SKILL.md`, schemas/samples/reference for 2 modes
**Commit:** `feat(think-temporal): add Temporal and Historical reasoning methods`

### Task 5: think-probabilistic (Bayesian, Evidential)

**Knowledge packs:** `docs/superpowers/knowledge-packs/{bayesian,evidential}.md`
**Files:** `skills/think-probabilistic/SKILL.md`, schemas/samples/reference for 2 modes
**Special note:** The Bayesian schema must constrain all probability fields to `{"type": "number", "minimum": 0, "maximum": 1}`. The worked example in the knowledge pack shows real Bayes' theorem arithmetic — preserve that in the SKILL.md.
**Commit:** `feat(think-probabilistic): add Bayesian and Evidential reasoning methods`

### Task 6: think-causal (Causal, Counterfactual)

**Knowledge packs:** `docs/superpowers/knowledge-packs/{causal,counterfactual}.md`
**Files:** `skills/think-causal/SKILL.md`, schemas/samples/reference for 2 modes
**Commit:** `feat(think-causal): add Causal and Counterfactual reasoning methods`

### Task 7: think-strategic (GameTheory, Optimization, Constraint)

**Knowledge packs:** `docs/superpowers/knowledge-packs/{gametheory,optimization,constraint}.md`
**Files:** `skills/think-strategic/SKILL.md`, schemas/samples/reference for 3 modes
**Commit:** `feat(think-strategic): add GameTheory, Optimization, and Constraint reasoning methods`

### Task 8: think-analytical (Analogical, FirstPrinciples, MetaReasoning, Cryptanalytic)

**Knowledge packs:** `docs/superpowers/knowledge-packs/{analogical,firstprinciples,metareasoning,cryptanalytic}.md`
**Files:** `skills/think-analytical/SKILL.md`, schemas/samples/reference for 4 modes
**Commit:** `feat(think-analytical): add Analogical, FirstPrinciples, MetaReasoning, and Cryptanalytic methods`

### Task 9: think-scientific (ScientificMethod, SystemsThinking, FormalLogic)

**Knowledge packs:** `docs/superpowers/knowledge-packs/{scientificmethod,systemsthinking,formallogic}.md`
**Files:** `skills/think-scientific/SKILL.md`, schemas/samples/reference for 3 modes
**Commit:** `feat(think-scientific): add ScientificMethod, SystemsThinking, and FormalLogic methods`

### Task 10: think-engineering (Engineering, Algorithmic)

**Knowledge packs:** `docs/superpowers/knowledge-packs/{engineering,algorithmic}.md`
**Files:** `skills/think-engineering/SKILL.md`, schemas/samples/reference for 2 modes
**Commit:** `feat(think-engineering): add Engineering and Algorithmic reasoning methods`

### Task 11: think-academic (Synthesis, Argumentation, Critique, Analysis)

**Knowledge packs:** `docs/superpowers/knowledge-packs/{synthesis,argumentation,critique,analysis}.md`
**Files:** `skills/think-academic/SKILL.md`, schemas/samples/reference for 4 modes
**Commit:** `feat(think-academic): add Synthesis, Argumentation, Critique, and Analysis methods`

### Task 12: think-advanced (Recursive, Modal, Stochastic)

**Knowledge packs:** `docs/superpowers/knowledge-packs/{recursive,modal,stochastic}.md`
**Files:** `skills/think-advanced/SKILL.md`, schemas/samples/reference for 3 modes
**Commit:** `feat(think-advanced): add Recursive, Modal, and Stochastic reasoning methods`

---

## Task 13: Update router skill and mode-index for all 34 modes

**Files:**
- Modify: `skills/think/SKILL.md`
- Modify: `skills/think/mode-index.md`

- [ ] **Step 1: Update `skills/think/SKILL.md`**

Find the "## Available Modes (v0.1.0)" section. Replace the 3-row table with a table listing all 34 modes grouped by category, pointing to the correct category skill for each. Change the section heading to "## Available Modes (v0.2.0)". Remove the "Only three modes are available in this version. The remaining 31 ship in later versions." note.

The new table should look like:

```markdown
## Available Modes (v0.2.0)

All 34 modes are available in this version.

| Mode | Category skill | Use when |
|---|---|---|
| `sequential` | `think-standard` | Breaking a complex problem into ordered steps... |
| `shannon` | `think-standard` | Information-theoretic decomposition... |
| `hybrid` | `think-standard` | Combining multiple modes for cross-cutting problems |
| `inductive` | `think-core` | Forming general principles from specific observations... |
| `deductive` | `think-core` | Drawing specific conclusions from established premises... |
| `abductive` | `think-core` | Finding the best explanation for surprising observations |
| ... | ... | ... |
```

Include ALL 34 modes. Keep each "Use when" under 80 characters.

Remove the "Unavailable mode in v0.1.0" subsection entirely (no modes are unavailable now).

- [ ] **Step 2: Update `skills/think/mode-index.md`**

Replace the 3-mode decision tree with a full 34-mode decision tree. Group questions by category. Example structure:

```markdown
## Decision Tree

Ask yourself in this order:

1. **Is this formal logical inference from stated premises?** → `deductive`
2. **Do you have multiple observations and need to find a pattern?** → `inductive`
3. **Do you have a surprising observation and need the best explanation?** → `abductive`
4. **Does the problem involve probability, evidence updates, or belief revision?**
   - Evidence-based belief update → `bayesian`
   - Multi-source evidence evaluation → `evidential`
5. **Does the problem involve cause-and-effect?**
   - Tracing what caused what → `causal`
   - "What if X had been different" → `counterfactual`
6. **Does the problem involve strategy, optimization, or constraints?**
   - Multi-agent strategic interaction → `gametheory`
   - Finding the best allocation → `optimization`
   - Satisfying a set of rules → `constraint`
...etc
```

Include all 34 modes. Update the "Example Mappings" table to include 2-3 examples per category. Keep the file under ~100 lines.

- [ ] **Step 3: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat(think): update router and mode-index for all 34 modes"
```

---

## Task 14: Update taxonomy.md for all 34 modes

**File:** Modify: `reference/taxonomy.md`

- [ ] **Step 1: Read current file**

Currently lists 3 modes in "v0.1.0 Modes" and 31 in "Future Modes".

- [ ] **Step 2: Replace with full 34-mode taxonomy**

Change heading from "v0.1.0 Modes" to "v0.2.0 Modes". Add an entry per mode grouped by category. Each entry follows the pattern:

```markdown
### <ModeName> (`<slug>`)
- **Category:** think-<category>
- **Shape:** <one line describing the problem shape>
- **Signals:** <comma-separated phrases that indicate this mode fits>
- **Anti-signals:** <comma-separated phrases that indicate a different mode fits better>
```

Remove the entire "Future Modes" section.

- [ ] **Step 3: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "docs(taxonomy): graduate all 34 modes from future to v0.2.0"
```

---

## Task 15: Update commands/think.md slash command for all 34 modes

**File:** Modify: `commands/think.md`

- [ ] **Step 1: Update mode list in the "What to do" section**

The current `/think` slash command mentions only sequential, inductive, deductive. Expand the mode list to include all 34 modes grouped by category, with one-line descriptions. Remove the "Unavailable modes" section (no longer needed).

- [ ] **Step 2: Update decision tree**

Replace the 3-branch decision tree with the full 34-mode version (same content as `mode-index.md` Step 2). Or: since the slash command already references `mode-index.md`, simplify by saying "Follow the decision tree in `skills/think/mode-index.md`" and remove the inline tree.

- [ ] **Step 3: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat(command): update /think command for all 34 modes"
```

---

## Task 16: Expand test harness for all 34 modes

**File:** Modify: `test/harness.py`

- [ ] **Step 1: Update SAMPLES tuple**

The current SAMPLES list has 4 entries. Expand to include valid samples for all 34 modes plus the 1 invalid sample. New list:

```python
SAMPLES = [
    ("sequential-valid.json", "sequential.json", True),
    ("shannon-valid.json", "shannon.json", True),
    ("hybrid-valid.json", "hybrid.json", True),
    ("inductive-valid.json", "inductive.json", True),
    ("deductive-valid.json", "deductive.json", True),
    ("abductive-valid.json", "abductive.json", True),
    ("mathematics-valid.json", "mathematics.json", True),
    ("physics-valid.json", "physics.json", True),
    ("computability-valid.json", "computability.json", True),
    ("temporal-valid.json", "temporal.json", True),
    ("historical-valid.json", "historical.json", True),
    ("bayesian-valid.json", "bayesian.json", True),
    ("evidential-valid.json", "evidential.json", True),
    ("causal-valid.json", "causal.json", True),
    ("counterfactual-valid.json", "counterfactual.json", True),
    ("gametheory-valid.json", "gametheory.json", True),
    ("optimization-valid.json", "optimization.json", True),
    ("constraint-valid.json", "constraint.json", True),
    ("analogical-valid.json", "analogical.json", True),
    ("firstprinciples-valid.json", "firstprinciples.json", True),
    ("metareasoning-valid.json", "metareasoning.json", True),
    ("cryptanalytic-valid.json", "cryptanalytic.json", True),
    ("scientificmethod-valid.json", "scientificmethod.json", True),
    ("systemsthinking-valid.json", "systemsthinking.json", True),
    ("formallogic-valid.json", "formallogic.json", True),
    ("engineering-valid.json", "engineering.json", True),
    ("algorithmic-valid.json", "algorithmic.json", True),
    ("synthesis-valid.json", "synthesis.json", True),
    ("argumentation-valid.json", "argumentation.json", True),
    ("critique-valid.json", "critique.json", True),
    ("analysis-valid.json", "analysis.json", True),
    ("recursive-valid.json", "recursive.json", True),
    ("modal-valid.json", "modal.json", True),
    ("stochastic-valid.json", "stochastic.json", True),
    ("sequential-invalid.json", "sequential.json", False),
]
```

35 entries total (34 valid + 1 invalid).

- [ ] **Step 2: Run harness**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/harness.py
```

Expected: `Results: 35 passed, 0 failed`

If ANY mode fails, investigate: is the schema too strict? Is the sample incomplete? Fix and re-run.

- [ ] **Step 3: Run all other tests**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/test_plugin_json.py && python test/test_skill_frontmatter.py
```

The frontmatter validator currently expects exactly 3 SKILL.md files. Update `test/test_skill_frontmatter.py` to expect 12:

```python
assert len(skills) == 12, f"Expected 12 SKILL.md files, found {len(skills)}"
```

Re-run and verify it passes.

- [ ] **Step 4: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "test: expand harness to cover all 34 modes"
```

---

## Task 17: Update CHANGELOG and tag v0.2.0

**Files:**
- Modify: `CHANGELOG.md`
- Git: Move v0.2.0 tag

- [ ] **Step 1: Add v0.2.0 entry to CHANGELOG.md**

Insert above the v0.1.0 entry:

```markdown
## [0.2.0] - 2026-04-12

### Added
- **31 new reasoning modes** across 10 new category skills, bringing total to 34 modes:
  - `think-mathematics`: Mathematics, Physics, Computability
  - `think-temporal`: Temporal, Historical
  - `think-probabilistic`: Bayesian, Evidential
  - `think-causal`: Causal, Counterfactual
  - `think-strategic`: GameTheory, Optimization, Constraint
  - `think-analytical`: Analogical, FirstPrinciples, MetaReasoning, Cryptanalytic
  - `think-scientific`: ScientificMethod, SystemsThinking, FormalLogic
  - `think-engineering`: Engineering, Algorithmic
  - `think-academic`: Synthesis, Argumentation, Critique, Analysis
  - `think-advanced`: Recursive, Modal, Stochastic
- Expanded `think-core` with Abductive; expanded `think-standard` with Shannon and Hybrid
- 31 new JSON Schemas in `test/schemas/`
- 31 new sample valid thoughts in `test/samples/`
- 31 new reference/output-format docs
- Updated `/think` command and router to know all 34 modes
- Expanded test harness to 35 cases (34 valid + 1 invalid)

### Changed
- `skills/think/SKILL.md`: Available Modes table now lists all 34 modes
- `skills/think/mode-index.md`: Decision tree covers all 34 modes
- `reference/taxonomy.md`: All 34 modes graduated from "Future" to "v0.2.0 Modes"
- `commands/think.md`: Mode list and decision tree updated

### Verified
- Automated: 35 schema validations pass, 12 SKILL.md frontmatter validations pass
- Manual smoke test deferred to post-release (v0.2.1 task)
```

- [ ] **Step 2: Commit CHANGELOG and tag**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add CHANGELOG.md && git commit -m "chore: release v0.2.0" && git tag -a v0.2.0 -m "v0.2.0: All 34 reasoning modes available"
```

- [ ] **Step 3: Final verification**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git log --oneline | head -20 && git tag && python test/test_plugin_json.py && python test/test_skill_frontmatter.py && python test/harness.py
```

All tests should pass. Tag `v0.2.0` should exist alongside `v0.1.0`.

---

## Self-Review Notes

**Spec coverage:**
- ✅ All 31 new modes (Tasks 1-12 cover them)
- ✅ Router update (Task 13)
- ✅ Taxonomy update (Task 14)
- ✅ Slash command update (Task 15)
- ✅ Test harness expansion (Task 16)
- ✅ CHANGELOG + tag (Task 17)

**Parallelization opportunities:**
- Tasks 1-12 are mostly independent and can run as parallel subagents (with care to avoid conflicting edits to shared files — each task only touches its own `skills/think-<category>/` directory plus new files in `test/schemas/`, `test/samples/`, `reference/output-formats/`)
- Tasks 13-16 depend on the outputs of Tasks 1-12 and must run sequentially after all category skills are in place
- Task 17 is the final gate

**Deferred to later plans:**
- Phase 4: visual-exporter agent, render-diagram.py, per-mode visual-grammar.md
- Phase 5: full manual smoke testing of all 34 modes end-to-end (partial coverage in this plan via automated schema validation)

**No placeholders verified.** Every task specifies knowledge pack paths, file paths, commit messages, and verification commands.
