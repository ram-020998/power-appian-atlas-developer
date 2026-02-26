# Action: Technical Debt

Identify and analyze technical debt — orphaned objects, unused code, high-complexity areas.

## Workflow

1. **Get overview**: `get_app_overview(app)` for coverage stats and bundle summary.
2. **Find orphans**: `list_orphans(app)` to get unbundled objects.
3. **Analyze orphans**: `get_orphan(app, uuid)` for code and dependencies of key orphans.
4. **Check stats**: `get_statistics(app, "orphan_summary")` for debt by type.
5. **Find complexity**: `get_statistics(app, "bundle_complexity", {"limit": 10})` for most complex bundles.
6. **Check tags**: `search_by_tags(app, ["complex", "integration_heavy"])` for flagged objects.
7. **Report**: Categorize debt and recommend prioritized cleanup.

## Orphan Categories

- **Dead code** — no dependencies, safe to delete
- **Missing from bundles** — has dependencies but not bundled, needs bundling
- **Legacy** — old implementations replaced by newer ones, needs migration plan

## Response Guidelines

- Group orphans by type and category
- Show dependency counts to assess if truly unused
- Recommend specific actions (delete, bundle, migrate)
- Prioritize by risk and effort
