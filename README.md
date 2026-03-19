# Chart Review Copilot — Release Readiness Framework

> **Author:** Bhavana Adarsha, Principal AI PM | IRIS Digi LLC  
> **Version:** v1.0 | March 2026  
> **Product:** LLM-powered clinical chart summarization, delivered via Epic CDS Hooks (patient-view)  
> **Tech Stack:** Azure OpenAI GPT-4 (BAA) · FHIR R4 · Epic SMART on FHIR · CDS Hooks

---

## Overview

This framework defines the gates, metrics, governance, and pilot design required to take the Chart Review Copilot from alpha testing through General Availability (GA) in a health system environment.

Clinical AI releases carry patient safety implications that conventional software releases do not. An incorrect drug dosage in a generated summary, a hallucinated lab result, or a fabricated allergy omission are not bugs to fix in a hotpatch — they are potential patient harm events. Every exit gate in this framework is tied to a patient safety or clinical trust threshold, not a feature checklist.

---

## What the Product Does

When a physician, NP, PA, or RN opens a patient chart in Epic, the Chart Review Copilot:

1. Retrieves structured patient data via FHIR R4 (medications, active problems, recent labs, allergies, last 3 encounter notes)
2. Sends this context — de-identified of direct identifiers before the LLM call — to GPT-4 via Azure OpenAI (HIPAA BAA in place)
3. Returns a structured CDS Card: **Summary** (patient snapshot in plain language), **Flags** (abnormal labs, drug interactions, overdue preventive care), and a **Confidence indicator**
4. Allows the clinician to accept, edit, or dismiss the summary before it touches the clinical record

---

## Release Milestone Map

| Stage | Duration | Users | Environment | Primary Goal |
|-------|----------|-------|-------------|--------------|
| **ALPHA** | 4 weeks | 3–5 internal clinical informaticists | Sandbox EHR; no real patient data | Validate model output quality; identify failure modes |
| **BETA** | 6 weeks | 8–15 clinical champions (volunteer clinicians) | Staging; de-identified real patient data | Measure clinician edit rate and trust baseline |
| **PILOT** | 12 weeks | 40–60 clinicians across 2–3 units | Production; real data; read-only output | Clinical validation; safety signal detection; governance evidence |
| **GA** | Ongoing | Full health system rollout | Production; write-back with clinician attestation | Scaled adoption; continuous monitoring; model governance |

**No stage may be skipped. A failed exit gate requires remediation before re-evaluation.**

---

## ALPHA — Exit Criteria

> *Internal sandbox validation. Goal: understand failure modes before any patient data exposure.*

| Criterion | Threshold |
|-----------|-----------|
| Hallucination Rate | < 5% on 100-case curated eval set |
| FHIR Retrieval Accuracy | 100% — all critical fields present (allergies, active meds, problem list) |
| Structured Output Compliance | > 95% of responses match required card schema |
| Critical Omission Rate | **Zero** omissions of allergy or medication data |
| Latency (p95) | < 4 seconds from chart open to card render |
| PHI Leak to LLM | Zero incidents of direct identifier in prompt payload (n=200 sample) |

**Note:** The 5% hallucination threshold at Alpha is intentionally lenient — this stage is about discovery, not certification. The model will hallucinate; the goal is to understand where and why.

---

## BETA — Exit Criteria

> *Clinical champion validation in staging. Goal: measure trust and edit rate baselines with real de-identified charts.*

| Criterion | Threshold |
|-----------|-----------|
| Hallucination Rate | < 2% on 300-case eval set |
| Clinician Edit Rate | < 40% of accepted summaries require substantive edits |
| Clinician Trust Score | > 3.5 / 5.0 — "I trust this summary to inform my clinical decision" |
| Critical Error Rate | **Zero** hallucinations of medication dose, allergy status, or diagnosis |
| NPS | > 0 (more promoters than detractors among Beta users) |
| IRB / Ethics Clearance | Written IRB determination (approval or exemption) received |
| BAA Executed | BAA with Azure OpenAI signed and on file |

**Feedback mechanisms:** In-app thumbs up/down · Weekly 5-question survey · Bi-weekly group debrief · Edit log capture

---

## PILOT — Design & Exit Criteria

