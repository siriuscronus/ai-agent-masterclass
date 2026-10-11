---
name: mck-ppt-design
description: >-
  Create professional, consultant-grade PowerPoint presentations from scratch
  using MckEngine (python-pptx wrapper) with McKinsey-style design. Use when
  user asks to create slides, pitch decks, business presentations, strategy
  decks, quarterly reviews, board meeting slides, or any professional PPTX.
  AI calls eng.cover(), eng.donut(), eng.timeline() etc — 67 high-level methods
  across 12 categories (structure, data, framework, comparison, narrative,
  timeline, team, charts, images, advanced viz, dashboards, visual storytelling),
  consistent typography, zero file-corruption issues, BLOCK_ARC native shapes
  for circular charts (donut, pie, gauge), production-hardened guard rails
  for spacing, overflow, legend consistency, title style uniformity,
  dynamic sizing for variable-count layouts, horizontal item overflow
  protection, chart rendering, and
  AI-generated cover images via Tencent Hunyuan 2.0 with professional cutout,
  cool grey-blue tint, and McKinsey-style Bézier ribbon decoration.
---

# McKinsey PPT Design — Harness Framework

> **Version**: 2.3.3-harness-v2 · **Engine**: MckEngine (python-pptx) · **Author**: [likaku](https://github.com/likaku/Mck-ppt-design-skill)
>
> **Required tools**: Read, Write, Bash · **Requires**: `pip install python-pptx lxml`

---

## Common Failure Modes (Read Before Every Generation, Before HARD RULES)

> Telling the AI what to do is useful. Telling it where previous attempts most often failed is even more effective.
> The three anti-patterns below have all been validated in real executions. Actively guard against them every time.

### Anti-pattern 1: Verbally declaring "gate passed" (most common)

**Incorrect approach**:
> "S4 QA found 7 errors. They are all judged to be engine-design behavior, so the gate passes and we can enter S5."

**Why this is wrong**: `passed` is being decided verbally by the AI instead of being derived by the program. No matter how reasonable the explanation sounds, this amounts to the AI issuing its own completion certificate.

**Correct approach**:
1. Run `python references/scripts/gate_check.py <pptx_path> <project_directory>`
2. Read `<project_directory>/gate_result.json`
3. Enter S5 only when `gate_result.json` contains `"passed": true`
4. If `"passed": false`, fix the issues listed under `user_code_errors`, render again, and run gate_check again

---

### Anti-pattern 2: Treating the S3 gate as passed after a mental review

**Incorrect approach**:
> "S3 content-gate precheck: API format is correct and text is within limits. Passed, entering S4." (without executing any code)

**Why this is wrong**: Three real API-format errors occurred in practice (`four_column` / `matrix_2x2` / `executive_summary` argument formats). A mental review does not reliably catch them. Code-based checks are mandatory.

**Correct approach**:
1. Run `python references/scripts/gate_check_s3.py <content_json_path> <project_directory>`
2. Read `<project_directory>/gate_s3.json`
3. Enter S4 only when `"passed": true`
4. If there are `fail` items, correct `content.json` and run gate_check_s3 again

---

### Anti-pattern 3: Using the engine_bug classification as a verbal escape hatch

**Incorrect approach**:
> "`peer_font_inconsistency` is internal engine behavior rather than a user-code problem, so it can be waived."

**Why this is wrong**: The classification itself may be correct, but if the AI makes the exemption verbally in the conversation, the AI effectively grants itself waiver authority, and the AI has an incentive to let itself pass.

**Correct approach**:
`gate_check.py` contains a hard-coded `ENGINE_BUG_WHITELIST` enumeration. Only error categories on that whitelist may be exempted. The whitelist is code, not an AI judgment. If you believe an error category belongs on the whitelist, modify the enumeration in `gate_check.py` instead of verbally granting an exemption.

---

## HARD RULES (Mandatory, No Bypass)

1. **Every generation must follow the five-stage workflow.** Do not skip prerequisite stages and generate directly from a one-line request.
2. **TaskCreate drives progress.** Create 5 tasks before starting and immediately mark each stage completed with TaskUpdate when that stage finishes.
3. **Load context on demand.** At each stage, read only the files required for that stage. Do not load all content from the legacy SKILL.md.
4. **Gates must be machine-readable.** S3 and S4 must run the corresponding gate_check scripts and read their JSON results. Do not decide verbally.
5. **Self-Refinement is mandatory.** After fixing a pattern-level problem, write the lesson to the corresponding file under `experiences/`.
6. **Engine path is fixed**: `sys.path.insert(0, os.path.expanduser('~/.workbuddy/skills/mck-ppt-design'))`

---

## Knowledge Routing (Context-Loading Index)

> **At the beginning of each stage, read the corresponding files.** Do not load everything at once.

| Stage | Must Read | Purpose |
|------|-----------|---------|
| S1 Requirements | `references/team/brand-guide.md` | Understand design constraints |
| S2 Structure | `references/framework/engine-api.md` + `references/layout-matrix.yaml` | Layout selection and capability boundaries |
| S3 Content | `references/framework/guard-rails.md` + `experiences/*.md` (when present) | Overflow-prevention rules + lessons learned |
| S4 Rendering | Relevant `references/layouts/*.md` files only | Layout implementation details |
| S5 Delivery | None | — |

**Complete routing table**: `references/INDEX.md`

---

## Five-Stage Generation Workflow

```text
+-------------+   +----------------+   +----------------+   +-------------------+   +-------------+
| S1 Require. |-->| S2 Structure * |-->| S3 Content *   |-->| S4 Render+QA **   |-->| S5 Delivery |
| brief.md    |   | outline.json   |   | content.json   |   | .pptx             |   | + Learning  |
+-------------+   +----------------+   +----------------+   +-------------------+   +-------------+
                       * = gate. If FAIL, fix it inside the current stage. Do not skip.
              S3/S4 gates must run gate_check scripts and read JSON results. Never declare pass verbally.
```

### Stage 1: Requirements Definition

**Read**: `references/team/brand-guide.md`

**Collect**:
- Audience (role / decision responsibility)
- Objective (decision / reporting / persuasion)
- Duration (minutes → approximately 1 minute per slide)
- Key information (maximum 5 core messages)
- Data sources (what data is available)

**Output**: Create `ppt-project-{slug}/brief.md` in the working directory.

**Gate**: `audience` + `goal` + `key_messages` must all be non-empty. AI self-assessment is acceptable for this gate.

---

### Stage 2: Structure Design

**Read**: `references/framework/engine-api.md`, `references/layout-matrix.yaml`

**Tasks**:
1. Determine the number of slides from presentation duration (approximately 1 minute per slide)
2. Choose a `layout` for every slide using the engine-api.md quick-reference table
3. Write one core `key_point` per slide as a complete sentence, not a label
4. Confirm every layout is within the supported capability boundaries in layout-matrix.yaml

**Output**: `ppt-project-{slug}/outline.json`

```json
{
  "brief": {"audience": "Board of Directors", "goal": "Strategy review", "duration_minutes": 15},
  "slides": [
    {"idx": 1, "layout": "cover", "title": "Q1 2026 Strategy Review", "key_point": ""},
    {"idx": 2, "layout": "toc", "title": "Agenda", "key_point": ""},
    {"idx": 3, "layout": "table_insight", "title": "Three market shifts are reshaping the strategic agenda", "key_point": ""}
  ]
}
```

**S2 Gate** (AI self-assessment):
- [ ] A `cover` slide exists
- [ ] Slide count fits the duration constraint (`count <= duration_minutes * 1.2`)
- [ ] Every layout is defined in `layout-matrix.yaml`
- [ ] Every Action Title is a complete sentence (`len > 10` and contains a verb)
- [ ] Number of `two_column_text` layouts ≤ 1

---

### Stage 3: Content Population

**Read**: `references/framework/guard-rails.md`, all existing `experiences/*.md` files

**Tasks**:
1. Fill every slide with concrete copy, numbers, and chart data
2. Ensure every slide has a `source`
3. Keep text volume within the `char_budget` defined in `layout-matrix.yaml`

**Output**: `ppt-project-{slug}/content.json`

**S3 Gate** (must be machine-readable; AI self-assessment is not allowed):

```bash
python ~/.workbuddy/skills/mck-ppt-design/references/scripts/gate_check_s3.py \
    <project_directory>/content.json  <project_directory>
```

Read `<project_directory>/gate_s3.json`:
- `"passed": true` → enter S4
- `"passed": false` → fix the issues listed in `fail_items` in `content.json`, then run the check again

---

### Stage 4: Rendering + QA

**Read**: The `references/layouts/*.md` files corresponding only to the layouts actually used

**Tasks**:
1. Generate the Python rendering script from `content.json`
2. Execute the script to generate the `.pptx`
3. Run the QA gate script

**S4 Gate** (must be machine-readable; do not verbally declare a pass):

```bash
python ~/.workbuddy/skills/mck-ppt-design/references/scripts/gate_check.py \
    <pptx_path>  <project_directory>
```

Read `<project_directory>/gate_result.json`:
- `"passed": true` → enter S5
- `"passed": false` → review `user_code_errors`, fix the rendering code, render again, and rerun the check

**Note**: `engine_bug` errors, including whitelisted entries such as `peer_font_inconsistency`, are automatically exempted by the script. AI does not need to make a verbal judgment.

**Output**: `<project_directory>/gate_result.json` + `.pptx`

---

### Stage 5: Delivery + Self-Refinement

**Tasks**:
1. Read `gate_result.json` and confirm `passed: true`. Never claim a pass if this file does not exist.
2. Deliver the `.pptx` file
3. **Self-Refinement**: determine whether any correction made during this run was pattern-level

**Self-Refinement Protocol**:

```text
After a correction, classify it as:

  ONE-TIME (specific adjustment for this deck only)
    → No need to persist the lesson

  PATTERN (reproducible across decks)
    → Must be written to experiences/
      - Overflow issues → experiences/overflow.md
      - Chart limitations → experiences/chart-limits.md
      - Layout pitfalls → experiences/layout-pitfalls.md
      - CJK rendering → experiences/cjk-issues.md

Format:
  ## Experience NNN: {title}
  **Date**: YYYY-MM-DD
  **Problem**: ...
  **Root Cause**: ...
  **Fix**: ...
  **Rule**: ... (how to prevent it at the gate level)
```

---

## Fast Track (Skip S2/S3 Gates for Simple Requests)

The AI may skip the S2 structure review and S3 content review only when **all** conditions below are met:
- Total slide count ≤ 5
- No data charts (`donut`, `bar`, `table`, etc.)
- The user explicitly requests something "quick", "needed immediately", or "simple"

**Still mandatory**: S1 requirements collection + S4 QA gate (run gate_check.py) + S5 delivery

---

## Checkpoint / Recovery

When the user says "continue working on that PPT":

```python
import os, json, glob

projects = glob.glob('ppt-project-*/')
for proj in projects:
    brief   = os.path.exists(f'{proj}brief.md')
    outline = os.path.exists(f'{proj}outline.json')
    content = os.path.exists(f'{proj}content.json')
    gate_s3 = os.path.exists(f'{proj}gate_s3.json')
    gate_s4 = os.path.exists(f'{proj}gate_result.json')
    pptx    = bool(glob.glob(f'{proj}*.pptx'))

    if not brief:           stage = 1
    elif not outline:       stage = 2
    elif not content:       stage = 3
    elif not gate_s3:       stage = '3-gate'   # content exists, but the gate has not run
    elif not pptx:          stage = 4
    elif not gate_s4:       stage = '4-gate'   # pptx exists, but the gate has not run
    else:                   stage = 5

    print(f"Project {proj}: currently at Stage {stage}")
```

---

## MckEngine Quick Template

```python
import sys, os
sys.path.insert(0, os.path.expanduser('~/.workbuddy/skills/mck-ppt-design'))
from mck_ppt import MckEngine
from mck_ppt.constants import *

eng = MckEngine(total_slides=12)
eng.cover(title='Title', subtitle='Subtitle', author='Author', date='2026')
eng.toc(items=[('1', 'Section One', 'One-sentence description'), ('2', 'Section Two', 'One-sentence description')])
# ... content slides ...
eng.closing(title='Thank You', message='We look forward to continuing the discussion')
eng.save('output/deck.pptx')  # automatically runs full_cleanup
```

**Detailed API**: `references/framework/engine-api.md`  
**Layout specifications**: `references/layouts/`  
**10 guard rails**: `references/framework/guard-rails.md`  
**Layout capability boundaries**: `references/layout-matrix.yaml`  
**Historical lessons learned**: `experiences/`  
**S3 gate script**: `references/scripts/gate_check_s3.py`  
**S4 gate script**: `references/scripts/gate_check.py`
