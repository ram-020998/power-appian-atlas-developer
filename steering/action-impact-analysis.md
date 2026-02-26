# Action: Impact Analysis

Analyze the impact of changing an Appian object — who depends on it, what it depends on, and how wide the blast radius is.

## Workflow

1. **Find the target object**: `search_objects(app, query)` to get UUID and full name.
2. **Trace dependencies**: `get_dependencies(app, object_name)` for `calls[]` and `called_by[]`.
3. **Assess scope**: For each consumer in `called_by[]`, check which bundles they belong to.
4. **Check reuse**: `get_statistics(app, "object_reuse")` or `get_object_enrichment(app, uuid)` for dependent_count.
5. **Batch load consumers**: `batch_get(app, "enrichments", [uuids])` for depth and complexity of impacted objects.
6. **Report**: Show dependency tree, bundle impact, and risk assessment.

## Response Guidelines

- Show the full dependency tree (both directions)
- Group impacted objects by bundle
- Highlight shared utilities with high dependent_count
- Flag cross-bundle dependencies
- Recommend testing scope based on impact