> *Controlled production deployment. Read-only mode. Goal: generate clinical evidence for governance sign-off.*

### Pilot Design

| Parameter | Detail |
|-----------|--------|
| **Duration** | 12 weeks (2 weeks onboarding/baseline; 8 weeks active; 2 weeks analysis) |
| **User Target** | 40–60 clinicians across 2–3 units (recommended: inpatient medicine + outpatient primary care + 1 specialty clinic) |
| **Patient Volume** | Minimum 1,000 unique charts reviewed with copilot active |
| **Deployment Mode** | Read-only: AI output is advisory only; no write-back to clinical record |
| **Comparison Group** | Within-subject: weeks 1–2 baseline without copilot, then active use |
| **Adverse Event Protocol** | Any incident where a clinician acts on a hallucinated summary → report to CISO and CMO within 24 hours; copilot suspended pending RCA |
| **Washout Option** | If hallucination rate > 2% at week 6 review → automatic 2-week suspension for model remediation |

### Pilot Feedback Collection

| Type | Method | Frequency |
|------|--------|-----------|
| Quantitative metrics | Automated telemetry (edit rate, dismissal rate, session time) | Real-time |
| Clinical trust survey | 5-item Likert survey in Epic after each session | Weekly |
| Adverse event log | Voluntary incident report form; escalation path to CMO | Continuous |
| Qualitative interviews | 30-min semi-structured interviews with 10 clinicians | Weeks 6 and 12 |
| Edit analysis | Structured diff analysis of all substantive edits | Bi-weekly |
| Time-to-ready | Epic timestamp delta from chart open to clinician ready | Automated weekly |

### Pilot Exit Criteria

| Criterion | Threshold |
|-----------|-----------|
| Hallucination Rate | < 1% on 1,000-case Pilot eval set (weekly sampled QA) |
| Critical Error Rate | **Zero** medication dose, allergy, or diagnosis hallucinations in full set |
| Clinician Edit Rate | < 30% substantive edits (improvement from Beta) |
| Clinician Trust Score | > 4.0 / 5.0 at week 12 exit survey |
| Documentation Time Savings | Clinician-reported savings ≥ 5 minutes per chart review session |
| Adverse Event Rate | **Zero** patient safety events attributable to copilot |
| Model Drift Baseline | Hallucination rate variance < 0.5% week-over-week |
| Governance Report | Pilot Report submitted to CMO / Clinical AI Committee |
| Legal / Compliance | HIPAA compliance review complete; Azure DUA renewed |

---

## GA — Post-Launch Monitoring

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| Hallucination Rate | < 1% | Suspend if > 2% in any week |
| Clinician Edit Rate | Trending toward < 25% | Investigate if > 40% for 2+ consecutive months |
| Card Dismissal Rate | < 35% | Investigate if > 50% for 2+ weeks |
| Adverse Event Reports | **Zero** | Any event → immediate pause + RCA |
| Model Drift | < 0.5% week-over-week variance | Retrain trigger if 3 consecutive rising weeks |
| System Uptime | > 99.5%; p95 latency < 4s | Page on-call if uptime < 99% or p95 > 6s |

**Write-back enabled at GA.** Clinicians can insert accepted summaries directly into Epic note fields via attestation click.

---

## Evaluation Metrics — Definitions

### Hallucination Rate
- **Definition:** Any claim in the generated summary that cannot be verified in the source FHIR data retrieved at generation time. Includes invented values, fabricated entries, misattributed data, and critical omissions.
- **Formula:** `(Hallucinated claims) / (Total claims in evaluated summaries) × 100%`
- **Evaluation:** Two independent clinical informaticists review each flagged claim. Both must agree for a claim to be coded as a hallucination.
- **Sample sizes:** Alpha: n=100 | Beta: n=300 | Pilot: n=100/week sampled

### Clinician Edit Rate
- **Definition:** % of accepted summaries that a clinician modifies in a substantive way before using or attesting to the content.
- **Formula:** `(Accepted summaries with ≥1 substantive edit) / (Total accepted summaries) × 100%`
- **Substantive edit:** Changes clinical meaning (e.g., changing a value, adding/removing a diagnosis). Typo fixes and reformatting are excluded.
- **Interpretation:** Rate trending **down** = trust improving. Rate trending **up** = model drift. Rate flat and high = fundamental quality problem requiring retraining.

