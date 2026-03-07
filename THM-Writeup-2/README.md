# SOC Lab #02 — Phishing Email Investigation

---

## Overview

Got an alert for an inbound email flagged by the SIEM for containing a suspicious external link. Went through the email artifacts, ran the URL through a reputation check, and closed it as a true positive. No escalation needed.

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

Pulled up Event ID 8815. The rule fired on an inbound email with an external link that tripped the filter. The SIEM also noted to check firewall and proxy logs to see if anyone actually tried to reach the URL.

![Alert case report showing Event ID 8815 flagged as phishing with medium severity](screenshots/alert_case_report.png)
*Figure 1: Case report for Event ID 8815 — Inbound Email Containing Suspicious External Link*

---

### Step 2 — Email Artifact Analysis

Went through the email header and body. A few things stood out right away.

| Artifact | Value | Finding |
|---|---|---|
| Sender | urgents@amazon.biz | Suspicious — legitimate Amazon uses @amazon.com |
| Recipient | h.harris@thetrydaily.thm | Internal employee targeted |
| Subject | Your Amazon Package Couldn't Be Delivered – Action Required | Classic urgency lure |
| Attachment | None | No malicious attachment |
| Direction | Inbound | External sender |
| Embedded URL | http://bit.ly/3sHkX3da12340 | URL-shortened link — masks true destination |

The sender was `urgents@amazon.biz`. Amazon doesn't send from `.biz`, it's `@amazon.com`. The subject line was the usual "Action Required" urgency bait, and the embedded link was a bit.ly short URL which already looks suspicious since it hides where it actually goes. The email was telling the user to click to confirm shipping info — textbook credential harvesting setup.

![Alert case report showing full email details including sender, recipient, subject, and embedded URL](screenshots/alert_case_report2.png)
*Figure 2: Full email artifact details for Event ID 8815*

---

### Step 3 — URL Reputation Analysis

Ran the bit.ly link through TryDetectThis to see if it was flagged.

**URL submitted:** `http://bit.ly/3sHkX3da12340`

**Result: MALICIOUS**

![TryDetectThis URL analysis confirming the embedded bit.ly link as MALICIOUS](screenshots/url_analysis_malicious.png)
*Figure 3: TryDetectThis URL/IP Security Check confirming MALICIOUS status for the embedded link*

Came back malicious. That confirmed it.

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

Pretty clear-cut phishing attempt. Fake Amazon sender domain, URL shortener hiding a malicious link, urgency subject line. URL confirmed malicious so I closed it as a true positive. No evidence the user clicked it so no escalation needed.

**Recommended defensive actions:**

- Block sender domain `amazon.biz` at the email gateway
- Quarantine the email from recipient's inbox
- Let `h.harris@thetrydaily.thm` know what happened and send over some phishing awareness guidance
- Add the bit.ly URL to the blocklist

---

## Key Takeaways

- `@amazon.biz` vs `@amazon.com` is an obvious tell once you look — always check if the sender domain actually matches the company they're claiming to be
- bit.ly and other URL shorteners should be treated as suspicious until you know where they actually go — run them through a rep tool before doing anything else
- Medium severity doesn't mean low importance. This was a real phish that could've led to stolen creds if the user clicked
- Not every true positive needs escalation — if there's no evidence the user interacted with it, containment at the email level is enough

---

*Write-up by Trystan Ruiz*
