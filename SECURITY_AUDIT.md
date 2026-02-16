# Security Audit Report: mattermost-mcp

**Audit Date:** 2026-02-16
**Auditor:** Michael Wegener (CTO, satware AG)
**Repository:** https://github.com/jane-alesi/mattermost-mcp.git
**Commit:** `1610cfa25225865350cc2426c6c17389996c45c7`
**Total Commits:** 7 (6 by Pablo Andrés Vélez Vidal, 1 by Jane Alesi)

---

## Executive Summary

**VERDICT: ✅ NO BACKDOORS OR MALICIOUS CODE DETECTED**

The codebase is free of data exfiltration, hidden endpoints, obfuscated payloads, and supply chain attacks. However, **3 security issues** require remediation before production use, including a **HIGH severity** vulnerability in the MCP SDK dependency and **credential leakage via debug logging**.

---

## Findings Summary

| ID | Severity | Category | Description | Status |
|----|----------|----------|-------------|--------|
| SEC-001 | **HIGH** | Dependency | `@modelcontextprotocol/sdk@0.7.0` has 2 known vulnerabilities | ⚠️ Requires Fix |
| SEC-002 | **HIGH** | Credential Leak | `client.ts` logs Authorization header (token) to stderr | ⚠️ Requires Fix |
| SEC-003 | **MEDIUM** | Access Control | HTTP monitoring endpoint has no authentication + CORS `*` | ⚠️ Requires Fix |
| SEC-004 | **LOW** | Information Disclosure | Excessive debug logging exposes internal state | ℹ️ Improve |
| SEC-005 | **INFO** | Configuration | Token stored in plaintext JSON config file | ℹ️ Accepted |

---

## Detailed Findings

### SEC-001: MCP SDK Dependency Vulnerabilities (HIGH)