### Factual Accuracy Against Ground Truth
- **Definition:** Measured during Alpha and Beta using a curated corpus where a clinical informaticist has written the correct reference summary.
- **Formula:** `(Correct claims) / (Total claims in ground truth) × 100%`
- **Complements hallucination rate:** Hallucination rate catches fabricated claims. Factual accuracy also catches omissions — critical data present in the chart but missing from the summary.

---

## Governance Sign-Off Checklist (Pre-GA)

All items must be checked with documented evidence before advancing to GA. Any unchecked item is a blocker.

### Section A: Patient Safety & Clinical Validation
- [ ] Hallucination rate < 1% confirmed on Pilot eval set (n=1,000) — *Clinical Informatics Lead*
- [ ] Zero critical errors (medication dose, allergy, diagnosis) in full Pilot dataset — *CMO*
- [ ] No adverse patient safety events attributable to copilot during Pilot — *CMO*
- [ ] Clinician edit rate < 30% at Pilot week 12 — *AI PM*
- [ ] Clinician trust score ≥ 4.0 / 5.0 at Pilot exit survey — *AI PM*
- [ ] Ground truth eval corpus reviewed and approved by clinical informaticist — *Clinical Informatics Lead*
- [ ] Model behavior reviewed for demographic bias (age, race, sex, insurance type) — *ML Engineer + CMO*

### Section B: Legal, Compliance & Privacy
- [ ] BAA with Azure OpenAI executed and current — *Legal / Compliance Officer*
- [ ] HIPAA Risk Assessment completed for production deployment — *CISO*
- [ ] IRB determination letter received (approval or exemption) — *Research / Compliance*
- [ ] De-identification layer validated: zero direct identifiers in LLM prompt logs (n=500 audit sample) — *CISO + ML Engineer*
- [ ] Data use agreement current for all FHIR data sources — *Legal*
- [ ] AI transparency disclosure prepared for patients (if required by state law) — *Legal + CMO*
- [ ] Staff training on PHI obligations in AI-assisted workflows complete — *CISO + HR*

### Section C: Technical & Operational Readiness
- [ ] Load tested: p95 latency < 4s at 2× expected concurrent users — *ML Engineer / DevOps*
- [ ] Uptime SLA confirmed (99.5%); monitoring and alerting configured — *DevOps*
- [ ] Rollback procedure documented and tested in staging — *ML Engineer*
- [ ] Model version pinned; no auto-updates without change management review — *ML Engineer*
- [ ] Audit logging active: every LLM call logged with prompt hash, user ID, timestamp (7-year retention) — *ML Engineer + CISO*
- [ ] Adverse event escalation path tested end-to-end — *AI PM + CMO office*
- [ ] Epic CDS Hooks integration smoke tested in production — *ML Engineer + Epic team*

### Section D: Clinical Governance & Communication
- [ ] Pilot Report reviewed and approved by Clinical AI Committee — *Committee Chair*
- [ ] CMO formal sign-off on clinical appropriateness of GA deployment — *CMO*
- [ ] Clinician training complete: appropriate reliance and override behaviors — *Clinical Informatics + AI PM*
- [ ] Clinician-facing disclosure shown in Epic UI at first use; acknowledgment logged — *Legal + Clinical Informatics*
- [ ] Communication plan sent to clinical leadership — *AI PM*
- [ ] Model monitoring dashboard reviewed by Clinical AI Committee — *AI PM*
- [ ] Maintenance and deprecation policy communicated to all clinical users — *AI PM + Clinical Informatics*

---

## Document Info

| | |
|--|--|
| **Author** | Bhavana Adarsha, Principal AI PM |
| **Company** | IRIS Digi LLC |
| **Portfolio** | [bhavana-adarsha.github.io](https://bhavana-adarsha.github.io) |
| **GitHub** | [github.com/bhavana-adarsha](https://github.com/bhavana-adarsha) |
| **LinkedIn** | [linkedin.com/in/bhav515](https://linkedin.com/in/bhav515) |
| **Version** | v1.0 — March 2026 |

---

*This document is a portfolio artifact demonstrating Principal-level AI Product Management competency in clinical AI release governance, safety metric design, and regulated product deployment.*
