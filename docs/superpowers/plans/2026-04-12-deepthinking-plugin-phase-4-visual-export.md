# deepthinking-plugin Phase 4: Visual Export Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add visual export capability to `deepthinking-plugin` so any structured thought produced by `/think <mode>` can be rendered as a diagram (Mermaid, DOT, ASCII, optionally SVG/PNG via external tools). Ship as v0.3.0 after all 34 modes support at least Mermaid + DOT export.

**Architecture:** A dedicated agent at `agents/visual-exporter.md` takes a structured thought (JSON) and a format name, reads per-mode diagram rules from `reference/visual-grammar.md`, and produces diagram source. For formats that need rendering (SVG/PNG), the agent invokes `scripts/render-diagram.py`, which wraps `graphviz` (`dot` binary) and `mermaid-cli` (`mmdc`). If those binaries aren't installed the script gracefully prints the source instead of failing.

**Tech Stack:** Markdown (visual-grammar rules per mode), Python (render-diagram.py wrapper), optional binaries (graphviz, @mermaid-js/mermaid-cli). No changes to the plugin's Node/TS runtime — there is none.

**Source knowledge:** The original deepthinking-mcp has 42 visual exporter files (24 mode-specific + 15 utility builders + 3 root) producing 11 formats. Those are the **reference design**, not code to port. We distill each mode-specific exporter into a ~1KB `reference/visual-grammar.md` section describing what the diagram SHOULD look like (node structure, edge semantics, colors, labels), so Claude can generate it directly without a builder class. The `src/export/visual/modes/<mode>.ts` files in `C:/Users/danie/Dropbox/Github/deepthinking-mcp/src/export/visual/modes/` are the authoritative reference for each mode's visual grammar.

**Working directory:** `C:/Users/danie/Dropbox/Github/deepthinking-plugin/` (already tagged v0.2.0).

## Scope

Phase 4 ships **two guaranteed formats per mode**: Mermaid and DOT. These are the formats Claude produces fluently from the reference rules. The other 9 formats (ASCII, SVG, GraphML, HTML, JSON, Markdown, Modelica, TikZ, UML) are **best-effort** via the render script or direct Claude generation where appropriate; they are not required for the phase to ship.

**Mode coverage:** All 34 modes get at least a Mermaid + DOT visual grammar entry. The 24 modes that had dedicated exporters in the MCP get richer grammar entries; the 10 that used generic exporters get default grammar rules.

**Out of scope for v0.3.0:** Interactive HTML dashboards, live diagram editors, native SVG rendering without external tools, any of the 14 fluent builder classes from the MCP (those were runtime TS code; we produce equivalent output via prompt rules).

## File Structure at End of Phase 4

New files (relative to plugin root):

```
agents/
└── visual-exporter.md               # Claude subagent that routes thought + format → diagram source

reference/
├── visual-grammar.md                # Shared grammar rules (node shapes, color palettes, conventions)
└── visual-grammar/                  # Per-mode rules (one file per mode)
    ├── sequential.md
    ├── shannon.md
    ├── hybrid.md
    ├── inductive.md
    ├── deductive.md
    ├── abductive.md
    ├── mathematics.md
    ├── physics.md
    ├── computability.md
    ├── temporal.md
    ├── historical.md
    ├── bayesian.md
    ├── evidential.md
    ├── causal.md
    ├── counterfactual.md
    ├── gametheory.md
    ├── optimization.md
    ├── constraint.md
    ├── analogical.md
    ├── firstprinciples.md
    ├── metareasoning.md
    ├── cryptanalytic.md
    ├── scientificmethod.md
    ├── systemsthinking.md
    ├── formallogic.md
    ├── engineering.md
    ├── algorithmic.md
    ├── synthesis.md
    ├── argumentation.md
    ├── critique.md
    ├── analysis.md
    ├── recursive.md
    ├── modal.md
    └── stochastic.md                # 34 per-mode grammar files

scripts/
└── render-diagram.py                # DOT → SVG/PNG and Mermaid → SVG/PNG wrapper

commands/
└── think-render.md                  # NEW slash command: /think-render [format] (renders the last JSON output in conversation)

test/
└── visual/                          # NEW test dir
    ├── validate-mermaid.py          # Verifies all 34 modes produce parseable Mermaid
    ├── validate-dot.py              # Verifies all 34 modes produce parseable DOT
    └── fixtures/                    # Reference Mermaid and DOT outputs per mode (optional, for regression)
```

