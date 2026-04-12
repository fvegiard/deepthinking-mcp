# deepthinking-plugin Phase 5: Smoke Tests + Expanded Formats + HTML Dashboards Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development where the plan uses parallelizable tasks; do the sequential pieces inline.

**Goal:** Ship v0.4.0 with three deliverables: (1) **automated end-to-end smoke tests** for all 34 modes via headless `claude -p`, (2) **9 additional visual-export format grammars** (ASCII, GraphML, HTML, JSON, Markdown, Modelica, TikZ, UML, Dashboard) added as shared grammar files, (3) **interactive HTML dashboard template** that takes a thought JSON and produces a standalone single-file HTML page with Mermaid rendering + JSON explorer.

**Architecture insight:** v0.3.0 already has per-mode grammar files (`reference/visual-grammar/<mode>.md`) covering Mermaid + DOT. The 9 additional formats are **format-level, not mode-level**: they describe how to encode ANY mode's thought into that format using the shared conventions. So we ship 9 format-grammar files at `reference/visual-grammar/formats/<format>.md`, each ~100-200 lines, explaining the encoding rules with generic examples. The visual-exporter agent consults both: `reference/visual-grammar/<mode>.md` for the semantic structure, and `reference/visual-grammar/formats/<format>.md` for the surface syntax.

**Tech Stack:** Python 3 (smoke test runner + dashboard renderer), markdown (format grammars), HTML/JS/Mermaid CDN (dashboard template). No new dependencies; `jsonschema` from v0.1.0 covers validation.

**Working directory:** `C:/Users/danie/Dropbox/Github/deepthinking-plugin/` (tagged v0.3.0).

## File Structure at End of Phase 5

```
test/smoke/
├── prompts.json              # 34 test prompts, one per mode
├── run-all-modes.py          # Headless runner: invokes claude -p for each, validates JSON
└── captured/                 # (gitignored) per-run captured outputs

reference/visual-grammar/formats/
├── ascii.md                  # ASCII art encoding rules
├── graphml.md                # GraphML (XML) encoding rules
├── html.md                   # Semantic HTML encoding rules
├── json.md                   # JSON (native + flattened) encoding rules
├── markdown.md               # Markdown (tables, lists, headings) encoding rules
├── modelica.md               # Modelica physical-system-modeling encoding rules
├── tikz.md                   # LaTeX TikZ encoding rules
├── uml.md                    # UML class/sequence/state diagram encoding rules
└── dashboard.md              # Interactive HTML dashboard encoding rules

reference/
└── html-dashboard-template.html   # Single-file HTML template with Mermaid CDN

scripts/
└── render-html-dashboard.py  # Injects thought JSON + mermaid source into the template

agents/visual-exporter.md     # UPDATED: knows about all 11 formats (mermaid, dot, + 9)
commands/think-render.md      # UPDATED: accepts all 11 formats

test/visual/
└── validate-formats.py       # Syntactic validator for the 9 new format grammars

CHANGELOG.md                  # v0.4.0 entry
README.md                     # Updated with new format list + dashboard section
```

## Parallelization

- **Stage 2 (smoke tests):** sequential (one run of the script that iterates 34 modes)
- **Stage 3 (9 format grammars):** parallelizable by batching into 3 subagent waves of 3 formats each
- **Stage 4 (dashboard):** sequential (template + script + integration)

---

## Stage 2: Automated Smoke Test Runner

### Task 2.1: Write `test/smoke/prompts.json`

**File:** Create: `test/smoke/prompts.json`

- [ ] **Step 1: Create the directory**

```bash
mkdir -p "C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/smoke/captured"
```

- [ ] **Step 2: Write the prompts file**

Write `test/smoke/prompts.json` with an array of 34 objects, each having `mode` and `prompt` fields. Use concrete, realistic scenarios per mode (not generic "test input"). Here are the 34 prompts to use verbatim:

```json
[
  {"mode": "sequential", "prompt": "Break down the steps to migrate our Postgres database to a new region."},
  {"mode": "shannon", "prompt": "Decompose the problem of reducing p99 latency on our user-search API."},
  {"mode": "hybrid", "prompt": "Should we rewrite the auth system in Rust? This requires comparing safety, performance, team expertise, and migration cost."},
  {"mode": "inductive", "prompt": "Given these three incidents, what pattern do they share? (1) Deploy on Mon 2026-04-06 failed with DB timeout. (2) Deploy on Wed 2026-04-08 failed with DB timeout. (3) Deploy on Fri 2026-04-10 failed with DB timeout."},
  {"mode": "deductive", "prompt": "If all users in the admin group can edit posts, and Alice is in the admin group, can Alice edit posts?"},
  {"mode": "abductive", "prompt": "Users on the analytics dashboard see 503 errors only on Tuesday mornings 9-10 AM. Database CPU spikes to 95% during that window. What is the best explanation?"},
  {"mode": "mathematics", "prompt": "Prove that the square root of 2 is irrational."},
  {"mode": "physics", "prompt": "Analyze the electromagnetic field tensor F_{mu-nu} in flat Minkowski space. What are its invariants and what do they represent physically?"},
  {"mode": "computability", "prompt": "Show why the Halting Problem is undecidable using diagonalization."},
  {"mode": "temporal", "prompt": "Incident timeline: 10:00 502 errors begin on API, 10:04 CPU alert fires on worker-3, 10:07 upstream deploy completes. What does the ordering tell us about causation?"},
  {"mode": "historical", "prompt": "Is our planned 2026 Postgres cross-region migration analogous to the 2024 incident where a similar migration caused 4 hours of downtime?"},
  {"mode": "bayesian", "prompt": "Is the 502 error rate increase caused by the new cache config? Use Bayesian inference with a sensible prior, evidence on (a) temporal correlation, (b) error code specificity, (c) isolated change window. Compute the posterior."},
  {"mode": "evidential", "prompt": "Sensor fusion scenario: radar says 'vehicle' with 0.7 confidence; camera says 'pedestrian' with 0.6 confidence; lidar says 'unknown' with 0.4 confidence. What is the most plausible classification using Dempster-Shafer?"},
  {"mode": "causal", "prompt": "Did the cache eviction policy change cause the p99 latency spike last week, or is it a confounder like the same-day memory limit change?"},
  {"mode": "counterfactual", "prompt": "Would we have avoided the production outage if we had rolled back the deployment before 2 AM?"},
  {"mode": "gametheory", "prompt": "Two competing microservices choose their cache TTL independently. Short TTL means fresher data but higher backend load; long TTL reduces load but risks stale data. Is there a Nash equilibrium?"},
  {"mode": "optimization", "prompt": "Allocate 100 CPU cores across 5 services to maximize total throughput, subject to each service needing at least 8 cores and service A needing twice as much as service B."},
  {"mode": "constraint", "prompt": "Assign 4 engineers (Alice, Bob, Carol, Dave) to 4 roles (backend, frontend, DevOps, QA). Alice cannot do DevOps. Bob must do backend or frontend. Carol refuses QA. Find any feasible assignment."},
  {"mode": "analogical", "prompt": "When designing our caching layer, is it more like a CPU cache (LRU, fast, small) or like a CDN (geographic distribution, eventual consistency)? Map the analogy and name where it breaks down."},
  {"mode": "firstprinciples", "prompt": "From first principles, what does 'authentication' actually require? Decompose to fundamentals and rebuild."},
  {"mode": "metareasoning", "prompt": "I've been using sequential reasoning for 6 thoughts on this problem without convergence. Am I in the right mode? What should I switch to, if anything?"},
  {"mode": "cryptanalytic", "prompt": "Given 50 authentication failures over 24 hours, all targeting the same 3 usernames, with password attempts from 47 distinct IPs, is this credential stuffing or user forgetfulness? Use Decibans-weighted evidence."},
  {"mode": "scientificmethod", "prompt": "Hypothesis: latency spikes are caused by GC pauses. Design an experiment to test this hypothesis with clear falsifiability criteria."},
  {"mode": "systemsthinking", "prompt": "Why does adding more developers to our team keep slowing down delivery? Identify the feedback loops and systems archetype at work."},
  {"mode": "formallogic", "prompt": "Prove using natural deduction: If the health check fails, the service is degraded. The service is not degraded. Therefore, the health check does not fail. (Modus Tollens)"},
  {"mode": "engineering", "prompt": "Should we use a managed database service or self-host? Trade off reliability, performance, cost, and operational burden."},
  {"mode": "algorithmic", "prompt": "Given N=1 million items arriving in a stream, find the top-K=100 most recent items efficiently. Choose the algorithm and data structure, analyze complexity."},
  {"mode": "synthesis", "prompt": "Synthesize findings from three studies on remote-work productivity: Study A (2021) says +13% productivity; Study B (2023) says no significant change; Study C (2024) says -8% for junior engineers only."},
  {"mode": "argumentation", "prompt": "Argue that we should adopt feature flags for all new releases. Use the Toulmin model: claim, grounds, warrant, backing, qualifier, rebuttal."},
  {"mode": "critique", "prompt": "Critique this proposed architecture: a monolithic Rails app with a single Postgres database for a team of 50 engineers building a fintech product with strict SLAs. What would a skeptical senior engineer ask?"},
  {"mode": "analysis", "prompt": "Analyze the failure mode of our authentication service. Use a layered approach: surface symptoms, structural issues, patterns across incidents, and synthesized insights."},
  {"mode": "recursive", "prompt": "Design a recursive solution for deeply nested config validation where each nested config follows the same rules as the parent. Specify base case, recursive case, and halting condition."},
  {"mode": "modal", "prompt": "In our RBAC system: in which worlds MUST Alice have admin access (necessity), and in which worlds COULD she have it (possibility)? Use deontic modality."},
  {"mode": "stochastic", "prompt": "Model the distribution of queue wait times for our ticket system using an M/M/1 queue. Arrival rate 10/hr, service rate 12/hr. Compute expected wait."}
]
```

