# SOC Lab #02 — Phishing Email Investigation
---

## Overview

This lab involved triaging an inbound phishing alert triggered by a suspicious email flagged by the SIEM. The investigation required analyzing email artifacts to identify indicators of compromise (IOCs), confirm malicious intent, and determine the appropriate response action.

---

## Alert Details

| Field | Value |
|---|---|
| Event ID | 8815 |
| Alert Rule | Inbound Email Containing Suspicious External Link |
| Incident Type | Phishing |
| Severity | Medium |
| Date Detected | February 1st, 2026 at 14:59 |
| Data Source | Email |

---

## Investigation

### Step 1 — Alert Triage

Upon receiving the alert, I reviewed the case report for Event ID 8815. The alert rule fired on an inbound email flagged for containing one or more external links with potentially suspicious characteristics. The SIEM recommended checking firewall and proxy logs to determine if any endpoints had attempted to access the embedded URLs.

![Alert case report showing Event ID 8815 flagged as phishing with medium severity](screenshots/alert_case_report.png)
*Figure 1: Case report for Event ID 8815 — Inbound Email Containing Suspicious External Link*

---

### Step 2 — Email Artifact Analysis

I reviewed the full email metadata and content to identify IOCs:

| Artifact | Value | Finding |
|---|---|---|
| Sender | urgents@amazon.biz | Suspicious — legitimate Amazon uses @amazon.com |
| Recipient | h.harris@thetrydaily.thm | Internal employee targeted |
| Subject | Your Amazon Package Couldn't Be Delivered – Action Required | Classic urgency lure |
| Attachment | None | No malicious attachment |
| Direction | Inbound | External sender |
| Embedded URL | http://bit.ly/3sHkX3da12340 | URL-shortened link — masks true destination |

**Key red flags identified:**

- **Sender domain spoofing** — The email claims to be from Amazon Delivery but originates from `urgents@amazon.biz`, not the legitimate `@amazon.com` domain. This is a classic impersonation technique.
- **Urgency and social engineering** — The subject line creates a false sense of urgency ("Action Required") to pressure the recipient into clicking without thinking.
- **URL shortening** — The bit.ly link obscures the true destination URL, a common tactic used to bypass email filters and hide malicious domains from recipients.
- **Credential harvesting setup** — The email body instructs the recipient to click the link to confirm shipping information, consistent with a credential phishing page.

---

### Step 3 — URL Reputation Analysis

I extracted the embedded URL from the email and submitted it to TryDetectThis, a URL/IP security analysis tool available in the Analyst VM, to determine its reputation and confirm malicious status.

**URL submitted:** `http://bit.ly/3sHkX3da12340`

**Result: MALICIOUS**

![TryDetectThis URL analysis confirming the embedded bit.ly link as MALICIOUS](screenshots/url_analysis_malicious.png)
*Figure 2: TryDetectThis URL/IP Security Check confirming MALICIOUS status for the embedded link*

The tool confirmed the URL as malicious, corroborating the phishing indicators identified during email artifact analysis.

---

## IOC Summary

| IOC Type | Value | Confidence |
|---|---|---|
| Malicious URL | http://bit.ly/3sHkX3da12340 | High |
| Spoofed Sender Domain | urgents@amazon.biz | High |
| Phishing Subject Line | Your Amazon Package Couldn't Be Delivered – Action Required | High |

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Observed Behavior |
|---|---|---|
| T1566.002 | Phishing: Spearphishing Link | Inbound email containing malicious URL targeting internal employee |
| T1036.005 | Masquerading: Match Legitimate Name | Sender spoofing Amazon domain to appear legitimate |
| T1204.001 | User Execution: Malicious Link | Recipient prompted to click URL to confirm shipping info |

---

## Verdict & Response

**Classification:** True Positive  
**Escalation Required:** No

Based on the email artifact analysis and URL reputation confirmation, this alert was classified as a **true positive phishing attempt**. The email impersonated Amazon Delivery using a lookalike sender domain, embedded a URL-shortened malicious link, and used urgency-based social engineering to prompt the recipient to click.

**Recommended defensive actions:**

- Block sender domain `amazon.biz` at the email gateway
- Quarantine the email from recipient's inbox
- Notify the targeted user `h.harris@thetrydaily.thm` with phishing awareness guidance
- Add the bit.ly URL to the blocklist

---

## Key Takeaways

- URL shorteners like bit.ly are a common evasion technique — always expand and analyze shortened links before trusting them
- Sender domain verification is one of the fastest ways to identify spoofed emails — `@amazon.biz` vs `@amazon.com` is a clear mismatch
- Medium severity alerts still require full triage — this was a confirmed phish that could have led to credential theft if the user clicked the link
- Not every true positive requires escalation — thorough documentation and containment actions are sufficient for straightforward phishing cases

---

*Write-up by Trystan Ruiz*
