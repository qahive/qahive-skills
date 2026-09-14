# Sync Local Test Cases with Stingerflow

Reads local test case definitions, queries the Stingerflow repository via the `using-stingerflow-mcp` skill, and synchronizes the two. It generates an `Implementation Plan` artifact before executing any changes to guarantee no duplicate test suites or cases are created, and that the remote repository remains fully up-to-date.

---

## Workflow Steps

### Step 1: Read Local Test Cases

- Scan the target local directory or file(s) containing the test case definitions.
- Extract key metadata for each test case: Suite Name, Test Case Title, Description, Steps, Expected Results, and any local ID tags.

### Step 2: Query Stingerflow (Remote State)

- Use the `using-stingerflow-mcp` tools to query existing test suites and test cases in the Stingerflow repository that match the local Suite Names or Test Case Titles.
- Retrieve the Stingerflow IDs, titles, and current versions of these existing remote cases.

### Step 3: Analyze and Map (Diffing)

- Compare the Local Test Cases against the Stingerflow Test Cases.
- **Match Criteria**: Match by existing Stingerflow ID (if tracked locally) OR by exact/fuzzy matching of the Test Case Title within the specific Test Suite.
- **Categorize Actions**:
  - **ADD (Create)**: Local test case does not exist in Stingerflow.
  - **UPDATE (Modify)**: Local test case exists in Stingerflow, but details (steps, description, expected results) differ.
  - **DELETE (Archive/Remove)**: Test case exists in Stingerflow for this suite but is no longer present in the local files. (Note: Verify if soft-delete or archiving is preferred over hard deletion).
  - **SKIP**: Local and remote match exactly; no action needed.

### Step 4: Create the Implementation Plan Artifact

- Generate a structured summary artifact named **`Implementation Plan`** (`implementation_plan.md` with `RequestFeedback: true` and `UserFacing: true`).
- The artifact must include a summary table of the planned operations to prevent duplication and ensure transparency.

### Step 5: Obtain User Approval Before Execution

- **STOP and wait** for the user's explicit confirmation (via clicking "Proceed" on the `Implementation Plan` or confirming in chat) before invoking any mutating tools (`creating_test_case`, `updating_test_case`, `creating_test_suite`, `creating_component`, etc.).
- Do NOT perform any mutating tool calls or update local files before approval is granted.

### Step 6: Execute Mutations & Update Local Files

- Once approved, execute the corresponding Stingerflow MCP tool calls.
- Update the local markdown test case file(s) with the resolved metadata (Workspace ID, Project ID, Test Suite ID, Test Case IDs, Codes, timestamps).
