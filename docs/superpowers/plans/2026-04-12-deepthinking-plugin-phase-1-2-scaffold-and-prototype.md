# deepthinking-plugin Phase 1-2: Scaffold + Prototype Modes Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create an installable Claude Code plugin scaffold for `deepthinking-plugin` and port three prototype reasoning modes (Sequential, Inductive, Deductive) end-to-end, producing a working installable plugin that handles `/think sequential | inductive | deductive "<problem>"` with structured JSON output.

**Architecture:** A plugin directory at `C:/Users/danie/Dropbox/Github/deepthinking-plugin/` containing `.claude-plugin/plugin.json`, a router skill (`skills/think/`), two category skills (`skills/think-standard/` holding Sequential, and `skills/think-core/` holding Inductive + Deductive), and a minimal reference directory. No Node.js runtime; all logic lives in SKILL.md instruction content.

**Tech Stack:** Markdown (SKILL.md files), JSON (plugin.json), Python 3 (test harness with `jsonschema` library), Claude Code CLI (for headless smoke tests).

**Source knowledge:** This plan extracts method knowledge from the existing deepthinking-mcp codebase at `C:/Users/danie/Dropbox/Github/deepthinking-mcp/`. Specifically:
- `src/types/core.ts` lines 263-376 (BaseThought, InductiveThought, DeductiveThought)
- `src/types/modes/sequential.ts` (SequentialThought)
- `src/modes/handlers/SequentialHandler.ts`, `InductiveHandler.ts`, `DeductiveHandler.ts`
- `docs/modes/SEQUENTIAL.md`, `docs/modes/INDUCTIVE.md`, `docs/modes/DEDUCTIVE.md`

This plan produces a **complete, testable milestone**: a plugin you can install and use for three reasoning modes. Subsequent plans will add the remaining 31 modes (Phase 3), the visual-exporter agent (Phase 4), and the full test suite (Phase 5).

---

## File Structure

After this plan completes, the following files exist at `C:/Users/danie/Dropbox/Github/deepthinking-plugin/`:

| File | Purpose | Approx size |
|---|---|---|
| `.claude-plugin/plugin.json` | Plugin manifest (name, version, description) | ~400 bytes |
| `skills/think/SKILL.md` | Router skill with decision tree and output-format contract | ~4-5 KB |
| `skills/think/mode-index.md` | Decision tree for auto-recommendation (lists only the 3 prototype modes in this plan) | ~1 KB |
| `skills/think-core/SKILL.md` | Category skill with Inductive + Deductive methods | ~6 KB |
| `skills/think-standard/SKILL.md` | Category skill with Sequential method (will also hold Shannon + Hybrid in later plan) | ~4 KB |
| `reference/output-formats/sequential.md` | JSON schema + worked example for Sequential | ~2 KB |
| `reference/output-formats/inductive.md` | JSON schema + worked example for Inductive | ~2 KB |
| `reference/output-formats/deductive.md` | JSON schema + worked example for Deductive | ~2 KB |
| `reference/taxonomy.md` | Initial mode-selection rules (3 modes) | ~1 KB |
| `test/harness.py` | Python test harness using `jsonschema` to validate structured outputs | ~2 KB |
| `test/schemas/sequential.json` | JSON Schema for Sequential thought | ~1 KB |
| `test/schemas/inductive.json` | JSON Schema for Inductive thought | ~1 KB |
| `test/schemas/deductive.json` | JSON Schema for Deductive thought | ~1 KB |
| `test/samples/sequential-valid.json` | A hand-crafted sample valid output | ~300 bytes |
| `test/samples/inductive-valid.json` | Sample valid output | ~400 bytes |
| `test/samples/deductive-valid.json` | Sample valid output | ~300 bytes |
| `test/samples/sequential-invalid.json` | Sample output that should fail validation | ~200 bytes |
| `README.md` | Install + usage instructions | ~2 KB |
| `CHANGELOG.md` | Version 0.1.0 entry | ~300 bytes |

---

## Task 1: Create plugin directory scaffold and initialize git

**Files:**
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/` (directory)
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/.gitignore`

- [ ] **Step 1: Create the plugin directory tree**

Run:
```bash
mkdir -p "C:/Users/danie/Dropbox/Github/deepthinking-plugin/.claude-plugin"
mkdir -p "C:/Users/danie/Dropbox/Github/deepthinking-plugin/skills/think"
mkdir -p "C:/Users/danie/Dropbox/Github/deepthinking-plugin/skills/think-core"
mkdir -p "C:/Users/danie/Dropbox/Github/deepthinking-plugin/skills/think-standard"
mkdir -p "C:/Users/danie/Dropbox/Github/deepthinking-plugin/reference/output-formats"
mkdir -p "C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/schemas"
mkdir -p "C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/samples"
```

Expected: All directories created silently. Verify with `ls C:/Users/danie/Dropbox/Github/deepthinking-plugin`.

- [ ] **Step 2: Initialize git repo**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git init && git branch -M master
```

Expected: `Initialized empty Git repository in C:/Users/danie/Dropbox/Github/deepthinking-plugin/.git/`

- [ ] **Step 3: Write `.gitignore`**

Write file `C:/Users/danie/Dropbox/Github/deepthinking-plugin/.gitignore`:

```
# Python
__pycache__/
*.pyc
.venv/
venv/

# OS
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/

# Test artifacts
test/outputs/
test/.pytest_cache/

# Build
dist/
*.log
```

- [ ] **Step 4: Install Python dependency `jsonschema`**

The test harness (Task 4) uses the `jsonschema` library. Install it now to avoid surprises later:

```bash
pip install jsonschema
```

Expected: `Successfully installed jsonschema-...` or `Requirement already satisfied: jsonschema`.

If `pip` is not available, try `python -m pip install jsonschema`.

- [ ] **Step 5: Commit scaffold**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "chore: initialize plugin scaffold directory"
```

Expected: `[master (root-commit) ...] chore: initialize plugin scaffold directory`

---

## Task 2: Write plugin.json manifest

**Files:**
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/.claude-plugin/plugin.json`

- [ ] **Step 1: Write the failing test (JSON parse validity)**

Write file `C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/test_plugin_json.py`:

```python
"""Smoke test: plugin.json parses and has required fields."""
import json
from pathlib import Path

PLUGIN_ROOT = Path(__file__).parent.parent
MANIFEST = PLUGIN_ROOT / ".claude-plugin" / "plugin.json"

def test_plugin_json_exists_and_parses():
    assert MANIFEST.exists(), f"Missing manifest at {MANIFEST}"
    with open(MANIFEST, 'r', encoding='utf-8') as f:
        data = json.load(f)
    assert data["name"] == "deepthinking-plugin"
    assert data["version"] == "0.1.0"
    assert "description" in data and len(data["description"]) > 10
    assert "author" in data

