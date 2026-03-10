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

## Investigation

Event ID 8817 was flagged on the SIEM at 20:00, one minute after the firewall alert from Lab #03, triggered by the same alert rule for an inbound email containing a suspicious link. Opening the case in the SOC platform revealed the full email artifact details for analysis.

The sender was `no-reply@m1crosoftsupport.co` and the recipient was `c.allen@thetrydaily.thm`, a confirmed internal employee. At a quick glance the sender domain appears to be Microsoft, but on closer inspection the domain uses the number `1` in place of the letter `i` in "microsoft", making it `m1crosoftsupport.co` instead of a legitimate Microsoft domain. This is a typosquatting technique where a threat actor registers a domain that visually resembles a trusted brand to deceive recipients who do not scrutinize the sender address carefully. Microsoft's legitimate security notification emails originate from domains like `@microsoft.com` or `@account.microsoft.com`, not from a `.co` TLD with a character substitution. You can also verify this against internal records and known vendor domains to confirm it is not a registered company account.

The subject line was: **"Unusual Sign-In Activity on Your Microsoft Account."**

This is a social engineering tactic targeting account security anxiety. It implies the recipient's account is currently being accessed by someone else, creating urgency to act immediately. The email body reinforced this by referencing a specific sign-in from IP `102.89.222.143` attributed to Lagos, Nigeria. Including a real-looking IP address and geographic location makes the email appear to be a genuine Microsoft security alert. This IP is likely fabricated or spoofed within the email body for psychological effect rather than being a real technical indicator.

The embedded URL was `https://m1crosoftsupport.co/login`, directing the user to a fake Microsoft login page hosted on the same typosquatted domain. The intent is credential harvesting: the user, believing their account is under threat, navigates to what looks like a Microsoft login page and submits their credentials, which are captured by the attacker.

![Alert and email details for Event ID 8817](screenshots/alert_email_details.png)

| Artifact | Value | Finding |
|---|---|---|
| Sender | no-reply@m1crosoftsupport.co | Typosquatting, "i" replaced with "1", not a Microsoft domain |
| Recipient | c.allen@thetrydaily.thm | Internal employee |
| Subject | Unusual Sign-In Activity on Your Microsoft Account | Fear and urgency lure targeting account security |
| Embedded URL | https://m1crosoftsupport.co/login | Fake Microsoft login page for credential harvesting |
| IP in Email Body | 102.89.222.143 | Lagos, Nigeria, used to add false legitimacy to the lure |

To confirm whether the URL was malicious or not, it was submitted to TryDetectThis, the scenario's internal threat intelligence platform, essentially the same thing as VirusTotal, which checks submissions against known threat feeds and reputation databases to return a malicious or clean verdict.

![TryDetectThis confirming MALICIOUS](screenshots/url_reputation_malicious.png)

As seen in the screenshot above, TryDetectThis returned a malicious verdict on `https://m1crosoftsupport.co/login`, confirming this is a credential harvesting phishing page. There is no evidence in the logs that `c.allen` visited the page or submitted any credentials, so the threat was contained at the email delivery level.

![Incident report](screenshots/incident_report.png)

---

## IOC Summary

| IOC Type | Value | Confidence |
|---|---|---|
| Typosquatted Domain | m1crosoftsupport.co | High |
| Malicious URL | https://m1crosoftsupport.co/login | High |
| IP in Email Body | 102.89.222.143 | Medium |

---

## MITRE ATT&CK

| Technique ID | Technique Name | Notes |
|---|---|---|
| T1566.002 | Phishing: Spearphishing Link | Malicious URL targeting internal employee |
| T1036.005 | Masquerading: Match Legitimate Name | Typosquatted domain impersonating Microsoft |
| T1078 | Valid Accounts | Credential harvesting via fake login page |
| T1204.001 | User Execution: Malicious Link | User prompted to click fake login link |

---

## Verdict

True positive. The sender domain `m1crosoftsupport.co` is a typosquatted impersonation of Microsoft and is not a legitimate Microsoft domain. The embedded URL was confirmed malicious by TryDetectThis and leads to a credential harvesting page. No evidence was found that `c.allen` clicked the link or submitted credentials. Actions taken: domain `m1crosoftsupport.co` blocked, email quarantined, URL `https://m1crosoftsupport.co/login` added to the blocklist, IP `102.89.222.143` added to the blocklist. `c.allen` was notified about the typosquatting technique. Alert closed, no escalation required.

---

*Write-up by Trystan Ruiz*