Additionally: update `skills/think/SKILL.md` and `commands/think.md` to mention that the output can be rendered via `/think-render` or by asking the visual-exporter agent directly.

## Parallelization Strategy

The 34 per-mode visual-grammar files (Tasks 2-11) are independent and can be authored in parallel by one subagent per category (10 subagents). Each subagent reads the relevant MCP visual exporter files and distills them into grammar rules. The shared grammar file, agent file, render script, new command, and tests run sequentially.

---

## Task 1: Shared visual-grammar.md

**File:** Create: `reference/visual-grammar.md`

- [ ] **Step 1: Write shared grammar conventions**

The file establishes conventions used by all per-mode grammar files:

- **Node shapes** — which shape means what (rectangle = concrete thing, ellipse = concept, diamond = decision, cylinder = data source, cloud = external system)
- **Color palette** — a small set of semantic colors (green = success/supporting, red = failure/contradicting, blue = neutral, orange = uncertainty, purple = meta)
- **Edge semantics** — solid = direct dependency, dashed = indirect/inferred, thick = strong, thin = weak, labeled = annotated
- **Label conventions** — every edge gets a short label; every node gets a short name plus an optional one-line description
- **Layout hints** — top-to-bottom for causal/sequential; left-to-right for comparisons; centered with spokes for hub-and-spoke

Content should be ~200-300 lines of prose + examples. Include 1 Mermaid and 1 DOT example showing the conventions in action.

- [ ] **Step 2: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "docs(visual): add shared visual grammar conventions"
```

---

## Task 2: Per-mode visual grammar — think-standard (Sequential, Shannon, Hybrid)

**Files:** Create:
- `reference/visual-grammar/sequential.md`
- `reference/visual-grammar/shannon.md`
- `reference/visual-grammar/hybrid.md`

**Source reference (read, don't port):**
- `C:/Users/danie/Dropbox/Github/deepthinking-mcp/src/export/visual/modes/sequential.ts`
- `C:/Users/danie/Dropbox/Github/deepthinking-mcp/src/export/visual/modes/shannon.ts`
- `C:/Users/danie/Dropbox/Github/deepthinking-mcp/src/export/visual/modes/hybrid.ts`

- [ ] **Step 1: Read the source exporters to understand the visual grammar**

Each .ts file contains a function that takes a thought and produces Mermaid/DOT/etc. output. Read them to see: node shapes used, edge labels, color choices, how revision is shown, how branches are drawn.

- [ ] **Step 2: Write each grammar file**

Per mode, write a markdown file with this structure:

```markdown
# Visual Grammar: <Mode>

How to render a `<mode>` thought as a diagram.

## Node Structure

- <thought/main concept> → <shape, color>
- <dependencies> → <shape, color>
- <revisions> → <shape, edge style>
- <branches> → <shape, edge style>

## Edge Semantics

- <depends-on> → solid arrow labeled "<label>"
- <revises> → dashed arrow labeled "revision"
- <branch> → thick arrow labeled "<branch id>"

## Mermaid Template

\`\`\`mermaid
graph TD
  t1["<content of thought 1>"] --> t2["<content of thought 2>"]
  t2 -.->|revision| t1
\`\`\`

## DOT Template

\`\`\`dot
digraph Sequential {
  rankdir=TB;
  t1 [shape=box, label="<content>"];
  t2 [shape=box, label="<content>"];
  t1 -> t2;
  t2 -> t1 [style=dashed, label="revision"];
}
\`\`\`

## Worked Example

For the sequential thought chain about DB migration (see `reference/output-formats/sequential.md` worked example), the Mermaid output is:

\`\`\`mermaid
graph TD
  t1[Inventory services] --> t2[Choose strategy]
  t2 --> t3[Provision target]
  t3 --> t4[Initialize replication]
  t4 --> t5[Execute cutover]
  t5 --> t6[Post-cutover validation]
\`\`\`

## Special Cases

- Empty/single thought: single node
- Thoughts with `isRevision: true` and `revisesThought: X`: dashed arrow back to X
- Thoughts with `branchFrom: X`: thick parallel edge from X
```