- [ ] **Step 3: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "test: add 34-mode smoke test prompts"
```

### Task 2.2: Write `test/smoke/run-all-modes.py`

**File:** Create: `test/smoke/run-all-modes.py`

Write a Python script that:
1. Loads `test/smoke/prompts.json`
2. For each entry: invokes `claude --plugin-dir <plugin_root> --bare -p "/deepthinking-plugin:think <mode> <prompt>"` via subprocess, captures stdout with a timeout of 180 seconds
3. Extracts the first JSON code block from the response
4. Validates it against `test/schemas/<mode>.json`
5. Saves output to `test/smoke/captured/<mode>-run.json` and a summary of validation failures
6. Reports total pass/fail count

Full script:

```python
"""
Automated smoke test runner for all 34 deepthinking-plugin modes.

Invokes `claude --plugin-dir <plugin_root> --bare -p` for each mode's test prompt,
extracts the JSON code block from Claude's response, and validates it against the
mode's JSON Schema. Writes per-mode captured outputs to test/smoke/captured/.

Usage:
    cd <plugin-root>
    python test/smoke/run-all-modes.py

Environment:
    SMOKE_TIMEOUT   seconds per mode (default 180)
    SMOKE_MODE      if set, only run this single mode (e.g. SMOKE_MODE=bayesian)
"""

import json
import os
import re
import subprocess
import sys
from pathlib import Path

try:
    import jsonschema
except ImportError:
    print("ERROR: jsonschema not installed. Run: pip install jsonschema")
    sys.exit(1)

PLUGIN_ROOT = Path(__file__).parent.parent.parent
SMOKE_DIR = PLUGIN_ROOT / "test" / "smoke"
CAPTURED_DIR = SMOKE_DIR / "captured"
SCHEMAS_DIR = PLUGIN_ROOT / "test" / "schemas"
PROMPTS_FILE = SMOKE_DIR / "prompts.json"

