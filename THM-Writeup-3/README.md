SOC Lab #03 — Firewall Alert: Access to Blacklisted External URL
---

## Overview

This lab involved triaging a high severity firewall alert triggered when an internal endpoint attempted to access a known malicious external URL. The investigation required analyzing firewall logs, performing IP and URL reputation checks, identifying the affected host, and determining whether the incident warranted escalation. This alert was a direct continuation of the phishing campaign identified in Lab #02, showing the downstream network impact after a user clicked a malicious link.

---

## Alert Details

| Field | Value |
|---|---|
| Event ID | 8816 |
| Alert Rule | Access to Blacklisted External URL Blocked by Firewall |
| Incident Type | Firewall |
| Severity | **High** |
| Date Detected | February 1st, 2026 at 19:59 |
| Data Source | Firewall |
| Action Taken | Blocked |

---

## Investigation

### Step 1 — Alert Triage

Upon receiving the alert, I reviewed the firewall log entry for Event ID 8816. The alert fired when an internal host attempted an outbound connection to an external IP address listed on the organization's threat intelligence blacklist. The firewall successfully blocked the connection, but the attempt itself indicated a potentially compromised endpoint.

![Firewall alert queue showing Event ID 8816 flagged as High severity with full connection details](screenshots/firewall_alert_details.png)
*Figure 1: Alert queue entry for Event ID 8816 — Access to Blacklisted External URL Blocked by Firewall*

**Firewall log breakdown:**

| Field | Value | Significance |
|---|---|---|
| Source IP | 10.20.2.17 | Internal endpoint — potential victim host |
| Source Port | 34257 | Ephemeral port — outbound connection initiated by host |
| Destination IP | 67.199.248.11 | External IP — blacklisted |
| Destination Port | 80 | HTTP — unencrypted web traffic |
| URL | http://bit.ly/3sHkX3da12340 | Same malicious URL from Lab #02 phishing email |
| Application | Web-browsing | User-initiated browser request |
| Protocol | TCP | Standard web connection |
| Rule Triggered | Blocked Websites | Firewall blacklist rule fired |

**Critical observation:** The URL `http://bit.ly/3sHkX3da12340` is the exact same malicious link embedded in the phishing email from Lab #02. This confirms the targeted user at `h.harris@thetrydaily.thm` clicked the link, causing their workstation at `10.20.2.17` to initiate an outbound connection to the attacker's infrastructure.

---

### Step 2 — IP Reputation Analysis

I extracted the destination IP `67.199.248.11` from the firewall log and submitted it to TryDetectThis to confirm its malicious status independently of the URL blacklist.

**IP submitted:** `67.199.248.11`

**Result: MALICIOUS**

![TryDetectThis IP analysis confirming destination IP 67.199.248.11 as MALICIOUS](screenshots/ip_reputation_malicious.png)
*Figure 2: TryDetectThis URL/IP Security Check confirming MALICIOUS status for destination IP 67.199.248.11*

---

### Step 3 — URL Reputation Confirmation

I additionally submitted the full URL to confirm the bit.ly link resolves to known malicious infrastructure, corroborating both the firewall blacklist hit and the IP reputation result.

**URL submitted:** `http://bit.ly/3sHkX3da12340`

**Result: MALICIOUS**

![TryDetectThis URL analysis confirming the bit.ly link as MALICIOUS](screenshots/url_reputation_malicious.png)
*Figure 3: TryDetectThis confirming MALICIOUS status for the embedded bit.ly URL*

Both the destination IP and the full URL independently confirmed as malicious — this rules out a false positive.

---

### Step 4 — Incident Report & Escalation Decision

With both reputation checks confirming malicious infrastructure and the source IP pointing to an internal workstation, I completed the incident report and determined escalation was required.

![Completed incident report showing True Positive classification, affected entities, escalation reasoning and remediation actions](screenshots/incident_report.png)
*Figure 4: Completed incident report — True Positive, escalated due to potential compromised workstation*

**Incident report summary:**

| Field | Detail |
|---|---|
| Time of Activity | Feb 1st 2026 @ 15:00 |
| Affected Entity | 10.20.2.17 |
| Classification | True Positive |
| Reason for True Positive | Internal computer attempted to access a known malicious IP and website |
| Reason for Escalation | Potential compromised user account and/or workstation |

---

## IOC Summary

| IOC Type | Value | Confidence |
|---|---|---|
| Malicious Destination IP | 67.199.248.11 | High |
| Malicious URL | http://bit.ly/3sHkX3da12340 | High |
| Affected Internal Host | 10.20.2.17 | High |
| Protocol/Port | TCP/80 | Medium |

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Observed Behavior |
|---|---|---|
| T1566.002 | Phishing: Spearphishing Link | User clicked malicious link from phishing email (Lab #02) |
| T1204.001 | User Execution: Malicious Link | Internal host initiated outbound connection to C2 infrastructure |
| T1071.001 | Application Layer Protocol: Web Protocols | Outbound HTTP connection over TCP/80 to attacker IP |
| T1090 | Proxy | Bit.ly URL shortener used to proxy/redirect to malicious destination |

---

## Verdict & Response

**Classification:** True Positive  
**Escalation Required:** Yes

The firewall alert was classified as a confirmed true positive. An internal workstation (`10.20.2.17`) attempted to reach known malicious infrastructure (`67.199.248.11`) via the same bit.ly URL embedded in the phishing email investigated in Lab #02. The firewall successfully blocked the connection, preventing the full compromise, but the attempt indicates the user clicked the malicious link.

**Recommended remediation actions:**

- Contact the user associated with `10.20.2.17` to determine if credentials or sensitive data were entered before the connection was blocked
- If user cannot be confirmed safe, isolate `10.20.2.17` from the network immediately
- Revoke and reset credentials for the affected account as a precaution
- Mitigate privileges on the affected account while investigation is ongoing
- Review endpoint logs on `10.20.2.17` for any additional suspicious activity or persistence mechanisms
- Confirm the bit.ly URL and destination IP `67.199.248.11` are blocked organization-wide

---

## Connection to Lab #02

This alert is a direct continuation of the phishing campaign documented in [SOC Lab #02](../THM-Writeup-2/README.md). The timeline tells the full story:

| Time | Event |
|---|---|
| 14:59 | Phishing email delivered to h.harris@thetrydaily.thm (Lab #02) |
| 15:00 | User clicks malicious link — outbound connection attempt from 10.20.2.17 |
| 19:59 | Firewall alert fires on blacklisted URL access (Lab #03) |

This demonstrates how a single unblocked phishing email can result in downstream network alerts — and why both email and firewall data sources must be correlated during SOC investigations.

---

## Key Takeaways

- **Firewall blocks are not the end of the investigation** — a blocked connection still means an internal host tried to reach malicious infrastructure, which requires investigation into why
- **Cross-alert correlation is critical** — connecting this firewall alert back to the phishing email in Lab #02 tells the full attack story and identifies the root cause
- **High severity alerts require escalation** — unlike the medium phishing alert, a potentially compromised workstation elevates the response level
- **Dual IOC validation** — confirming both the URL and the destination IP independently as malicious removes any doubt about a false positive before escalating

---

*Write-up by Trystan Ruiz | TryHackMe SOC Simulator | Blue Team Portfolio*