Each file should be ~100-200 lines. The Sequential grammar should show revision arrows and branching clearly; Shannon should show stage progression with uncertainty labels on edges; Hybrid should show mode switches at each node.

- [ ] **Step 3: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat(visual-grammar): add rules for think-standard (Sequential, Shannon, Hybrid)"
```

---

## Tasks 3-11: Per-mode visual grammar for remaining 9 categories

Each task follows the **same pattern** as Task 2: read the source exporters in `C:/Users/danie/Dropbox/Github/deepthinking-mcp/src/export/visual/modes/` for the category's modes, distill into grammar files.

### Task 3: think-core (Inductive, Deductive, Abductive)

**Grammar files:** `reference/visual-grammar/{inductive,deductive,abductive}.md`
**Source exporters:** `src/export/visual/modes/{inductive,deductive,abductive}.ts` in the deepthinking-mcp repo

- Inductive: observations as leaf nodes pointing to a pattern node which points to the generalization node; counterexamples as red nodes pointing to the generalization with a contradicting edge style; confidence shown as edge thickness or node fill intensity
- Deductive: premises as stacked nodes flowing into a conclusion; validity check labels the conclusion node green/red; logic form labels the edge
- Abductive: observations at the top; candidate hypotheses as a ranked column with scores; the best explanation highlighted; evidence edges supporting/contradicting each hypothesis

**Commit:** `feat(visual-grammar): add rules for think-core (Inductive, Deductive, Abductive)`

### Task 4: think-mathematics (Mathematics, Physics, Computability)

**Grammar files:** `reference/visual-grammar/{mathematics,physics,computability}.md`
**Source exporters:** `src/export/visual/modes/{mathematics,physics,computability}.ts`

- Mathematics: theorem nodes with proof steps flowing between them; axioms as leaf nodes with double border
- Physics: tensor objects as rectangles with rank labels; conservation laws as thick edges; field-theory context as node grouping
- Computability: Turing machine states as circles; problems as rectangles; reductions as labeled arrows between problems; complexity classes as node fills (P=green, NP=yellow, undecidable=red)

**Commit:** `feat(visual-grammar): add rules for think-mathematics (Mathematics, Physics, Computability)`

### Task 5: think-temporal (Temporal, Historical)

**Grammar files:** `reference/visual-grammar/{temporal,historical}.md`
**Source exporters:** `src/export/visual/modes/{temporal,historical}.ts`

- Temporal: timeline with events on a horizontal axis; intervals as rectangles; Allen relations as labeled brackets between intervals; causal edges (with delay label) as arrows
- Historical: episodes as boxed groups with source nodes below; reliability shown as node border thickness; pattern arrows connecting similar episodes across time periods

**Commit:** `feat(visual-grammar): add rules for think-temporal (Temporal, Historical)`

### Task 6: think-probabilistic (Bayesian, Evidential)

**Grammar files:** `reference/visual-grammar/{bayesian,evidential}.md`
**Source exporters:** `src/export/visual/modes/{bayesian,evidential}.ts`

- Bayesian: hypothesis as root; alternatives as siblings; evidence nodes with P(E|H) and P(E|¬H) labels as children of the hypothesis; prior shown on the root node; posterior as an updated root; bayesFactor label on the edge from evidence to hypothesis
- Evidential: multiple source nodes at the top; belief/plausibility intervals as box-and-whisker style labels; conflict visualized as intersecting mass assignments

**Commit:** `feat(visual-grammar): add rules for think-probabilistic (Bayesian, Evidential)`

### Task 7: think-causal (Causal, Counterfactual)

**Grammar files:** `reference/visual-grammar/{causal,counterfactual}.md`
**Source exporters:** `src/export/visual/modes/{causal,counterfactual}.ts`

- Causal: full causal graph with nodes as variables, directed edges as cause→effect, mechanisms as edge labels, confounders as diamond nodes pointing to both cause and effect
- Counterfactual: actual scenario and counterfactual scenario as parallel columns; intervention points marked with a scissors icon; diverging outcomes; causal chain traced with thick arrows

**Commit:** `feat(visual-grammar): add rules for think-causal (Causal, Counterfactual)`

### Task 8: think-strategic (GameTheory, Optimization, Constraint)

**Grammar files:** `reference/visual-grammar/{gametheory,optimization,constraint}.md`
**Source exporters:** `src/export/visual/modes/{gametheory,optimization,constraint}.ts`

- GameTheory: player nodes with action branches; payoff matrix as a Mermaid table (classes `gg`); Nash equilibria highlighted with bold borders
- Optimization: objective function as a diamond; constraints as hexagons; feasible region implied by constraint intersection; optimal solution as a gold-filled node
- Constraint: variable nodes with domain labels; constraints as edges between variables; assignments highlighted when found

**Commit:** `feat(visual-grammar): add rules for think-strategic (GameTheory, Optimization, Constraint)`

### Task 9: think-analytical (Analogical, FirstPrinciples, MetaReasoning, Cryptanalytic)

**Grammar files:** `reference/visual-grammar/{analogical,firstprinciples,metareasoning,cryptanalytic}.md`
**Source exporters:** `src/export/visual/modes/{analogical,firstprinciples,metareasoning,cryptanalytic}.ts`

- Analogical: source and target as left/right columns; mapping edges between them; limitations shown as red X marks on failed mappings
- FirstPrinciples: principles as foundation layer; derivations built upward in a tree; each layer labeled with its reasoning step
- MetaReasoning: current mode as center node; alternative modes as spokes; mode switch arrows; confidence/utility scores on each mode option
- Cryptanalytic: pattern candidates as columns; Decibans bars (horizontal) showing cumulative evidence; threshold line; winner highlighted

**Commit:** `feat(visual-grammar): add rules for think-analytical (Analogical, FirstPrinciples, MetaReasoning, Cryptanalytic)`

### Task 10: think-scientific (ScientificMethod, SystemsThinking, FormalLogic)

**Grammar files:** `reference/visual-grammar/{scientificmethod,systemsthinking,formallogic}.md`
**Source exporters:** `src/export/visual/modes/{scientificmethod,systemsthinking,formallogic}.ts`

- ScientificMethod: hypothesis → prediction → experiment → observation → revision as a cycle; falsifiability checkpoint as a decision diamond
- SystemsThinking: stocks as rectangles; flows as labeled arrows with valve icons; reinforcing loops with R labels; balancing loops with B labels; leverage points as star markers
- FormalLogic: natural deduction tree with inference rules on each step; assumptions at leaves; discharged assumptions crossed out

**Commit:** `feat(visual-grammar): add rules for think-scientific (ScientificMethod, SystemsThinking, FormalLogic)`

### Task 11: think-engineering (Engineering, Algorithmic)

**Grammar files:** `reference/visual-grammar/{engineering,algorithmic}.md`
**Source exporters:** `src/export/visual/modes/{engineering,algorithmic}.ts`

- Engineering: alternatives as columns; criteria as rows; weighted scores as cell colors; trade study winner highlighted
- Algorithmic: input→algorithm→output as the main flow; complexity boxes (time/space) annotated; data structures as database-cylinder nodes; CLRS category tag

**Commit:** `feat(visual-grammar): add rules for think-engineering (Engineering, Algorithmic)`

### Task 12: think-academic (Synthesis, Argumentation, Critique, Analysis)

**Grammar files:** `reference/visual-grammar/{synthesis,argumentation,critique,analysis}.md`
**Source exporters:** `src/export/visual/modes/{synthesis,argumentation,critique,analysis}.ts`

- Synthesis: sources as leaf nodes with reliability badges; synthesized claims as root nodes; edges weighted by evidence strength
- Argumentation: Toulmin diagram — claim at center, grounds→warrant→backing chain, qualifier attached, rebuttal branching off
- Critique: strengths/weaknesses as parallel lists; Socratic questions as callout boxes; actionable suggestions in a separate column
- Analysis: 4-layer pyramid (surface, structural, patterns, synthesis); coverage dial at center

**Commit:** `feat(visual-grammar): add rules for think-academic (Synthesis, Argumentation, Critique, Analysis)`

### Task 13: think-advanced (Recursive, Modal, Stochastic)

**Grammar files:** `reference/visual-grammar/{recursive,modal,stochastic}.md`
**Source exporters:** `src/export/visual/modes/{recursive,modal,stochastic}.ts`

- Recursive: recursion tree showing base case at the bottom and recursive calls branching upward; halting condition as a decision diamond
- Modal: possible-worlds diagram with accessibility relations as directed edges; current world highlighted; necessity/possibility labels on propositions
- Stochastic: state diagram (Markov chain) with transition probabilities on edges; stationary distribution shown as node size; Monte Carlo samples as overlay

**Commit:** `feat(visual-grammar): add rules for think-advanced (Recursive, Modal, Stochastic)`

---

## Task 14: Write the visual-exporter agent

**File:** Create: `agents/visual-exporter.md`

- [ ] **Step 1: Write the agent definition**

```markdown
---
name: visual-exporter
description: "Convert a structured deepthinking thought into diagram source code. Use when the user asks to visualize, render, draw, or diagram the output of a recent /think invocation. Supports Mermaid, DOT, ASCII, JSON, and Markdown formats natively; SVG/PNG via the scripts/render-diagram.py wrapper."
tools: Read, Write, Bash
---

