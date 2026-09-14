# Defect Management Workflow

This guide details the end-to-end workflows, standards, and rules for creating and revising defects in Stingerflow via MCP tools.

---

## 1. Defect Severity Matrix

Use the following criteria when assigning or updating the `severity` field:

| Severity   | Definition                                                                                               | Examples                                                                                                                 |
| :--------- | :------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------- |
| `critical` | Complete feature/system breakdown, crash, data corruption, or security vulnerability with no workaround. | App crashes on launch; payment transaction double-charges; database records corrupted.                                   |
| `high`     | Major feature broken or severely impaired with no acceptable workaround; core workflow blocked.          | User unable to submit defect form; test execution results fail to save; login fails for specific auth provider.          |
| `medium`   | Non-critical functionality defect, edge case, or functional issue with an available workaround.          | Filter dropdown fails to reset on clear; pagination shows duplicate items on page 3; export CSV missing optional column. |
| `low`      | Minor cosmetic, styling, typo, or slight UI alignment issue that does not impact functional flow.        | Misaligned button margin; typo in tooltip; minor color contrast imperfection.                                            |

---

## 2. Defect Status Lifecycle

Defects progress through the following lifecycle states (`status`):

- **`open`**: Initial state when defect is reported and logged.
- **`in_progress`**: Defect is acknowledged and actively being investigated or resolved.
- **`resolved`**: Fix has been implemented or deployed, ready for verification/re-testing.
- **`closed`**: Verification completed successfully; defect is verified fixed.
- **`reopen`**: Verification failed during testing, or the defect regressed after fix.
- **`invalid`**: Works as designed, duplicate defect, or not reproducible.

---

## 3. Creating a New Defect (`creating_defect`)

Follow this step-by-step workflow when creating a new defect:

### Step 1: Resolve Context, Components, and Tags

- **Context**: Resolve `workspaceId` and `projectId` from context (see [SKILL.md](../SKILL.md)).
- **Components Resolution**:
  - Call `listing_components` to inspect available components in the project:
    ```json
    {
      "workspaceId": "...",
      "projectId": "..."
    }
    ```
  - **Strict Semantic Matching**:
    - **Exact / Genuine Match**: If an existing component is genuinely and directly related to the defect's domain (e.g., `Authentication` for login/session issues), auto-select its ObjectId in `components: ["..."]`.
    - **Weak / Loosely Related or Missing Match**: If existing components do **not** directly match the defect's functional area (e.g., only `Enrollment` and `Authentication` exist, but the defect is about `Payment` or `Booking`), **DO NOT force-fit** a weak match. Instead:
      1. Proactively suggest creating a new, accurately named component (e.g., `Payment` or `Booking`) via `creating_component`.
      2. Include an **FYI Note** explaining what existing components were found and why they were not selected (e.g., _"Found existing components `[Authentication, Enrollment]`, but neither directly covers payment transactions. Suggesting to create `Payment`."_).
      3. Propose creating the new component in the `Implementation Plan`:
         | Field | Value |
         | :--- | :--- |
         | **Component Name** | `Payment` |
         | **Description** | `Payment transactions and gateway integration` |
         _Require user confirmation via the Implementation Plan before calling `creating_component`._
- **Tags** (Optional): Only include tags if explicitly specified or provided by the user (e.g., user specifies `["staging", "v1.2.0"]`). Never auto-generate, guess, or infer tags from domain keywords, environment, or feature names. If not explicitly provided by the user, **always default to an empty array `[]`**.

### Step 2: Formulate an Actionable Title

- Structure title as `Specific Failure or Symptom`.
- Avoid vague titles (e.g., prefer `Bluestone Admin level Operation login เข้า Admin Web Portal ได้เฉพาะ company SYS` over `Login broken`).
- **Review & Title Confirmation via `ask_question`**:
  - Review the defect title provided by the user. If an improved, more specific, or standardized title can be formulated, invoke `ask_question` to let the user confirm or select their preferred title.
  - **Mandatory Original Option**: The options list in `ask_question` **MUST always include 1 option containing the user's exact original defect title** (e.g., Option 1: `(Recommended) [Refined title]`, Option 2: `[Original title submitted by user]`).

