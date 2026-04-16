# Chart Review Copilot — Release Readiness Framework
 
> **Author:** Bhavana Adarsha, Principal AI PM | IRIS Digi LLC
> **Version:** v2.0 | April 2026
> **Product:** LLM-powered clinical chart summarization, delivered via Epic CDS Hooks (patient-view)
> **Tech Stack:** Azure OpenAI GPT-4 (BAA) · FHIR R4 · Epic SMART on FHIR · CDS Hooks
 
---
 
## Overview
 
This framework defines the gates, metrics, governance, pilot design, and adoption strategy required to take the Chart Review Copilot from alpha testing through General Availability (GA) and sustained use in a health system environment.
 
Clinical AI releases carry patient safety implications that conventional software releases do not. An incorrect drug dosage in a generated summary, a hallucinated lab result, or a fabricated allergy omission are not bugs to fix in a hotpatch they are potential patient harm events. Every exit gate in this framework is tied to a patient safety or clinical trust threshold, not a feature checklist.
 
A technically sound release that clinicians do not use is not a successful release. Adoption is treated as a first-class outcome alongside safety and quality.
 
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
 
## Adoption Plan
 
> A technically sound release that clinicians do not use is not a successful release. This section defines how we train, onboard, and measure sustained adoption — meeting clinicians in their existing workflow rather than asking them to learn a new one.
 
### The Adoption Principle
 
Clinicians are time-constrained professionals with low tolerance for tools that slow them down. The Chart Review Copilot earns adoption by proving its value inside the existing workflow — not by requiring behavior change upfront. The training strategy reflects this: we show clinicians how the Copilot fits into what they already do, and we let the time savings make the case.
 
---
 
### Champion Identification
 
Before GA, identify 3–5 clinical champions per unit — clinicians who participated in the Pilot and showed consistent engagement and positive trust scores. Champions serve as:
 
- Peer advocates during unit-level rollout
- First responders for questions from new users
- Feedback conduits back to the product team
- Voices in clinical leadership communications
Champions are not power users chosen for technical enthusiasm. They are trusted peers chosen for credibility with their colleagues.
 
---
 
### Training Plan
 
Training is delivered in three layers. Each layer is designed to fit inside the clinician's existing schedule without requiring dedicated training time.
 
**Layer 1 — Shadow Session (15 minutes, one-time)**
 
Before a clinician uses the Copilot independently, a champion or clinical informatics lead sits with them during a normal chart review session. The clinician follows their existing workflow. The champion shows the Copilot card alongside — no pressure to use it, just observation.
 
Goal: Remove the unknown. The clinician sees exactly what appears, when it appears, and what it asks of them.
 
**Layer 2 — Side-by-Side Comparison (first two weeks of use)**
 
During the clinician's first two weeks of active use, they continue their normal chart review process and use the Copilot in parallel. They are asked one question at the end of each session:
 
> *"Did the Copilot summary save you time compared to your usual review? If yes, roughly how many minutes?"*
 
This is captured via a single in-app prompt — one tap, no forms.
 
Goal: Build personal evidence of time savings. Clinicians who see their own time savings data become self-motivated adopters.
 
**Layer 3 — Micro-Reference Card (always available)**
 
A single-page quick reference card is embedded in the Epic UI and available as a printed card at workstations. It covers:
 
- What the Copilot does and does not do
- How to read the confidence indicator
- How to accept, edit, or dismiss a summary
- How to report a concern
Goal: Eliminate the need to remember training. Everything a clinician needs is one click away.
 
---
 
### Time Savings Communication
 
Time savings data collected during the Pilot is shared with clinical leadership and unit champions before GA rollout. The message is kept concrete and personal:
 
> *"During the 12-week Pilot, clinicians reported saving an average of X minutes per chart review. For a clinician reviewing 15 charts per shift, that is Y minutes per shift returned to patient care."*
 
This framing — time returned to patient care, not efficiency gained for the health system — resonates with clinicians and creates pull rather than push adoption.
 
---
 
### Adoption Metrics
 
| Metric | Definition | Target at 90 Days Post-GA |
|--------|------------|--------------------------|
| Activation Rate | % of onboarded clinicians who use the Copilot at least once in first 7 days | > 80% |
| Weekly Active Rate | % of onboarded clinicians using the Copilot at least once per week | > 60% |
| Card Acceptance Rate | % of Copilot summaries accepted (vs dismissed) | > 65% |
| Time-to-First-Use | Days from access granted to first Copilot use | < 3 days median |
| Self-Reported Time Savings | Clinician-reported minutes saved per chart review session | ≥ 5 minutes |
| Champion Engagement | % of champions who conducted at least 2 shadow sessions in first 30 days | 100% |
| Feedback Submission Rate | % of weekly active users who submit at least one feedback response per month | > 40% |
 
---
 
### Feedback Loop Back to Product
 
Adoption without a feedback loop is a one-way broadcast. Every piece of clinician feedback — edit logs, survey responses, champion debriefs — feeds back to the product roadmap on a defined cadence:
 
| Cadence | Activity |
|---------|----------|
| Weekly | Edit log review — identify recurring edits that signal model gaps |
| Bi-weekly | Champion debrief — qualitative themes from peer conversations |
| Monthly | Adoption metrics review with clinical leadership |
| Quarterly | Roadmap update communicated to all active users |
 
Clinicians who see their feedback reflected in product updates become long-term advocates. Closing the loop is not a courtesy — it is a retention strategy.
 
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
### Section E: Adoption Readiness
- [ ] Clinical champions identified and briefed for each rollout unit — *AI PM*
- [ ] Shadow session schedule confirmed with champions — *AI PM + Clinical Informatics*
- [ ] Micro-reference card finalized and embedded in Epic UI — *AI PM + Epic team*
- [ ] Time savings data from Pilot compiled and shared with clinical leadership — *AI PM*
- [ ] Adoption metrics dashboard live and reviewed — *AI PM*
- [ ] Feedback loop cadence confirmed with product team — *AI PM*
---
 
## Document Info
 
| | |
|--|--|
| **Author** | Bhavana Adarsha, Principal AI PM |
| **Company** | IRIS Digi LLC |
| **Portfolio** | [bhavana-adarsha.github.io](https://bhavana-adarsha.github.io) |
| **GitHub** | [github.com/bhavana-adarsha](https://github.com/bhavana-adarsha) |
| **LinkedIn** | [linkedin.com/in/bhav515](https://linkedin.com/in/bhav515) |
| **Version** | v2.0 — April 2026 |
 
---
 
*This document is a portfolio artifact demonstrating Principal-level AI Product Management competency in clinical AI release governance, safety metric design, regulated product deployment, and adoption strategy.*
