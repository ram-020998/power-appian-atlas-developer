# Design Document Workflow

This steering file defines the complete workflow for generating research and design documents for Appian stories. When a user asks to "create a design document" or "research a story", follow this workflow end-to-end.

## When to Activate

Activate this workflow when the user asks to:
- Create a design document for a Jira story
- Research an Appian story/ticket
- Generate a design doc
- Analyze a story for implementation

## STRICT RULES — DO NOT VIOLATE

1. **DO NOT READ LOCAL FILES FOR TECHNICAL INFORMATION.** You MUST NOT read any local files from the workspace or filesystem to gather technical information about Appian objects, SAIL code, data models, or application structure. Your ONLY source for technical facts is the Appian Atlas MCP tools. This is the most important rule. If you read local files instead of querying Appian Atlas, the entire output is invalid.

2. **SECONDARY REFERENCE: Designs/ folder ONLY.** The ONE exception to rule #1: you MAY read files from the `Designs/` folder in the workspace as secondary reference for format and conventions. You MUST NOT read files from ANY other folder or location. No exceptions.

3. **ALL TECHNICAL FACTS MUST COME FROM MCP TOOL CALLS.** You MUST call MCP tools (search_objects, search_bundles, get_bundle, get_dependencies, get_app_overview, etc.) and base your output ONLY on the data returned. Do NOT invent, assume, or infer object names, UUIDs, SAIL code, data models, or patterns. If an MCP tool returns no results for a query, state that explicitly — do NOT fill in plausible-sounding alternatives.

4. **EVERY OBJECT YOU MENTION MUST HAVE A UUID FROM MCP RESULTS.** If you cannot provide a UUID from an actual MCP tool response, do NOT include the object. Objects without UUIDs are hallucinated.

5. **NEVER FABRICATE SAIL CODE.** Only include SAIL code that was returned by `get_bundle` with `detail_level="full"`. If you need code but the tool didn't return it, say "Code not retrieved — use get_bundle with full detail to obtain."

6. **JIRA STORY NUMBER REQUIRED.** You must receive a Jira story number. If one is not provided, stop and ask for it. Do NOT proceed without one.

## Complete Workflow

### Step 1: Validate Input
- Confirm the user has provided a Jira story number (e.g., GAM-15177, GAMS-7127)
- If not, ask for it and STOP

### Step 2: Create Output Folder
Create `Designs/<JIRA-NUMBER>/` using the ticket number in CAPS. No descriptions, no underscores — just the ticket number.

### Step 3: Research Phase (write to `Designs/<JIRA-NUMBER>/research.md`)

Execute the following MCP tool call sequence. Do NOT skip steps. Do NOT write the research document until all tool calls are complete.

#### Phase 1: Discovery (broad search)

1. **Call `get_app_overview`** for the target application to understand its structure.

2. **Call `search_objects`** for EACH keyword extracted from the story. Try at least 3-5 keyword variations. For example, if the story mentions "evaluation summary", search for:
   - "evaluation summary"
   - "EvaluationSummary"
   - "evalSummary"
   - "summary"
   - "evaluation"
   Also search for specific object types when the story implies them (e.g., `object_type="Interface"` for UI stories).

3. **Call `search_bundles`** for the same keywords to find related bundles.

#### Phase 2: Deep dive (targeted investigation)

4. **For each relevant object found**, call `get_dependencies` to trace inbound and outbound dependencies.

5. **For the most critical bundles** (directly related to the story), call `get_bundle` with `detail_level="full"` to get SAIL code. Limit to 2-3 bundles max.

6. **For supporting bundles**, call `get_bundle` with `detail_level="summary"`.

#### Phase 3: Gap analysis

7. **Review what you found vs. what the story requires.** If the story mentions objects or patterns you couldn't find via MCP, explicitly note them as "NOT FOUND in Appian Atlas" — do NOT invent them.

8. **Search for related patterns.** If the story requires a new pattern (e.g., "add a tab"), search for existing similar implementations.

#### Research Document Format

Write `Designs/<JIRA-NUMBER>/research.md` with these sections:

**Story Summary** — Brief restatement of the ticket.

**MCP Query Log** — Log of MCP tool calls and results:
```
- search_objects("SourceSelection", "evaluation summary") → 5 results
- search_objects("SourceSelection", "vendor analysis") → 0 results
- get_bundle("SourceSelection", "AS_GSS_...", "full") → retrieved with SAIL code
```

**Relevant Objects (from MCP)** — Table with columns:
| Object Name | Type | UUID | Source (MCP tool call) | Role/Purpose |

EVERY row must have a real UUID. No exceptions.

**Bundle Analysis** — Impacted bundles with IDs and relevance.

**Dependency Map** — Key dependency chains from `get_dependencies` results.

**SAIL Code Snippets (from get_bundle)** — Only code returned by MCP. Cite bundle ID and object name.

**Data Layer** — CDTs, record types found via MCP.