**Affected:** `@modelcontextprotocol/sdk@0.7.0`
**CVEs:**
- [GHSA-w48q-cv73-mx4w](https://github.com/advisories/GHSA-w48q-cv73-mx4w) — DNS rebinding protection not enabled by default
- [GHSA-8r9q-7v3j-jr4g](https://github.com/advisories/GHSA-8r9q-7v3j-jr4g) — ReDoS vulnerability

**Impact:** An attacker on the local network could exploit DNS rebinding to access the MCP server's HTTP transport. The ReDoS vulnerability could cause denial of service.

**Remediation:** Upgrade to `@modelcontextprotocol/sdk@1.26.0` or later.

```bash
npm audit fix --force
```

### SEC-002: Authorization Token Logged to stderr (HIGH)

**File:** `src/client.ts`, lines 35-36

```typescript
console.error(`Fetching channels from URL: ${url.toString()}`);
console.error(`Using headers: ${JSON.stringify(this.headers)}`);
```

**Impact:** `this.headers` contains `{ Authorization: 'Bearer t6s51ji48ifcfmy789xj4noe8w' }`. Every channel fetch call logs the System Admin token to stderr. Any process capturing stderr (log aggregators, supervisors, terminal scrollback) gains admin access to Mattermost.

**Remediation:** Remove or redact header logging. Replace with:
```typescript
console.error(`Fetching channels from URL: ${url.toString()}`);
// NEVER log headers — contains Authorization token
```

### SEC-003: Unauthenticated HTTP Monitoring Endpoint (MEDIUM)

**File:** `src/index.ts`, HTTP server section

```typescript
res.setHeader('Access-Control-Allow-Origin', '*');
```

**Impact:** The HTTP server on port 3456 accepts requests from any origin with no authentication. Any process on localhost (or any origin via CORS) can:
- Trigger monitoring runs (`/run-monitoring`)
- Query server status (`/status`)

While limited to localhost by default, the CORS wildcard means a malicious website opened in any browser on the same machine can trigger monitoring.

**Remediation:**
1. Remove CORS wildcard — bind to `127.0.0.1` only
2. Add a shared secret or bearer token for HTTP trigger
3. Or remove the HTTP server entirely (use MCP tool trigger only)

### SEC-004: Excessive Debug Logging (LOW)

**Files:** `src/index.ts` (35+ console.error calls), `src/monitor/index.ts` (30+ calls), `src/client.ts` (6 calls)

**Impact:** Debug-level logging in production exposes internal state including user IDs, channel IDs, usernames, and operational details to anyone with access to process stderr.

**Remediation:** Replace `console.error` with a structured logger with configurable log levels. Set production default to `warn`.

### SEC-005: Plaintext Token in Config File (INFO — Accepted)

**File:** `config.local.json` (gitignored)

**Impact:** The Mattermost Personal Access Token is stored in plaintext. This is standard practice for MCP servers and mitigated by `.gitignore` excluding `config.local.json` from version control.

**Recommendation:** Consider supporting environment variable override: `MATTERMOST_TOKEN` env var taking precedence over config file.

---

## Clean Areas (No Issues Found)

| Area | Result |
|------|--------|
| Hardcoded external URLs/IPs | ✅ None — all URLs derived from config |
| eval/Function/base64/obfuscation | ✅ None found in any source file |
| child_process/exec/spawn | ✅ None in project code |
| npm lifecycle hooks (postinstall) | ✅ None — only `build`, `start`, `dev` scripts |
| Hidden files/directories | ✅ Only standard `.gitignore` and `.idea/` |
| Outbound network calls | ✅ All go to configured Mattermost URL only |
| Shell scripts | ✅ Clean — only `localhost` curl and `node build/index.js` |
| Supply chain (npm audit) | ⚠️ 1 high from MCP SDK (see SEC-001) |
| Dependencies | ✅ All well-known packages (node-fetch, node-cron, zod) |
| Build output | ✅ Compiled JS matches TypeScript source |
| Git history | ✅ 7 clean commits, no force-pushes, no suspicious changes |

---

## Dependency Inventory

| Package | Version | Weekly Downloads | Risk |
|---------|---------|-----------------|------|
| `@modelcontextprotocol/sdk` | 0.7.0 | N/A (Anthropic official) | ⚠️ Outdated |
| `node-fetch` | 3.3.2 | 40M+ | ✅ Low |
| `node-cron` | 3.0.3 | 2M+ | ✅ Low |
| `zod` | 3.24.2 | (transitive) | ✅ Low |
| `typescript` | 5.8.2 | (dev) | ✅ Low |

---

## Recommendations (Priority Order)

1. **IMMEDIATE:** Fix SEC-002 — Remove token logging from `client.ts`
2. **IMMEDIATE:** Fix SEC-001 — Upgrade `@modelcontextprotocol/sdk` to ≥1.26.0
3. **SHORT-TERM:** Fix SEC-003 — Add authentication to HTTP endpoint or remove it
4. **SHORT-TERM:** Fix SEC-004 — Replace console.error with structured logger
5. **MEDIUM-TERM:** Add environment variable support for token (SEC-005)

---

## Methodology

- Full manual code review of all 13 TypeScript source files
- Pattern scanning: `eval`, `base64`, `exec`, `spawn`, `child_process`, `Buffer.from`, `atob`, `btoa`, `Function(`, `process.env`
- `npm audit` for known vulnerabilities
- `npm ls --all` for full dependency tree inspection
- `git log` for commit history and author verification
- Hidden file discovery via `find`
- Build output verification (compiled JS matches source)
- Shell script audit for hidden commands

**References:**
- [OWASP Top 10 for Agentic Applications (2026)](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [medium.com](https://medium.com/@emergentcap/hardening-claude-code-a-security-review-framework-and-the-prompt-that-does-it-for-you-c546831f2cec) — Hardening Claude Code security framework
- [thehackernews.com](https://thehackernews.com/2025/09/first-malicious-mcp-server-found.html) — Malicious MCP server postmark-mcp incident
- [dev.to](https://dev.to/snyk/malicious-mcp-server-on-npm-postmark-mcp-harvests-emails-3go6) — Snyk analysis of postmark-mcp supply chain attack