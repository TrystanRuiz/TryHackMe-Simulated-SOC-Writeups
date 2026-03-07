# SOC Lab #01 — Inbound Email Containing Suspicious External Link

**Platform:** TryHackMe SOC Simulator
**Date:** February 1, 2026
**Outcome:** False Positive, closed no escalation

---

## Alert Details

| Field | Value |
|---|---|
| Event ID | 8814 |
| Alert Rule | Inbound Email Containing Suspicious External Link |
| Severity | Medium |
| Date Detected | February 1st, 2026 at 14:57 |
| Data Source | Email |

---

## Investigation

Pulled up Event ID 8814. Inbound email flagged for a suspicious external link.

Sender was `onboarding@hrconnex.thm`, recipient was `j.garcia@thetrydaily.thm`. Subject was "Action Required: Finalize Your Onboarding Profile." No attachment. The embedded URL had a user-specific path: `https://hrconnex.thm/onboarding/15400654060/j.garcia`.

![Case report for Event ID 8814](screenshots/thm1.png)

| Artifact | Value | Finding |
|---|---|---|
| Sender | onboarding@hrconnex.thm | HR onboarding platform |
| Recipient | j.garcia@thetrydaily.thm | Internal employee |
| Subject | Action Required: Finalize Your Onboarding Profile | Standard onboarding CTA |
| Attachment | None | |
| Embedded URL | https://hrconnex.thm/onboarding/15400654060/j.garcia | User-specific onboarding link |

Ran the URL through TryDetectThis.

![TryDetectThis showing SAFE](screenshots/thm3.png)

Came back clean.

![Incident report](screenshots/thm2.png)

---

## Verdict

False positive. Legitimate HR onboarding email sent to Julia Garcia. URL scanned clean, no indicators of malicious activity. Closed with rationale documented.

---

*Write-up by Trystan Ruiz*
