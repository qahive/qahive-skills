# Generate Access Links for Stingerflow

This reference defines the URL structure and generation rules for direct access links to Stingerflow entities. Use these URL patterns when generating artifacts, summary reports, implementation plans, notifications, or chat responses.

---

## Base Domain

```
https://www.stingerflow.com
```

---

## URL Patterns & Entities

| Entity           | Pattern                                                                              | Identifier Used                                                     | Path Suffix / Notes                       | Example Link                                                                                                 |
| :--------------- | :----------------------------------------------------------------------------------- | :------------------------------------------------------------------ | :---------------------------------------- | :----------------------------------------------------------------------------------------------------------- |
| **Main Project** | `https://www.stingerflow.com/projects/{project code}`                                | `{project code}` (e.g. `decom01`)                                   | Plural `/projects/`                       | [decom01](https://www.stingerflow.com/projects/decom01)                                                      |
| **Test Case**    | `https://www.stingerflow.com/project/{project code}/test-cases/{testcase code}/edit` | `{project code}`, `{testcase code}` (e.g. `DECOM01-T1001`)          | Singular `/project/`, requires `/edit`    | [DECOM01-T1001](https://www.stingerflow.com/project/decom01/test-cases/DECOM01-T1001/edit)                   |
| **Defect**       | `https://www.stingerflow.com/project/{project code}/defects/{defect Id}`             | `{project code}`, `{defect Id}` (e.g. `6ac6055d892dd26c17ea9499`)   | Singular `/project/`, defect ObjectId     | [Defect Details](https://www.stingerflow.com/project/decom01/defects/6ac6055d892dd26c17ea9499)               |
| **Test Run**     | `https://www.stingerflow.com/project/{project code}/test-runs/{test run Id}/execute` | `{project code}`, `{test run Id}` (e.g. `6ac5d19d20cb51f698205d06`) | Singular `/project/`, requires `/execute` | [Test Run Execution](https://www.stingerflow.com/project/decom01/test-runs/6ac5d19d20cb51f698205d06/execute) |

---

## Detailed Specifications

### 1. Main Project URL

Navigates to the overview/dashboard of the specified project.

- **Route Format**: `https://www.stingerflow.com/projects/{project code}`
- **Path Rule**: Uses the plural `projects`.
- **Example**:
  ```
  https://www.stingerflow.com/projects/decom01
  ```

### 2. Test Case Link (Edit / Detail View)

Test case details must be accessed via the `/edit` route.

- **Route Format**: `https://www.stingerflow.com/project/{project code}/test-cases/{testcase code}/edit`
- **Path Rule**:
  - Uses singular `project`.
  - Uses `{testcase code}` (e.g. `DECOM01-T1001`, uppercase project prefix + `-T` + number).
  - Requires the `/edit` suffix at the end.
- **Example**:
  ```
  https://www.stingerflow.com/project/decom01/test-cases/DECOM01-T1001/edit
  ```

### 3. Defect Link (View Detail)

Navigates directly to the defect detail view.

- **Route Format**: `https://www.stingerflow.com/project/{project code}/defects/{defect Id}`
- **Path Rule**:
  - Uses singular `project`.
  - Uses `{defect Id}` (the 24-character hexadecimal MongoDB ObjectId string, e.g. `6ac6055d892dd26c17ea9499`).
- **Example**:
  ```
  https://www.stingerflow.com/project/decom01/defects/6ac6055d892dd26c17ea9499
  ```

### 4. Test Run Link (Execution / Detail View)

Navigates to the test run execution view.

- **Route Format**: `https://www.stingerflow.com/project/{project code}/test-runs/{test run Id}/execute`
- **Path Rule**:
  - Uses singular `project`.
  - Uses `{test run Id}` (the 24-character hexadecimal MongoDB ObjectId string, e.g. `6ac5d19d20cb51f698205d06`).
  - Requires the `/execute` suffix at the end.
- **Example**:
  ```
  https://www.stingerflow.com/project/decom01/test-runs/6ac5d19d20cb51f698205d06/execute
  ```

---

## Important Rules & Gotchas

1. **Plural vs. Singular `project`**:
   - The main project overview uses **plural** `/projects/{project code}`.
   - All sub-resources (test cases, defects, test runs) use **singular** `/project/{project code}/...`.
2. **Access Suffixes**:
   - Test cases require `/edit` to view/edit details.
   - Test runs require `/execute` to view/execute runs.
3. **Identifier Types**:
   - Test Cases use the human-readable **Test Case Code** (e.g. `DECOM01-T1001`).
   - Defects and Test Runs use the internal **MongoDB ObjectId** string (e.g. `6ac6055d...`).