TIMEOUT = int(os.environ.get("SMOKE_TIMEOUT", "180"))
SINGLE_MODE = os.environ.get("SMOKE_MODE")

# Match JSON code blocks in Claude's output.
JSON_BLOCK_RE = re.compile(r"```json\s*\n(.*?)\n```", re.DOTALL)


def extract_json_block(text):
    """Return the first ```json ... ``` block parsed as Python, or None."""
    for match in JSON_BLOCK_RE.finditer(text):
        candidate = match.group(1).strip()
        try:
            return json.loads(candidate)
        except json.JSONDecodeError:
            continue
    return None


def run_mode(mode, prompt):
    """Invoke claude -p for one mode. Return (captured_text, exit_code, timed_out)."""
    cmd = [
        "claude",
        "--plugin-dir",
        str(PLUGIN_ROOT),
        "--bare",
        "-p",
        f'/deepthinking-plugin:think {mode} "{prompt}"',
    ]
    try:
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            encoding="utf-8",
            timeout=TIMEOUT,
            errors="replace",
        )
        return result.stdout, result.returncode, False
    except subprocess.TimeoutExpired:
        return "", -1, True


def validate_thought(thought, mode):
    """Validate a parsed thought object against the mode's schema."""
    schema_path = SCHEMAS_DIR / f"{mode}.json"
    if not schema_path.exists():
        return False, f"missing schema at {schema_path}"
    with open(schema_path, "r", encoding="utf-8") as f:
        schema = json.load(f)
    try:
        jsonschema.validate(thought, schema)
        return True, "ok"
    except jsonschema.ValidationError as e:
        return False, e.message


def main():
    CAPTURED_DIR.mkdir(parents=True, exist_ok=True)
    with open(PROMPTS_FILE, "r", encoding="utf-8") as f:
        prompts = json.load(f)

    if SINGLE_MODE:
        prompts = [p for p in prompts if p["mode"] == SINGLE_MODE]
        if not prompts:
            print(f"ERROR: no prompt for mode '{SINGLE_MODE}'")
            return 1

    results = []
    for entry in prompts:
        mode = entry["mode"]
        prompt = entry["prompt"]
        print(f"[{mode}] running (timeout {TIMEOUT}s)...", flush=True)

        captured, exit_code, timed_out = run_mode(mode, prompt)

        # Save raw capture
        raw_path = CAPTURED_DIR / f"{mode}-raw.txt"
        raw_path.write_text(captured, encoding="utf-8")

        if timed_out:
            print(f"  FAIL: timeout after {TIMEOUT}s")
            results.append((mode, False, "timeout"))
            continue

        if exit_code != 0 and not captured:
            print(f"  FAIL: exit {exit_code} with no output")
            results.append((mode, False, f"exit {exit_code}"))
            continue

        thought = extract_json_block(captured)
        if thought is None:
            print("  FAIL: no JSON code block found in output")
            results.append((mode, False, "no JSON block"))
            continue

        # Save parsed JSON for inspection
        json_path = CAPTURED_DIR / f"{mode}-parsed.json"
        json_path.write_text(json.dumps(thought, indent=2), encoding="utf-8")

        ok, reason = validate_thought(thought, mode)
        if ok:
            print("  PASS")
            results.append((mode, True, "ok"))
        else:
            print(f"  FAIL: schema violation: {reason}")
            results.append((mode, False, reason))

    print()
    passed = sum(1 for _, ok, _ in results if ok)
    failed = len(results) - passed
    print(f"===== Summary: {passed}/{len(results)} passed, {failed} failed =====")
    for mode, ok, reason in results:
        marker = "PASS" if ok else "FAIL"
        print(f"  {marker:4s}  {mode:20s}  {reason}")

    return 0 if failed == 0 else 1


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 4: Gitignore the captured directory**

Add to `.gitignore`:

```
test/smoke/captured/
```

