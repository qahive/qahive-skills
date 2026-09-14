# Sync Test Automation Status

Use this workflow when an automated test script has been created or identified and the corresponding StingerFlow test case must be synchronized (`automationStatus: "automated"`).

---

## Input

- **Automation Test Script**: File path or name of the automated test script (e.g. `tests/e2e/auth/login.spec.ts` or `apps/api/test/auth.e2e-spec.ts`).
- **Optional Test Case ID or Reference**: Explicit Test Case ID (ObjectId), Test Case Code (e.g., `AIM-T101`), or scenario annotation in the test script.

---

## Workflow Steps

### Step 1: Identify the Automation Test Script

- Locate and inspect the target automated test script file.
- Parse the test file to extract:
  - Test scenario titles (`test('...', ...)` or `it('...', ...)`) and `describe` blocks.
  - Test tags, annotations, or comments referencing StingerFlow test cases (e.g., `@TC-001`, `@AIM-T101`, `// StingerFlow ID: 66e...`).
  - Executed steps, assertions, test fixtures, and verified business expectations.

### Step 2: Determine the Corresponding StingerFlow Test Case

- **Direct Reference (Highest Precedence)**: Check if the test script explicitly tags or documents the StingerFlow Test Case ID or Code.
- **Search & Semantic Matching**: If no explicit ID is present, search StingerFlow by test scenario name or suite context using `listing_test_cases` with the `search` parameter:
  ```json
  {
    "workspaceId": "6543a1b2c3d4e5f6a7b8c9d0",
    "projectId": "699e681311a4a7d86d802a2c",
    "search": "Valid User Login with Correct Credentials"
  }
  ```

### Step 3: Query the Test Case from StingerFlow

- Fetch the detailed test case definition using `fetching_test_case` (if `testCaseId` is known) or inspect the result from `listing_test_cases`:
  ```json
  {
    "workspaceId": "6543a1b2c3d4e5f6a7b8c9d0",
    "projectId": "699e681311a4a7d86d802a2c",
    "testCaseId": "66e1f2a3b4c5d6e7f8a9b0c1"
  }
  ```
- Inspect current fields: `automationStatus`, `toBeAutomated`, `status`, `testSteps`, and `testSuiteId`.

### Step 4: Validate Script-to-Test-Case Mapping (Ambiguity Check)

- Validate that the test script and the StingerFlow test case map 1-to-1 with high confidence:
  - Verify that the actions and assertions in the script align with the test steps and expected results in StingerFlow.
  - Verify that the test scenario type (e.g., `happy_path`, `negative`, `edge`) and test type (`e2e` or `api`) correspond.
- **Ambiguity Rule**: If multiple test cases match or if the mapping is uncertain, **DO NOT proceed automatically**. Prompt the user or ask for clarification using `ask_question` to confirm the exact Test Case ID.

### Step 5: Update the Test Case Automation Status

- Prepare the update payload setting `automationStatus` to `"automated"` and `toBeAutomated` to `false`:
  ```json
  {
    "workspaceId": "6543a1b2c3d4e5f6a7b8c9d0",
    "projectId": "699e681311a4a7d86d802a2c",
    "testCaseId": "66e1f2a3b4c5d6e7f8a9b0c1",
    "automationStatus": "automated",
    "toBeAutomated": false
  }
  ```
- Call `updating_test_case` via the StingerFlow MCP tool.

### Step 6: Re-query the Test Case

- Fetch the test case again via `fetching_test_case` using the `testCaseId`:
  ```json
  {
    "workspaceId": "6543a1b2c3d4e5f6a7b8c9d0",
    "projectId": "699e681311a4a7d86d802a2c",
    "testCaseId": "66e1f2a3b4c5d6e7f8a9b0c1"
  }
  ```

### Step 7: Verify Final State

- Assert that `automationStatus` is confirmed as `"automated"` and `toBeAutomated === false` in the returned response.
- Report the synchronization confirmation (Test Case Code, Title, Script Path, and Verified Status) back to the user.

---

## Safety Rules

- **Do Not Set `automationStatus="automated"` When Mapping is Ambiguous**: Never guess or force a match if multiple test cases share similar names or if coverage is only partial.
- **Do Not Infer Automation Status from Filename Alone**: Never mark a test case automated based solely on script filename without inspecting the test content, scenario coverage, and assertions.
- **Prefer Explicit Identifiers**: Prefer explicit Test Case IDs (`testCaseId`), Codes (`AIM-Txxx`), or annotations over heuristic name matching.
- **Always Verify Post-Update**: Always re-query and verify the final state in StingerFlow after updating to guarantee the remote repository accurately reflects the automation status.
