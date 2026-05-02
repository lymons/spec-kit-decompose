# /speckit.decompose.select

Interactively select the next feature to specify from the feature map. This command is read-only — it does not modify files or invoke `/speckit.specify`.

## Prerequisites

A feature map must exist at `.specify/decompose/feature-map.md`. If it does not exist, tell the user to run `/speckit.decompose <source-document>` first.

## Procedure

### Step 1: Read the feature map

Read `.specify/decompose/feature-map.md`. Parse the feature inventory table to extract each feature's number, name, priority, complexity, dependencies, and status.

### Step 2: Check for source drift

If `.specify/decompose/source-hash.txt` exists, read the stored hash. Then read the source document path from the feature map header and compute its current SHA-256 hash. If the hashes differ, warn the user:

> ⚠️ The source document has changed since the last decomposition. Consider re-running `/speckit.decompose` to pick up changes.

Continue with the selection flow regardless — the warning is informational.

### Step 3: Identify selectable features

A feature is selectable if:
1. Its status is `pending` (no spec artifacts exist yet)
2. All of its dependencies have status `specified`, `planned`, `tasked`, or `done`

Build the list of selectable features. Also build a list of blocked features (pending but with unmet dependencies) for context.

### Step 4: Present the selection

Show the user:

1. **Selectable features** ordered by priority (must-have first), then by feature number:
   ```
   Ready to specify:
     [1] F1: User Authentication (must-have, medium)
     [3] F7: CSV Export (could-have, small)
   ```

2. **Blocked features** with the reason they're blocked:
   ```
   Blocked (dependencies not yet specified):
     F2: Project Dashboard — waiting on F1
     F3: Kanban Board — waiting on F1, F2
   ```

3. **Completed features** for context:
   ```
   Already in progress or done:
     [specified] F0: Foundation
   ```

4. **Recommendation**: Suggest the highest-priority selectable feature as the default choice.

Ask the user which feature to specify next. Accept a feature number or name.

### Step 5: Validate the selection

If the user selects a feature:

1. Confirm all dependencies are satisfied. If not (e.g., the user typed a blocked feature number), explain which dependencies are missing and suggest specifying those first.
2. If valid, show the feature's detail section from the feature map, including:
   - Summary
   - Source sections
   - Boundary rationale
   - The pre-drafted specify input

### Step 6: Output the specify prompt

Present the pre-drafted specify input in a copy-friendly format:

```
Suggested /speckit.specify input:

  /speckit.specify Build user authentication with email/password registration,
  login, session management, and three roles: admin, member, viewer. Admins
  can manage team membership. Sessions expire after 24 hours of inactivity.
```

Tell the user they can use this as-is or modify it, then run `/speckit.specify` with their chosen prompt.

**Do not invoke `/speckit.specify` yourself.** The user controls when and how they run that command.

### Step 7: Remind about status tracking

After the user has the specify prompt, remind them:

> After running `/speckit.specify`, run `/speckit.decompose.status` to update the feature map with the new status.

## Edge cases

- **All features complete**: Tell the user all features have been specified or are in progress. The decomposition is done.
- **No selectable features but pending features exist**: All remaining features are blocked. Show the dependency chain and explain what needs to happen to unblock them.
- **Feature map is malformed**: If the feature inventory table can't be parsed, tell the user the feature map may have been edited in a way that broke the expected structure. Suggest re-running `/speckit.decompose` or fixing the table format.
