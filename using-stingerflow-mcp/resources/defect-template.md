# Defect Templates for Stingerflow (HTML Format)

> **Important**: Stingerflow rich-text editor stores and renders defect descriptions as **HTML**. Always construct the `description` string using standard HTML tags (`<p>`, `<strong>`, `<ul>`, `<ol>`, `<li>`, `<code>`, `<pre>`, `<a>`). Each section header is formatted as `<p><strong>Header Name</strong></p>`, followed by its list or content, and separated by `<p></p>`.

---

## 1. New Defect Description Template (HTML)

Use the following HTML structure when populating the `description` parameter in `creating_defect`:

### Standard HTML Template

```html
<p><strong>Environment</strong></p>
<ul>
  <li>Environment: [e.g. Staging / UAT / Production]</li>
  <li>Browser: [e.g. Chrome 128.0.0.0 / Safari 18.0]</li>
  <li>OS: [e.g. macOS Sequoia / Windows 11]</li>
</ul>
<p></p>
<p><strong>Preconditions</strong></p>
<ul>
  <li>[User roles, prerequisite state, e.g. User account exists with valid password containing special character <code>!</code>]</li>
</ul>
<p></p>
<p><strong>Test Data</strong></p>
<ul>
  <li>Username: <code>[e.g. testuser@example.com]</code></li>
  <li>Password: <code>[e.g. User@123!]</code></li>
  <li>URL: [e.g. https://staging.example.com/login]</li>
</ul>
<p></p>
<p><strong>Steps to Reproduce</strong></p>
<ol>
  <li>Navigate to the Login page</li>
  <li>Enter registered email/username <code>testuser@example.com</code></li>
  <li>Enter valid password containing special character <code>!</code> (e.g., <code>User@123!</code>)</li>
  <li>Click the "Login" button</li>
</ol>
<p></p>
<p><strong>Actual Result</strong></p>
<ul>
  <li>[Observed incorrect behavior or error message, e.g. Authentication fails with invalid credential error or client/server validation failure.]</li>
</ul>
<p></p>
<p><strong>Expected Result</strong></p>
<ul>
  <li>[Expected behavior per requirements, e.g. User successfully logs in and is redirected to the dashboard.]</li>
</ul>
<p></p>
<p><strong>Attachment & Evidence</strong></p>
<p><a href="[Evidence URL, e.g. Jam / Loom / Screenshot URL]" rel="noopener noreferrer" target="_blank">[Evidence URL]</a></p>
<p></p>
```

### Compact Escaped JSON String Format

```json
"description": "<p><strong>Environment</strong></p><ul><li>Environment: Staging</li><li>Browser: Chrome</li><li>OS: Mac</li></ul><p></p><p><strong>Preconditions</strong></p><ul><li>User account exists with valid password containing special character <code>!</code> (e.g., <code>User@123!</code>)</li></ul><p></p><p><strong>Test Data</strong></p><ul><li>Username: <code>testuser@example.com</code></li><li>Password: <code>User@123!</code></li></ul><p></p><p><strong>Steps to Reproduce</strong></p><ol><li>Navigate to the Login page</li><li>Enter registered email/username <code>testuser@example.com</code></li><li>Enter valid password containing special character <code>!</code> (e.g., <code>User@123!</code>)</li><li>Click the \"Login\" button</li></ol><p></p><p><strong>Actual Result</strong></p><ul><li>Authentication fails with invalid credential error or client/server validation failure.</li></ul><p></p><p><strong>Expected Result</strong></p><ul><li>User successfully logs in and is redirected to the dashboard.</li></ul><p></p><p><strong>Attachment & Evidence</strong></p><p><a href=\"https://jam.dev/c/7db6effc-6662-4266-b254-5bb8b84b7a44\" rel=\"noopener noreferrer\" target=\"_blank\">https://jam.dev/c/7db6effc-6662-4266-b254-5bb8b84b7a44</a></p><p></p>"
```

---

## 2. Defect Revision / Update Note Template (HTML)

When updating an existing defect via `updating_defect`, preserve the existing HTML description and append a dated revision section:

### Revision HTML Snippet

```html
<p></p>
<p><strong>Revision Notes [YYYY-MM-DD]</strong></p>
<ul>
  <li><strong>Author / Role</strong>: [e.g. QA / Agent / Developer]</li>
  <li><strong>Status Change</strong>: [e.g. open -> in_progress / in_progress -> resolved]</li>
  <li><strong>Root Cause / Investigation</strong>: [Summary of root cause findings or reproduction details]</li>
  <li><strong>Fix Summary</strong>: [Details of fix, PR link, or commit hash]</li>
  <li><strong>Verification Instructions</strong>: [Steps for QA to verify the resolution]</li>
</ul>
<p></p>
```

### Example Payload for Updating Defect

```json
{
  "workspaceId": "699adbd09ebb3d49fa58cf2f",
  "projectId": "699adbe39ebb3d49fa58cf3d",
  "defectId": "6a969bba24130308828a9b20",
  "status": "resolved",
  "description": "<Original HTML Description Content><p></p><p><strong>Revision Notes [2026-09-01]</strong></p><ul><li><strong>Status Change</strong>: in_progress -> resolved</li><li><strong>Fix Summary</strong>: Fixed special character escaping in auth controller password validator (PR #204).</li><li><strong>Verification Instructions</strong>: Verify login succeeds with passwords containing special characters like `!`, `@`, `#` in Staging.</li></ul><p></p>"
}
```
