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

Event ID 8816 was flagged on the SIEM at 19:59, approximately five hours after the phishing email from Lab #02 was delivered to `h.harris@thetrydaily.thm`. The alert rule triggered when the firewall detected and blocked an outbound connection from an internal host attempting to reach a blacklisted external URL. Opening the case revealed the full firewall log details for analysis.

The source IP was `10.20.2.17`, which is `h.harris`'s internal workstation, attempting an outbound HTTP connection to destination IP `67.199.248.11` on TCP port 80. The URL logged by the firewall was `http://bit.ly/3sHkX3da12340`, which is the exact same shortened URL that was embedded in the phishing email from Lab #02. This confirms that `h.harris` clicked the malicious link at some point after it was delivered at 14:59.

It is important to note that a blocked connection does not guarantee that no data was transmitted. The bit.ly link would have attempted to redirect to a downstream destination, and depending on timing, partial traffic may have occurred before the firewall rule engaged. Endpoint logs from `10.20.2.17` would be needed to fully confirm what was or was not communicated.

![Firewall log for Event ID 8816](screenshots/firewall_alert_details.png)

| Field | Value | Notes |
|---|---|---|
| Source IP | 10.20.2.17 | h.harris's internal workstation |
| Destination IP | 67.199.248.11 | Blacklisted external IP |
| Destination Port | 80 | Unencrypted HTTP traffic |
| URL | http://bit.ly/3sHkX3da12340 | Same URL from Lab #02 phishing email |
| Action | Blocked | Firewall prevented connection from completing |

Both the destination IP and the URL were submitted to TryDetectThis independently, the scenario's internal threat intelligence platform, essentially the same thing as VirusTotal, which checks submissions against known threat feeds and reputation databases to return a malicious or clean verdict.

![TryDetectThis confirming 67.199.248.11 as MALICIOUS](screenshots/ip_reputation_malicious.png)

![TryDetectThis confirming the URL as MALICIOUS](screenshots/url_reputation_malicious.png)

As seen in the screenshots above, both the destination IP and the URL returned malicious verdicts. The full incident report was completed and the event was escalated given that a user had actively clicked a confirmed malicious link and their workstation had initiated an outbound connection to a known malicious IP.

![Incident report](screenshots/incident_report.png)

---

## Connection to Lab #02

| Time | Event |
|---|---|
| 14:59 | Phishing email with malicious bit.ly link delivered to h.harris |
| ~15:00 | h.harris clicks the link; 10.20.2.17 initiates outbound connection to 67.199.248.11 |
| 19:59 | Firewall alert fires; outbound connection blocked and logged |

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

True positive, escalated. The firewall blocked the outbound connection, but `h.harris` did click the malicious link from the Lab #02 phishing email. The destination behind the bit.ly shortener resolved to `67.199.248.11`, a confirmed malicious IP. Recommended follow-up actions: contact `h.harris` and advise them of the incident, isolate workstation `10.20.2.17` pending further investigation, pull endpoint logs to determine if any data was transmitted before the block, reset `h.harris`'s credentials as a precaution, and confirm `67.199.248.11` and `http://bit.ly/3sHkX3da12340` are blocked organization-wide.

---

*Write-up by Trystan Ruiz*
