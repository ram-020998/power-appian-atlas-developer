# Action: Design Document

This is the complete workflow for generating research and design documents for Appian stories. Follow every step in order. Do NOT skip steps.

## STRICT RULES — DO NOT VIOLATE

1. **DO NOT READ LOCAL FILES FOR TECHNICAL INFORMATION.** You MUST NOT read any local files from the workspace or filesystem to gather technical information about Appian objects, SAIL code, data models, or application structure. Your ONLY source for technical facts is the Appian Atlas MCP tools. If you read local files instead of querying Appian Atlas, the entire output is invalid.

2. **SECONDARY REFERENCE: Designs/ folder ONLY.** The ONE exception to rule #1: you MAY read files from the `Designs/` folder in the workspace as secondary reference for format and conventions. You MUST NOT read files from ANY other folder or location. No exceptions.

3. **ALL TECHNICAL FACTS MUST COME FROM MCP TOOL CALLS.** Base your output ONLY on data returned by MCP tools. Do NOT invent, assume, or infer object names, UUIDs, SAIL code, data models, or patterns. If an MCP tool returns no results, state that explicitly — do NOT fill in plausible-sounding alternatives.

4. **EVERY OBJECT YOU MENTION MUST HAVE A UUID FROM MCP RESULTS.** If you cannot provide a UUID from an actual MCP tool response, do NOT include the object. Objects without UUIDs are hallucinated.

5. **NEVER FABRICATE SAIL CODE.** Only include SAIL code returned by `get_bundle` with `detail_level="full"`.

6. **JIRA STORY NUMBER REQUIRED.** If the user has not provided one, stop and ask. Do NOT proceed without it.

## Step 1: Validate Input

Confirm the user has provided a Jira story number (e.g., GAM-15177, GAMS-7127). If not, ask for it and STOP.

## Step 2: Create Output Folder

Create `Designs/<JIRA-NUMBER>/` using the ticket number in CAPS. No descriptions, no underscores — just the ticket number.

## Step 3: Research Phase

Execute the following MCP tool call sequence. Do NOT skip steps. Do NOT write research.md until all tool calls are complete.

### Phase 1: Discovery

1. **Call `get_app_overview`** for the target application.

2. **Call `search_objects`** for EACH keyword from the story. Try at least 3-5 keyword variations. Example for "evaluation summary":
   - "evaluation summary"
   - "EvaluationSummary"
   - "evalSummary"
   - "summary"
   - "evaluation"
   Also filter by `object_type` when the story implies it (e.g., `"Interface"` for UI stories).

3. **Call `search_bundles`** for the same keywords.

### Phase 2: Deep Dive

4. **For each relevant object**, call `get_dependencies` to trace inbound/outbound.

5. **For 2-3 critical bundles**, call `get_bundle` with `detail_level="full"` to get SAIL code.

6. **For supporting bundles**, call `get_bundle` with `detail_level="summary"`.

### Phase 3: Gap Analysis

7. If the story mentions objects not found via MCP, note them as "NOT FOUND in Appian Atlas" — do NOT invent them.

8. Search for related patterns (e.g., if story says "add a tab", search for existing tab implementations).

### Write research.md

Write `Designs/<JIRA-NUMBER>/research.md` with these sections:

**Story Summary** — Brief restatement of the ticket.

**MCP Query Log** — Every MCP call and its result count:
```
- search_objects("SourceSelection", "evaluation summary") → 5 results
- search_objects("SourceSelection", "vendor analysis") → 0 results
- get_bundle("SourceSelection", "AS_GSS_...", "full") → retrieved with SAIL code
```

**Relevant Objects (from MCP)** — Table:
| Object Name | Type | UUID | Source (MCP tool call) | Role/Purpose |

EVERY row must have a real UUID. No exceptions.

**Bundle Analysis** — Impacted bundles with IDs.

**Dependency Map** — Key chains from `get_dependencies`.

**SAIL Code Snippets** — Only code from `get_bundle` with `detail_level="full"`. Cite bundle ID and object name.