# Visual Exporter

You convert structured reasoning thoughts (JSON objects produced by the `/think` command) into diagram source code.

## Your Inputs

You are invoked with:
1. A JSON thought (or array of thoughts for sequential/shannon chains). The thought's `mode` field names the reasoning mode.
2. A target format: `mermaid`, `dot`, `ascii`, `json`, `markdown`, `svg`, or `png`.
3. Optional: a destination file path.

## Your Workflow

1. **Identify the mode** from the thought's `mode` field.
2. **Load the mode's visual grammar** from `reference/visual-grammar/<mode>.md`. Also read `reference/visual-grammar.md` for shared conventions.
3. **Generate the diagram source** following the grammar's templates, substituting the actual field values from the thought into node labels, edge labels, colors, and shapes.
4. **For `mermaid`, `dot`, `ascii`, `json`, or `markdown`**: emit the source directly as a code block.
5. **For `svg` or `png`**: call `scripts/render-diagram.py` with the generated Mermaid or DOT source. If the script is not available or the required binaries (`dot`, `mmdc`) aren't installed, fall back to emitting the source.

## Per-Mode Grammar Lookup

- `sequential`, `shannon`, `hybrid` → `reference/visual-grammar/<mode>.md`
- `inductive`, `deductive`, `abductive` → same pattern
- ... (all 34 modes)

