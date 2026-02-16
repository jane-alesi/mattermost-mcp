# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Security
- **SEC-002 (HIGH):** `client.ts` logs Authorization header (Bearer token) to stderr on every channel fetch — requires immediate remediation
- **SEC-001 (HIGH):** `@modelcontextprotocol/sdk@0.7.0` has 2 known vulnerabilities (DNS rebinding GHSA-w48q-cv73-mx4w, ReDoS GHSA-8r9q-7v3j-jr4g) — upgrade to ≥1.26.0
- **SEC-003 (MEDIUM):** HTTP monitoring endpoint has no authentication and CORS wildcard `*` — exploitable from any browser on the same machine

### Added
- `SECURITY_AUDIT.md` — Full security audit report with methodology and findings
- `CHANGELOG.md` — Project changelog
- `UPGRADE_PLAN.md` — Modernization and security remediation plan

## [1.0.0] - 2025-03-10 to 2026-02-15

### Added
- Mattermost MCP server with stdio transport
- Channel listing and history tools (`mattermost_list_channels`, `mattermost_get_channel_history`)
- Message posting tools (`mattermost_post_message`, `mattermost_reply_to_thread`)
- Reaction tool (`mattermost_add_reaction`)
- Thread replies tool (`mattermost_get_thread_replies`)
- User listing and profile tools (`mattermost_get_users`, `mattermost_get_user_profile`)
- Topic monitoring system with keyword-based analysis
- Cron-based monitoring scheduler (`node-cron`)
- HTTP trigger endpoint for monitoring (`/run-monitoring`)
- CLI interface for manual monitoring trigger
- `config.local.json` support for sensitive configuration (gitignored)
- Utility scripts: `analyze-channel.js`, `get-last-message.js`, `view-channel-messages.js`
- Shell scripts: `run-monitoring.sh`, `run-monitoring-http.sh`, `trigger-monitoring.sh`

### Contributors
- Pablo Andrés Vélez Vidal (original author, 6 commits)
- Jane Alesi (1 commit — monitoring guard fix)