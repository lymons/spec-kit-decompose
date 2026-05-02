# spec-kit-decompose

A Spec Kit extension that bridges the gap between product-level documents and feature-level `/speckit.specify` calls.

## Problem

Spec Kit's `/speckit.specify` command operates at feature scope: one feature in, one spec out, one branch created. When a team has a large PRD, tech spec, or requirements document, there's no structured way to decompose it into the discrete features that `/speckit.specify` expects.

This extension fills that gap with a three-command decomposition pipeline.

## Commands

### `/speckit.decompose <path>`

Parse a source document and produce a feature map at `.specify/decompose/feature-map.md`.

The command identifies candidate features, classifies priorities, estimates complexity, extracts dependency relationships, and detects shared foundational infrastructure. Each feature gets a pre-drafted prompt for `/speckit.specify`.

Supported source formats: MoSCoW tables, checkbox lists, numbered roadmaps, epic/story hierarchies, free-form prose PRDs, and technical spec documents.

### `/speckit.decompose.select`

Interactively select the next feature to specify. Shows features in dependency-safe order, validates that prerequisites are met, and outputs a copy-friendly `/speckit.specify` prompt.

This command is read-only — it does not invoke `/speckit.specify` or modify any files. The user stays in control of when and how they run the core command.

### `/speckit.decompose.status`

Show decomposition progress across all features. Scans `.specify/specs/` to detect which features have been specified, planned, tasked, or completed — and auto-updates the feature map to keep it in sync.

Status is derived from which artifacts exist:

| Artifact | Status |
|----------|--------|
| No spec directory | `pending` |
| `spec.md` exists | `specified` |
| `plan.md` exists | `planned` |
| `tasks.md` exists | `tasked` |
| All tasks checked off | `done` |

## Installation

Install via the Spec Kit extension manager:

```
/speckit.install spec-kit-decompose
```

Or clone into your extensions directory:

```
git clone https://github.com/chrishaveard/spec-kit-decompose.git ~/.speckit/extensions/spec-kit-decompose
```

Requires Spec Kit ≥ 0.4.0.

## Workflow

```
1. Write your PRD / tech spec / requirements doc

2. /speckit.decompose docs/PRD.md
   → Produces .specify/decompose/feature-map.md
   → Review and edit the feature map as needed

3. /speckit.decompose.select
   → Pick the next feature (dependency-safe)
   → Get a pre-drafted /speckit.specify prompt

4. /speckit.specify <prompt>
   → Spec Kit does its thing

5. /speckit.decompose.status
   → See progress, auto-update the feature map
   → Repeat from step 3
```

## File structure

```
spec-kit-decompose/
  extension.yml               # Extension manifest
  README.md
  LICENSE
  CHANGELOG.md
  commands/
    decompose.md              # /speckit.decompose
    decompose-select.md       # /speckit.decompose.select
    decompose-status.md       # /speckit.decompose.status
  templates/
    feature-map-template.md   # Template for the generated feature map
```

Artifacts created at runtime:

```
.specify/
  decompose/
    feature-map.md            # The decomposition artifact
    source-hash.txt           # SHA-256 of the source doc at decompose time
```

## Design decisions

**No auto-invocation of `/speckit.specify`**: The select command stops short of calling specify. This avoids coupling to spec-kit internals and preserves the user's ability to review and edit the prompt.

**Feature map as a durable file**: The feature map is editable, version-controllable, and survives across agent sessions. If you disagree with how features were split, edit the file directly.

**SHA-256 source tracking**: When the source document changes, the status command warns you. It doesn't do semantic diffing — just cheap change detection so you know to re-decompose.

**Foundation feature (000)**: Shared infrastructure (auth, database, build tooling) is front-loaded as Feature 000 to avoid duplicate scaffolding across business features.

**Auto-updating status**: The status command writes updated statuses back to the feature map, keeping it in sync with reality without manual edits.

## License

MIT