### Step 3: Determine Severity and Status

- Set `severity` according to the Severity Matrix (`critical`, `high`, `medium`, or `low`). Defaults to `medium` if unspecified.
- Set `status` to `open` (default).

### Step 4: Gather Missing Details & Construct Structured Description in HTML

- **Mandatory Information Gathering via `ask_question`**:
  Before constructing the HTML description and drafting the Implementation Plan, check if the user provided the necessary defect details. If any of the following 4 core items are missing or not provided in the request:
  1. **Test Data** (e.g. account credentials, order IDs, product names/SKUs, transaction numbers)
  2. **Steps to Reproduce** (exact reproduction steps)
  3. **Actual Result** (observed error, unexpected UI behavior, error logs)
  4. **Expected Result** (expected behavior per requirements)

  👉 **You MUST invoke the `ask_question` tool** to ask the user for the missing details before preparing the Implementation Plan.
  👉 **CRITICAL**: Each missing item must be asked as a **separate, distinct question** in the `ask_question` call (e.g., Question 1 for Test Data, Question 2 for Steps to Reproduce, Question 3 for Actual Result, Question 4 for Expected Result). **Never bundle Steps to Reproduce, Expected Result, and Actual Result together into a single question.**

- Use the standard QA HTML format defined in [resources/defect-template.md](../resources/defect-template.md):
  - **Environment**: Environment, Browser, OS
  - **Preconditions**: User roles, prerequisites
  - **Test Data**: URLs, test credentials, test entity IDs
  - **Steps to Reproduce**: `<ol><li>...</li></ol>`
  - **Actual Result**: Observed incorrect behavior
  - **Expected Result**: Expected behavior per requirements
  - **Attachment & Evidence**: Error logs, screenshots, API payloads

### Step 5: Create Implementation Plan and Request User Confirmation

- **MANDATORY**: Before calling `creating_defect`, create an **`Implementation Plan`** artifact (`implementation_plan.md` with `RequestFeedback: true` and `UserFacing: true`) containing the structured summary table and details:

| Field                   | Value                                                                                                 |
| :---------------------- | :---------------------------------------------------------------------------------------------------- |
| **Action**              | Create New Defect (`creating_defect`)                                                                 |
| **Workspace ID**        | `699adbd09ebb3d49fa58cf2f`                                                                            |
| **Project ID**          | `699adbe39ebb3d49fa58cf3d`                                                                            |
| **Title**               | `User unable to login with valid password containing special character '!'`                           |
| **Severity**            | `high`                                                                                                |
| **Status**              | `open`                                                                                                |
| **Components**          | `Authentication` (`6a96a4c665cee131cc31c3a7`)                                                         |
| **Tags**                | `["staging", "v1.2.0"]`                                                                               |
| **Assignee ID**         | _(Unassigned or User ID)_                                                                             |
| **Description Preview** | Environment: Staging, Browser: Chrome / Mac<br>Steps: Enter special char password '!' -> Login fails. |

- **Wait for User Confirmation**: Do NOT execute `creating_defect` or any mutating tools until the user explicitly confirms (e.g., clicks the "Proceed" button on the Implementation Plan or gives explicit confirmation in chat).

### Step 6: Execute Tool Call

- After user confirms, call `creating_defect` via Stingerflow MCP:
  ```json
  {
    "workspaceId": "699adbd09ebb3d49fa58cf2f",
    "projectId": "699adbe39ebb3d49fa58cf3d",
    "title": "User unable to login with valid password containing special character '!'",
    "description": "<p>...</p>",
    "severity": "high",
    "status": "open",
    "components": ["6a96a4c665cee131cc31c3a7"],
    "tags": ["staging", "v1.2.0"]
  }
  ```

### Step 7: Output Summary

- Present the newly created defect's Code (e.g. `AIM-D1020`), Title, Severity, and Status back to the user.

---

## 4. Revising an Existing Defect (`updating_defect`)

Follow this workflow when modifying, reassigning, or transitioning defect status:

### Step 1: Locate and Inspect Defect