if __name__ == "__main__":
    test_plugin_json_exists_and_parses()
    print("PASS: plugin.json is valid")
```

- [ ] **Step 2: Run test to verify it fails**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/test_plugin_json.py
```

Expected: `AssertionError: Missing manifest at ...plugin.json` (file does not exist yet).

- [ ] **Step 3: Write plugin.json**

Write file `C:/Users/danie/Dropbox/Github/deepthinking-plugin/.claude-plugin/plugin.json`:

```json
{
  "name": "deepthinking-plugin",
  "version": "0.1.0",
  "description": "Structured reasoning skills for Claude Code. Teaches Claude 34 reasoning methods (Bayesian, causal, game theory, etc.) as prompt-based skills that produce typed thought outputs. Ports the deepthinking-mcp methodology into native Claude Code skills.",
  "author": {
    "name": "Daniel Simon Jr.",
    "url": "https://github.com/danielsimonjr/deepthinking-plugin"
  },
  "license": "MIT",
  "keywords": [
    "reasoning",
    "thinking",
    "bayesian",
    "causal",
    "mcp-replacement"
  ],
  "repository": {
    "type": "git",
    "url": "https://github.com/danielsimonjr/deepthinking-plugin"
  }
}
```

- [ ] **Step 4: Run test to verify it passes**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/test_plugin_json.py
```

Expected: `PASS: plugin.json is valid`

- [ ] **Step 5: Commit**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat: add plugin.json manifest"
```

---

## Task 3: Write README.md and CHANGELOG.md

**Files:**
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/README.md`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/CHANGELOG.md`

- [ ] **Step 1: Write README.md**

Write file `C:/Users/danie/Dropbox/Github/deepthinking-plugin/README.md`:

```markdown
# deepthinking-plugin

Structured reasoning methods for Claude Code. This plugin teaches Claude 34 reasoning modes (Bayesian inference, causal analysis, game theory, etc.) as native skills. Unlike the original `deepthinking-mcp` TypeScript server, no external runtime is required — Claude learns each method from skill content and produces the structured output directly.

## Status

**v0.1.0 — Prototype.** Three modes implemented: Sequential, Inductive, Deductive. The remaining 31 modes ship in subsequent versions.

## Install

Install locally for development:

    claude --plugin-dir "C:/Users/danie/Dropbox/Github/deepthinking-plugin"

## Usage

Invoke a specific mode:

    /think sequential "Break down the steps to migrate this database"
    /think inductive "Given these three incidents, what pattern do they share?"
    /think deductive "If all users in the admin group can edit posts, and Alice is in the admin group, can Alice edit posts?"

Or let the router auto-recommend:

    /think "Why did the last three deployments fail?"

Each invocation produces a structured JSON output matching the mode's schema. See `reference/output-formats/` for per-mode schemas and examples.

## Testing

    cd test && python harness.py

Runs JSON schema validation over the sample outputs.

## Migration from deepthinking-mcp

This plugin replaces the MCP server at `C:/Users/danie/Dropbox/Github/deepthinking-mcp`. The main differences:

- No Node.js process
- No persistent session state
- Claude reasons natively instead of calling tools
- Structured output enforced by skill templates, not Zod validators

See `docs/superpowers/specs/2026-04-12-deepthinking-plugin-migration-design.md` in the source repo for the full rationale.
```

- [ ] **Step 2: Write CHANGELOG.md**

Write file `C:/Users/danie/Dropbox/Github/deepthinking-plugin/CHANGELOG.md`:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

## [0.1.0] - 2026-04-12

### Added
- Initial plugin scaffold with `.claude-plugin/plugin.json` manifest
- Router skill `think` with auto-recommendation for the 3 prototype modes
- Category skill `think-standard` with Sequential mode
- Category skill `think-core` with Inductive and Deductive modes
- Reference output-format files for Sequential, Inductive, Deductive
- Python test harness (`test/harness.py`) with JSON Schema validation
- Sample thoughts (valid and invalid) for smoke testing
- README and installation instructions
```

- [ ] **Step 3: Commit**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "docs: add README and CHANGELOG"
```

---

## Task 4: Create test harness and JSON schemas for the 3 prototype modes

**Files:**
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/harness.py`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/schemas/sequential.json`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/schemas/inductive.json`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/schemas/deductive.json`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/samples/sequential-valid.json`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/samples/inductive-valid.json`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/samples/deductive-valid.json`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/test/samples/sequential-invalid.json`

- [ ] **Step 1: Write JSON Schema for Sequential**

Write file `test/schemas/sequential.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SequentialThought",
  "type": "object",
  "required": ["mode", "thoughtNumber", "totalThoughts", "content", "nextThoughtNeeded"],
  "properties": {
    "mode": { "const": "sequential" },
    "thoughtNumber": { "type": "integer", "minimum": 1 },
    "totalThoughts": { "type": "integer", "minimum": 1 },
    "content": { "type": "string", "minLength": 1 },
    "nextThoughtNeeded": { "type": "boolean" },
    "isRevision": { "type": "boolean" },
    "revisesThought": { "type": "string" },
    "revisionReason": { "type": "string" },
    "buildUpon": { "type": "array", "items": { "type": "string" } },
    "dependencies": { "type": "array", "items": { "type": "string" } },
    "branchFrom": { "type": "string" },
    "branchId": { "type": "string" },
    "needsMoreThoughts": { "type": "boolean" }
  },
  "additionalProperties": false
}
```

- [ ] **Step 2: Write JSON Schema for Inductive**

Write file `test/schemas/inductive.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "InductiveThought",
  "type": "object",
  "required": ["mode", "observations", "generalization", "confidence"],
  "properties": {
    "mode": { "const": "inductive" },
    "observations": {
      "type": "array",
      "minItems": 1,
      "items": { "type": "string" }
    },
    "pattern": { "type": "string" },
    "generalization": { "type": "string", "minLength": 1 },
    "confidence": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "counterexamples": {
      "type": "array",
      "items": { "type": "string" }
    },
    "sampleSize": { "type": "integer", "minimum": 1 }
  },
  "additionalProperties": false
}
```

- [ ] **Step 3: Write JSON Schema for Deductive**

Write file `test/schemas/deductive.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "DeductiveThought",
  "type": "object",
  "required": ["mode", "premises", "conclusion", "validityCheck"],
  "properties": {
    "mode": { "const": "deductive" },
    "premises": {
      "type": "array",
      "minItems": 1,
      "items": { "type": "string" }
    },
    "conclusion": { "type": "string", "minLength": 1 },
    "logicForm": { "type": "string" },
    "validityCheck": { "type": "boolean" },
    "soundnessCheck": { "type": "boolean" }
  },
  "additionalProperties": false
}
```

