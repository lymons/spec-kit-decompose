# /speckit.decompose.status

Show decomposition progress and auto-update the feature map statuses based on existing spec artifacts.

## Prerequisites

A feature map must exist at `.specify/decompose/feature-map.md`. If it does not exist, tell the user to run `/speckit.decompose <source-document>` first.

## Procedure

### Step 1: Read the feature map

Read `.specify/decompose/feature-map.md`. Parse the feature inventory table to extract each feature's number, name, slug (kebab-case directory name), and current status.

The slug is derived from the feature number and name: feature number zero-padded to 3 digits, a hyphen, and the name in kebab-case. For example, feature 1 "User Authentication" maps to directory `001-user-authentication`.

### Step 2: Check for source drift

If `.specify/decompose/source-hash.txt` exists, read the stored hash. Read the source document path from the feature map header and compute its current SHA-256 hash. If the hashes differ, include a warning in the status output:

> ⚠️ Source document has changed since last decomposition. Run `/speckit.decompose` to refresh.

### Step 3: Scan spec artifacts and determine status

For each feature in the inventory, check the `.specify/specs/` directory for a matching subdirectory. Determine status by which artifacts exist:

| Condition | Status |
|-----------|--------|
| No directory exists under `.specify/specs/` for this feature | `pending` |
| `spec.md` exists in the feature directory | `specified` |
| `plan.md` exists in the feature directory | `planned` |
| `tasks.md` exists in the feature directory | `tasked` |
| `tasks.md` exists and all task checkboxes are checked | `done` |

Status is determined by the **highest-stage artifact** present. The progression is: pending → specified → planned → tasked → done. If `plan.md` exists but `spec.md` does not, the status is still `planned` — the assumption is that the spec stage may have been skipped or the file was removed. Report this as a note but don't block on it.

For the `done` check: read `tasks.md` and look for Markdown checkboxes. If every `- [ ]` has been changed to `- [x]`, the feature is done. If the file has no checkboxes, treat it as `tasked`.

### Step 4: Auto-update the feature map

Compare the detected status of each feature with the status recorded in the feature map's inventory table. If any statuses have changed:

1. Update the Status column in the feature inventory table for each changed feature
2. Update the Status field in the corresponding Feature Details section
3. Write the updated feature map back to `.specify/decompose/feature-map.md`
4. Note which features were updated in the status output

This is the key difference from a read-only status command: the feature map stays in sync with reality without manual edits.

### Step 5: Present the status report

Display a clear progress report:

```
Feature Map Status (source: docs/PRD.md)
=========================================

  [done]      000 — Foundation
  [specified] 001 — User Authentication
  [planned]   002 — Project Dashboard
  [pending]   003 — Kanban Board
  [pending]   004 — Task Comments
  [pending]   005 — Drag-and-Drop Reordering
  [pending]   006 — Email Notifications
  [pending]   007 — CSV Export

Progress: 3/8 features in-flight or complete
Next recommended: F3 (Kanban Board) — all dependencies satisfied
```

Include:

1. **Each feature** with its status indicator and name
2. **Progress summary**: count of features that are not `pending` vs total
3. **Next recommendation**: the highest-priority feature whose dependencies are all satisfied (status ≥ `specified`)
4. **Source drift warning** if applicable (from Step 2)
5. **Auto-update notes**: list any features whose status was just updated, e.g.:
   ```
   Updated: F1 pending → specified, F0 tasked → done
   ```

### Step 6: Suggest next action

Based on the current state, suggest what to do next:

- If there are selectable features: "Run `/speckit.decompose.select` to choose the next feature to specify."
- If all features are specified or beyond: "All features have been specified. Continue with `/speckit.plan` and `/speckit.tasks` for individual features."
- If all features are done: "All features are complete. The decomposition is finished."
- If source drift was detected: "Consider running `/speckit.decompose` to refresh the feature map from the updated source document."

## Edge cases

- **No `.specify/specs/` directory**: All features are pending. This is normal if no features have been specified yet.
- **Spec directory exists but doesn't match any feature**: Ignore it — it may be from a previous decomposition or manually created.
- **Feature map has been manually edited with non-standard statuses**: Preserve any status value the auto-update doesn't override. Only update features where artifact detection yields a different status.
- **Feature directory name doesn't match expected slug**: Try a fuzzy match — match on the feature number prefix (e.g., `001-`) regardless of the name portion. If no match is found, leave the feature as its current status.
