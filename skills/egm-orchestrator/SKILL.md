---
name: egm-orchestrator
description: >-
  Operational playbook for EGM Conductor covering automatic intent routing,
  workspace parameter inference, adaptive review presets, and chaining across
  the 7 skills and 10 Goldfish personas of the Elephant-Goldfish Model (EGM) plugin.
---

# EGM Orchestrator Playbook

This playbook defines how **EGM Conductor** translates natural-language prompts into direct execution of the installed **Elephant-Goldfish Model (EGM)** plugin skills.

## Base EGM Plugin Inventory

- **Default install root**: `~/.gemini/config/plugins/egm/` (if installed elsewhere, locate the `egm` plugin directory first).
- **Skills** (all declare `disable-model-invocation: true` — read the corresponding `SKILL.md` before executing):
  - `skills/egm-brainstorm/SKILL.md`
  - `skills/egm-draft/SKILL.md`
  - `skills/egm-review/SKILL.md`
  - `skills/egm-review-loop/SKILL.md`
  - `skills/egm-resolve/SKILL.md`
  - `skills/egm-bootstrap/SKILL.md`
  - `skills/egm-drift-report/SKILL.md`
- **Document Standards** (`resources/document_standards/`):
  - `heilmeier-standard.md`
  - `feature-spec-standard.md`
  - `zoom-out-standard.md`
  - `implementation-readiness-standard.md`
  - `re-implementation-standard.md`
- **Goldfish Subagents** (`invoke_subagent` `TypeName`):
  - `egm-zoom-out`, `egm-comprehension`, `egm-design`, `egm-style`, `egm-structural`, `egm-codebase-details`, `egm-implementation-readiness`, `egm-inventive-catalyst`, `egm-synthesizer`, `egm-bootstrap-orchestrator`.

## 5-Step Execution Protocol

### Step 1 — Triage (Strict Spec-First)

- **Bypass** only for pure questions or trivial few-line edits with no architectural impact. Output:
  `[EGM → bypass | reason: <short reason>]`
- **Invoke EGM** for any logic change, multi-file modification, ideation, design, review, documentation bootstrap, or drift audit.

### Step 2 — Silent Workspace Probing

Before asking the user for any path or parameter, inspect the active workspace root:

1. `./concepts_brief.md` (default output of `egm-brainstorm`).
2. `./design_draft.md` (default output of `egm-draft`).
3. `./docs/designs/` (versioned design specs and `SITEMAP.md`).
4. Recent review or audit reports (`consolidated_review_report.md`, `goldfish_review_report.md`, `*_DRIFT_REPORT.md`).
5. Internal links inside the target document to populate `GoldfishContext` automatically (external links outside the workspace and URLs still require explicit user approval per `egm-review`).

Ask the user for a path only when multiple candidate files are genuinely ambiguous or none exist.

### Step 3 — Intent Routing and Parameter Pre-Fill

| Detected intent | Target skill | Automatically inferred parameters |
| :--- | :--- | :--- |
| New feature, logic change without a spec, open-ended idea | `egm-brainstorm` | Prompt context + relevant repo files. Ask open-ended plain-text questions (never `ask_question`) until the *readiness gate*. |
| Turn a brief or notes into a design doc | `egm-draft` | `Context File`: `./concepts_brief.md` (if present) or conversation notes. Pause section-by-section only at architectural decision gates. |
| Review a design doc (general request) | `egm-review` | `TargetFile`: `./design_draft.md` or referenced doc. **Adaptive mode**: if a `## Detailed Implementation Plan` (or synonym) header exists, use `Step=uninterrupted`; otherwise use `Step=feature_spec`. |
| Validate readiness to implement | `egm-review` | `Step=implementation` (Stage 9: `egm-implementation-readiness` + `egm-codebase-details`). |
| Step-by-step interactive review | `egm-review` | `Step=interactive` (only when the user explicitly requests per-stage checkpoints). |
| Targeted review (specific section, lens, or non-standard doc: ADR, PRD, README) | `egm-review-loop` | Pre-fill all 5 parameters (`TargetFile`, `SemanticTarget`, `Reviewers`, `Standards`, `GoldfishContext`) to skip 5-turn manual intake. |
| Resolve a hard architectural trade-off or contradiction | `egm-brainstorm` or `egm-review-loop` | Explicitly include `egm-inventive-catalyst` (TRIZ). |
| Triage and apply review findings | `egm-resolve` | `TargetFile` + `LineRange=full file` + `FindingsList` from the latest review report. |
| Map or document a codebase subtree | `egm-bootstrap` | `path`: requested directory (or repo root). If the user wants to preserve human-maintained READMEs, set `target_filename="EGM.md"`. Enforce the `>15` subdirectories safety gate. |
| Check code vs. spec divergence | `egm-drift-report` | Target directory: `docs/designs/` (or specified path). |

### Step 4 — Automatic Post-Review Chaining

Whenever `egm-review` (`uninterrupted`, `feature_spec`, or `implementation`) or `egm-review-loop` produces findings:

1. **Do not stop at the raw report** (unless the user explicitly asks for raw Goldfish output only).
2. Run `egm-resolve` immediately in the main session (*Elephant*), preserving decision context.
3. Deliver `goldfish_review_report.md` with each item categorized as:
   - `PROPOSE FIX` (ready to apply upon approval)
   - `CONTEST` (backed by verifiable file/symbol evidence)
   - `ESCALATE` (requires user decision or external context)
   - `DECLINE` (nits logged without edits)

### Step 5 — Guided Handoff (1 Line)

End every completed stage with a single line indicating the next natural step:

- After `egm-brainstorm` → *"Brief saved to `concepts_brief.md`. Ready to draft the technical spec (`egm-draft`) whenever you are."*
- After `egm-draft` → *"Draft saved to `design_draft.md`. Ready to run the Goldfish review + triage (`egm-review` → `egm-resolve`)."*
- After `egm-resolve` → *"Fixes applied. Next natural step: promote the spec to `docs/designs/<feature>.md` and start implementation."*
- After `egm-bootstrap` → *"Contracts generated. Optionally create a `<project>-elephant` context skill to speed up future sessions."*
- After `egm-drift-report` → *"Report saved to `{YYYY-MM-DD}_DRIFT_REPORT.md`. Specify which divergences to fix in code and which to update in specs."*

## Safety Invariants

- Never simulate a Goldfish review in the same conversation context; if fresh subagents cannot be spawned, stop and inform the user.
- Never run `git add` or `git commit` during `egm-bootstrap` or documentation flows.
- Never silently resolve architectural decisions with multiple defensible trade-offs — present up to 3 options with mechanisms, trade-offs, and failure conditions for the author to choose.
