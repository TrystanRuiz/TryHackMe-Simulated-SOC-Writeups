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

## Investigation

Pulled up Event ID 8816. Internal host attempted an outbound connection to a blacklisted IP, firewall blocked it.

URL in the log was `http://bit.ly/3sHkX3da12340` — same link from the phishing email in Lab #02. h.harris clicked it.

![Firewall log for Event ID 8816](screenshots/firewall_alert_details.png)

| Field | Value | Notes |
|---|---|---|
| Source IP | 10.20.2.17 | h.harris's workstation |
| Destination IP | 67.199.248.11 | Blacklisted external IP |
| Destination Port | 80 | HTTP |
| URL | http://bit.ly/3sHkX3da12340 | Same URL from Lab #02 |
| Action | Blocked | |

Ran the destination IP through TryDetectThis.

![TryDetectThis confirming 67.199.248.11 as MALICIOUS](screenshots/ip_reputation_malicious.png)

Also ran the URL.

![TryDetectThis confirming the URL as MALICIOUS](screenshots/url_reputation_malicious.png)

Both malicious. Filled out the incident report and escalated.

![Incident report](screenshots/incident_report.png)

---

## Connection to Lab #02

| Time | Event |
|---|---|
| 14:59 | Phishing email delivered to h.harris |
| 15:00 | User clicks link, 10.20.2.17 initiates outbound connection |
| 19:59 | Firewall alert fires |

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
| T1566.002 | Phishing: Spearphishing Link | User clicked link from Lab #02 phishing email |
| T1204.001 | User Execution: Malicious Link | Outbound connection from internal host |
| T1071.001 | Application Layer Protocol: Web Protocols | HTTP over TCP/80 |
| T1090 | Proxy | bit.ly redirecting to malicious destination |

---

## Verdict

True positive, escalated. Firewall blocked the connection but we don't know if the user entered anything before it was cut. Recommended: contact h.harris, isolate 10.20.2.17 if needed, reset credentials, pull endpoint logs, confirm IP and URL are blocked org-wide.

---

*Write-up by Trystan Ruiz*
