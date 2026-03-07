# SOC Lab #03 — Firewall Alert: Access to Blacklisted External URL

**Platform:** TryHackMe SOC Simulator
**Date:** February 1, 2026
**Outcome:** True Positive, escalated

---

## Alert Details

| Field | Value |
|---|---|
| Event ID | 8816 |
| Alert Rule | Access to Blacklisted External URL Blocked by Firewall |
| Severity | High |
| Date Detected | February 1st, 2026 at 19:59 |
| Data Source | Firewall |
| Action Taken | Blocked |

---

## What Happened

High severity firewall alert — internal host tried to reach a blacklisted external URL and got blocked. This came in a few hours after the phishing alert from Lab #02.

Pulled up the firewall log for Event ID 8816 and immediately recognized the URL: `http://bit.ly/3sHkX3da12340` — same link from the phishing email. So h.harris clicked it.

![Firewall alert queue showing Event ID 8816 with full connection details](screenshots/firewall_alert_details.png)

**Log breakdown:**

| Field | Value | Notes |
|---|---|---|
| Source IP | 10.20.2.17 | Internal host — h.harris's workstation |
| Destination IP | 67.199.248.11 | Blacklisted external IP |
| Destination Port | 80 | HTTP |
| URL | http://bit.ly/3sHkX3da12340 | Same URL from Lab #02 phishing email |
| Action | Blocked | Firewall caught it |

Ran the destination IP through TryDetectThis to confirm independently.

![TryDetectThis confirming 67.199.248.11 as MALICIOUS](screenshots/ip_reputation_malicious.png)

Also ran the URL itself.

![TryDetectThis confirming the bit.ly URL as MALICIOUS](screenshots/url_reputation_malicious.png)

Both malicious. No doubt about a false positive here.

The firewall blocked the connection but that doesn't mean nothing happened — we don't know if the user entered credentials before the block kicked in. That's why this needed to go up.

![Completed incident report](screenshots/incident_report.png)

| Field | Detail |
|---|---|
| Affected Host | 10.20.2.17 |
| Classification | True Positive |
| Reason for Escalation | Potential compromised workstation/account |

---

## How This Connects to Lab #02

| Time | Event |
|---|---|
| 14:59 | Phishing email delivered to h.harris (Lab #02) |
| 15:00 | User clicks the link — 10.20.2.17 initiates outbound connection |
| 19:59 | Firewall alert fires (Lab #03) |

The phishing email from Lab #02 is what caused this alert. Two separate alerts, one attack chain.

---

## IOC Summary

| IOC Type | Value | Confidence |
|---|---|---|
| Malicious Destination IP | 67.199.248.11 | High |
| Malicious URL | http://bit.ly/3sHkX3da12340 | High |
| Affected Internal Host | 10.20.2.17 | High |

---

## MITRE ATT&CK

| Technique ID | Technique Name | Notes |
|---|---|---|
| T1566.002 | Phishing: Spearphishing Link | User clicked link from phishing email in Lab #02 |
| T1204.001 | User Execution: Malicious Link | Outbound connection initiated from internal host |
| T1071.001 | Application Layer Protocol: Web Protocols | HTTP over TCP/80 to attacker IP |
| T1090 | Proxy | bit.ly used to redirect to malicious destination |

---

## Verdict

True positive, escalated. The firewall blocked the connection but a blocked connection still means something triggered it. Recommended actions: contact h.harris and find out if anything was entered, isolate 10.20.2.17 if needed, reset credentials, and pull endpoint logs. Make sure the IP and URL are blocked org-wide.

Big takeaway from this one — working the phishing ticket and the firewall ticket separately would've missed the full picture. Correlating them told the whole story.

---

*Write-up by Trystan Ruiz*
