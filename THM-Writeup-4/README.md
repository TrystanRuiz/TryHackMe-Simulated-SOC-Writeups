# SOC Lab #04 — Microsoft Account Impersonation Phishing
**Platform:** TryHackMe SOC Simulator  
**Category:** Phishing Analysis  
**Difficulty:** Medium  
**Date:** February 1, 2026  
**Outcome:** ✅ True Positive — No Escalation Required

---

## Overview

This lab involved triaging an inbound phishing alert where a threat actor impersonated Microsoft's account security team to trick an internal employee into clicking a malicious login link. The investigation required analyzing email artifacts, identifying domain spoofing and typosquatting techniques, performing URL reputation analysis, and determining the appropriate containment response.

---

## Alert Details

| Field | Value |
|---|---|
| Event ID | 8817 |
| Alert Rule | Inbound Email Containing Suspicious External Link |
| Incident Type | Phishing |
| Severity | Medium |
| Date Detected | February 1st, 2026 at 20:00 |
| Data Source | Email |

---

## Investigation

### Step 1 — Alert Triage

Upon receiving the alert, I reviewed the case report for Event ID 8817. The alert fired on an inbound email flagged for containing a suspicious external link. The SIEM recommended checking firewall and proxy logs to determine if any endpoints had attempted to access the embedded URL.

![Alert queue showing Event ID 8817 flagged as medium severity phishing with full email artifact details](screenshots/alert_email_details.png)
*Figure 1: Alert queue entry for Event ID 8817 — Inbound Email Containing Suspicious External Link*

---

### Step 2 — Email Artifact Analysis

I reviewed the full email metadata and content to identify IOCs:

| Artifact | Value | Finding |
|---|---|---|
| Sender | no-reply@m1crosoftsupport.co | Typosquatting — "m1crosoft" uses number "1" instead of letter "i" |
| Recipient | c.allen@thetrydaily.thm | Internal employee targeted |
| Subject | Unusual Sign-In Activity on Your Microsoft Account | Fear/urgency lure |
| Attachment | None | No malicious attachment |
| Direction | Inbound | External sender |
| Embedded URL | https://m1crosoftsupport.co/login | Typosquatted domain mimicking Microsoft |
| IP in Email Body | 102.89.222.143 | Foreign IP — Lagos, Nigeria |

**Key red flags identified:**

- **Typosquatting domain** — Both the sender domain and embedded URL use `m1crosoftsupport.co` replacing the letter "i" with the number "1" — a classic typosquatting technique designed to visually fool recipients into thinking it's legitimate Microsoft infrastructure
- **Fake TLD** — Legitimate Microsoft communications come from `@microsoft.com` — the `.co` TLD is a common indicator of a spoofed domain
- **Urgency and fear lure** — The subject line "Unusual Sign-In Activity" creates alarm to pressure the recipient into acting without verifying the sender
- **Foreign IP disclosure** — The email body references a sign-in attempt from Lagos, Nigeria (`102.89.222.143`) to add legitimacy and increase the sense of urgency
- **Credential harvesting setup** — The embedded link directs to a fake Microsoft login page designed to steal credentials

---

### Step 3 — URL Reputation Analysis

I extracted the embedded URL from the email and submitted it to TryDetectThis to confirm its malicious status before taking action.

**URL submitted:** `https://m1crosoftsupport.co/login`

**Result: MALICIOUS**

![TryDetectThis URL analysis confirming https://m1crosoftsupport.co/login as MALICIOUS](screenshots/url_reputation_malicious.png)
*Figure 2: TryDetectThis URL/IP Security Check confirming MALICIOUS status for the typosquatted Microsoft login URL*

The reputation check confirmed the URL as malicious, corroborating the typosquatting indicators identified during email artifact analysis.

---

### Step 4 — Incident Report & Closure

With the URL confirmed malicious and the phishing technique clearly identified, I completed the incident report. No escalation was required as there was no evidence the user clicked the link or that any endpoint accessed the malicious URL.

![Completed incident report showing True Positive classification, affected entities, and remediation actions](screenshots/incident_report.png)
*Figure 3: Completed incident report — True Positive, no escalation required*

**Incident report summary:**

| Field | Detail |
|---|---|
| Time of Activity | Feb 1st 2026 @ 15:01 |
| Affected Entities | Charlotte Allen, c.allen@thetrydaily.thm |
| Classification | True Positive |
| Reason for True Positive | Impersonation of a company via malicious link |
| Reason for No Escalation | No evidence of user interaction with the link |
| Attack Indicators | Fake company name, URL contains company name misspelling (typosquatting) |

---

## IOC Summary

| IOC Type | Value | Confidence |
|---|---|---|
| Typosquatted Sender Domain | m1crosoftsupport.co | High |
| Malicious URL | https://m1crosoftsupport.co/login | High |
| Foreign IP Referenced in Email | 102.89.222.143 | Medium |
| Targeted User | c.allen@thetrydaily.thm | High |

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Observed Behavior |
|---|---|---|
| T1566.002 | Phishing: Spearphishing Link | Inbound email with malicious URL targeting internal employee |
| T1036.005 | Masquerading: Match Legitimate Name | Typosquatted domain impersonating Microsoft |
| T1078 | Valid Accounts | Credential harvesting page designed to steal Microsoft account credentials |
| T1204.001 | User Execution: Malicious Link | Recipient prompted to click fake Microsoft login link |

---

## Verdict & Response

**Classification:** True Positive  
**Escalation Required:** No

This alert was classified as a confirmed true positive phishing attempt. The threat actor impersonated Microsoft's account security team using a typosquatted domain (`m1crosoftsupport.co`) to deliver a credential harvesting link to internal employee Charlotte Allen. The URL was confirmed malicious via reputation analysis.

**Recommended defensive actions:**

- Block sender domain `m1crosoftsupport.co` at the email gateway
- Add `m1crosoftsupport.co` and `102.89.222.143` to the organization's threat intelligence blocklist
- Quarantine the email from `c.allen@thetrydaily.thm`'s inbox
- Notify Charlotte Allen with phishing awareness guidance — specifically around typosquatted domains
- No endpoint investigation required as no evidence of link interaction was found

---

## Typosquatting — Analyst Note

This lab is a good example of **typosquatting**, one of the most common phishing techniques used to impersonate major brands. The substitution of the letter "i" with the number "1" in `m1crosoftsupport` is subtle enough that a distracted user reading quickly could easily miss it — especially when combined with an urgent subject line.

**Quick detection checklist for typosquatting:**
- Does the sender domain exactly match the legitimate company domain?
- Are any letters replaced with visually similar numbers (i→1, o→0)?
- Does the TLD match what the company actually uses (.com, .org, etc.)?
- Does the embedded URL domain match the sender domain?

Verifying these four things takes under 30 seconds and catches the majority of brand impersonation phishing attempts.

---

## Key Takeaways

- **Typosquatting is subtle but detectable** — character substitution is one of the first things to check when a sender claims to be a major brand
- **URL reputation tools are essential** — confirming the link as malicious before any action removes ambiguity and supports the true positive classification
- **Not every true positive needs escalation** — if there's no evidence of user interaction or endpoint compromise, containment at the email gateway level is sufficient
- **Fear-based lures are extremely common** — "Unusual Sign-In Activity," "Your account has been compromised," and "Action Required" subject lines should always trigger extra scrutiny

---

*Write-up by Trystan Ruiz | TryHackMe SOC Simulator | Blue Team Portfolio*
