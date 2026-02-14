# Appian Atlas Power - Developer

Technical deep-dive into Appian applications for developers who need implementation details, SAIL code, and dependency analysis.

## What This Power Does

This power provides technical, implementation-focused responses when exploring Appian applications through the Appian Atlas MCP server. Perfect for developers building and maintaining Appian solutions.

## Features

- **Technical identifiers**: Shows UUIDs and full object names
- **SAIL code access**: Loads complete implementation details
- **Dependency tracing**: Full `calls[]` and `called_by[]` analysis
- **Technical terminology**: Expression Rules, CDTs, Process Models, etc.
- **Code quality insights**: Identifies technical debt, orphaned objects, high coupling

## Installation

1. In Kiro IDE, go to the **Powers** tab
2. Click **Add Power**
3. Enter this URL:
   ```
   https://github.com/ram-020998/power-appian-atlas-developer
   ```

## Prerequisites

You must have the **appian-atlas MCP server** installed. See the [gam-knowledge-base repository](https://github.com/ram-020998/gam-knowledge-base) for installation instructions.

## Other Personas

- **Product Owner**: [power-appian-atlas-product-owner](https://github.com/ram-020998/power-appian-atlas-product-owner) - Business-focused, no technical jargon
- **UX Designer**: [power-appian-atlas-ux-designer](https://github.com/ram-020998/power-appian-atlas-ux-designer) - UI/UX focused with interface analysis

## Example Usage

```
User: "How does the Add Vendors action work?"

Response: "The Add Vendors action (Record Action) triggers process model AS_GSS_PM_addVendors.

Technical flow:
1. Entry form: AS_GSS_FM_addVendors (UUID: _a-0000e30a...)
   - Calls: AS_GSS_BL_validateVendors for validation
2. Process model: AS_GSS_PM_addVendors
   - Writes to SourceSelection record
   
[Shows full SAIL code and dependency graph]"
```

## License

MIT
