## Hi there 👋

Welcome to **Lucid Labs** — a Brisbane-based Microsoft Partner specialising in cloud transformation, security, and enterprise automation.

We build solutions that help organisations unlock the full potential of their data, analytics, and AI capabilities.

🔗 **VS Code Extension:** [Lucid Labs Theme](https://marketplace.visualstudio.com/items?itemName=lucidlabs.lucid-labs-theme)
💼 **Website:** [lucidlabs.com.au](https://lucidlabs.com.au)

---

## Repository Portfolio

Every active repo in this org maps to one of five capability tracks. New repos must justify which track they belong to before being created.

### 1. Tenant Ops & Automation

Microsoft 365, Azure, and Partner Center management — scripts, scheduled runbooks, and the self-hosted CIPP portal.

| Repo | Purpose |
|---|---|
| [lucidazure](https://github.com/LucidLabsAU/lucidazure) | PowerShell automation for M365 + Azure tenant management; includes `automation/runbooks/` for all Azure Automation runbooks |
| [CIPP](https://github.com/LucidLabsAU/CIPP) | Self-hosted CIPP portal fork for multi-tenant M365 management via GDAP |

### 2. Audit & Governance

Evidence collection, specialisation audits, and customer audit deliverables. Follows a template → customer-instance pattern.

| Repo | Purpose |
|---|---|
| [azure-tenant-audit-template](https://github.com/LucidLabsAU/azure-tenant-audit-template) | Reusable PowerShell audit collector template (M365 + Azure cross-tenant) |
| [azure-audit](https://github.com/LucidLabsAU/azure-audit) | Analytics on Azure Specialisation V2.8.1 audit evidence system |
| [progenesis-azure-audit](https://github.com/LucidLabsAU/progenesis-azure-audit) | Customer instance — Progenesis tenant audit |

### 3. Data Platform

Fabric/Power BI tooling, customer data platforms, semantic glossary, and data integration.

| Repo | Purpose |
|---|---|
| [fabric-tools](https://github.com/LucidLabsAU/fabric-tools) | Fabric/Power BI CLI + MCP tools and audit playbooks |
| [enterprise-data-platform](https://github.com/LucidLabsAU/enterprise-data-platform) | Reusable Fabric medallion template (Bronze → Silver → Gold) |
| [PDP-DEV](https://github.com/LucidLabsAU/PDP-DEV) | Customer instance — Perfection Fresh Australia data platform |
| [lucid-data-intelligence](https://github.com/LucidLabsAU/lucid-data-intelligence) | Semantic glossary engine |
| [mesh](https://github.com/LucidLabsAU/mesh) | Multi-tenant NFP data integration platform (Employment Hero ↔ Sage Intacct) |

### 4. Sales / Partner / Internal Ops

Dynamics 365, Pax8, partner motion, time tracking, and internal productivity MCP tooling.

| Repo | Purpose |
|---|---|
| [lucid-hub](https://github.com/LucidLabsAU/lucid-hub) | D365 ↔ Xero integration middleware (quotes, invoices, payroll, webhooks) |
| [productivity-mcp](https://github.com/LucidLabsAU/productivity-mcp) | MCP server for Microsoft 365 + Graph API + D365 + branded PowerPoint decks |
| [pax8inter](https://github.com/LucidLabsAU/pax8inter) | Pax8 → Dynamics 365 subscription/invoice sync |
| [salesCRM](https://github.com/LucidLabsAU/salesCRM) | Dynamics 365 (CRM 6 AU) environment configuration |
| [dynatime](https://github.com/LucidLabsAU/dynatime) | Multi-platform timesheet mobile app (iOS / watchOS / macOS) |
| [bidManagement](https://github.com/LucidLabsAU/bidManagement) | Bid response MCP server (document extraction, knowledge base, response drafting) |

### 5. Brand & Content

Marketing, training content, design system, and linked-in distribution.

| Repo | Purpose |
|---|---|
| [website](https://github.com/LucidLabsAU/website) | Corporate marketing site (React + Vite) |
| [linkedin-content](https://github.com/LucidLabsAU/linkedin-content) | Dual-channel LinkedIn posting engine (Lucid Labs + personal) |
| [exam-study-guides](https://github.com/LucidLabsAU/exam-study-guides) | Interactive Microsoft certification study guides |
| [lucid-labs-vscode-theme](https://github.com/LucidLabsAU/lucid-labs-vscode-theme) | Theme factory — branded VS Code extensions from templates |

---

## Template → Customer Instance Pattern

Two repos are reusable templates; customer work forks from them:

- **Audit deliverables:** `azure-tenant-audit-template` → customer instances (`progenesis-azure-audit`, …)
- **Data platform builds:** `enterprise-data-platform` → customer instances (`PDP-DEV`, …)

Resist the temptation to fork-and-modify outside these patterns.

---

_This map replaces the pre-consolidation layout. Repos retired during consolidation: `lucidazure-runbooks` (absorbed into `lucidazure`), `partnerBuilder` (reference docs salvaged to `.github/docs/partner-program/`), `lucy` (paused), `pfa-training-pitch` (pptx logic moved to `productivity-mcp`), `powerbi-report-toolkit` (stale clone of `fabric-tools`)._