- [ ] **Step 5: Run the script**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/smoke/run-all-modes.py
```

Expected: 34/34 passed. If any fail, investigate per-mode (may be schema too strict, prompt ambiguous, or Claude produced malformed JSON — fix and re-run with `SMOKE_MODE=<failing_mode>` to iterate quickly).

- [ ] **Step 6: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "test(smoke): add automated runner that validates all 34 modes end-to-end"
```

---

## Stage 3: 9 Format Grammar Files

### Task 3.1: Create `reference/visual-grammar/formats/` directory and write 3 format grammars (ASCII, JSON, Markdown)

**Files:**
- `reference/visual-grammar/formats/ascii.md`
- `reference/visual-grammar/formats/json.md`
- `reference/visual-grammar/formats/markdown.md`

Each file follows this pattern:

```markdown
# Format Grammar: <Format Name>

How to encode a deepthinking-plugin thought into <format>.

## Format Overview

<1-2 paragraphs explaining the format, its audience, and strengths>

## Encoding Rules

<how to map thought fields to format elements>

## Template

\`\`\`<format>
<generic template>
\`\`\`

## Worked Example

Using a [mode] thought as input:

\`\`\`json
<input thought excerpt>
\`\`\`

Produces:

\`\`\`<format>
<encoded output>
\`\`\`

## Per-Mode Considerations

<notes on which modes work well/poorly in this format>

## Rendering Tools

<external tools needed, if any, with install commands>
```

Content specifics:

**ascii.md**: Explain how to render thoughts as text trees using `├──`, `│`, `└──` characters; show templates for sequential (vertical chain), bayesian (hierarchical with probability labels), and causal (boxed nodes with ASCII arrows). Include the `tree(1)` style and the `box-drawing` style. No external tools.

**json.md**: Explain that the thought itself IS JSON — this format grammar covers three subformats:
1. **Native**: the thought object as-is (`JSON.stringify(thought, null, 2)`)
2. **Flattened**: paths like `hypothesis.prior.probability = 0.3` as flat key-value pairs
3. **JSON Lines**: for sequential/shannon chains, one thought per line
Show templates and examples for each.

**markdown.md**: Explain encoding thoughts as markdown with headings for each field, tables for structured sub-objects (e.g., Bayesian evidence array → table), and bullet lists for simple arrays. Show templates for at least 3 mode styles (sequential → numbered list of thoughts, bayesian → table of evidence with posterior at the bottom, causal → headings with mechanism tables).

**Commit:** `feat(visual-grammar/formats): add ASCII, JSON, and Markdown format grammars`

### Task 3.2: Write 3 more format grammars (GraphML, HTML, TikZ)

**Files:**
- `reference/visual-grammar/formats/graphml.md`
- `reference/visual-grammar/formats/html.md`
- `reference/visual-grammar/formats/tikz.md`

**graphml.md**: XML-based graph exchange format. Explain the `<node>` and `<edge>` elements, `<data key="...">` for custom attributes (probability, confidence, mechanism), `<graph edgedefault="directed">`. Show a template and a worked example using a Bayesian thought. Mention yEd, Gephi, and Cytoscape as consumers.

**html.md**: Semantic HTML with minimal inline styling. Use `<article>` for the top-level thought, `<section>` for each field group, `<table>` for structured sub-objects, `<details>/<summary>` for collapsible evidence lists. Include a small inline `<style>` block for readable defaults. **Not** the interactive dashboard — that's `dashboard.md`. This is static semantic HTML a browser can render directly.

**tikz.md**: LaTeX graphics using TikZ. Explain `\tikz` / `\begin{tikzpicture}`, node shapes (`draw=black, fill=blue!20, rectangle, rounded corners`), edges with labels, matrices for payoff diagrams. Show a worked example using a causal thought. Mention Overleaf and local LaTeX as consumers.

**Commit:** `feat(visual-grammar/formats): add GraphML, HTML, and TikZ format grammars`

### Task 3.3: Write 3 more format grammars (Modelica, UML, Dashboard)

**Files:**
- `reference/visual-grammar/formats/modelica.md`
- `reference/visual-grammar/formats/uml.md`
- `reference/visual-grammar/formats/dashboard.md`