**Data Layer** — CDTs, record types found via MCP.

**Existing Patterns** — Similar implementations found via MCP.

**Not Found / Gaps** — What the story requires but was NOT found. Critical for design phase.

**Key Findings** — Observations, risks, recommendations.

## Step 4: Validate Research

After writing research.md, verify:
- Does it have an "MCP Query Log" section?
- Do objects have UUIDs?
- Is there a "Not Found / Gaps" section?

If research lacks MCP-sourced data, re-run searches with different keywords before proceeding.

## Step 5: Design Phase

Read research.md, then write `Designs/<JIRA-NUMBER>/design.md` using ONLY objects confirmed in the research (with UUIDs). Clearly mark new objects.

### Design Document Format — EXACT TEMPLATE

```markdown
# [<TICKET-ID> | GSS: <Title>](<jira-link>) {#<ticket-id-lowercase>-|-gss:-<title-lowercase-hyphenated>}

| People Designer: <name> Design Reviewer: Developer: Peer Reviewer: | Helpful Links [Principles & Rules](https://docs.google.com/document/d/1DQ4TmCCQ9wybii1eGhXk1ZiLx0Ldj8w8x1wCNVG7lLM/edit#heading=h.xhblfpgm25om) Example Expression Rule Design Doc Common patterns: [Async process](https://docs.google.com/document/d/12ccIJMZvi8p9EgsVkbI_jG1GFRAMLPc_NNgAl7L9dBU/edit#) UI pattern/map configuration [Example Interface Design Doc](#<ticket-id-lowercase>-|-gss:-<title-lowercase-hyphenated>) Example Decision Design Doc |
| :---- | :---- |

### Before Development

| Detailed Design Notes <all object instructions here as continuous block> *Note: SDX configuration needs to be updated based on the UI changes. Make sure the validations and conditions in the screens are retained and verified. Make sure to replicate any accessibility, font or size changes mentioned in the mockup.* |
| :---- |
|  |

| Mock Ups <links or "To be provided"> |
| :---- |

| Test Cases <continuous text, each starting with "Verify if"> |
| :---- |

### After Development

| Code Changes & Peer Review  |
| :---- |

### 

| Deployment |
```

### Detailed Design Notes Format

All instructions in a SINGLE table cell as continuous text.

**NEW objects:**
```
AS_GSS_<ObjectName> (New)
Create this new <type> which <purpose>.
<Implementation details as continuous text>
Add below rule inputs to the interface
<rule inputs, one per line>
```

**EXISTING objects** (must have UUID in research — just use name in design):
```
AS_GSS_<ObjectName>
<Specific instructions — what to change and where>
<Reference line numbers from research SAIL code>
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

**Closing note** — ALWAYS at the end:
```
*Note: SDX configuration needs to be updated based on the UI changes. Make sure the validations and conditions in the screens are retained and verified. Make sure to replicate any accessibility, font or size changes mentioned in the mockup.*
```

### Test Cases Format

Continuous text, each starting with "Verify if":
```
Verify if the empty state screen is as per the mockup provided.
Verify if the Labels, Headers and Buttons are as per the new design.
Verify if existing validations are retained after the UI changes.
```

### Mock Ups

Links with descriptive labels from the story. If none, write "To be provided".

### After Development

Leave empty — filled post-implementation.

## Step 6: Summary

Report to user:
- Folder created at `Designs/<JIRA-NUMBER>/`
- Both documents generated with paths
- Research quality: object count with UUIDs, gaps noted
- Key findings or concerns

## Anti-Hallucination Rules

- If `search_objects` returns 0 results, say "No objects found for '<keyword>'"
- If `get_dependencies` returns an error, say "Dependencies not available for '<object>'"
- NEVER write "the standard pattern is..." without citing a specific MCP result
- NEVER invent constant names, expression rule names, or interface names
- New objects: mark as "TO BE CREATED" with no UUID
- Existing objects in design: ONLY those with UUIDs from research