## Verification Before Emitting

- The generated source is syntactically valid (balanced braces, proper arrow syntax).
- Every required field from the thought is represented somewhere in the diagram.
- Confidence/uncertainty values are visually encoded (node fill intensity, edge thickness, or explicit labels).
- The diagram would parse in the respective tool.

## Output Format

Always output:
1. One sentence explaining which format you produced and why (e.g., "Mermaid chosen for quick rendering in markdown-capable viewers.").
2. A code block with the diagram source.
3. If a file was written, the destination path.
4. If an external tool was needed and missing, a note on how to install it (with the correct command).
```

- [ ] **Step 2: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat(agent): add visual-exporter subagent"
```

---

## Task 15: Write the render-diagram.py script

**File:** Create: `scripts/render-diagram.py`

- [ ] **Step 1: Write the script**

```python
"""
render-diagram.py — wraps external diagram-rendering binaries.

Usage:
    python render-diagram.py --format mermaid --output out.svg < input.mmd
    python render-diagram.py --format dot --output out.png < input.dot

Input is read from stdin. If the required binary is not installed, the
script prints the source to stdout and exits 0 (graceful degradation).

Required binaries (optional):
    - graphviz (`dot` on PATH) — for DOT → SVG/PNG
    - @mermaid-js/mermaid-cli (`mmdc` on PATH) — for Mermaid → SVG/PNG
"""

import argparse
import shutil
import subprocess
import sys
import tempfile
from pathlib import Path


def render_dot(source: str, output: Path, fmt: str) -> int:
    if not shutil.which("dot"):
        print("# graphviz not installed. Install with: winget install Graphviz.Graphviz", file=sys.stderr)
        print(source)
        return 0
    with tempfile.NamedTemporaryFile("w", suffix=".dot", delete=False, encoding="utf-8") as f:
        f.write(source)
        tmp = Path(f.name)
    try:
        result = subprocess.run(
            ["dot", f"-T{fmt}", str(tmp), "-o", str(output)],
            capture_output=True,
            text=True,
            encoding="utf-8",
        )
        if result.returncode != 0:
            print(f"ERROR from dot: {result.stderr}", file=sys.stderr)
            return result.returncode
        print(f"Wrote {output}")
        return 0
    finally:
        tmp.unlink(missing_ok=True)


def render_mermaid(source: str, output: Path, fmt: str) -> int:
    if not shutil.which("mmdc"):
        print("# mermaid-cli not installed. Install with: npm install -g @mermaid-js/mermaid-cli", file=sys.stderr)
        print(source)
        return 0
    with tempfile.NamedTemporaryFile("w", suffix=".mmd", delete=False, encoding="utf-8") as f:
        f.write(source)
        tmp = Path(f.name)
    try:
        result = subprocess.run(
            ["mmdc", "-i", str(tmp), "-o", str(output)],
            capture_output=True,
            text=True,
            encoding="utf-8",
        )
        if result.returncode != 0:
            print(f"ERROR from mmdc: {result.stderr}", file=sys.stderr)
            return result.returncode
        print(f"Wrote {output}")
        return 0
    finally:
        tmp.unlink(missing_ok=True)


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--format", choices=["mermaid", "dot"], required=True, help="Source format")
    parser.add_argument("--output", required=True, type=Path, help="Output file path")
    parser.add_argument("--render-as", choices=["svg", "png"], default="svg", help="Output format (svg/png)")
    args = parser.parse_args()

    source = sys.stdin.read()
    if not source.strip():
        print("ERROR: no source on stdin", file=sys.stderr)
        return 1

    if args.format == "dot":
        return render_dot(source, args.output, args.render_as)
    if args.format == "mermaid":
        return render_mermaid(source, args.output, args.render_as)
    return 1


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 2: Test the script with a simple input**

Create a temporary test:

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin"
echo 'digraph { a -> b; b -> c }' | python scripts/render-diagram.py --format dot --output /tmp/test.svg --render-as svg || true
```

