---
layout: default
title: Security, Privacy & Data Residency — Limit Guard for Jira Cloud
permalink: /privacy
---

# Security, Privacy & Data Residency Policy

**Effective Date:** November 23, 2026  
**Vendor / Legal Entity:** The Peripety Company, LLC (State of Maine, USA)  
**Product:** Limit Guard for Jira Cloud  
**Support Portal:** [The Peripety Company Support Portal](https://theperipetycompany.atlassian.net/servicedesk/customer/portal/1)  
**Support & Security Email:** [support@theperipetycompany.com](mailto:support@theperipetycompany.com)

---

## 1. Runs on Atlassian (100% Pure Forge · Zero External Egress)
Limit Guard for Jira Cloud is engineered as a **100% Pure Forge** application (**Runs on Atlassian** eligible) operating exclusively on Atlassian's managed cloud infrastructure:
- **Runtime:** Atlassian Forge `nodejs22.x`
- **User Interface:** Native Forge UI Kit (`@forge/react`)
- **Database:** Tenant-isolated Forge SQL (`@forge/sql`, managed MySQL/TiDB)
- **Background Processing:** Forge Async Events (`@forge/events`)
- **Zero External Servers & Zero External Data Egress:** `manifest.yml` declares **zero** `external.fetch` permissions and **zero** Atlassian Connect modules. The Peripety Company, LLC operates no external backend servers for Limit Guard, and customer data never leaves Atlassian Cloud.

---

## 2. Strictly Read-Only Permission Model (v1)
Limit Guard v1 is **strictly read-only**. It requests **zero** `write:*`, `delete:*`, or `manage:*` OAuth 2.0 scopes and enforces read-only HTTP methods (`GET` plus allow-listed read-only `POST` exclusively for `/rest/api/3/search/jql` and `/rest/api/3/workflowscheme/read`).

| OAuth 2.0 Scope | Purpose |
| :--- | :--- |
| `storage:app` | Forge SQL schema migrations and tenant-isolated storage operations |
| `read:jira-work` | Read projects (spaces), issue search (`/rest/api/3/search/jql`), saved filters, and `/rest/api/3/mypermissions` |
| `read:field:jira`, `read:field-configuration:jira`, `read:field-configuration-scheme:jira` | Measure Fields per Space (Limit 1) and map custom fields |
| `read:issue-type:jira`, `read:issue-type-scheme:jira` | Measure Work Types per Space (Limit 2) |
| `read:field.option:jira`, `read:custom-field-contextual-configuration:jira` | Measure Field Options per Field (Limit 3) |
| `read:issue-security-scheme:jira`, `read:issue-security-level:jira` | Measure Work Item Security Levels per Space (Limit 4) |
| `read:permission:jira`, `read:permission-scheme:jira` | Measure Permission Grants per Individual Permission (Limit 5) |
| `read:project-version:jira` | Measure Active Releases/Versions per Space (Limit 6) |
| `read:workflow:jira`, `read:workflow-scheme:jira`, `read:status:jira` | Measure Workflows per Scheme (Limit 7), Statuses per Workflow (Limit 8), and workflow dependencies |
| `read:project.component:jira` | Measure Components per Space (Limit 9) and component dependencies |
| `read:priority:jira`, `read:priority-scheme:jira` | Measure Priorities per Space (Limit 10) |
| `read:screen:jira`, `read:screen-tab:jira`, `read:screen-field:jira` | Screen blast-radius dependency mapping (gracefully degrades if restricted) |
| `read:board-scope:jira-software`, `read:board-scope.admin:jira-software` | Agile board blast-radius dependency mapping (gracefully degrades if restricted/unlicensed) |

---

## 3. Access Control & `ADMINISTER` Permission Enforcement
- **Strict Admin-Only Access:** Every interactive Forge resolver (`getHealth`, `triggerScan`, `getDashboardData`, `exportCleanupPack`) is unconditionally protected by `defineAdmin`, which verifies Jira's global `ADMINISTER` permission via `GET /rest/api/3/mypermissions?permissions=ADMINISTER` in `asUser()` context before executing any logic.
- **Database-Backed Worker Authorization:** Asynchronous scan workers (`scan-worker`) verify every job against a valid `PENDING` or `RUNNING` record in the tenant's Forge SQL `scan_runs` table rather than trusting event payload flags.
- **System Trigger Isolation:** Lifecycle (`avi:forge:installed:app`, `avi:forge:upgraded:app`) and nightly scheduled triggers are bound solely to Forge system triggers and never exposed as user-callable resolvers or web triggers.

---

## 4. Data Minimization, Zero PII in Logs & 90-Day Retention
- **What is Stored (Forge SQL only, inside your Atlassian tenant):** Aggregated numeric limit counts, utilization percentages, severity labels, Jira configuration metadata (custom field, status, component, workflow, filter, screen, and board IDs/names), and bloat watchlist issue keys with comment counts and estimated payload sizes.
- **What is Never Stored or Collected:** Issue summaries, descriptions, comment text, attachment contents, user emails, display names, Atlassian `accountId`s, or authentication tokens. Limit Guard does **not** store personal data.
- **Zero PII in Logs:** Application logs record strictly anonymized event names, scan UUIDs, HTTP status codes, durations, and route-templated paths (numeric IDs and keys are redacted to `{id}`).
- **SQL Injection & Export Hardening:** 100% of Forge SQL queries use parameterized `sql.prepare(...).bindParams(...)` bindings. Exported CSV Cleanup Packs neutralize spreadsheet formula-injection prefixes (`=`, `+`, `-`, `@`, `\t`, `\r`) per OWASP guidance and run entirely in-tenant with zero external network calls.
- **Retention & Deletion:** Scan snapshots older than **90 days** are automatically purged during nightly scheduled maintenance. Uninstalling Limit Guard permanently removes the tenant's Forge SQL database per Atlassian Forge lifecycle policy.

---

## 5. Data Residency & Contact
- **Data Residency:** Automatically inherits Atlassian Cloud's tenant-pinned data residency region through Forge SQL and Forge Functions.
- **Privacy, Security & InfoSec Inquiries:** Submit a ticket via [The Peripety Company Support Portal](https://theperipetycompany.atlassian.net/servicedesk/customer/portal/1) or email [support@theperipetycompany.com](mailto:support@theperipetycompany.com). We respond to all inquiries within 1 business day.
