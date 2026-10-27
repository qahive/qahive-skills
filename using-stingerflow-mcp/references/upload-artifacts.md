# Upload Artifacts & Evidence via Signed URLs

This reference details the procedure for generating pre-signed Google Cloud Storage upload URLs and uploading artifacts (test logs, defect screenshots, videos, and test reports) via the `generating_upload_url` MCP tool.

---

## 1. Overview

The `generating_upload_url` MCP tool allows AI agents to securely request one-time, time-limited Google Cloud Storage pre-signed PUT URLs. This pattern ensures that large diagnostic files (such as console logs, Playwright traces, and screenshots) are uploaded directly to cloud storage without overloading the application server or streaming heavy binary payloads across JSON-RPC connections.

---

## 2. Generating the Signed Upload URL

### Tool Invocation: `generating_upload_url`

- **Parameters**:
  - `filename` (string, required): Original filename with extension (e.g., `error-screenshot.png`, `playwright-report.zip`, `execution-log.txt`).
- **Context Requirement**:
  - Does **not** require `workspaceId` or `projectId`.
  - Requires standard authenticated MCP session (JWT bearer token).

### Example Payload:

```json
{
  "filename": "defect-login-failure.png"
}
```

### Response Schema:

The tool returns an MCP text content envelope containing the `UploadFileURLDto` JSON:

```json
{
  "uploadUrl": "https://storage.googleapis.com/tmt-bucket-stg/tmp/defect-login-failure.png?X-Goog-Algorithm=GOOG4-RSA-SHA256&X-Goog-Credential=...&X-Goog-Signature=...",
  "objectPath": "tmp/defect-login-failure.png",
  "fullPath": "https://storage.googleapis.com/tmt-bucket-stg/tmp/defect-login-failure.png",
  "expiresIn": 120
}
```

| Field | Description |
| :--- | :--- |
| `uploadUrl` | The V4 pre-signed PUT URL used to directly upload the file. |
| `objectPath` | Unique destination path within the storage bucket. |
| `fullPath` | The public URL of the uploaded asset for linking/embedding. |
| `expiresIn` | URL validity duration in seconds (default: 120 seconds). |

---

## 3. Direct HTTP PUT Upload

After receiving the `uploadUrl`, the AI agent or script must upload the file's binary content directly using an HTTP `PUT` request before the URL expires (default: 120 seconds).

> [!IMPORTANT]
> **Matching Content-Type is Mandatory**: The `Content-Type` header sent with the HTTP PUT request **must** match the MIME type signed by the backend. If omitted or mismatched, Google Cloud Storage will reject the request with `403 SignatureDoesNotMatch`.

---

### macOS / Linux (bash / zsh)

#### One-Liner Command:

```bash
curl -X PUT \
  -H "Content-Type: image/png" \
  --data-binary "@/path/to/defect-login-failure.png" \
  "<uploadUrl>"
```

#### Reusable Shell Script (`upload-to-gcs.sh`):

```bash
#!/usr/bin/env bash
# Usage: ./upload-to-gcs.sh <file_path> <upload_url> [content_type]
set -euo pipefail

FILE_PATH="${1:?Error: file_path is required}"
UPLOAD_URL="${2:?Error: upload_url is required}"
CONTENT_TYPE="${3:-}"

if [ ! -f "$FILE_PATH" ]; then
  echo "Error: File '$FILE_PATH' does not exist." >&2
  exit 1
fi

# Detect Content-Type by extension if not explicitly provided
if [ -z "$CONTENT_TYPE" ]; then
  case "${FILE_PATH##*.}" in
    png)  CONTENT_TYPE="image/png" ;;
    jpg|jpeg) CONTENT_TYPE="image/jpeg" ;;
    gif)  CONTENT_TYPE="image/gif" ;;
    webp) CONTENT_TYPE="image/webp" ;;
    svg)  CONTENT_TYPE="image/svg+xml" ;;
    pdf)  CONTENT_TYPE="application/pdf" ;;
    txt)  CONTENT_TYPE="text/plain" ;;
    html) CONTENT_TYPE="text/html" ;;
    json) CONTENT_TYPE="application/json" ;;
    zip)  CONTENT_TYPE="application/zip" ;;
    mp4)  CONTENT_TYPE="video/mp4" ;;
    mov)  CONTENT_TYPE="video/quicktime" ;;
    *)    CONTENT_TYPE="application/octet-stream" ;;
  esac
fi

echo "Uploading '$FILE_PATH' ($CONTENT_TYPE) to Google Cloud Storage..."
HTTP_STATUS=$(curl -s -w "%{http_code}" -o /dev/null -X PUT \
  -H "Content-Type: $CONTENT_TYPE" \
  --data-binary "@$FILE_PATH" \
  "$UPLOAD_URL")

if [ "$HTTP_STATUS" -eq 200 ]; then
  echo " Upload successful (HTTP $HTTP_STATUS)"
else
  echo " Upload failed with HTTP status: $HTTP_STATUS" >&2
  exit 1
fi
```