If `dot` is on PATH: expect `Wrote /tmp/test.svg`
If `dot` is NOT on PATH: expect the source printed to stdout with an install hint on stderr. Exit code 0.

- [ ] **Step 3: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat(scripts): add render-diagram.py wrapper for graphviz and mmdc"
```

---

## Task 16: Add /think-render slash command

**File:** Create: `commands/think-render.md`

- [ ] **Step 1: Write the command**

```markdown
---
description: "Render the most recent structured thought output (from /think) as a diagram. Usage: /think-render [format] where format is mermaid, dot, ascii, svg, or png (defaults to mermaid)."
argument-hint: "[mermaid|dot|ascii|svg|png]"
---

The user invoked `/think-render` with arguments: `$ARGUMENTS`

## What to do

1. **Find the most recent /think output in this conversation.** Look for a JSON code block matching one of the 34 mode schemas. If multiple recent outputs exist, use the latest.
2. **Parse the mode** from the JSON's `mode` field.
3. **Determine the target format** from `$ARGUMENTS` (default to `mermaid` if empty).
4. **Invoke the visual-exporter agent** with the thought JSON and target format. The agent reads `reference/visual-grammar/<mode>.md` and produces the diagram source.
5. **If format is `svg` or `png`:** after the agent emits source, call `scripts/render-diagram.py` to render it. Use a temp file or redirect source via stdin.
6. **Return** the diagram source as a code block, plus the rendered file path if applicable.

## If no recent /think output exists

Respond: "No recent `/think` output found in this conversation. Run `/think <mode> "<problem>"` first, then use `/think-render` to visualize the result."
```

- [ ] **Step 2: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat(command): add /think-render slash command"
```