**modelica.md**: Physical-systems modeling language. Primarily useful for Physics, Systems Thinking, and Stochastic modes. Explain `model Foo ... end Foo;`, `Real`, `Integer`, equations with `=`, and `connect(a, b)` for flows. Show a worked example using a systemsthinking thought (feedback loops as Modelica equations).

**uml.md**: Unified Modeling Language. Support three UML diagram types: **class diagrams** (for analytical modes with entities + relationships), **sequence diagrams** (for temporal, sequential, scientific method), and **state diagrams** (for computability, stochastic). Show templates for each and a worked example using a scientific-method thought as a sequence diagram.

**dashboard.md**: References the interactive HTML dashboard template (built in Stage 4) and explains what the dashboard can do — mode-adaptive rendering, client-side Mermaid rendering, JSON explorer. This file is essentially the docstring for the `/think-render dashboard` flow. It points to `reference/html-dashboard-template.html` and `scripts/render-html-dashboard.py`.

**Commit:** `feat(visual-grammar/formats): add Modelica, UML, and Dashboard format grammars`

### Task 3.4: Update `agents/visual-exporter.md` to know all 11 formats

**File:** Modify: `agents/visual-exporter.md`

Update the `description` field in frontmatter to mention all 11 formats. Update the workflow section to explain that the agent consults BOTH `reference/visual-grammar/<mode>.md` (semantic structure) AND `reference/visual-grammar/formats/<format>.md` (surface syntax) when producing output for any format other than mermaid/dot. Mention the dashboard format specifically and note it invokes `scripts/render-html-dashboard.py`.

**Commit:** `feat(agent): update visual-exporter to route 11 formats via format grammar files`

### Task 3.5: Update `commands/think-render.md` to accept all 11 formats

**File:** Modify: `commands/think-render.md`

Update `description` and `argument-hint` to include all 11 formats. Update the "determine target format" step to accept `ascii`, `graphml`, `html`, `json`, `markdown`, `modelica`, `tikz`, `uml`, `dashboard` in addition to existing `mermaid`, `dot`, `svg`, `png`. For `dashboard`: explain that the command invokes `scripts/render-html-dashboard.py` with the captured thought JSON as input.

**Commit:** `feat(command): expand /think-render to accept all 11 formats`

---

## Stage 4: Interactive HTML Dashboard

### Task 4.1: Write `reference/html-dashboard-template.html`

**File:** Create: `reference/html-dashboard-template.html`

A single-file HTML template with:

1. **CDN-loaded Mermaid** (`https://cdn.jsdelivr.net/npm/mermaid@latest/dist/mermaid.min.js`) for client-side diagram rendering
2. **Placeholder tokens** that the Python renderer will substitute:
   - `{{MODE}}` — the thought's mode slug
   - `{{MODE_DISPLAY}}` — human-readable mode name
   - `{{THOUGHT_JSON}}` — the full thought as JSON (for the viewer)
   - `{{MERMAID_SOURCE}}` — the mermaid diagram source for this thought
   - `{{TIMESTAMP}}` — ISO timestamp of when the dashboard was generated
