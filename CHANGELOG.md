# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.3.0] - 2026-10-27

### Added
- **`using-stingerflow-mcp`**: Added signed upload URL generation and artifact attachment workflows.
  - Integration with `generating_upload_url` MCP tool for uploading evidence assets directly to Google Cloud Storage (GCS).
  - Shell (`upload-to-gcs.sh`) and PowerShell (`Upload-ToGcs.ps1`) upload scripts and curl commands for direct HTTP PUT uploads.
  - Defect HTML template and guidelines for embedding uploaded images (`<img>`) and non-image evidence attachments (`<a>`).

## [0.2.0] - 2026-10-27

### Added
- **`using-stingerflow-mcp`**: Added direct access link generation reference and updated skill instructions.
  - Direct URL pattern resolution for Stingerflow resources (Projects, Test Suites, Test Cases, Defect reports, Test Runs).
  - Integration guidelines for report summaries and Slack/chat notifications.

## [0.1.0] - 2026-09-14

### Added
- **`using-stingerflow-mcp`**: Initial release for Stingerflow MCP integration.
  - Core MCP workflow, context resolution, and mutation safety rules.
  - Defect management guide, status lifecycle, and rich HTML templates.
  - Test case and test automation status synchronization workflows.
