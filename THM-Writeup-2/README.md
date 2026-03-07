# SOC Lab #02 — Phishing Email Investigation

**Platform:** TryHackMe SOC Simulator
**Date:** February 1, 2026
**Outcome:** True Positive, closed no escalation

---

## Alert Details

| Field | Value |
|---|---|
| Event ID | 8815 |
| Alert Rule | Inbound Email Containing Suspicious External Link |
| Severity | Medium |
| Date Detected | February 1st, 2026 at 14:59 |
| Data Source | Email |

---

## Investigation

Pulled up Event ID 8815. Inbound email flagged for a suspicious external link.

Checked the sender first — `urgents@amazon.biz`. Amazon sends from @amazon.com, not .biz. Subject was "Your Amazon Package Couldn't Be Delivered – Action Required." No attachment, just a bit.ly link asking the user to confirm shipping info.

![Alert case report showing Event ID 8815](screenshots/alert_case_report.png)

![Email artifact details](screenshots/alert_case_report2.png)

| Artifact | Value | Finding |
|---|---|---|
| Sender | urgents@amazon.biz | Spoofed — Amazon uses @amazon.com |
| Recipient | h.harris@thetrydaily.thm | Internal employee |
| Subject | Your Amazon Package Couldn't Be Delivered – Action Required | Urgency lure |
| Attachment | None | |
| Embedded URL | http://bit.ly/3sHkX3da12340 | URL-shortened link |

Ran the bit.ly link through TryDetectThis.

![TryDetectThis confirming MALICIOUS](screenshots/url_analysis_malicious.png)

Came back malicious.

---

## IOC Summary

| IOC Type | Value | Confidence |
|---|---|---|
| Malicious URL | http://bit.ly/3sHkX3da12340 | High |
| Spoofed Sender Domain | urgents@amazon.biz | High |

---

## MITRE ATT&CK

| Technique ID | Technique Name | Notes |
|---|---|---|
| T1566.002 | Phishing: Spearphishing Link | Malicious URL sent via email |
| T1036.005 | Masquerading: Match Legitimate Name | Sender spoofing Amazon domain |
| T1204.001 | User Execution: Malicious Link | User prompted to click link |

---

## Verdict

True positive. Spoofed sender domain, malicious URL confirmed. No evidence h.harris clicked it — blocked the domain, quarantined the email, added the URL to the blocklist.

---

*Write-up by Trystan Ruiz*
