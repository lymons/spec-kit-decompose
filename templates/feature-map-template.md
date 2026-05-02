# Feature Map

**Source**: {{SOURCE_PATH}}
**Generated**: {{DATE}}
**Total Features Identified**: {{FEATURE_COUNT}}
**Decomposition Strategy**: {{STRATEGY}}

---

## Feature Inventory

| # | Feature | Priority | Complexity | Dependencies | Status |
|---|---------|----------|------------|--------------|--------|
{{FEATURE_ROWS}}

## Dependency Graph

{{DEPENDENCY_GRAPH}}

## Foundational Feature

{{#IF_FOUNDATION}}
Cross-cutting infrastructure was detected that multiple features depend on.
A "Feature 000 — Foundation" spec is recommended before any business features.

**Recommended foundation scope**:
{{FOUNDATION_SCOPE}}
{{/IF_FOUNDATION}}

{{#IF_NO_FOUNDATION}}
No shared foundational infrastructure was detected. Features can be specified independently in dependency order.
{{/IF_NO_FOUNDATION}}

---

## Feature Details

{{#EACH_FEATURE}}
### F{{NUMBER}}: {{NAME}}

**Source sections**: {{SOURCE_SECTIONS}}
**Priority**: {{PRIORITY}}
**Complexity**: {{COMPLEXITY}}
**Dependencies**: {{DEPENDENCIES}}
**Status**: {{STATUS}}

**Summary**: {{SUMMARY}}

**Boundary rationale**: {{BOUNDARY_RATIONALE}}

**Estimated specify input**:
> {{SPECIFY_INPUT}}

---
{{/EACH_FEATURE}}