3. **Layout sections** (using CSS flexbox for responsive):
   - Header with mode badge + generation timestamp
   - Left pane: rendered Mermaid diagram (the visual-exporter's Mermaid output)
   - Right pane: collapsible JSON explorer showing the thought fields
   - Bottom: "Export" buttons (copy JSON to clipboard, download as JSON, print)
4. **No external CSS/JS dependencies** beyond the Mermaid CDN
5. **Dark/light mode** toggled via `prefers-color-scheme`

Target size: 300-500 lines (including inline CSS + JS + templates).

**Commit:** `feat(dashboard): add interactive HTML dashboard template`

### Task 4.2: Write `scripts/render-html-dashboard.py`

**File:** Create: `scripts/render-html-dashboard.py`

A Python script that:
1. Takes `--thought <path-to-thought.json>` and `--output <dashboard.html>` arguments, plus optional `--mermaid <path-to-mermaid-source>` for pre-rendered Mermaid (otherwise it generates a minimal fallback mermaid from the thought)
2. Loads the HTML template at `reference/html-dashboard-template.html`
3. Substitutes the placeholder tokens
4. Writes the final HTML to `--output`
5. Prints the output path on success

Keep the script under 100 lines. No dependencies beyond Python stdlib.

**Commit:** `feat(dashboard): add render-html-dashboard.py renderer`

### Task 4.3: Integration test for the dashboard

**File:** Create: `test/visual/test-dashboard.py`

A test that:
1. Reads an existing sample fixture (`test/samples/bayesian-valid.json`)
2. Writes a minimal placeholder Mermaid source for it
3. Calls `scripts/render-html-dashboard.py` with both
4. Verifies the output HTML file exists and contains the expected placeholders filled in (e.g., the mode slug, the JSON content)
5. Cleans up the output file

**Commit:** `test(dashboard): add integration test for html dashboard renderer`

---

## Stage 5: Release v0.4.0

### Task 5.1: Run full test suite

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin"
python test/test_plugin_json.py
python test/test_skill_frontmatter.py
python test/harness.py
python test/visual/validate-mermaid.py
python test/visual/validate-dot.py
python test/visual/test-dashboard.py
```

All must pass. The smoke test from Stage 2 is run separately (it's expensive and shouldn't be in the default test loop).

### Task 5.2: Update README.md

Add sections:
- **Automated smoke tests**: how to run `test/smoke/run-all-modes.py`
- **11 export formats**: list them with one-line descriptions
- **Interactive dashboard**: how `/think-render dashboard` works

### Task 5.3: Update CHANGELOG.md

Add v0.4.0 entry above v0.3.0:

```markdown
## [0.4.0] - 2026-04-12

### Added
- Automated end-to-end smoke test runner (`test/smoke/run-all-modes.py`) that invokes `claude -p` for each of the 34 modes and validates the JSON output against its schema
- 9 new format grammar files at `reference/visual-grammar/formats/{ascii,graphml,html,json,markdown,modelica,tikz,uml,dashboard}.md`
- Interactive HTML dashboard template at `reference/html-dashboard-template.html` with Mermaid CDN rendering
- `scripts/render-html-dashboard.py` — injects a thought into the template
- `/think-render dashboard` support in `commands/think-render.md`
- Updated `agents/visual-exporter.md` to route 11 total formats

### Verified
- All 34 modes pass automated smoke tests end-to-end (captured outputs validate against schemas)
- All format grammar files have valid syntax
- Dashboard renders a sample thought as a standalone HTML file
```

### Task 5.4: Commit and tag v0.4.0

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "chore: release v0.4.0 with smoke tests, formats, and dashboards" && git tag -a v0.4.0 -m "v0.4.0: Automated smoke tests + 9 additional formats + interactive HTML dashboard"
```

---

## Self-Review Notes

**Spec coverage:**
- ✅ Automated smoke tests for all 34 modes (Stage 2)
- ✅ 9 additional format grammars (Stage 3)
- ✅ Interactive HTML dashboard (Stage 4)
- ✅ Release tag (Stage 5)

**Key trade-offs:**
- Format grammars are **shared** (format-level) rather than **per-mode-per-format** (34 × 9 = 306 files). This is an order-of-magnitude reduction in surface area without losing functionality — the visual-exporter agent combines the mode's semantic grammar (`visual-grammar/<mode>.md`) with the format's syntactic grammar (`visual-grammar/formats/<format>.md`) at invocation time.
- Dashboard template is **mode-adaptive** rather than **mode-specific** — one template handles all modes via client-side Mermaid rendering + JSON explorer. Future phases can add mode-specific widgets as enhancements.
- Smoke tests use `claude -p` with `--bare` to minimize startup overhead and avoid session quota concerns.

**No placeholders verified.** Every task specifies exact content or generation rules. The format grammar files and dashboard template are the only "describe what to produce, don't show it verbatim" tasks — but they're bounded by size limits and have per-format specifics.