---

### Windows (PowerShell)

#### One-Liner Command:

```powershell
Invoke-RestMethod -Method Put -Uri "<uploadUrl>" -InFile "C:\path\to\defect-login-failure.png" -ContentType "image/png"
```

#### Reusable PowerShell Script (`Upload-ToGcs.ps1`):

```powershell
<#
.SYNOPSIS
  Uploads a local file to Google Cloud Storage using a pre-signed V4 PUT URL.
.EXAMPLE
  .\Upload-ToGcs.ps1 -FilePath ".\defect-login-failure.png" -UploadUrl "https://storage.googleapis.com/..."
#>
[CmdletBinding()]
param(
    [Parameter(Mandatory = $true, Position = 0)]
    [ValidateScript({ Test-Path $_ -PathType Leaf })]
    [string]$FilePath,

    [Parameter(Mandatory = $true, Position = 1)]
    [string]$UploadUrl,

    [Parameter(Mandatory = $false, Position = 2)]
    [string]$ContentType
)

$resolvedPath = Resolve-Path $FilePath

# Detect Content-Type by extension if not explicitly provided
if (-not $ContentType) {
    $ext = [System.IO.Path]::GetExtension($resolvedPath).ToLower()
    $mimeMap = @{
        '.png'  = 'image/png'
        '.jpg'  = 'image/jpeg'
        '.jpeg' = 'image/jpeg'
        '.gif'  = 'image/gif'
        '.webp' = 'image/webp'
        '.svg'  = 'image/svg+xml'
        '.pdf'  = 'application/pdf'
        '.txt'  = 'text/plain'
        '.html' = 'text/html'
        '.json' = 'application/json'
        '.zip'  = 'application/zip'
        '.mp4'  = 'video/mp4'
        '.mov'  = 'video/quicktime'
    }
    $ContentType = if ($mimeMap.ContainsKey($ext)) { $mimeMap[$ext] } else { 'application/octet-stream' }
}

Write-Host "Uploading '$resolvedPath' ($ContentType) to Google Cloud Storage..." -ForegroundColor Cyan

try {
    $response = Invoke-WebRequest -Method Put -Uri $UploadUrl -InFile $resolvedPath -ContentType $ContentType -UseBasicParsing
    if ($response.StatusCode -eq 200) {
        Write-Host " Upload successful (HTTP $($response.StatusCode))" -ForegroundColor Green
    } else {
        Write-Warning " Upload completed with status code: $($response.StatusCode)"
    }
} catch {
    Write-Error " Upload failed: $_"
    throw
}
```

---

### Supported MIME Types Reference Table:

The backend automatically resolves `Content-Type` based on the filename extension:
- **Images**: `.png` (`image/png`), `.jpg`/`.jpeg` (`image/jpeg`), `.gif` (`image/gif`), `.webp` (`image/webp`), `.svg` (`image/svg+xml`)
- **Documents & Logs**: `.txt` (`text/plain`), `.pdf` (`application/pdf`), `.html` (`text/html`), `.json` (`application/json`)
- **Archives**: `.zip` (`application/zip`)
- **Videos**: `.mp4` (`video/mp4`), `.mov` (`video/quicktime`)
- **Fallback**: `application/octet-stream` for other extensions

---

## 4. Embedding Uploaded Evidence into Defects

Once the file is uploaded, use the returned `fullPath` in the defect description HTML under the **Attachment & Evidence** section:

### For Screenshots / Images:

Embed inline using `<img src="..." />`. Do **not** add a duplicate `<a href="...">` anchor link for images.

```html
<p><strong>Attachment & Evidence</strong></p>
<p><img src="https://storage.googleapis.com/tmt-bucket-stg/tmp/defect-login-failure.png" alt="defect-login-failure.png" /></p>
```

### For Non-Image Attachments (Logs, Test Reports, Archives):

Use `<a href="..." target="_blank">` text links only for non-image files (e.g., `.txt`, `.zip`, `.pdf`, or external video/screen recording links like Jam/Loom):

```html
<p><strong>Attachment & Evidence</strong></p>
<p><a href="https://storage.googleapis.com/tmt-bucket-stg/tmp/execution-log.txt" rel="noopener noreferrer" target="_blank">execution-log.txt</a></p>
```

---

## 5. End-to-End Workflow Summary

```
1. Core Details Clarified -> 2. Prompt for Attachments (ask_question) -> 3. Collect Local File/URL -> 4. Call generating_upload_url -> 5. HTTP PUT to uploadUrl -> 6. Embed into HTML (img/a) -> 7. Implementation Plan Artifact -> 8. Execute create/update defect
```
