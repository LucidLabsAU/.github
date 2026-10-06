# Lucid Labs Responsible Disclosure Policy

Lucid Labs Pty Ltd (ABN 33 678 306 539) (**Lucid Labs**, **we**) values the
work of the security research community. This policy describes how to
report a vulnerability in any Lucid Labs system, what we will do with the
report, and the safe-harbour commitment we make in return.

This is the org-level policy. It applies to every repository in the
**LucidLabsAU** GitHub organisation and to every system listed in the
Scope section.

A machine-readable summary lives at
`https://lucidlabs.com.au/.well-known/security.txt` (RFC 9116).

## Scope

The following systems are **in scope** for responsible-disclosure
reports:

- The Lucid Labs website and any subdomain of `lucidlabs.com.au`.
- The **Voiceprint** SaaS application and its production hosting.
- Any other Lucid Labs-published SaaS product, listed on
  `https://lucidlabs.com.au` at the time of report.
- Source code in any **public** repository in the
  [LucidLabsAU GitHub organisation](https://github.com/LucidLabsAU).
- The Lucid Labs Microsoft Marketplace and AppSource listings published
  under the LucidLabsAU publisher identity.

The following are **out of scope**:

- **Customer tenants** that Lucid Labs operates as a Microsoft Partner.
  Report findings about a customer tenant directly to that customer or
  to the relevant cloud provider (Microsoft Security Response Center at
  `https://msrc.microsoft.com/report`).
- **Microsoft-hosted infrastructure** the customer or product depends
  on (Azure, Microsoft 365, Entra, Power Platform). Report to MSRC.
- **Cloudflare-hosted infrastructure**. Report via Cloudflare's HackerOne
  program at `https://hackerone.com/cloudflare`.
- **Third-party libraries and frameworks** we depend on. Report
  upstream; we will update once a patched version is released.
- **Private repositories** in the LucidLabsAU organisation.
- **Social engineering, phishing, or physical attacks** against Lucid
  Labs personnel, contractors, premises, or customers.
- **Denial-of-service attacks** of any kind.
- **Spam, content-injection, or volumetric testing** of our endpoints.

## How to report

Email **`security@lucidlabs.com.au`** with:

- a description of the vulnerability;
- the system affected (URL, repository, product version, commit SHA);
- steps to reproduce — short, scripted, and complete enough that we can
  reproduce without additional context;
- the impact you assess and the user(s) or data at risk;
- any suggested remediation you have in mind; and
- your name (or pseudonym) and whether you would like credit in the
  resulting advisory.

If the vulnerability is high-severity and you would prefer encrypted
communication, request a PGP key in your first email and we will provide
one.

**Do not** open public GitHub issues, Discussions, or pull requests for
suspected vulnerabilities. **Do not** post details on social media until
the coordinated disclosure timeline has elapsed (see below).

## What you can expect from us

| Stage | Within |
| ----- | ------ |
| Acknowledgement of your report | 2 business days |
| Initial triage and severity assessment | 5 business days |
| Status update on confirmed vulnerabilities | every 14 days until resolution |
| Coordinated public disclosure | 90 calendar days from acknowledgement, extendable by mutual agreement |

We will:

- assess the report under our internal severity matrix (aligned with the
  Lucid Labs Cyber Incident Response Plan);
- patch the affected system within the SLA defined by our Patching
  Policy (Critical 7 days, High 30 days, Medium 90 days);
- issue a CVE through GitHub Security Advisories where appropriate;
- credit the reporter in the resulting advisory, by name or pseudonym,
  unless asked not to;
- notify affected customers under the Notifiable Data Breaches scheme
  where applicable.

We do **not** currently run a paid bug bounty program. We acknowledge
reports publicly through GitHub Security Advisories and our website
Hall of Fame.

## Safe harbour

Lucid Labs will not pursue civil or criminal action against, or refer
to law enforcement, any researcher who:

(a) discovers a vulnerability through **good-faith** security research
    against an **in-scope** system;

(b) follows this policy, including the disclosure timeline and the
    out-of-scope restrictions;

(c) accesses only the minimum data necessary to demonstrate the
    vulnerability, and does **not** access, modify, exfiltrate, delete,
    or otherwise process customer or personal data beyond that minimum;

(d) avoids degrading the availability of any service for any user;

(e) does **not** attempt to social-engineer, phish, or physically
    access Lucid Labs personnel or premises;

(f) does **not** publicly disclose the vulnerability before the agreed
    disclosure date; and

(g) reports the vulnerability promptly using the channel in this
    policy.

This commitment is consistent with the Australian Cyber Security
Centre's guidance on coordinated vulnerability disclosure and applies
to the maximum extent permitted by Australian law, including the
_Cybercrime Act 2001_ (Cth) and the _Privacy Act 1988_ (Cth). It does
not waive any rights against actions that fall outside (a)–(g).

If you are unsure whether your planned research is in scope, **ask
first** at `security@lucidlabs.com.au`. We are happy to confirm scope
in writing before you start.

## Out-of-scope findings we will not accept

We will respectfully decline reports of the following unless you can
demonstrate a real, exploitable impact:

- Missing security headers without a concrete exploitation path
  (HSTS preload, CSP report-only mode, etc.).
- SPF / DKIM / DMARC misconfigurations on non-mail-sending subdomains.
- Banner / version disclosure (server software, framework versions).
- Click-jacking on pages with no sensitive actions.
- Self-XSS, XSS only exploitable in unsupported browsers, or XSS
  requiring the victim to disable browser security features.
- TLS configuration findings on services we do not directly control
  (Azure Front Door, Azure Container Apps default certificates).
- Rate-limit findings on endpoints intended to be unauthenticated
  (search, sitemap, marketing forms).
- Reports generated by automated scanning tools without manual
  verification.
- Findings against systems we have already publicly disclosed (check
  our published advisories first).

## Coordinated disclosure timeline

The default disclosure timeline is **90 calendar days** from the date
we acknowledge your report. During that window:

- Days 0–14: we triage, confirm, and assess severity.
- Days 14–60: we develop and test a patch.
- Days 60–90: we coordinate release timing with you, prepare the
  advisory, and notify affected customers under the NDB scheme where
  required.
- Day 90: we publish a GitHub Security Advisory and you may publish
  your write-up.

The timeline can be **extended by mutual agreement** for findings that
require complex remediation, or **shortened** by mutual agreement where
a faster public advisory protects users. We will not unilaterally
shorten the timeline without informing you first.

If we have not patched by day 90 and have not agreed an extension with
you, you are free to publish. We ask that you do so responsibly,
without weaponised exploit code.

## Hall of fame

We list researchers who have helped us at
`https://lucidlabs.com.au/security#thanks` (once published).

## Contact

- **Reports:** `security@lucidlabs.com.au`
- **Reports (web form):** `https://lucidlabs.com.au/#contact`
- **Machine-readable:** `https://lucidlabs.com.au/.well-known/security.txt`
- **Postal:** Lucid Labs Pty Ltd, contact via email for current address

## Policy versioning

| Version | Date         | Change                                                                                          |
| ------- | ------------ | ----------------------------------------------------------------------------------------------- |
| 1.0     | 6 Oct 2026   | Initial release. Replaces the prior single-page SECURITY.md with a full coordinated-disclosure policy aligned with the Lucid Labs Cyber Incident Response Plan, Patching Policy and Data Retention Policy, and with the published `/.well-known/security.txt`. |

This policy is reviewed annually by the CTO, or earlier following any
material change to the Lucid Labs ISMS, the regulatory environment, or
the in-scope product portfolio.