- [ ] **Step 4: Write sample valid outputs**

Write file `test/samples/sequential-valid.json`:

```json
{
  "mode": "sequential",
  "thoughtNumber": 1,
  "totalThoughts": 3,
  "content": "First, identify the three services involved in the migration: API, workers, and DB.",
  "nextThoughtNeeded": true,
  "dependencies": []
}
```

Write file `test/samples/inductive-valid.json`:

```json
{
  "mode": "inductive",
  "observations": [
    "Deploy on Monday 2026-04-06 failed with DB connection timeout",
    "Deploy on Wednesday 2026-04-08 failed with DB connection timeout",
    "Deploy on Friday 2026-04-10 failed with DB connection timeout"
  ],
  "pattern": "All recent deploys fail at the DB connection phase",
  "generalization": "Recent production deploys consistently fail at the DB connection phase, suggesting a configuration or network issue introduced prior to 2026-04-06",
  "confidence": 0.85,
  "sampleSize": 3
}
```

Write file `test/samples/deductive-valid.json`:

```json
{
  "mode": "deductive",
  "premises": [
    "All users in the admin group can edit posts",
    "Alice is in the admin group"
  ],
  "conclusion": "Alice can edit posts",
  "logicForm": "modus ponens",
  "validityCheck": true,
  "soundnessCheck": true
}
```

Write file `test/samples/sequential-invalid.json` (missing required `mode` field):

```json
{
  "thoughtNumber": 1,
  "totalThoughts": 3,
  "content": "Missing mode field — should fail validation",
  "nextThoughtNeeded": true
}
```

- [ ] **Step 5: Write the test harness**

Write file `test/harness.py`:

```python
"""
Test harness for deepthinking-plugin.
Validates sample thought outputs against JSON schemas.
"""
import json
import sys
from pathlib import Path

try:
    import jsonschema
except ImportError:
    print("ERROR: jsonschema library not installed. Run: pip install jsonschema")
    sys.exit(1)

TEST_DIR = Path(__file__).parent
SCHEMA_DIR = TEST_DIR / "schemas"
SAMPLE_DIR = TEST_DIR / "samples"

# Map each sample file to its expected schema + expected outcome
SAMPLES = [
    ("sequential-valid.json", "sequential.json", True),
    ("inductive-valid.json", "inductive.json", True),
    ("deductive-valid.json", "deductive.json", True),
    ("sequential-invalid.json", "sequential.json", False),
]

def load_schema(name):
    with open(SCHEMA_DIR / name, 'r', encoding='utf-8') as f:
        return json.load(f)

def load_sample(name):
    with open(SAMPLE_DIR / name, 'r', encoding='utf-8') as f:
        return json.load(f)

def run_tests():
    passed = 0
    failed = 0
    for sample_name, schema_name, should_pass in SAMPLES:
        try:
            sample = load_sample(sample_name)
            schema = load_schema(schema_name)
            jsonschema.validate(sample, schema)
            # Validation succeeded
            if should_pass:
                print(f"PASS: {sample_name} validates against {schema_name}")
                passed += 1
            else:
                print(f"FAIL: {sample_name} should have failed validation but passed")
                failed += 1
        except jsonschema.ValidationError as e:
            # Validation failed
            if not should_pass:
                print(f"PASS: {sample_name} correctly rejected by {schema_name}")
                passed += 1
            else:
                print(f"FAIL: {sample_name} should have validated but got: {e.message}")
                failed += 1
    print()
    print(f"Results: {passed} passed, {failed} failed")
    return 0 if failed == 0 else 1

if __name__ == "__main__":
    sys.exit(run_tests())
```

- [ ] **Step 6: Run test harness (should pass all 4 cases)**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/harness.py
```

Expected output:
```
PASS: sequential-valid.json validates against sequential.json
PASS: inductive-valid.json validates against inductive.json
PASS: deductive-valid.json validates against deductive.json
PASS: sequential-invalid.json correctly rejected by sequential.json

Results: 4 passed, 0 failed
```

If jsonschema is not installed, install it first:
```bash
pip install jsonschema
```

- [ ] **Step 7: Commit**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "test: add JSON Schema test harness for 3 prototype modes"
```

---

## Task 5: Write reference/output-formats for the 3 prototype modes

