# Upgrade Plan: mattermost-mcp Modernization

**Created:** 2026-02-16
**Author:** Michael Wegener (CTO, satware AG)
**Branch:** `feat/modernize-2026` (to be created)

---

## Overview

Modernize the mattermost-mcp project from its current prototype state to production-grade quality, addressing all security findings and bringing the codebase up to 2026 standards.

---

## Phase 0: Security Remediation (IMMEDIATE — before any other work)

**Priority:** CRITICAL — these must be fixed before continued use with admin token.

### 0.1 Fix Token Leakage (SEC-002)
- [ ] Remove `console.error` lines 35-36 in `src/client.ts` that log headers containing the Bearer token
- [ ] Audit all other `console.error` calls for sensitive data exposure
- [ ] Verify no other code path logs the token

### 0.2 Upgrade MCP SDK (SEC-001)
- [ ] Upgrade `@modelcontextprotocol/sdk` from `0.7.0` to `≥1.26.0`
- [ ] Handle breaking changes (v0.7 → v1.x is a major version jump)
- [ ] Update imports and API usage as needed
- [ ] Test all MCP tools still work after upgrade

### 0.3 Secure HTTP Endpoint (SEC-003)
- [ ] Remove CORS wildcard `Access-Control-Allow-Origin: *`
- [ ] Bind HTTP server to `127.0.0.1` only
- [ ] Add bearer token authentication to `/run-monitoring` endpoint
- [ ] Or: remove HTTP server entirely (use MCP tool `mattermost_run_monitoring` instead)

### 0.4 Fix Debug Logging (SEC-004)
- [ ] Replace all `console.error` debug logging with structured logger
- [ ] Add configurable log levels (error, warn, info, debug)
- [ ] Default to `warn` in production

---

## Phase 1: Package Manager Migration

### 1.1 Migrate npm → pnpm
- [ ] Delete `package-lock.json`
- [ ] Delete `node_modules/`
- [ ] Run `pnpm install`
- [ ] Verify `pnpm-lock.yaml` generated
- [ ] Update shell scripts to use `pnpm` commands
- [ ] Add `packageManager` field to `package.json`

---

## Phase 2: TypeScript Strict Mode

### 2.1 Enable Strict TypeScript
- [ ] Set `"strict": true` in `tsconfig.json`
- [ ] Enable `"noUncheckedIndexedAccess": true`
- [ ] Enable `"useUnknownInCatchVariables": true`
- [ ] Enable `"noImplicitOverride": true`
- [ ] Fix all resulting type errors
- [ ] Replace `any` types with proper types

### 2.2 Modernize TypeScript Config
- [ ] Target `ES2022` or later (Node 24 LTS)
- [ ] Use `"module": "NodeNext"` with ESM
- [ ] Or keep CommonJS if MCP SDK requires it
- [ ] Add path aliases if helpful

---

## Phase 3: Dependency Modernization

### 3.1 Remove node-fetch
- [ ] Replace `node-fetch` with native `fetch()` (available since Node 18)
- [ ] Remove `@types/node-fetch`
- [ ] Update all import statements
- [ ] Test all API calls still work

### 3.2 Update All Dependencies
- [ ] Update `node-cron` to latest
- [ ] Update `typescript` to latest 5.x
- [ ] Update `@types/node` to match Node 24 LTS
- [ ] Run `pnpm audit` — verify 0 vulnerabilities

---

## Phase 4: Code Quality Tooling

### 4.1 ESLint 9 Flat Config
- [ ] Install `eslint`, `@eslint/js`, `typescript-eslint`
- [ ] Create `eslint.config.js` with strict TypeScript rules
- [ ] Run `eslint --fix .` for auto-fixable issues
- [ ] Fix remaining lint errors manually
- [ ] Add `lint` script to `package.json`

### 4.2 Vitest 4 Testing Framework
- [ ] Install `vitest` and `@vitest/coverage-v8`
- [ ] Create `vitest.config.ts` with coverage thresholds
- [ ] Add `test` and `test:coverage` scripts to `package.json`
- [ ] Target ≥80% code coverage

### 4.3 Gitleaks
- [ ] Add `.gitleaks.toml` configuration
- [ ] Run `gitleaks detect` on repository
- [ ] Create baseline if needed for existing false positives

---

## Phase 5: Test Suite

### 5.1 Unit Tests
- [ ] `src/client.ts` — Mock fetch, test all API methods
- [ ] `src/config.ts` — Test config loading, validation, defaults
- [ ] `src/monitor/analyzer.ts` — Test topic matching logic
- [ ] `src/monitor/scheduler.ts` — Test scheduling behavior
- [ ] `src/tools/*.ts` — Test tool definitions and handlers

### 5.2 Integration Tests
- [ ] MCP server startup and tool registration
- [ ] Tool execution with mocked Mattermost API responses
- [ ] Monitoring workflow end-to-end with mocked data

### 5.3 Coverage Target
- [ ] Achieve ≥80% line coverage
- [ ] Achieve ≥80% branch coverage

---

## Phase 6: Code Improvements

### 6.1 Environment Variable Support
- [ ] Support `MATTERMOST_URL`, `MATTERMOST_TOKEN`, `MATTERMOST_TEAM_ID` env vars
- [ ] Env vars take precedence over config file values
- [ ] Document in README

### 6.2 Input Validation
- [ ] Add Zod schemas for all tool input parameters
- [ ] Validate config file structure at startup
- [ ] Sanitize user inputs before passing to API

### 6.3 Error Handling
- [ ] Create consistent error response format
- [ ] Add proper error types/classes
- [ ] Ensure all async operations have try/catch

### 6.4 Clean Up Utility Scripts
- [ ] Remove or move `analyze-channel.js`, `get-last-message.js`, `view-channel-messages.js` to `scripts/` directory
- [ ] Convert to TypeScript or document as development utilities

---

## Phase 7: Documentation

### 7.1 README Updates
- [ ] Add security section with token handling guidance
- [ ] Add development setup instructions (pnpm)
- [ ] Add testing instructions
- [ ] Add architecture overview
- [ ] Document all MCP tools with examples

### 7.2 Contributing Guide
- [ ] Create `CONTRIBUTING.md` with development workflow
- [ ] Document code style, testing requirements, PR process

---

## Definition of Done

Each phase is complete when:
1. All checklist items are checked
2. `pnpm test` passes with ≥80% coverage
3. `pnpm lint` passes with 0 errors
4. `gitleaks detect` passes
5. `pnpm build` succeeds
6. All MCP tools tested and working against Mattermost

---

## Estimated Effort

| Phase | Effort | Priority |
|-------|--------|----------|
| Phase 0: Security | 2-3 hours | **CRITICAL** |
| Phase 1: pnpm | 30 min | HIGH |
| Phase 2: TS Strict | 1-2 hours | HIGH |
| Phase 3: Dependencies | 1 hour | HIGH |
| Phase 4: Tooling | 1-2 hours | MEDIUM |
| Phase 5: Tests | 3-4 hours | MEDIUM |
| Phase 6: Code Quality | 2-3 hours | MEDIUM |
| Phase 7: Docs | 1-2 hours | LOW |
| **Total** | **~12-18 hours** | |