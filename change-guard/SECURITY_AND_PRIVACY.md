---
layout: default
title: Security, Privacy & Data Residency — Change Guard for Jira Cloud
permalink: /change-guard/privacy
---

# Security, Privacy & Data Residency Policy

**Effective Date:** November 24, 2026  
**Vendor / Legal Entity:** The Peripety Company, LLC (State of Maine, USA)  
**Product:** Change Guard for Jira Cloud  
**Support Portal:** [The Peripety Company Support Portal](https://theperipetycompany.atlassian.net/servicedesk/customer/portals)  
**Support & Security Email:** [support@theperipetycompany.com](mailto:support@theperipetycompany.com)

---

## 1. Runs on Atlassian (100% Pure Forge · Zero External Egress)
Change Guard for Jira Cloud is engineered as a **100% Pure Forge** application (**Runs on Atlassian** compliant) operating exclusively on Atlassian's managed cloud infrastructure:
- **Runtime:** Atlassian Forge `nodejs22.x`
- **User Interface:** Native Forge UI Kit (`@forge/react`)
- **Database:** Tenant-isolated Forge SQL (`@forge/sql`, managed MySQL/TiDB)
- **Background Processing:** Forge Async Events (`@forge/events`)
- **Zero External Servers & Zero External Data Egress:** `manifest.yml` declares **zero** `external.fetch` permissions and **zero** Atlassian Connect modules. The Peripety Company, LLC operates no external backend servers for Change Guard, and customer audit evidence never leaves Atlassian Cloud.

---

## 2. Strictly Read-Only Permission Model
Change Guard v1 is **strictly read-only**. It requests **zero** `write:*`, `delete:*`, or `manage:*` OAuth 2.0 scopes and enforces read-only HTTP methods at runtime (`GET` plus allow-listed read-only `POST` exclusively for `/rest/api/3/search/jql`).

| OAuth 2.0 Scope | Purpose |
| :--- | :--- |
| `storage:app` | Forge SQL schema migrations and tenant-isolated audit storage |
| `read:jira-work` | Read projects, issues, JQL search (`/rest/api/3/search/jql`), paginated changelogs, remote links, issue links, attachments/comments metadata, and `/rest/api/3/mypermissions` |
| `read:jira-user` | Read Jira account IDs for Segregation of Duties (Approver ≠ Assignee/Deployer) verification (never logged) |
| `read:permission:jira` | Verify Jira global `ADMINISTER` permission before granting access |
| `read:field:jira` | Read custom field definitions for audit profile configuration |
| `read:status:jira` | Read workflow status definitions for audit profile configuration |
| `read:servicedesk-request` | Read-only JSM request approvals (`/rest/servicedeskapi/request/{key}/approval`); gracefully degrades on Jira Software-only sites |

---

## 3. Access Control & Cryptographic Audit Integrity
- **Strict Admin-Only Access:** Every interactive Forge resolver is unconditionally wrapped in `defineAdmin`, verifying Jira's global `ADMINISTER` permission via `GET /rest/api/3/mypermissions?permissions=ADMINISTER` in `asUser()` context and enforcing Paid via Atlassian licensing in production.
- **Database-Backed Worker Authorization:** Asynchronous audit queue workers (`audit-worker`) verify every job against an authoritative `PENDING` or `RUNNING` record in the tenant's Forge SQL `audit_runs` table and read ticket keys strictly from persisted `audit_population` rows — never trusting event payload flags or key lists.
- **Tamper-Evident Evidence Binders:** Population manifests and Evidence Binders use deterministic **RFC 8785 (JCS)** canonical JSON sorted by UTF-8 bytes and hashed with **SHA-256**, enabling auditors to independently verify zero cherry-picking or post-export alteration.
- **System Trigger Isolation:** Lifecycle (`avi:forge:installed:app`, `avi:forge:upgraded:app`) and daily maintenance triggers are bound solely to Forge system triggers and never exposed as user-callable resolvers or web triggers.

---

## 4. Data Minimization, Zero PII in Logs & Retention
- **What is Stored (Forge SQL only, inside your Atlassian tenant):** Audit profile configurations, locked audit run metadata (JQL query, audit period, initiator account ID, SHA-256 digests, sample seed), locked population ticket identifiers, 6-control evaluation outcomes and rule codes, and per-ticket changelog/JSM approval metadata. Atlassian `accountId` values stored for Segregation of Duties and audit-trail attribution are opaque platform identifiers retained strictly inside the customer's tenant-isolated Forge SQL database for up to 365 days and are never exported to vendor servers or written to application logs.
- **What is Never Stored or Collected:** Raw issue descriptions, summaries, comment text, attachment binary contents, user emails, display names, or authentication tokens. Free-text changelog fields (`description`, `summary`, `comment`, `environment`) are redacted to length markers (`[TEXT_REDACTED (len:N)]`) upon ingestion; comment keyword checks execute strictly in-memory.
- **Zero PII in Logs:** Application logs emit structured JSON through a strict key-allowlist logger that redacts non-allowlisted keys, emails, tokens, account IDs, and issue keys, and templates all Jira/JSM paths to `{id}`. Unexpected runtime/SQL errors are sanitized to fixed error codes at handler boundaries.
- **SQL Injection, XSS & CSV Formula Hardening:** 100% of Forge SQL queries use parameterized `sql.prepare(...).bindParams(...)` bindings. Exported CSV files neutralize OWASP formula triggers (`=`, `+`, `-`, `@`, `\t`, `\r`, NFKC full-width triggers, and Unicode format characters `\p{Cf}`). Printable HTML reports strictly escape all interpolated strings and load zero external assets.
- **Retention & Deletion:** Audit runs and associated population, evaluation, and changelog records older than **365 days** (supporting annual SOC 2 / ISO 27001 / DORA audit cycles) are automatically purged during daily maintenance. Uninstalling Change Guard permanently removes the tenant's Forge SQL database per Atlassian Forge lifecycle policy.

---

## 5. Data Residency & Contact
- **Data Residency:** Automatically inherits Atlassian Cloud's tenant-pinned data residency region through Forge SQL and Forge Functions.
- **Privacy, Security & InfoSec Inquiries:** Submit a ticket via [The Peripety Company Support Portal](https://theperipetycompany.atlassian.net/servicedesk/customer/portals) or email [support@theperipetycompany.com](mailto:support@theperipetycompany.com). We respond to all inquiries within 1 business day.