**Files:**
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/reference/output-formats/sequential.md`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/reference/output-formats/inductive.md`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/reference/output-formats/deductive.md`

- [ ] **Step 1: Write `reference/output-formats/sequential.md`**

Write file:

````markdown
# Sequential Thought — Output Format

General-purpose iterative reasoning with revision and branching support.

## JSON Schema

```json
{
  "mode": "sequential",
  "thoughtNumber": <integer ≥1>,
  "totalThoughts": <integer ≥1>,
  "content": "<the thought content as natural language>",
  "nextThoughtNeeded": <boolean>,
  "isRevision": <boolean, optional>,
  "revisesThought": "<id of the thought being revised, optional>",
  "revisionReason": "<why revising, optional>",
  "buildUpon": ["<thought id>", ...],
  "dependencies": ["<thought id>", ...],
  "branchFrom": "<thought id to branch from, optional>",
  "branchId": "<branch identifier, optional>",
  "needsMoreThoughts": <boolean, optional>
}
```

## Required Fields

- `mode` — always the literal string `"sequential"`
- `thoughtNumber` — 1-indexed position in the chain
- `totalThoughts` — total count (may grow as reasoning continues)
- `content` — the actual thought text
- `nextThoughtNeeded` — `true` if more thinking is required

## Worked Example

Input: "Break down the steps to migrate this database."

Output:

```json
{
  "mode": "sequential",
  "thoughtNumber": 1,
  "totalThoughts": 3,
  "content": "First, identify the three services involved in the migration: API, workers, and DB.",
  "nextThoughtNeeded": true,
  "dependencies": []
}
```

## Verification Checklist

Before emitting, verify:
- `mode` is exactly `"sequential"`
- `thoughtNumber` ≤ `totalThoughts`
- `content` is non-empty natural language
- `nextThoughtNeeded` is a boolean, not a string
- If `isRevision` is true, `revisesThought` must be set
````

- [ ] **Step 2: Write `reference/output-formats/inductive.md`**

Write file:

````markdown
# Inductive Thought — Output Format

Reasoning from specific observations to general principles.

## JSON Schema

```json
{
  "mode": "inductive",
  "observations": ["<specific case 1>", "<specific case 2>", ...],
  "pattern": "<identified pattern across observations, optional>",
  "generalization": "<the general principle derived>",
  "confidence": <number 0-1>,
  "counterexamples": ["<known exception 1>", ...],
  "sampleSize": <integer, optional>
}
```

## Required Fields

- `mode` — always `"inductive"`
- `observations` — at least one specific observation
- `generalization` — the inferred general principle
- `confidence` — strength of the inference in [0, 1]

## Worked Example

Input: "Given these three incidents, what pattern do they share?"

Context observed:
- Deploy on Monday 2026-04-06 failed with DB connection timeout
- Deploy on Wednesday 2026-04-08 failed with DB connection timeout
- Deploy on Friday 2026-04-10 failed with DB connection timeout

Output:

```json
{
  "mode": "inductive",
  "observations": [
    "Deploy on Monday 2026-04-06 failed with DB connection timeout",
    "Deploy on Wednesday 2026-04-08 failed with DB connection timeout",
    "Deploy on Friday 2026-04-10 failed with DB connection timeout"
  ],
  "pattern": "All recent deploys fail at the DB connection phase",
  "generalization": "Recent production deploys consistently fail at the DB connection phase, suggesting a configuration or network issue introduced prior to 2026-04-06",
  "confidence": 0.85,
  "sampleSize": 3
}
```

## Verification Checklist

Before emitting, verify:
- `mode` is exactly `"inductive"`
- `observations` has at least 1 entry
- `confidence` is in [0, 1]
- `confidence` should be lower when `sampleSize` is small or when `counterexamples` exist
- `generalization` is logically supported by the observations (not a leap)
````

- [ ] **Step 3: Write `reference/output-formats/deductive.md`**

Write file:

````markdown
# Deductive Thought — Output Format

Reasoning from general principles to specific conclusions. Classic logical inference (modus ponens, modus tollens, syllogisms).

## JSON Schema

```json
{
  "mode": "deductive",
  "premises": ["<general principle 1>", "<general principle 2>", ...],
  "conclusion": "<the specific conclusion>",
  "logicForm": "<e.g., 'modus ponens', 'modus tollens', 'syllogism', optional>",
  "validityCheck": <boolean>,
  "soundnessCheck": <boolean, optional>
}
```

## Required Fields

- `mode` — always `"deductive"`
- `premises` — at least one premise
- `conclusion` — the derived conclusion
- `validityCheck` — does the conclusion logically follow from the premises (true/false)?

## Key Distinction: Validity vs Soundness

- **Valid** means: IF the premises are true, the conclusion must be true. Validity is about logical form.
- **Sound** means: valid AND the premises are actually true. Soundness is about real-world truth.

## Worked Example

Input: "If all users in the admin group can edit posts, and Alice is in the admin group, can Alice edit posts?"

Output:

```json
{
  "mode": "deductive",
  "premises": [
    "All users in the admin group can edit posts",
    "Alice is in the admin group"
  ],
  "conclusion": "Alice can edit posts",
  "logicForm": "modus ponens",
  "validityCheck": true,
  "soundnessCheck": true
}
```

## Verification Checklist

Before emitting, verify:
- `mode` is exactly `"deductive"`
- `premises` has at least 1 entry
- `validityCheck` reflects whether the conclusion *necessarily* follows (not whether it's plausible)
- If you cannot verify premises are true in the real world, omit `soundnessCheck` or set it to the user's stated assumption
````

- [ ] **Step 4: Commit**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "docs: add output-format references for 3 prototype modes"
```

---

## Task 6: Write the router skill (`skills/think/SKILL.md`) and mode-index

