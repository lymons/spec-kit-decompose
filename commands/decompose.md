# /speckit.decompose

Decompose a source document into discrete, dependency-ordered features suitable for `/speckit.specify`.

## Input

The user provides a path to a source document:

```
/speckit.decompose <path-to-source-document>
```

If no path is provided, ask the user to specify one. The source document is typically a PRD, technical spec, requirements doc, or similar product-level document.

## Procedure

### Step 1: Read and understand the source document

Read the file at the provided path. If the file does not exist or is empty, tell the user and stop.

Identify the document format. The following formats are common — use the appropriate parsing strategy:

- **MoSCoW tables**: Look for tables with columns like Feature, Priority, Status where priority values are Must/Should/Could/Won't. Extract each row as a candidate feature.
- **Checkbox lists**: Lines matching `- [ ] Feature name` or `- [x] Feature name`. Each checkbox item is a candidate feature. Checked items may indicate already-completed work.
- **Numbered roadmap sections**: Headings like `### Must Have`, `### Should Have`, `### Nice to Have` with feature descriptions beneath. Each described unit is a candidate feature.
- **Epic/story hierarchies**: Headings like `## Epic: ...` with nested `### Story: ...`. Epics may map to features, or stories may, depending on granularity. Prefer story-level unless stories are trivially small.
- **Free-form prose PRDs**: Narrative descriptions without structured lists. Extract features by identifying distinct functional capabilities, user-facing behaviors, or system components described in the prose. This requires more judgment — err on the side of fewer, larger features rather than many tiny ones.
- **Technical spec documents**: Architecture descriptions, API designs, system diagrams. Identify implementable units: services, modules, API endpoints, data models. Group related endpoints into a single feature rather than one feature per endpoint.

If the format is ambiguous or you are not confident in the extraction, present the candidate features to the user and ask them to confirm, merge, split, or remove entries before proceeding.

### Step 2: Identify features and dependencies

For each candidate feature:

1. **Name it** with a short, descriptive title (2-5 words).
2. **Classify priority** using signals from the source document:
   - MoSCoW labels → map directly to must-have / should-have / could-have / won't-have
   - Numbered priorities → map to must-have (1) / should-have (2) / could-have (3)
   - Checkbox status → checked = done, unchecked = pending (priority unclear, ask user)
   - Roadmap ordering → items listed first are higher priority
   - Explicit priority fields → use as-is
   - No signal found → ask the user to assign priority
3. **Estimate complexity** as small / medium / large based on:
   - small: Single-concern feature, limited UI, straightforward data model
   - medium: Multiple concerns, moderate UI, some integration points
   - large: Cross-cutting, complex UI, significant data modeling or external integrations
4. **Identify dependencies**: Which other features must exist before this one can be built? Express as feature references (F1, F2, etc.). A feature depends on another if it consumes that feature's API, data model, or UI components.
5. **Record source sections**: Note which sections, headings, or line ranges of the source document this feature was derived from.
6. **Write a boundary rationale**: One sentence explaining why this is a separate feature and where the boundary was drawn.
7. **Draft a specify input**: Write a 2-4 sentence prompt suitable for `/speckit.specify`. This should describe what to build and why, at the right abstraction level — what and why, not how. Include key acceptance criteria or constraints from the source document.

### Step 3: Detect foundational infrastructure

Look for cross-cutting concerns that multiple features depend on:

- Database schema, connection pooling, migration framework
- Authentication/authorization middleware
- Project scaffolding, build tooling, CI/CD pipeline
- Shared component library, design system, or design tokens
- API gateway, routing, error handling patterns
- Logging, monitoring, observability infrastructure

If two or more features share a dependency on infrastructure that doesn't exist yet, recommend a "Feature 000 — Foundation" that front-loads this work. List the specific foundation scope items.

If no shared infrastructure is detected (e.g., features are independent microservices with their own stacks), skip the foundation recommendation.

### Step 4: Build the dependency graph

Produce a topological ordering of features based on the dependencies identified in Step 2. The graph should be rendered as an ASCII tree showing which features block which others.

Validate the graph:
- If there are circular dependencies, flag them to the user and ask for resolution.
- If a feature has no dependencies and is not depended on by anything, it can be specified in any order — note this.

### Step 5: Compute the source hash

Compute the SHA-256 hash of the source document content. This will be stored alongside the feature map so that future commands can detect if the source has changed.

### Step 6: Write the feature map

Create the directory `.specify/decompose/` if it does not exist.

Write `.specify/decompose/feature-map.md` using the template at `templates/feature-map-template.md` as a structural guide. Fill in all sections:

- Header metadata (source path, date, feature count, decomposition strategy)
- Feature inventory table with all features numbered sequentially (000 for foundation if applicable, then 001+)
- Dependency graph
- Foundation section (if applicable)
- Feature detail sections for every feature

Write `.specify/decompose/source-hash.txt` containing only the SHA-256 hex string.

### Step 7: Present results to the user

Show the user:
1. A summary: how many features were identified, whether a foundation was recommended
2. The feature inventory table
3. The dependency graph
4. The recommended next step: "Run `/speckit.decompose.select` to choose a feature to specify, or edit `.specify/decompose/feature-map.md` to adjust the decomposition."

## Error handling

- **File not found**: Tell the user the path doesn't exist. Suggest checking the path.
- **Empty file**: Tell the user the document is empty. Nothing to decompose.
- **No features detected**: Tell the user you couldn't identify discrete features. Ask them to point out which sections describe distinct features.
- **Ambiguous structure**: Present your best-guess feature list and ask for confirmation before writing the feature map.
- **Existing feature map**: If `.specify/decompose/feature-map.md` already exists, warn the user that re-running will overwrite it. Ask for confirmation. Mention they can use version control to preserve the previous version.

## Decomposition strategy

Record which strategy was used in the feature map header. Common strategies:

- **domain-boundary**: Features correspond to distinct business domains or bounded contexts
- **user-journey**: Features map to discrete user workflows or user stories
- **component**: Features map to system components or services
- **incremental-delivery**: Features are sliced for incremental delivery (vertical slices through the stack)
- **hybrid**: A mix of the above — note which approach was used for which features

Choose the strategy that best fits the source document's structure. If the document is organized by user stories, use user-journey. If it's organized by system components, use component. For most PRDs, domain-boundary or user-journey will be the best fit.