---

## Task 17: Add visual export tests

**Files:** Create: `test/visual/validate-mermaid.py`, `test/visual/validate-dot.py`

- [ ] **Step 1: Write `test/visual/validate-mermaid.py`**

```python
"""
Validate that every mode's visual grammar contains at least one parseable
Mermaid code block. This is a syntactic smoke test — it does not invoke mmdc.
"""

import re
import sys
from pathlib import Path

GRAMMAR_DIR = Path(__file__).parent.parent.parent / "reference" / "visual-grammar"

# Minimal Mermaid parser: we check that the source starts with a recognized
# graph declaration and that braces/brackets balance.
MERMAID_START = re.compile(r"^(graph|flowchart|sequenceDiagram|classDiagram|stateDiagram|erDiagram|gantt|journey|pie|gitGraph)\s", re.MULTILINE)

def extract_mermaid_blocks(text: str):
    return re.findall(r"```mermaid\n(.*?)\n```", text, re.DOTALL)

def validate_mermaid(source: str) -> tuple[bool, str]:
    if not MERMAID_START.search(source):
        return False, "no recognized mermaid graph declaration"
    if source.count("[") != source.count("]"):
        return False, "unbalanced square brackets"
    if source.count("(") != source.count(")"):
        return False, "unbalanced parentheses"
    if source.count("{") != source.count("}"):
        return False, "unbalanced curly braces"
    return True, "ok"

def main():
    passed = 0
    failed = 0
    skipped = 0
    for grammar_file in sorted(GRAMMAR_DIR.glob("*.md")):
        content = grammar_file.read_text(encoding="utf-8")
        blocks = extract_mermaid_blocks(content)
        if not blocks:
            print(f"SKIP: {grammar_file.name} has no mermaid block")
            skipped += 1
            continue
        all_ok = True
        for i, block in enumerate(blocks):
            ok, reason = validate_mermaid(block)
            if not ok:
                print(f"FAIL: {grammar_file.name} block {i+1}: {reason}")
                all_ok = False
        if all_ok:
            print(f"PASS: {grammar_file.name} ({len(blocks)} blocks)")
            passed += 1
        else:
            failed += 1
    print()
    print(f"Results: {passed} passed, {failed} failed, {skipped} skipped")
    return 0 if failed == 0 else 1

if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 2: Write `test/visual/validate-dot.py`**

```python
"""
Validate that every mode's visual grammar contains at least one parseable
DOT code block. Syntactic check only — does not invoke graphviz.
"""

import re
import sys
from pathlib import Path

GRAMMAR_DIR = Path(__file__).parent.parent.parent / "reference" / "visual-grammar"

DOT_START = re.compile(r"^(strict\s+)?(digraph|graph)\s+\w*\s*\{", re.MULTILINE)

def extract_dot_blocks(text: str):
    return re.findall(r"```dot\n(.*?)\n```", text, re.DOTALL)

def validate_dot(source: str) -> tuple[bool, str]:
    if not DOT_START.search(source):
        return False, "no digraph/graph declaration"
    if source.count("{") != source.count("}"):
        return False, "unbalanced curly braces"
    return True, "ok"

def main():
    passed = 0
    failed = 0
    skipped = 0
    for grammar_file in sorted(GRAMMAR_DIR.glob("*.md")):
        content = grammar_file.read_text(encoding="utf-8")
        blocks = extract_dot_blocks(content)
        if not blocks:
            print(f"SKIP: {grammar_file.name} has no dot block")
            skipped += 1
            continue
        all_ok = True
        for i, block in enumerate(blocks):
            ok, reason = validate_dot(block)
            if not ok:
                print(f"FAIL: {grammar_file.name} block {i+1}: {reason}")
                all_ok = False
        if all_ok:
            print(f"PASS: {grammar_file.name} ({len(blocks)} blocks)")
            passed += 1
        else:
            failed += 1
    print()
    print(f"Results: {passed} passed, {failed} failed, {skipped} skipped")
    return 0 if failed == 0 else 1