**Files:**
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/skills/think/SKILL.md`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/skills/think/mode-index.md`
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/reference/taxonomy.md`

- [ ] **Step 1: Write `skills/think/SKILL.md`**

Write file with the following exact content:

````markdown
---
name: think
description: Route a reasoning task to the appropriate thinking mode. Use when the user invokes `/think <mode?> "<problem>"` or asks for structured reasoning, critical analysis, Bayesian inference, causal analysis, inductive/deductive reasoning, hypothesis formation, or any form of disciplined thinking. Supports explicit mode selection (e.g., `/think bayesian ...`) and auto-recommendation (`/think ...`).
---

# Think — Reasoning Router

This skill routes a reasoning task to the appropriate category skill containing the relevant method(s). Your job as the router is:

1. **Parse the invocation.** Look for a mode name in the arguments. If the user wrote `/think bayesian "..."`, the mode is `bayesian`.
2. **Load the right category skill.** Each category skill holds 2-4 reasoning methods. Use the mapping in `mode-index.md`.
3. **Apply the method.** Once the category skill is loaded, follow its instructions to produce structured output matching the mode's JSON schema.
4. **Emit the structured output.** Every reasoning output MUST be a valid JSON object matching the schema in `reference/output-formats/<mode>.md`.

## Available Modes (v0.1.0)

Only three modes are available in this version. The remaining 31 ship in later versions.

| Mode | Category skill | Use when |
|---|---|---|
| `sequential` | `think-standard` | Breaking a complex problem into ordered steps, iterative planning, revision-friendly workflows |
| `inductive` | `think-core` | Forming general principles from specific observations; pattern recognition across cases |
| `deductive` | `think-core` | Drawing specific conclusions from established premises; formal logical inference |

## Invocation Patterns

### Explicit mode

    /think sequential "Break down the steps to migrate this database"
    /think inductive "Given these three incidents, what pattern do they share?"
    /think deductive "If all users in admin can edit posts and Alice is in admin, can Alice edit posts?"

If the user provided an explicit mode name, load the corresponding category skill and apply that method.

### Auto-recommend (no mode specified)

    /think "Why did the last three deployments fail?"

If no mode is specified, read `mode-index.md` for the decision tree and pick the mode whose shape best matches the problem. Explain your choice to the user in one sentence before producing the structured output.

### Unavailable mode in v0.1.0

If the user names a mode not yet available (e.g., `/think bayesian`), respond:

> "The `<mode>` mode is not yet available in v0.1.0 of deepthinking-plugin. Currently available: sequential, inductive, deductive. The router will auto-recommend one of these for your problem instead."

Then proceed with auto-recommendation.

## Output Format Contract

Every reasoning output from this plugin follows a consistent pattern:

1. **One sentence of meta** explaining which mode was chosen (only for auto-recommend cases).
2. **A JSON code block** containing the structured thought matching the mode's schema.
3. **A short natural-language summary** (2-4 sentences) explaining the reasoning for humans.

Example of a well-formed response to `/think inductive "..."`:

    Using inductive reasoning to generalize from the three observed incidents.

    ```json
    {
      "mode": "inductive",
      "observations": ["..."],
      "generalization": "...",
      "confidence": 0.85,
      "sampleSize": 3
    }
    ```

    The three deploy failures all occurred at the same phase with identical symptoms, which supports a strong generalization that the issue is upstream of DB connection handling. Confidence is 0.85 because the sample is small (n=3) and we have not yet ruled out confounders.

## Decision Tree Reference

For auto-recommendation, consult `skills/think/mode-index.md`.

## Output-Format References

For the JSON schema and verification checklist for each mode, see:
- `reference/output-formats/sequential.md`
- `reference/output-formats/inductive.md`
- `reference/output-formats/deductive.md`
````

- [ ] **Step 2: Write `skills/think/mode-index.md`**

Write file:

````markdown
# Mode Index — Auto-Recommendation Decision Tree (v0.1.0)

When the user invokes `/think` without specifying a mode, use this decision tree to pick the most appropriate one.

## Decision Tree

Ask yourself:

1. **Does the user want to generalize from specific cases?**
   Signals: multiple observations provided, "what pattern", "in general", "these all show", "trend"
   → **Inductive**

2. **Does the user want to apply established rules to derive a conclusion?**
   Signals: "if X then Y", "given these rules", "can we conclude", explicit premises
   → **Deductive**

3. **Does the user want to break a complex task into ordered steps or iterate with revision?**
   Signals: "break down", "plan", "steps to", "how should I approach", multi-step workflows
   → **Sequential**

If none of the above clearly fit, default to **Sequential** and explain you're treating the task as iterative planning.

## Example Mappings

| User prompt | Recommended mode | Reason |
|---|---|---|
| "Break down the steps to migrate this database" | sequential | Explicit "break down" and multi-step workflow |
| "Given these three incidents, what pattern do they share?" | inductive | Multiple observations, asking for pattern |
| "If all users in admin can edit posts and Alice is in admin, can Alice edit posts?" | deductive | Explicit premises, formal inference |
| "Why did the last three deployments fail?" | inductive | "The last three" implies pattern-finding from observations |
| "What should our caching strategy be for this API?" | sequential | Multi-step design decision |
| "Plan the refactor of the auth module" | sequential | Explicit "plan" and multi-step work |

## Explain Your Choice

When auto-recommending, always start your response with one sentence naming the chosen mode and why. Example:

> "Using inductive reasoning to identify a pattern across the three deployment failures."

This transparency lets the user override you with `/think <mode>` if they prefer a different frame.
````

- [ ] **Step 3: Write `reference/taxonomy.md`**

Write file:

````markdown
# Reasoning Mode Taxonomy

This file is the canonical reference for which mode applies to which problem shape. It will grow as more modes are added in future versions.

## v0.1.0 Modes

### Sequential (`sequential`)
- **Category:** think-standard
- **Shape:** Iterative planning with revision
- **Signals:** "plan", "break down", "steps to", "how to approach"
- **Anti-signals:** Formal logical inference (use deductive), pattern-from-cases (use inductive)

### Inductive (`inductive`)
- **Category:** think-core
- **Shape:** Observations → pattern → generalization
- **Signals:** Multiple specific examples, "pattern", "trend", "in general", "these all..."
- **Anti-signals:** Premise-driven inference (use deductive), single-case analysis (use sequential)

### Deductive (`deductive`)
- **Category:** think-core
- **Shape:** Premises → logical inference → conclusion
- **Signals:** "if X then Y", explicit rules, syllogisms, "can we conclude"
- **Anti-signals:** Uncertain premises (consider bayesian in a later version), observation-driven reasoning (use inductive)

## How to Read This File

When a category skill is loaded, the skill can read from this taxonomy to (a) confirm the problem fits its mode, and (b) recommend switching to a different mode if it does not fit.

## Future Modes

Not yet implemented in v0.1.0 (ship in future versions):
- Core: shannon, hybrid, mathematics, physics, computability
- Causal: causal, counterfactual, temporal, historical
- Probabilistic: bayesian, evidential
- Strategic: gametheory, optimization, constraint
- Analytical: analogical, firstprinciples, metareasoning, cryptanalytic
- Scientific: scientificmethod, systemsthinking, formallogic
- Engineering: engineering, algorithmic
- Academic: synthesis, argumentation, critique, analysis
- Advanced: recursive, modal, stochastic
````

- [ ] **Step 4: Commit**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat: add router skill, mode index, and taxonomy for 3 prototype modes"
```

---

## Task 7: Write `skills/think-standard/SKILL.md` with Sequential method

**Files:**
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/skills/think-standard/SKILL.md`

- [ ] **Step 1: Write the category skill**

Write file with this exact content:

````markdown
---
name: think-standard
description: Standard sequential and iterative reasoning methods. Use when the user invokes `/think sequential` or asks to break down a complex task into ordered steps, iterate on a plan, or revise earlier thinking. In v0.1.0 this skill contains only the Sequential method; Shannon and Hybrid ship in future versions.
---

# think-standard — Standard Sequential Reasoning Methods

This category skill contains the **Sequential** reasoning method. Shannon and Hybrid will join this skill in a future version.

---

## Sequential Reasoning

Sequential reasoning is general-purpose iterative thinking: break a problem into a chain of thoughts, each building on the previous, with the ability to revise or branch as understanding deepens.

### When to Use

- Breaking a complex task into ordered steps
- Planning a multi-step workflow (refactor, migration, investigation)
- Iterating on an approach where earlier thoughts may need revision
- Situations that benefit from explicit step-tracking

**Do not use Sequential** when:
- The user wants a pattern from multiple observations → use Inductive
- The user wants formal logical inference from premises → use Deductive
- The user wants a probabilistic belief update → (use Bayesian in a future version)

### How to Think Sequentially

1. **Assess total scope.** Estimate `totalThoughts` — how many steps you expect this will take. It's fine to adjust later.
2. **Produce thought N.** Each thought is a single logical step. Keep it focused on one idea or action.
3. **Track dependencies.** If thought N builds on an earlier thought, reference its id in `dependencies`.
4. **Revise if needed.** If a later thought invalidates an earlier one, set `isRevision: true`, set `revisesThought` to the earlier thought's id, and explain in `revisionReason`.
5. **Branch if exploring alternatives.** Use `branchFrom` and `branchId` to explore a parallel line of thinking without abandoning the main chain.
6. **Emit the structured output.** Follow the schema in `reference/output-formats/sequential.md`.
7. **Continue if more is needed.** Set `nextThoughtNeeded: true` when the reasoning is not complete. Set it to `false` only when the chain has reached a conclusion.

### Output Format

See `reference/output-formats/sequential.md` for the authoritative JSON schema, worked example, and verification checklist.

### Quick Template

```json
{
  "mode": "sequential",
  "thoughtNumber": 1,
  "totalThoughts": <your estimate>,
  "content": "<this thought as natural language>",
  "nextThoughtNeeded": <true if more steps remain>,
  "dependencies": [<ids of earlier thoughts this builds on>]
}
```

### Verification Before Emitting

- `mode` is exactly `"sequential"`
- `thoughtNumber` ≤ `totalThoughts`
- `content` is a complete sentence, not a stub
- `nextThoughtNeeded` is a boolean
- If `isRevision: true`, then `revisesThought` is set

### Worked Example

Input: "Break down the steps to migrate our Postgres database to a new region."

Output (thought 1 of a chain):

```json
{
  "mode": "sequential",
  "thoughtNumber": 1,
  "totalThoughts": 4,
  "content": "First, inventory the services that touch the database. We need to know everything that will need its connection string updated: API, workers, cron jobs, and any dashboards.",
  "nextThoughtNeeded": true,
  "dependencies": []
}
```

Natural-language summary: "I'm treating this as a four-step migration plan. Step 1 is service inventory — without knowing what connects to the DB, any migration plan is incomplete."

---

## Future Methods (not in v0.1.0)

- **Shannon** — Information-theoretic problem decomposition. Uses entropy to measure reducible uncertainty.
- **Hybrid** — Combines multiple modes for cross-cutting problems.

These ship in a future version of this skill.
````

- [ ] **Step 2: Commit**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat(think-standard): add Sequential reasoning method"
```

