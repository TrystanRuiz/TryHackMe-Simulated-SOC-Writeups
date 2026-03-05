# SOC Lab #01 — Inbound Email Containing Suspicious External Link
**Platform:** TryHackMe SOC Simulator
**Category:** Phishing Analysis
**Difficulty:** Medium
**Date:** February 1, 2026
**Outcome:** False Positive — Closed, No Escalation Required

---

## Overview

This lab involved triaging a phishing alert triggered by an inbound email flagged by the SIEM for containing a suspicious external link. The investigation required reviewing email artifacts and running the embedded URL through an internal security scanner to determine whether the activity was malicious or a legitimate false positive.

---

## Alert Details

| Field | Value |
|---|---|
| Event ID | 8814 |
| Alert Rule | Inbound Email Containing Suspicious External Link |
| Incident Type | Phishing |
| Severity | Medium |
| Date Detected | February 1st, 2026 at 14:57 |
| Data Source | Email |

---

## Investigation

### Step 1 — Alert Triage

Upon receiving the alert, I reviewed the case report for Event ID 8814. The alert rule fired on an inbound email flagged for containing one or more external links with potentially suspicious characteristics. The SIEM recommended checking firewall and proxy logs to determine whether any endpoints had attempted to access the embedded URLs.

![Case report for Event ID 8814 flagged as phishing with medium severity](screenshots/thm1.png)
*Figure 1: Case report for Event ID 8814 — Inbound Email Containing Suspicious External Link*

---

### Step 2 — Email Artifact Analysis

I reviewed the full email metadata and content to identify potential IOCs:

| Artifact | Value | Finding |
|---|---|---|
| Sender | onboarding@hrconnex.thm | HR onboarding service — requires verification |
| Recipient | j.garcia@thetrydaily.thm | Internal employee targeted |
| Subject | Action Required: Finalize Your Onboarding Profile | Urgency language — common phishing lure |
| Attachment | None | No malicious attachment |
| Direction | Inbound | External sender |
| Embedded URL | https://hrconnex.thm/onboarding/15400654060/j.garcia | Contains recipient-specific path — warrants scanning |

**Observations:**

- The email is addressed to **Julia Garcia** (`j.garcia@thetrydaily.thm`) and references completing an onboarding profile for TheTryDaily.
- The sender domain `hrconnex.thm` is consistent with an HR onboarding platform rather than a known threat domain.
- The URL contains a user-specific path (`/j.garcia`) which is typical of legitimate onboarding workflows.
- No attachment was present and the email body contained no overt social engineering beyond a standard call-to-action.

---

### Step 3 — URL Reputation Analysis

I extracted the embedded URL and submitted it to TryDetectThis, the internal URL/IP security analysis tool available in the Analyst VM, to confirm whether the link was malicious.

**URL submitted:** `https://hrconnex.thm/onboarding/15400654060/j.garcia`

**Result: SAFE**

![TryDetectThis URL analysis showing the embedded link returned clean with no malicious activity detected](screenshots/thm3.png)
*Figure 2: TryDetectThis URL/IP Security Check — no malicious activity found*

The tool confirmed the URL as clean, consistent with a legitimate HR onboarding email.

---

## Verdict & Response

**Classification:** False Positive
**Escalation Required:** No

Based on the email artifact analysis and URL reputation scan, this alert was classified as a **false positive**. The email was a legitimate onboarding communication sent to Julia Garcia via the HR platform `hrconnex.thm`. The embedded link was scanned and returned no indicators of malicious activity.

**Closure rationale documented:**

- Time of Activity: Feb 1st 2026 @ 14:57
- Related Entities: Julia Garcia, `j.garcia@thetrydaily.thm`
- Reason: User was sent a legitimate email to complete an onboarding process. URL was scanned internally and no malicious activity was found.

![Incident report showing false positive classification with closure rationale filled out](screenshots/thm2.png)
*Figure 3: Incident report — classified as False Positive and closed*

---

## Key Takeaways

- Not every phishing alert is a true positive — urgency language and external links alone are not sufficient to confirm malice
- URL scanning tools are a critical step in the triage process to confirm or rule out malicious intent
- Recipient-specific URLs in onboarding emails are a normal pattern — context matters when analyzing email artifacts
- Proper documentation of the false positive rationale is essential for audit trails and tuning alert rules to reduce noise

---

*Write-up by Trystan Ruiz | TryHackMe SOC Simulator | Blue Team Portfolio*
