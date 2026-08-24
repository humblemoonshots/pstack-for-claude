---
name: setup-pstack
description: Configure which models pstack uses per role. Detects your available models and writes a config file that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", or changing pstack's model choices.
---

# Setup pstack

Write `.claude/pstack-models.md`, a project-local config file that sets pstack's model per role. The skills read it when present and fall back to their inline defaults when a line is absent, so this is an override layer, not a requirement.

## Steps

### 1. Detect available models

Enumerate the model values you can pass to an `Agent` subagent in this session; that is the dependable source. The aliases are `opus`, `fable`, `sonnet`, and `haiku`; full model IDs (`claude-opus-5`, `claude-sonnet-5`) also resolve, and `inherit` runs the role on the parent chat model. Your organization may restrict the set via `availableModels`, so confirm rather than assume. If you cannot detect any, ask the user which models they have access to. Never write a real model value you have not confirmed is available. The alias `inherit` is always valid.

### 2. Load current state

The default role-to-model mapping is the shape shown in step 5 below. If `.claude/pstack-models.md` already exists, read it and treat its values as the current choices. Otherwise start from those defaults.

### 3. Map and confirm

Show every role with its current model, marking any value not in the detected set as needing a choice. Ask whether to accept as-is or change specific roles, offering the detected models plus `inherit` (meaning: this role runs on the parent chat model) as the options. Prefer AskUserQuestion over free text. For panel roles (how critics, arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. Repeating a model in a panel list is legitimate: two runs of the same model from different framings still diverge. `arena cross-judge pool` is also a list, but Arena selects one value from it whose model differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every model value written must be in the detected set; `inherit` always passes. If a chosen value is not available, stop and ask again. A config pointing at a model the user cannot use breaks every delegation that reads it.

### 5. Write the config

Write `.claude/pstack-models.md` with one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit` as a value: the role runs on the parent chat model (omit the Agent call's `model`).
# Alias entries in a panel list still count toward its fan-out.
feature, refactoring: sonnet
bug-fix: opus
perf-issue: opus
hillclimb: opus
judgment and prose: fable
hardest tasks: fable
how explorer: sonnet
how explainer: fable
how critics: opus, fable, sonnet
why investigators: sonnet
why synthesizer: fable
reflect tooling: opus
reflect judgment, divergent, synthesizer: fable
arena runners: opus, fable, sonnet
arena cross-judge pool: opus, fable, sonnet
swarm workers: sonnet
architect runners: opus, fable, sonnet
interrogate reviewers: opus, fable, sonnet
```

Reasoning effort is a session-level setting in Claude Code, not a per-call one: `Agent` takes `model` but not `effort`. When a role needs deeper reasoning than the session is running at, raise it with `/model` before the fan-out, or give that role a dedicated subagent definition in `.claude/agents/` with an `effort` field.

### 6. Confirm

Tell the user the config was written and that skills pick it up on their next invocation. Re-running this skill updates it. If the project commits `.claude/`, the config is shared with the team; say so.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, an existing harness, or Claude Code's bundled `/run` and `/verify`). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed — project, user, or plugin). On no, move on without pushing.