- If user provides defect code (e.g. `AIM-D1016`) or MongoDB ObjectId:
  - Call `fetching_defect` with `identifier`:
    ```json
    {
      "workspaceId": "699adbd09ebb3d49fa58cf2f",
      "projectId": "699adbe39ebb3d49fa58cf3d",
      "identifier": "AIM-D1016"
    }
    ```
- If user provides keywords/title:
  - Call `listing_defects` with `search`:
    ```json
    {
      "workspaceId": "699adbd09ebb3d49fa58cf2f",
      "projectId": "699adbe39ebb3d49fa58cf3d",
      "search": "special character"
    }
    ```
- Extract the defect's MongoDB `_id` (`defectId`) from the response.

### Step 2: Determine Updates Required

- **Status transitions**: Check valid lifecycle transition (e.g., `open` -> `in_progress` -> `resolved`).
- **Severity adjustments**: Re-evaluate if priority or impact changed.
- **Assignee changes**: Pass `assigneeId` (or `null` to unassign).
- **Tags & Components**: Pass updated array of tag strings (e.g., `["staging", "v1.2.0"]`) or component IDs (auto-match via `listing_components` or suggest creating new via `creating_component`).

### Step 3: Update Description with Revision Notes in HTML

- **Default (Preserve & Append)**: Preserve original HTML description and append a dated revision section:
  ```html
  <p></p>
  <p><strong>Revision Notes [YYYY-MM-DD]</strong></p>
  <ul>
    <li><strong>Author / Role</strong>: QA / Agent</li>
    <li><strong>Status Change</strong>: open -> resolved</li>
    <li>
      <strong>Root Cause / Investigation</strong>: Company scope query lacked wildcard matching for
      Bluestone Admin level.
    </li>
    <li><strong>Fix / Verification</strong>: Fixed in PR #204. Verified on UAT.</li>
  </ul>
  <p></p>
  ```
- **Explicit Rewrite**: Only completely replace the description if the user explicitly instructs a full rewrite.

### Step 4: Create Implementation Plan and Request User Confirmation

- **MANDATORY**: Before calling `updating_defect`, create an **`Implementation Plan`** artifact (`implementation_plan.md` with `RequestFeedback: true` and `UserFacing: true`) containing the structured comparison table of changes and planned revision details:

| Field           | Current Value                            | Proposed Value                                |
| :-------------- | :--------------------------------------- | :-------------------------------------------- |
| **Defect Code** | `AIM-D1016` (`6a969bba24130308828a9b20`) | -                                             |
| **Status**      | `in_progress`                            | `resolved`                                    |
| **Severity**    | `high`                                   | `high` (unchanged)                            |
| **Components**  | `[]`                                     | `Authentication` (`6a96a4c665cee131cc31c3a7`) |
| **Tags**        | `["staging"]`                            | `["staging", "v1.2.0"]`                       |
| **Assignee**    | `user-123`                               | `null` (unassigned)                           |
| **Description** | `<Original HTML>`                        | Appending Revision Note `[2026-09-03]`        |

- **Wait for User Confirmation**: Do NOT execute `updating_defect` or any mutating tools until the user explicitly confirms (e.g., clicks the "Proceed" button on the Implementation Plan or gives explicit confirmation in chat).

### Step 5: Execute Tool Call

- After user confirms, call `updating_defect` via Stingerflow MCP:
  ```json
  {
    "workspaceId": "699adbd09ebb3d49fa58cf2f",
    "projectId": "699adbe39ebb3d49fa58cf3d",
    "defectId": "6a969bba24130308828a9b20",
    "status": "resolved",
    "description": "<Original HTML Description Content><p></p><p><strong>Revision Notes [2026-09-01]</strong></p><ul><li><strong>Status Change</strong>: in_progress -> resolved</li><li><strong>Fix Summary</strong>: Fixed special character escaping in auth controller password validator (PR #204).</li><li><strong>Verification Instructions</strong>: Verify login succeeds with passwords containing special characters like `!`, `@`, `#` in Staging.</li></ul><p></p>"
  }
  ```

### Step 6: Output Summary

- Report the updated status, modified fields, and confirmation to the user.

---

## 5. Reference Templates

Full templates and HTML examples are maintained in [resources/defect-template.md](../resources/defect-template.md).