if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 3: Run both tests**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/visual/validate-mermaid.py && python test/visual/validate-dot.py
```

Expected: 34 passed, 0 failed for each. If any grammar file has a syntax issue, fix it and re-run.

- [ ] **Step 4: Commit**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "test(visual): add syntactic validators for mermaid and dot grammar files"
```

---

## Task 18: Update README and CHANGELOG for v0.3.0, tag it

- [ ] **Step 1: Update README.md**

Read the current README. In the "Usage" section, add a new subsection after the mode list:

```markdown
### Rendering diagrams

After any `/think` invocation, render the output as a diagram:

    /think-render                  # default: mermaid
    /think-render dot              # DOT source
    /think-render svg              # SVG (requires `dot` on PATH)
    /think-render png              # PNG (requires `mmdc` on PATH)

Install optional diagram binaries:

    # Graphviz (for DOT → SVG/PNG)
    winget install Graphviz.Graphviz       # Windows
    brew install graphviz                   # macOS
    apt install graphviz                    # Linux

    # Mermaid CLI (for Mermaid → SVG/PNG)
    npm install -g @mermaid-js/mermaid-cli
```

- [ ] **Step 2: Update CHANGELOG.md**

Insert above v0.2.0:

```markdown
## [0.3.0] - <date>

### Added
- **Visual export for all 34 modes** via `/think-render [format]` slash command
- `agents/visual-exporter.md` — subagent that converts structured thoughts to diagram source
- `reference/visual-grammar.md` — shared node/edge/color conventions
- `reference/visual-grammar/*.md` — 34 per-mode grammar files with Mermaid and DOT templates
- `scripts/render-diagram.py` — wraps `graphviz` (DOT) and `mmdc` (Mermaid) binaries; gracefully degrades to printing source if binaries aren't installed
- `test/visual/validate-{mermaid,dot}.py` — syntactic validators verifying every grammar file has parseable code blocks
- README section documenting optional external binary installation

### Changed
- `skills/think/SKILL.md` and `commands/think.md`: mention `/think-render` for visualization

### Verified
- Every mode's visual grammar parses as Mermaid (34 files)
- Every mode's visual grammar parses as DOT (34 files)
- `render-diagram.py` runs without crashing when binaries are missing (graceful degradation)
```

- [ ] **Step 3: Final test run**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/test_plugin_json.py && python test/test_skill_frontmatter.py && python test/harness.py && python test/visual/validate-mermaid.py && python test/visual/validate-dot.py
```

All must pass. Also update `test/test_skill_frontmatter.py` if needed — the count may still be 13 (router + 12 categories), since no new category skills are added in Phase 4. No change needed unless a new category was created.

- [ ] **Step 4: Commit and tag**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "chore: release v0.3.0 with visual export" && git tag -a v0.3.0 -m "v0.3.0: Visual export via visual-exporter agent + render-diagram script"
```

- [ ] **Step 5: Verify final state**

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git log --oneline | head -25 && git tag
```

Tag `v0.3.0` should exist alongside `v0.1.0` and `v0.2.0`.

---

## Self-Review Notes

**Spec coverage:**
- ✅ Visual-exporter agent (Task 14)
- ✅ Shared grammar conventions (Task 1)
- ✅ 34 per-mode grammar files (Tasks 2-13)
- ✅ Render script for external tools (Task 15)
- ✅ `/think-render` slash command (Task 16)
- ✅ Visual export tests (Task 17)
- ✅ Documentation + v0.3.0 tag (Task 18)

**Parallelization opportunities:**
- Tasks 2-13 are 12 independent per-category grammar authoring tasks. Dispatch as parallel subagents (similar to Phase 3 Stage 3).
- Tasks 1, 14, 15, 16, 17, 18 run sequentially.

**Deferred (post-v0.3.0):**
- Native SVG generation without external binaries (would require a pure-Python SVG builder)
- Interactive HTML dashboards (complex, better as v0.4.0+)
- Round-trip validation (rendering → parsing → re-rendering) to catch subtle grammar bugs
- Other 9 formats (ASCII, GraphML, HTML, JSON, Markdown, Modelica, TikZ, UML) — Claude can generate these ad-hoc but they don't get dedicated grammar files yet

**No placeholders verified.** Every task specifies concrete file paths, exact content to write, validation commands, and commit messages.
