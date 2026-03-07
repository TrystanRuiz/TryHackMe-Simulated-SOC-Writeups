# SOC Lab #04 — Microsoft Account Impersonation Phishing

**Platform:** TryHackMe SOC Simulator
**Date:** February 1, 2026
**Outcome:** True Positive, closed no escalation

---

## Alert Details

| Field | Value |
|---|---|
| Event ID | 8817 |
| Alert Rule | Inbound Email Containing Suspicious External Link |
| Severity | Medium |
| Date Detected | February 1st, 2026 at 20:00 |
| Data Source | Email |

---

## What Happened

Same alert rule as Lab #02. Pulled up Event ID 8817 — inbound email with a suspicious link.

Sender was `no-reply@m1crosoftsupport.co`. The "i" in Microsoft is replaced with a "1". If you're reading fast you'd probably miss it, which is the whole point. Real Microsoft emails come from @microsoft.com, not some .co domain with a character swap in the name.

Subject line was "Unusual Sign-In Activity on Your Microsoft Account" — classic fear lure. The email body also referenced a sign-in attempt from Lagos, Nigeria with an IP (`102.89.222.143`) attached to it. That detail is just there to make it feel real and get the user to click without thinking.

The embedded link goes to `https://m1crosoftsupport.co/login` — same typosquatted domain, fake Microsoft login page to harvest credentials.

![Full alert and email details for Event ID 8817](screenshots/alert_email_details.png)

**Email artifacts:**

| Artifact | Value | Finding |
|---|---|---|
| Sender | no-reply@m1crosoftsupport.co | "i" replaced with "1" — typosquatting |
| Recipient | c.allen@thetrydaily.thm | Internal employee |
| Subject | Unusual Sign-In Activity on Your Microsoft Account | Fear/urgency lure |
| Embedded URL | https://m1crosoftsupport.co/login | Fake Microsoft login page |
| IP in Email Body | 102.89.222.143 | Lagos, Nigeria — used for social engineering |

Ran the URL through TryDetectThis.

![TryDetectThis confirming the URL as MALICIOUS](screenshots/url_reputation_malicious.png)

Malicious. Closed it out.

![Completed incident report](screenshots/incident_report.png)

| Field | Detail |
|---|---|
| Affected User | Charlotte Allen, c.allen@thetrydaily.thm |
| Classification | True Positive |
| Reason for No Escalation | No evidence the user interacted with the link |

---

## IOC Summary

| IOC Type | Value | Confidence |
|---|---|---|
| Typosquatted Sender Domain | m1crosoftsupport.co | High |
| Malicious URL | https://m1crosoftsupport.co/login | High |
| Foreign IP in Email Body | 102.89.222.143 | Medium |

---

## MITRE ATT&CK

| Technique ID | Technique Name | Notes |
|---|---|---|
| T1566.002 | Phishing: Spearphishing Link | Malicious URL targeting internal employee |
| T1036.005 | Masquerading: Match Legitimate Name | Typosquatted domain impersonating Microsoft |
| T1078 | Valid Accounts | Credential harvesting targeting Microsoft login |
| T1204.001 | User Execution: Malicious Link | User prompted to click fake login link |

---

## Verdict

True positive. Typosquatted domain, confirmed malicious URL, fake Microsoft login page. No evidence Charlotte clicked it so no escalation — blocked the domain, quarantined the email, added the IP to the blocklist.

The character swap (i to 1) is subtle enough that it's worth specifically flagging to the user so they know what to look for next time.

---

*Write-up by Trystan Ruiz*