---

## Task 8: Write `skills/think-core/SKILL.md` with Inductive and Deductive methods

**Files:**
- Create: `C:/Users/danie/Dropbox/Github/deepthinking-plugin/skills/think-core/SKILL.md`

- [ ] **Step 1: Write the category skill**

Write file with this exact content:

````markdown
---
name: think-core
description: Core reasoning triad — Inductive, Deductive, and Abductive. Use when the user invokes `/think inductive`, `/think deductive`, or `/think abductive`, or asks to generalize from examples, derive conclusions from rules, or find the best explanation for observations. In v0.1.0 this skill contains Inductive and Deductive; Abductive ships in a future version.
---

# think-core — Core Reasoning Triad

This category skill contains two of the three fundamental reasoning patterns: **Inductive** (specific → general) and **Deductive** (general → specific). **Abductive** (inference to the best explanation) will join this skill in a future version.

---

## Inductive Reasoning

Inductive reasoning moves from specific observations to general principles. You observe several cases, identify a pattern, and form a generalization whose confidence depends on the breadth and consistency of the observations.

### When to Use

- You have multiple specific examples and want to extract a pattern
- Finding trends across incidents, deployments, user behaviors, or measurements
- Forming a hypothesis that more observations could strengthen or refute
- Moving from particular cases to a rule

**Do not use Inductive** when:
- You have premises and want formal logical inference → use Deductive
- You have only one case → use Sequential or ask for more observations
- The generalization is already stated and you're evaluating it → use Deductive or Bayesian (future)

### How to Reason Inductively

1. **List every observation.** Do not summarize prematurely. Each observation is a concrete, specific case.
2. **Look for the invariant.** What is true in every observation? What varies?
3. **State the pattern.** A short phrase naming the invariant.
4. **Form the generalization.** One sentence extending the pattern beyond the observed cases.
5. **Assess confidence.** A number in [0, 1] reflecting strength of inference. Consider:
   - **Sample size** — more observations → higher confidence (within reason)
   - **Homogeneity** — if observations vary in relevant ways, the generalization is stronger
   - **Counterexamples** — any known cases where the pattern fails
6. **Note counterexamples.** Known exceptions lower confidence and narrow the generalization's scope.

### Output Format

See `reference/output-formats/inductive.md` for the authoritative JSON schema.

### Quick Template

```json
{
  "mode": "inductive",
  "observations": ["<case 1>", "<case 2>", "<case 3>"],
  "pattern": "<invariant in short form>",
  "generalization": "<general principle in one sentence>",
  "confidence": <0.0 to 1.0>,
  "counterexamples": [<optional known exceptions>],
  "sampleSize": <optional integer>
}
```

### Verification Before Emitting

- `mode` is exactly `"inductive"`
- `observations` has at least one entry (ideally ≥3)
- `confidence` is in [0, 1]
- `confidence` is lower when `sampleSize` is small OR `counterexamples` are present
- `generalization` is logically supported by the observations, not a leap
- `pattern` (if included) actually appears in every observation

### Worked Example

Input: "Given these three incidents, what pattern do they share? (1) Deploy on Mon 2026-04-06 failed with DB connection timeout. (2) Deploy on Wed 2026-04-08 failed with DB connection timeout. (3) Deploy on Fri 2026-04-10 failed with DB connection timeout."

Output:

```json
{
  "mode": "inductive",
  "observations": [
    "Deploy on Monday 2026-04-06 failed with DB connection timeout",
    "Deploy on Wednesday 2026-04-08 failed with DB connection timeout",
    "Deploy on Friday 2026-04-10 failed with DB connection timeout"
  ],
  "pattern": "All recent deploys fail at the DB connection phase",
  "generalization": "Recent production deploys consistently fail at the DB connection phase, suggesting a configuration or network issue introduced prior to 2026-04-06",
  "confidence": 0.85,
  "sampleSize": 3
}
```

Natural-language summary: "Three identical failures with no variation across day-of-week is a strong signal of a structural issue, not flakiness. Confidence is 0.85 (not 1.0) because n=3 is still small and we have not yet ruled out external factors like upstream DNS changes."

---

## Deductive Reasoning

Deductive reasoning moves from established premises to specific conclusions. If the premises are true and the form is valid, the conclusion must be true.

### When to Use

- You have explicit rules or premises and want to derive a conclusion
- Applying a policy to a specific case
- Formal logical inference (modus ponens, modus tollens, syllogisms)
- Verifying whether a claim follows from stated assumptions

**Do not use Deductive** when:
- You have uncertain premises (the conclusion inherits that uncertainty — consider Bayesian in a future version)
- You are deriving rules from observations → use Inductive
- The conclusion is plausible but not logically forced → you may be pattern-matching, not deducing

### Validity vs. Soundness

Two separate questions:

- **Validity** — Does the conclusion logically follow from the premises? (Logical form)
- **Soundness** — Validity PLUS the premises are actually true in the real world. (Form + truth)

A deduction can be valid but unsound. Example:
- Premise 1: "All unicorns can fly."
- Premise 2: "Sparky is a unicorn."
- Conclusion: "Sparky can fly."

This is **valid** (the logic is impeccable) but **unsound** (the premises are false).

### How to Reason Deductively

1. **List the premises.** Each as a clear statement.
2. **Identify the logical form.** Common forms:
   - **Modus ponens:** If P then Q. P. Therefore Q.
   - **Modus tollens:** If P then Q. Not Q. Therefore not P.
   - **Hypothetical syllogism:** If P then Q. If Q then R. Therefore if P then R.
   - **Categorical syllogism:** All A are B. X is A. Therefore X is B.
