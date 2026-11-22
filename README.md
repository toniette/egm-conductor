# EGM Conductor

An Antigravity orchestrator plugin for the [Elephant-Goldfish Model (EGM)](https://github.com/google/elephant-goldfish-model). It turns natural-language prompts into the right EGM skills, adaptive presets, and isolated Goldfish reviews while eliminating repetitive manual intake.

## Prerequisites

Install the official **Elephant-Goldfish Model (`egm`)** plugin first:

```sh
agy plugin install https://github.com/google/elephant-goldfish-model
```

## Install

```sh
agy plugin install https://github.com/toniette/egm-conductor
```

## What It Includes

- **Always-on orchestration rule (`rules/AGENTS.md`)**: Gives the primary agent the **EGM Conductor** persona, strict spec-first triage, discreet 1-line routing status (`[EGM → ...]`), and strict preservation of the *Elephant* vs. *Goldfish* memory boundary.
- **Operational playbook (`skills/egm-orchestrator/SKILL.md`)**: Progressively loaded routing table covering all 7 EGM skills (`egm-brainstorm`, `egm-draft`, `egm-review`, `egm-review-loop`, `egm-resolve`, `egm-bootstrap`, `egm-drift-report`), silent workspace probing, adaptive `egm-review` presets, and automatic post-review chaining into `egm-resolve`.

## Behavior Summary

- **Language-adaptive**: Responds in the user's prompt language; writes design artifacts in the repository's predominant language (defaulting to English for new projects).
- **Strict spec-first triage**: Routes logic changes and multi-file work through EGM; bypasses EGM only for direct questions and trivial few-line edits.
- **Maximum fluidity**: Auto-infers `TargetFile`, `GoldfishContext`, `Step`, and `Reviewers` from the workspace and pauses only at mandatory EGM gates (brainstorm readiness, architectural trade-offs, and `>15` subdirectories in `egm-bootstrap`).
- **Adaptive review + auto-triage**: Uses `Step=uninterrupted` for complete design docs and `Step=feature_spec` when no implementation plan exists yet, then automatically runs `egm-resolve` so findings arrive categorized (`PROPOSE FIX`, `CONTEST`, `ESCALATE`, `DECLINE`).

## License

[Apache 2.0](LICENSE)
