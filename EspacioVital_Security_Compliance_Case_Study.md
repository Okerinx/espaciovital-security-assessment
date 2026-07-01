# Security & Privacy Risk Assessment — EspacioVital

**A GRC case study on protecting special-category health data in a pre-launch digital health platform**

*Author: Oker Romero · Role: Security & Compliance (self-directed assessment) · Platform owner: RomeroWellness*

---

## 1. Executive Summary

EspacioVital is a pre-launch digital health service (HIV prevention and sexual health, Bucaramanga, Colombia) that currently operates a **public waitlist collecting personal contact data**. Because the service's purpose directly concerns a person's health and sexual life, *any* record in the waitlist constitutes **special-category (sensitive) personal data** under Colombian Law 1581 of 2012 — independent of how little contact information is stored.

This assessment treats the waitlist not as "a marketing email list" but as **a re-identifiable record of individuals associated with HIV prevention**, where a confidentiality breach could cause discrimination or real-world harm. I performed an asset and data-flow review, a qualitative risk assessment, a control-gap analysis mapped to ISO/IEC 27001:2022 Annex A, and a prioritized remediation roadmap suitable for a small WordPress-based platform before public launch.

**Headline finding:** the highest risk is not technical sophistication of an attacker — it is the combination of *sensitive data + a low-maturity CMS environment + no documented privacy controls* during the pre-launch window, when controls are easiest and cheapest to put in place.

---

## 2. Engagement Context & Scope

| Item | Detail |
|------|--------|
| Platform | EspacioVital (`espaciovital.lat`) |
| Stage | Pre-launch (waitlist active) |
| Technology | WordPress, Astra theme, cookie-consent plugin |
| In scope | Waitlist data collection, CMS administration, hosting, third-party processors, consent & privacy notices |
| Out of scope | Active exploitation/penetration testing (this is a defensive risk & compliance review of an owned asset) |
| Frameworks used | ISO/IEC 27001:2022 (Annex A), NIST CSF 2.0 (functions), Law 1581/2012 + Decree 1377/2013 (Colombia) |

---

## 3. System Overview & Data Flows

A visitor lands on the public page and submits contact details to join the waitlist. At a high level, the data flow is:

```
Visitor → Web form (WordPress) → [storage: DB and/or email notification] → Operator (wp-admin)
                                       �‍─ possible third-party form/email/analytics processors
```

**Key questions a GRC review must answer (data-flow mapping):**
- What fields are collected, and is each one *necessary* for a waitlist? (data minimization)
- Where do submissions land — database, admin email inbox, a third-party form service?
- Who can access wp-admin, and how is that access protected?
- Which third parties (hosting, cookie/analytics, form plugin) process the data, and is there a contract governing them?

The answers to these questions define the real attack surface and the compliance obligations.

---

## 4. Data Classification

| Data element | Classification | Why it matters |
|--------------|----------------|----------------|
| Contact identifier (email/phone/name) | **Sensitive (special-category)** *by association* | Linked to an HIV-prevention/sexual-health service → reveals health/sexual-life information about an identifiable person |
| Browsing/analytics metadata on the page | Potentially sensitive | A tracker firing on this specific page can leak the *sensitivity by association* (i.e., "this device visited an HIV-prevention page") |
| Operator credentials (wp-admin) | Confidential | Compromise exposes the entire waitlist |

**Core insight:** on most websites a name + email is ordinary PII. Here, the *context* upgrades it to special-category data. This single reframing drives the entire control set.

---

## 5. Regulatory & Legal Mapping

**Colombia — Law 1581 of 2012 & Decree 1377 of 2013 (primary):**
- Health and sexual-life data are explicitly **datos sensibles** (Art. 5).
- Processing sensitive data requires **explicit, informed, prior authorization**, and a data subject can never be *obliged* to provide sensitive data.
- A privacy notice (*aviso de privacidad*) and a data-processing policy must describe purpose, rights, and the channel to exercise them (habeas data).
- Verify current obligations regarding the **National Database Registry (RNBD)** with the SIC (Superintendencia de Industria y Comercio), as thresholds for who must register have changed over time.

**GDPR (only if any EU data subjects are reached):** this would be **Article 9 special-category data**, requiring an explicit lawful basis and heightened safeguards.

**Takeaway:** consent for this platform cannot be a single bundled checkbox. It must be **explicit and specific to the sensitive purpose**, separated from generic terms acceptance.