3. **Check validity.** Does the conclusion follow by logical form alone?
4. **Check soundness** (if you can). Are the premises actually true?
5. **State the conclusion** explicitly.

### Output Format

See `reference/output-formats/deductive.md` for the authoritative JSON schema.

### Quick Template

```json
{
  "mode": "deductive",
  "premises": ["<premise 1>", "<premise 2>"],
  "conclusion": "<the derived conclusion>",
  "logicForm": "<e.g., modus ponens>",
  "validityCheck": <true or false>,
  "soundnessCheck": <true, false, or omit if unknown>
}
```

### Verification Before Emitting

- `mode` is exactly `"deductive"`
- `premises` has at least one entry
- `validityCheck` is `true` ONLY if the conclusion *necessarily* follows from the premises
- If `validityCheck: false`, the conclusion is not actually entailed — flag this in the natural-language summary
- `soundnessCheck` is `true` only if you can actually verify the premises are true; otherwise omit

### Worked Example

Input: "If all users in the admin group can edit posts, and Alice is in the admin group, can Alice edit posts?"

Output:

```json
{
  "mode": "deductive",
  "premises": [
    "All users in the admin group can edit posts",
    "Alice is in the admin group"
  ],
  "conclusion": "Alice can edit posts",
  "logicForm": "modus ponens",
  "validityCheck": true,
  "soundnessCheck": true
}
```

Natural-language summary: "Classic modus ponens. The premises entail the conclusion by logical form, and both premises are stated as given, so soundness is also true. Alice can edit posts."

---

## Future Methods (not in v0.1.0)

- **Abductive** — Inference to the best explanation. Generate candidate hypotheses for a surprising observation and evaluate them against parsimony, explanatory power, plausibility, and testability.

This ships in a future version of this skill.
````

- [ ] **Step 2: Commit**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "feat(think-core): add Inductive and Deductive reasoning methods"
```

---

## Task 9: Install plugin locally and verify it loads

**Files:**
- No file changes. This task verifies Claude Code can load the plugin.

- [ ] **Step 1: Verify the plugin directory is well-formed**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && find . -type f -not -path './.git/*' | sort
```

Expected file list (exact):
```
./.claude-plugin/plugin.json
./.gitignore
./CHANGELOG.md
./README.md
./reference/output-formats/deductive.md
./reference/output-formats/inductive.md
./reference/output-formats/sequential.md
./reference/taxonomy.md
./skills/think-core/SKILL.md
./skills/think-standard/SKILL.md
./skills/think/SKILL.md
./skills/think/mode-index.md
./test/harness.py
./test/samples/deductive-valid.json
./test/samples/inductive-valid.json
./test/samples/sequential-invalid.json
./test/samples/sequential-valid.json
./test/schemas/deductive.json
./test/schemas/inductive.json
./test/schemas/sequential.json
./test/test_plugin_json.py
```

If any file is missing, go back to the relevant task and fix it.

- [ ] **Step 2: Run the test harness**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/harness.py
```

Expected:
```
PASS: sequential-valid.json validates against sequential.json
PASS: inductive-valid.json validates against inductive.json
PASS: deductive-valid.json validates against deductive.json
PASS: sequential-invalid.json correctly rejected by sequential.json

Results: 4 passed, 0 failed
```

- [ ] **Step 3: Run the plugin.json smoke test**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/test_plugin_json.py
```

Expected: `PASS: plugin.json is valid`

- [ ] **Step 4: Validate each SKILL.md has valid frontmatter**

Write file `test/test_skill_frontmatter.py`:

```python
"""Validate each SKILL.md has the required YAML frontmatter."""
import re
from pathlib import Path

PLUGIN_ROOT = Path(__file__).parent.parent
SKILLS_DIR = PLUGIN_ROOT / "skills"

REQUIRED_KEYS = ["name", "description"]
FRONTMATTER_RE = re.compile(r'^---\s*\n(.*?)\n---\s*\n', re.DOTALL)

def check_skill(skill_md):
    content = skill_md.read_text(encoding='utf-8')
    match = FRONTMATTER_RE.match(content)
    assert match, f"{skill_md}: no YAML frontmatter found"
    frontmatter = match.group(1)
    for key in REQUIRED_KEYS:
        assert re.search(rf'^{key}:\s*.+', frontmatter, re.MULTILINE), \
            f"{skill_md}: missing required key '{key}'"
    # Description should be substantial (>20 chars)
    desc_match = re.search(r'^description:\s*(.+)', frontmatter, re.MULTILINE)
    desc_text = desc_match.group(1).strip() if desc_match else ""
    assert len(desc_text) > 20, f"{skill_md}: description too short"

def main():
    skills = sorted(SKILLS_DIR.rglob("SKILL.md"))
    assert len(skills) == 3, f"Expected 3 SKILL.md files, found {len(skills)}"
    for s in skills:
        check_skill(s)
        print(f"PASS: {s.relative_to(PLUGIN_ROOT)}")
    print(f"\nAll {len(skills)} skills have valid frontmatter.")

if __name__ == "__main__":
    main()
```

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/test_skill_frontmatter.py
```

Expected:
```
PASS: skills/think-core/SKILL.md
PASS: skills/think-standard/SKILL.md
PASS: skills/think/SKILL.md

All 3 skills have valid frontmatter.
```

- [ ] **Step 5: Commit the skill frontmatter test**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add -A && git commit -m "test: add SKILL.md frontmatter validator"
```

- [ ] **Step 6: Attempt to install the plugin into Claude Code**

Run:
```bash
claude --plugin-dir "C:/Users/danie/Dropbox/Github/deepthinking-plugin" --version
```

Expected: Claude Code starts and prints its version. If Claude Code is not installed on PATH, this step is skipped — document this in the task notes but do not fail the plan.

If Claude Code is on PATH and the plugin loaded, verify plugin visibility:
```bash
claude --plugin-dir "C:/Users/danie/Dropbox/Github/deepthinking-plugin" plugin list
```

Expected: `deepthinking-plugin` appears in the output.

Note: If `claude plugin list` is not a supported subcommand in your Claude Code version, fall back to: open Claude Code interactively with the `--plugin-dir` flag, then invoke `/think` and verify the router skill loads.

---

## Task 10: Manual end-to-end smoke test of the three modes

**Files:**
- No new files. This task runs Claude Code interactively and captures outputs.

- [ ] **Step 1: Launch Claude Code with the plugin**

Run (in a separate terminal, since this is interactive):
```bash
claude --plugin-dir "C:/Users/danie/Dropbox/Github/deepthinking-plugin"
```