**Existing Patterns** — Similar implementations found via MCP with bundle IDs.

**Not Found / Gaps** — What the story requires but was NOT found. Critical for the design phase.

**Key Findings** — Observations, risks, recommendations.

### Step 4: Validate Research Quality

After writing research.md, verify:
- Does it contain a "MCP Query Log" section?
- Do objects have UUIDs?
- Is there a "Not Found / Gaps" section?

If the research lacks MCP-sourced data, re-run searches with different keywords before proceeding.

### Step 5: Design Phase (write to `Designs/<JIRA-NUMBER>/design.md`)

Read the research document, then produce the design document using ONLY objects confirmed in the research (with UUIDs) and clearly marking new objects.

#### Design Document Format

You MUST follow this EXACT format:

```markdown
# [<TICKET-ID> | GSS: <Title>](<jira-link>) {#<ticket-id-lowercase>-|-gss:-<title-lowercase-hyphenated>}

| People Designer: <name> Design Reviewer: Developer: Peer Reviewer: | Helpful Links [Principles & Rules](https://docs.google.com/document/d/1DQ4TmCCQ9wybii1eGhXk1ZiLx0Ldj8w8x1wCNVG7lLM/edit#heading=h.xhblfpgm25om) Example Expression Rule Design Doc Common patterns: [Async process](https://docs.google.com/document/d/12ccIJMZvi8p9EgsVkbI_jG1GFRAMLPc_NNgAl7L9dBU/edit#) UI pattern/map configuration [Example Interface Design Doc](#<ticket-id-lowercase>-|-gss:-<title-lowercase-hyphenated>) Example Decision Design Doc |
| :---- | :---- |

### Before Development

| Detailed Design Notes <all object instructions here as a continuous block> *Note: SDX configuration needs to be updated based on the UI changes. Make sure the validations and conditions in the screens are retained and verified. Make sure to replicate any accessibility, font or size changes mentioned in the mockup.* |
| :---- |
|  |

| Mock Ups <links to mockups or "To be provided"> |
| :---- |

| Test Cases <test scenarios as continuous text> |
| :---- |

### After Development

| Code Changes & Peer Review  |
| :---- |

### 

| Deployment |
```

#### Writing "Detailed Design Notes"

All object instructions go inside a SINGLE table cell as one continuous block. No sub-tables, no bullet points.

**For NEW objects:**
```
AS_GSS_<ObjectName> (New)
Create this new <type> which <purpose>.
<Implementation details as continuous text>
Add below rule inputs to the interface
<list of rule inputs, one per line>
```

**For EXISTING objects** (must have UUID in research doc — just use the name in design doc):
```
AS_GSS_<ObjectName>
<Specific instructions — what to change and where>
<Reference line numbers from research SAIL code when available>
```

**Field layout diagrams** — include when rearranging UI fields:
```
The layout is attached below for reference.
Factors
Factor Id		Title
Due Date		Factor Chair
Rating Method	Instructions
Description
```

**Closing note** — ALWAYS include at the end:
```
*Note: SDX configuration needs to be updated based on the UI changes. Make sure the validations and conditions in the screens are retained and verified. Make sure to replicate any accessibility, font or size changes mentioned in the mockup.*
```

#### Writing "Test Cases"

Continuous text, one sentence per test case, each starting with "Verify if":
```
Verify if the empty state screen is as per the mockup provided.
Verify if the Labels, Headers and Buttons across all sections are as per the new design.
Verify if existing validations are retained after the UI changes.
```

#### Writing "Mock Ups"

Include links from the story with descriptive labels. If none provided, write "To be provided".

#### "After Development" Sections

Leave empty — filled post-implementation.

### Step 6: Summary

Report to the user:
- Folder created at `Designs/<JIRA-NUMBER>/`
- Both documents generated
- Research quality: how many objects had UUIDs, any gaps noted
- Key findings or concerns

## Anti-Hallucination Rules

- If `search_objects` returns 0 results, say "No objects found for '<keyword>'"
- If `get_dependencies` returns an error, say "Dependencies not available for '<object>'"
- NEVER write "the standard pattern is..." or "GSS typically uses..." without citing a specific MCP result
- NEVER invent constant names, expression rule names, or interface names
- If the story references objects that don't exist yet, mark them as "TO BE CREATED" with no UUID
- When referencing existing objects in the design, ONLY use objects with UUIDs from the research
- If research quality is poor (no UUIDs), query MCP directly before writing the design

## Key Principles

- List objects in logical order (new shared components first, then parents, then children)
- Be specific: field names, parameter names, line numbers from SAIL code
- Reference mockups: "as per the mockup", "as per the new UI design"
- Call out when validations/conditions must be retained
- Keep instructions imperative: "Update the label", "Remove the section", "Add a condition"
- Mention property file updates (branding, i18n) when UI text changes
- Note SDX/automation test impacts for UI changes
