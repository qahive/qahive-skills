---
name: using-stingerflow-mcp
description: Guidelines and context resolution for interacting with Stingerflow via MCP tools, including resolving workspace and project contexts, creating and revising defects, and synchronizing test cases and test automation status.
---

# Using Stingerflow MCP

## When to use this skill

Trigger this skill any time a request involves:

- Listing, fetching, searching, creating, updating, exporting, or deleting **test cases**
- Synchronizing test case **automation status** when automation scripts are created or identified
- Listing, fetching, creating, or updating **test suites**
- Listing, fetching, creating, or updating **components**
- Listing, fetching, searching, creating, or updating **defects** (logging bugs, updating defect status, adding investigation notes, reassigning)
- Listing, fetching, creating, or updating **test runs**
- Any Stingerflow MCP tool call

This applies even if the user doesn't say "Stingerflow" or "MCP" by name — e.g. "add a test case for login flow", "show me test suites", "log a defect", or "update defect AIM-D1015 status to resolved" should trigger this skill.

## Core Rules

- Prefer `listing_test_cases`, `listing_defects`, or `listing_test_runs` with `search` param over multiple separate calls when looking for something by name or partial match.
- Never fabricate a `workspaceId` or `projectId` — always resolve via config file or explicit user input first.
- Always inspect/fetch an existing entity first before applying updates or revisions.
- **Defect Descriptions MUST be HTML formatted**: Stingerflow uses a rich-text editor that stores and renders HTML (`<ul>`, `<ol>`, `<li>`, `<strong>`, `<p>`, `<span>`, `<code>`). Never send raw Markdown in the `description` parameter.
- **Strict Component Matching & Proactive Suggestion**: When resolving components, only assign existing components if they genuinely match the defect domain. If existing components are only loosely related or missing, do not force-fit; explain why in an FYI note and propose creating a new component.
- **Do NOT Auto-Generate or Infer Tags**: Tags (`tags`) must default to an empty array `[]` unless explicitly provided or requested by the user. Never invent, hallucinate, or auto-generate tags based on domain keywords (e.g. `booking`, `payment`), environment, or feature names.
- **Defect Title Review & Clarification via `ask_question`**: Review the defect title submitted by the user. If a clearer, more specific, or standardized defect title is suggested, use `ask_question` to let the user choose. **The options MUST always include 1 option with the user's exact original defect title**, alongside any recommended clearer title(s).
- **Gather Missing Defect Details via `ask_question`**: When creating a defect (`creating_defect`), if the user has not provided **Test Data**, **Steps to Reproduce**, **Actual Result**, or **Expected Result**, you MUST use the `ask_question` tool to ask the user for the missing details before preparing the Implementation Plan. **Each missing item (Steps to Reproduce, Expected Result, Actual Result, Test Data) MUST be prompted as its own separate question** (do not combine Steps to Reproduce, Expected Result, and Actual Result into a single question).
- **User Confirmation via Implementation Plan**: Before calling any mutating tools in Stingerflow (including `creating_test_case`, `updating_test_case`, `creating_test_suite`, `updating_test_suite`, `creating_component`, `creating_defect`, `updating_defect`, `creating_test_run`, or `updating_test_run`), or executing test case synchronization, you MUST create an **`Implementation Plan`** artifact (`implementation_plan.md` with `RequestFeedback: true` and `UserFacing: true`) summarizing the planned entity creations/updates, target suite and component mappings, and schema normalizations, and **WAIT for explicit user confirmation** (e.g. clicking "Proceed" or chat approval) before executing any mutations.

## Resolving Default Context

Before calling any tool that requires `workspaceId` or `projectId`:

1. Check [`.mcp/stingerflow-config.json`](/.mcp/stingerflow-config.json) at the project root.
2. If the file exists, use its `workspaceId` and `projectId` as defaults for the tool call.
3. If the user's request explicitly names a different workspace or project, use theirs instead — explicit user input always overrides the config file.
4. If no config file exists and the tool call requires these fields, use the `ask_question` tool for `workspaceId` and `projectId` rather than guessing or omitting the field, and save into [`.mcp/stingerflow-config.json`](/.mcp/stingerflow-config.json).

```json
// .mcp/stingerflow-config.json
{
  "workspaceId": "6543a1b2c3d4e5f6a7b8c9d0",
  "projectId": "699e681311a4a7d86d802a2c"
}
```

---

## Domain Reference Guides

For detailed procedures, schema requirements, and workflows, consult the corresponding domain guide:

- **Defect Management**: [references/defect-management.md](references/defect-management.md)
  - Severity Matrix (`critical`, `high`, `medium`, `low`)
  - Status Lifecycle (`open`, `in_progress`, `resolved`, `closed`, `reopen`, `invalid`)
  - Creating Defect Workflow (`creating_defect`)
  - Revising & Updating Defect Workflow (`updating_defect`)
  - Defect HTML Templates & Payloads: [resources/defect-template.md](resources/defect-template.md)
- **Test Case Synchronization**: [references/sync-test-cases.md](references/sync-test-cases.md)
  - Sync Local Test Cases with Stingerflow
  - Step-by-step diffing, mapping, and categorization (`ADD`, `UPDATE`, `DELETE`, `SKIP`)
  - Implementation Plan artifact generation before execution
- **Sync Test Automation Status**: [references/sync-test-automation-status.md](references/sync-test-automation-status.md)
  - Synchronize test case automation status (`automationStatus: "automated"`) for automated scripts
  - Script-to-test-case validation and ambiguity rules
  - Step-by-step query, update, and post-update verification
