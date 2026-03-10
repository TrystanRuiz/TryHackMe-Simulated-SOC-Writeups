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

Event ID 8815 was flagged on the SIEM due to an alert rule firing for an inbound email containing a suspicious external link. Opening the case in the SOC platform revealed the full email artifact details for analysis.

The sender was `urgents@amazon.biz` and the recipient was `h.harris@thetrydaily.thm`. The recipient is a confirmed internal employee. The sender domain `amazon.biz` is not a legitimate Amazon domain. Amazon exclusively sends from `@amazon.com` and has no association with the `.biz` TLD, making this a clear indicator of sender domain spoofing where a threat actor registers a lookalike domain to impersonate a trusted brand. You can also verify sender legitimacy against internal records (which I did) by confirming whether the sending domain matches any known official vendor accounts.

The subject line was: **"Your Amazon Package Couldn't Be Delivered - Action Required."**

This is a social engineering lure designed to create urgency around a missed package delivery, a scenario plausible enough for almost any recipient. The goal is to pressure the user into clicking the link without scrutinizing the email.

There was no file attachment. The email contained a single embedded URL: `http://bit.ly/3sHkX3da12340`.

The use of a bit.ly URL shortener is a red flag in this context. Legitimate Amazon delivery notifications link directly to `amazon.com` domains, not third-party URL shorteners. Shorteners are commonly abused in phishing campaigns because they hide the true destination, making it impossible to identify where the link actually leads without following it.

![Alert case report showing Event ID 8815](screenshots/alert_case_report.png)

![Email artifact details](screenshots/alert_case_report2.png)

| Artifact | Value | Finding |
|---|---|---|
| Sender | urgents@amazon.biz | Spoofed sender domain, not a legitimate Amazon domain |
| Recipient | h.harris@thetrydaily.thm | Internal employee |
| Subject | Your Amazon Package Couldn't Be Delivered - Action Required | Urgency-based social engineering lure |
| Attachment | None | No malicious file risk |
| Embedded URL | http://bit.ly/3sHkX3da12340 | URL shortener hiding true destination |

To confirm whether the URL was malicious or not, it was submitted to TryDetectThis, the scenario's internal threat intelligence platform, essentially the same thing as VirusTotal, which checks submissions against known threat feeds and reputation databases to return a malicious or clean verdict.

![TryDetectThis confirming MALICIOUS](screenshots/url_analysis_malicious.png)

As seen in the screenshot above, TryDetectThis returned a malicious verdict on the bit.ly URL. Combined with the spoofed sender domain, this confirms a true positive. There is no evidence in the logs that `h.harris` clicked the link, so the threat was contained at the email delivery level.

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

True positive. The sender domain `amazon.biz` is spoofed and has no affiliation with Amazon. The embedded bit.ly URL was confirmed malicious by TryDetectThis. No evidence was found that `h.harris` clicked the link. Actions taken: domain `amazon.biz` blocked, email quarantined, URL `http://bit.ly/3sHkX3da12340` added to the blocklist. Alert closed, no escalation required.

---

*Write-up by Trystan Ruiz*
