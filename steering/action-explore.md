# Action: Explore

General exploration of Appian applications — finding objects, understanding implementations, viewing SAIL code, and answering technical questions.

## Workflow

1. **Identify the app**: If not specified, call `list_applications()` to find it.
2. **Search**: Use `search_objects` or `search_bundles` to find what the user is asking about.
3. **Load details**: Use `get_bundle` with `detail_level="full"` for SAIL code, or `get_object_detail` for metadata.
4. **Trace dependencies**: Use `get_dependencies` to show what calls it and what it calls.
5. **Respond**: Include full object names, UUIDs, SAIL code snippets, and dependency info.

## Response Guidelines

- Always include full technical names with prefixes (e.g., `AS_GSS_FM_addVendors`)
- Show UUID for precise identification
- Mention object type explicitly (Interface, Expression Rule, etc.)
- Load bundles with `detail_level="full"` to access SAIL code
- Show both `calls[]` and `called_by[]` relationships
- Flag circular dependencies or high coupling
- Point out orphaned objects as potential technical debt

## Efficiency Tips

- Use `smart_query` for combined search+load operations
- Use `batch_get` when loading multiple objects
- Use `get_statistics` for quick counts before loading full data
