# Phishing Email Analysis

> A SOC-style analysis of two phishing emails — email-header forensics, IOC extraction, and MITRE ATT&CK mapping. Samples are simulated for safe study.

## Overview

I analyzed two phishing emails the way a Tier-1 SOC analyst would: inspecting the raw headers, verifying email authentication (SPF / DKIM / DMARC), checking sender reputation, comparing the visible brand against the actual sending domain, and examining the malicious link and attachment. Both messages are credential-harvesting attempts using brand impersonation and urgency — one a Microsoft 365 "unusual sign-in" link lure, the other a DHL "overdue invoice" with a malicious HTML attachment. For each I reached a verdict, listed recommended actions, extracted indicators of compromise, and mapped the activity to MITRE ATT&CK.

Because real phishing payloads are dangerous to distribute, the samples in this repo are **simulated**: links are non-functional, malicious domains are fictional, sender IPs use the RFC 5737 documentation ranges, and the attachment payload is redacted.

## Skills demonstrated

- Email-header forensics (Received chain, From vs. Reply-To, Return-Path)
- Email authentication analysis: SPF, DKIM, and DMARC alignment
- Phishing triage and verdict-setting (malicious / suspicious / benign)
- Indicator of Compromise (IOC) extraction and defanging
- Threat mapping with the MITRE ATT&CK framework
- Translating technical findings into prioritized defender actions

## Tools used

Raw email header analysis, MXToolbox (SPF/DKIM/DMARC), urlscan.io, VirusTotal, AbuseIPDB, MITRE ATT&CK Navigator

## What I did

1. Collected the raw `.eml` source for each email and read the full header set.
2. Verified SPF, DKIM, and DMARC results and checked alignment against the spoofed `From` brand.
3. Compared the display name and header `From` against the real envelope/sending domain and the `Reply-To`.
4. Analyzed the payload — the credential-harvesting link (Email 1) and the HTML attachment (Email 2) — and how each evades reputation checks.
5. Assigned a verdict and recommended containment/response actions per email.
6. Consolidated all IOCs and mapped the techniques to MITRE ATT&CK.

## Key findings

- **Malicious (Email 1) —** Microsoft 365 credential phish. Fails SPF, DKIM, and DMARC; `Reply-To` differs from `From`; "Verify my account" button points to a look-alike domain hosting a cloned login page that harvests credentials. Classic urgency + account-suspension threat.
- **Malicious (Email 2) —** DHL "overdue invoice" phish. SPF softfail, no DKIM, DMARC fail; sender domain is unrelated to DHL. Ships an `.htm` attachment that renders a fake Office 365 login locally and POSTs credentials to an attacker server — bypassing URL reputation because there is no link in the body.
- **Common thread —** Both rely on brand impersonation, urgency/threats, and authentication failures. Failed SPF/DKIM/DMARC was the single most decisive technical tell.

## Recommendations

- Enforce DMARC `p=reject` and block the listed sender domains and IPs at the secure email gateway.
- Strip or quarantine high-risk attachment types (`.htm` / `.html`), which are rarely legitimate.
- Brief finance/AP staff specifically on invoice and payment lures; verify charges through official portals, never via the email.
- On any click/credential entry: force password reset, revoke active sessions and tokens, and audit for new MFA methods or inbox rules.
- Run user-awareness training focused on urgency cues and sender/link mismatch.

## What I learned

The most useful takeaway was how much email authentication does the heavy lifting: once you read the `Authentication-Results` header, a convincing-looking message often falls apart immediately. The DHL sample also showed why attackers move from links to attachments — an HTML attachment sidesteps URL-reputation defenses entirely, so detection has to shift to the gateway and the endpoint. Writing the verdicts reinforced that good analysis isn't just "is this bad?" but "what exactly do we block, and what do we do if someone already clicked?"

## Files in this repo

- `report.pdf` — full analysis report (per-email breakdown, IOCs, MITRE ATT&CK mapping)
- `samples/phishing_sample_1_m365.eml` — simulated Microsoft 365 credential phish (malicious link)
- `samples/phishing_sample_2_invoice.eml` — simulated DHL invoice phish (malicious HTML attachment)

## Disclaimer / ethics

All samples in this repository are **simulated training material created for educational purposes**. They contain no working malicious links, no live payloads, and no real victim or attacker data. Domains are fictional, IP addresses use the RFC 5737 documentation ranges, and reputation figures and file hashes are illustrative placeholders. Nothing here is intended for, or usable in, an actual phishing campaign.

---

**Author:** Norbert Eigner — [LinkedIn](https://www.linkedin.com/in/norbert-eigner/)