- [ ] **Step 2: Test Sequential mode**

In the Claude Code session, run:

```
/think sequential "Break down the steps to migrate our Postgres database to a new region."
```

Expected: Claude responds with a JSON block matching the Sequential schema. Verify:
- `mode` is `"sequential"`
- `thoughtNumber` and `totalThoughts` are present and consistent
- `content` is a natural-language sentence
- `nextThoughtNeeded` is a boolean

Save the JSON block to `test/captured/sequential-run1.json` (create the `captured/` directory).

- [ ] **Step 3: Validate the captured Sequential output**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python -c "
import json, jsonschema
with open('test/captured/sequential-run1.json') as f:
    data = json.load(f)
with open('test/schemas/sequential.json') as f:
    schema = json.load(f)
jsonschema.validate(data, schema)
print('PASS: captured sequential output is valid')
"
```

Expected: `PASS: captured sequential output is valid`

If it fails, read the error, update `skills/think-standard/SKILL.md` to clarify the expected format, and re-test.

- [ ] **Step 4: Test Inductive mode**

In Claude Code:

```
/think inductive "Given these three incidents, what pattern do they share? (1) Deploy on Mon 2026-04-06 failed with DB connection timeout. (2) Deploy on Wed 2026-04-08 failed with DB connection timeout. (3) Deploy on Fri 2026-04-10 failed with DB connection timeout."
```

Expected: Claude responds with a JSON block matching the Inductive schema.

Save to `test/captured/inductive-run1.json` and validate:

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python -c "
import json, jsonschema
with open('test/captured/inductive-run1.json') as f:
    data = json.load(f)
with open('test/schemas/inductive.json') as f:
    schema = json.load(f)
jsonschema.validate(data, schema)
print('PASS: captured inductive output is valid')
"
```

Expected: `PASS: captured inductive output is valid`

- [ ] **Step 5: Test Deductive mode**

In Claude Code:

```
/think deductive "If all users in the admin group can edit posts, and Alice is in the admin group, can Alice edit posts?"
```

Save to `test/captured/deductive-run1.json` and validate:

```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python -c "
import json, jsonschema
with open('test/captured/deductive-run1.json') as f:
    data = json.load(f)
with open('test/schemas/deductive.json') as f:
    schema = json.load(f)
jsonschema.validate(data, schema)
print('PASS: captured deductive output is valid')
"
```

Expected: `PASS: captured deductive output is valid`

- [ ] **Step 6: Test auto-recommendation**

In Claude Code:

```
/think "Why did the last three deployments fail?"
```

Expected: Claude opens with a meta sentence like "Using inductive reasoning to identify a pattern..." and produces a valid Inductive JSON block.

Save to `test/captured/auto-run1.json` and verify the `mode` field is `"inductive"`:

Write file `test/verify_auto_run.py`:

```python
"""Verify auto-recommended mode is inductive for the test prompt."""
import json
from pathlib import Path

CAPTURED = Path(__file__).parent / "captured" / "auto-run1.json"

def main():
    with open(CAPTURED, 'r', encoding='utf-8') as f:
        data = json.load(f)
    actual = data.get('mode')
    if actual == 'inductive':
        print('PASS: router auto-recommended inductive')
        return 0
    print(f'FAIL: expected inductive, got {actual}')
    return 1

if __name__ == "__main__":
    import sys
    sys.exit(main())
```

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/verify_auto_run.py
```

Expected: `PASS: router auto-recommended inductive`

If the router picked a different mode, investigate whether the prompt is ambiguous or the decision tree in `skills/think/mode-index.md` needs a clearer rule. Update the mode-index, restart Claude Code, and rerun.

Expected: `PASS: router auto-recommended inductive`

- [ ] **Step 7: Add the `captured/` directory to `.gitignore`**

Edit `C:/Users/danie/Dropbox/Github/deepthinking-plugin/.gitignore` to add:

```
# Captured test runs (regenerated per session)
test/captured/
```

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git add .gitignore && git commit -m "chore: gitignore captured test runs"
```

---

## Task 11: Tag milestone v0.1.0 and verify all tests pass

**Files:**
- No file changes.

- [ ] **Step 1: Run the full test suite**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && python test/test_plugin_json.py && python test/test_skill_frontmatter.py && python test/harness.py
```

Expected: All three test runs pass.

- [ ] **Step 2: Review the commit history**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git log --oneline
```

Expected: Roughly 10 commits from this plan, ending with the most recent work.

- [ ] **Step 3: Tag v0.1.0**

Run:
```bash
cd "C:/Users/danie/Dropbox/Github/deepthinking-plugin" && git tag -a v0.1.0 -m "v0.1.0: Plugin scaffold + 3 prototype modes (Sequential, Inductive, Deductive)"
```

Expected: No output. Verify with `git tag`.

- [ ] **Step 4: Confirm completion**

The plan is complete when:
1. The plugin directory exists at `C:/Users/danie/Dropbox/Github/deepthinking-plugin/`
2. All files listed in the File Structure section exist
3. All three tests (`test_plugin_json.py`, `test_skill_frontmatter.py`, `harness.py`) pass
4. Claude Code can load the plugin via `--plugin-dir` and responds to `/think sequential|inductive|deductive|<auto>`
5. Captured outputs for all three modes validate against their schemas
6. Tag `v0.1.0` exists in git

Report any failures to the user before claiming completion.

---

## Self-Review Notes

**Spec coverage check:**
- ✅ Plugin layout (Task 1)
- ✅ plugin.json manifest (Task 2)
- ✅ Router skill with decision tree (Task 6)
- ✅ mode-index.md (Task 6)
- ✅ reference/taxonomy.md (Task 6)
- ✅ Category skills for Sequential (Task 7) and Inductive + Deductive (Task 8)
- ✅ Output format references (Task 5)
- ✅ Verification checklists embedded in each skill
- ✅ Test harness with JSON Schema validation (Task 4)
- ✅ Sample valid and invalid thoughts (Task 4)
- ✅ Manual end-to-end smoke tests (Task 10)
- ✅ Plugin installs locally and is loadable by Claude Code (Task 9)
- ✅ v0.1.0 tagged (Task 11)

**Deferred to later plans:**
- Phase 3: Extracting the remaining 31 modes (batch work parallelizable across category skills)
- Phase 4: visual-exporter agent, render-diagram.py script, visual-grammar.md reference
- Phase 5: Full test suite (34 smoke tests, 20 router tests, 68 visual tests), skill-reviewer passes, plugin-validator pass
- Phase 6: Optional deprecation of deepthinking-mcp

**No placeholders verified.** Every code block, schema, and test case in this plan is complete and ready to copy directly.