---

## 6. Threat & Risk Assessment (Risk Register)

*Qualitative scale: Likelihood (L) and Impact (I) rated Low / Medium / High.*

| ID | Risk | L | I | Rating | Primary concern |
|----|------|---|---|--------|-----------------|
| R1 | Disclosure of waitlist contacts (who is associated with HIV-prevention interest) | M | High | **High** | Confidentiality / human harm |
| R2 | Operator (wp-admin) account compromise via weak/single-factor auth | M | High | **High** | Full data exposure |
| R3 | Third-party processor (form/email/analytics) handles data without a contract/DPA | M | Medium | **Medium** | Loss of control over data |
| R4 | Consent collected is not explicit/specific for sensitive data | High | Medium | **High** | Legal non-compliance |
| R5 | No defined retention/deletion → data kept indefinitely | High | Medium | **Medium** | Excess exposure window |
| R6 | Unknown encryption-at-rest / backup location for the database | M | Medium | **Medium** | At-rest exposure |
| R7 | Analytics/marketing trackers on the sensitive page leak visit-by-association | M | Medium | **Medium** | Privacy leakage to third parties |
| R8 | No incident-response plan or breach-notification process | M | High | **Medium** | Slow, non-compliant breach handling |

---

## 7. Control Gap Analysis — ISO/IEC 27001:2022 Annex A

| Risk | Recommended control | Annex A reference |
|------|---------------------|-------------------|
| R1, R7 | Privacy & protection of PII; data minimization | A.5.34 |
| R2 | Secure authentication (MFA) + least-privilege admin roles | A.8.5, A.5.15 |
| R3 | Supplier/third-party information-security agreements (DPA) | A.5.19–A.5.21 |
| R4 | Documented, explicit, purpose-specific consent | A.5.34 + Law 1581 |
| R5 | Defined retention schedule & secure deletion | A.5.10, A.8.10 |
| R6 | Cryptography (TLS in transit + encryption at rest) & backup controls | A.8.24, A.8.13 |
| R8 | Incident management & breach notification process | A.5.24–A.5.26 |

---

## 8. Prioritized Remediation Roadmap

**Now (before launch / pre-public scale-up):**
- Enforce **MFA on all wp-admin accounts** and remove unused admin users (least privilege).
- **Minimize collected fields** — collect only what a waitlist needs (e.g., one contact channel), nothing extra.
- Replace bundled consent with an **explicit, specific authorization** for sensitive-health processing, separated from general terms.
- Publish a **privacy notice + data policy** that names sensitive-data handling, purpose, retention, and how to exercise habeas-data rights.

**Within 30 days:**
- Inventory **all third-party processors** (hosting, form, email, analytics) and put **data-processing agreements** in place; remove any non-essential trackers from the sensitive page (R7).
- Confirm **TLS everywhere** and **encryption at rest** for the database/backups; document backup location and access.
- Define a **retention & deletion schedule** and a process to honor deletion requests.

**Within 90 days:**
- Verify and, if required, complete **RNBD registration** with the SIC.
- Document a lightweight **incident-response & breach-notification plan** proportionate to the platform.
- Run a brief **pre-launch readiness review** confirming all High risks (R1, R2, R4) are mitigated to acceptable residual levels.

---

## 9. Residual Risk & Conclusion

With the "Now" and 30-day controls in place, the dominant High risks (waitlist disclosure, admin compromise, weak consent) drop to acceptable residual levels for a pre-launch service. The most cost-effective security decision available to EspacioVital is **timing**: implementing these controls *before* the database fills with sensitive records is dramatically cheaper and lower-risk than retrofitting them after a public launch.

This assessment demonstrates the GRC core loop end to end: **classify data → map legal obligations → assess risk → select controls against a recognized framework → produce a prioritized, business-aware remediation plan.**

---

### Skills demonstrated (note for reviewers)

- Sensitive-data classification and the *context-upgrades-PII* judgment that distinguishes a mature GRC analyst.
- Regulatory mapping across **Colombian Law 1581/2012** and **GDPR Art. 9**.
- Qualitative **risk assessment** and risk-register construction.
- **ISO/IEC 27001:2022 Annex A** control mapping.
- Pragmatic, **risk-prioritized remediation** scoped to a real small-business technology stack.
