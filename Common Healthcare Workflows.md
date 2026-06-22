# Common Healthcare Workflows

> Understanding clinical workflows helps you build better integrations. Here are the workflows you'll encounter most.

> **Why this page matters:** Healthcare integrations almost always model a real-world workflow — a patient walking in, a doctor placing an order, a lab returning a result. If you understand the workflow, the messages and APIs make sense. If you don't, the technical specs look arbitrary and confusing. Read this page once to ground every other page in this wiki.

> **Universal analogy:** Each workflow below is essentially an **event-driven pipeline**. A real-world event happens (patient admitted, lab ordered, claim filed), one system publishes a message, other systems react. Mentally, replace "HL7v2 message" with "webhook payload" and "EHR" with "your monolith" — that's the shape of every workflow on this page.

## 1. Patient Registration / ADT Flow

```
Patient arrives → Registration → Admit (A01) → Transfer (A02) → Discharge (A03)
                      |
                 Demographics entered
                 Insurance verified
                 MRN assigned
```

**Messages generated:** ADT^A04 (register), ADT^A01 (admit), ADT^A02 (transfer), ADT^A03 (discharge), ADT^A08 (update)

**Why it matters:** ADT messages are the most common HL7v2 messages. Every downstream system needs to know where the patient is.

> **Analogy:** ADT messages are like **user lifecycle events** in a SaaS app: `user.created`, `user.updated`, `user.deleted`. Except instead of a user signing up, it's a patient being admitted. And instead of a webhook to Stripe, it's an HL7v2 message to the billing system, the lab, the pharmacy, the bed management system, and 15 other services — all simultaneously.

---

## 2. Lab Order → Result Flow

```
Doctor orders lab test
    |
    v (ORM^O01 order message)
Lab Information System (LIS) receives order
    |
    v
Lab tech collects specimen, runs test
    |
    v (ORU^R01 result message)
EHR receives result, displays to doctor
    |
    v
Doctor reviews and signs off
```

**Messages:** ORM^O01 (order), ORU^R01 (result)
**Key:** Results go through states: Preliminary → Final → Corrected → Cancelled

> **Analogy:** This is like an **async job queue**. The doctor submits a job (order), the lab processes it asynchronously, and when done, publishes the result back. The status transitions (Preliminary → Final → Corrected) are like job states in Sidekiq or Celery: `queued → processing → completed → amended`.

---

## 3. Referral Workflow

```
PCP decides patient needs specialist
    |
    v
PCP creates referral in EHR
    |
    v
CCDA Referral Note generated
    |
    v (via Direct, HIE, or fax)
Specialist receives referral + clinical summary
    |
    v
Specialist sees patient
    |
    v
Specialist sends Consultation Note back to PCP
```

**Tech:** CCDA documents via [[Direct Messaging Protocol|Direct]], [[Health Information Exchange (HIE)|HIE]], or [[Kno2]]. Increasingly FHIR Task resources.

> **Analogy:** A referral is like **transferring a customer ticket between two SaaS companies**. Your PCP "files a ticket" with notes attached, sends it to a specialist (different company, different system), and the specialist eventually closes the loop with their findings. The CCDA is the ticket attachment carrying patient history; the Direct/HIE/Kno2 layer is the transport that lets two unrelated CRMs talk securely.

---

## 4. Prescription / e-Prescribing

```
Doctor orders medication in EHR
    |
    v (NCPDP SCRIPT message)
e-Prescribing network (Surescripts)
    |
    v
Pharmacy receives prescription
    |
    v
Pharmacy dispenses, sends confirmation back
```

**Standard:** NCPDP SCRIPT (not HL7v2)
**Network:** Surescripts (the dominant e-prescribing network)

> **Analogy:** e-Prescribing is like **Stripe Connect for medications**. The doctor's EHR doesn't talk to each pharmacy directly — they all route through Surescripts, the same way merchants don't talk to each bank directly but route through Stripe. One integration with the network, reach every pharmacy in the country. NCPDP SCRIPT is the message format Surescripts speaks — like Stripe's API but for prescriptions.

---

## 5. Insurance Eligibility Check

```
Patient arrives for visit
    |
    v
Front desk runs eligibility check
    |
    v (X12 270 - Eligibility Inquiry)
Payer/Clearinghouse
    |
    v (X12 271 - Eligibility Response)
System shows: active coverage, copay amount, deductible status
```

**Standard:** X12 270/271
**Via:** Clearinghouses (Change Healthcare, Availity, Trizetto)

> **Analogy:** Eligibility checks are like a **real-time API call to verify a credit card before charging**. Before serving the patient, you ping the payer ("does this card work? what's the limit?") and get back coverage status, copay, and deductible. The 270 is your request, the 271 is the response. The clearinghouse is the payment processor sitting between you and dozens of insurance companies, normalizing the API.

---

## 6. Claims / Billing Flow

```
Patient visit completed
    |
    v
Coder assigns diagnosis (ICD-10) and procedure (CPT) codes
    |
    v (X12 837 - Claim)
Clearinghouse → Payer (insurance company)
    |
    v (X12 835 - Remittance/Payment)
Payment posted to practice management system
    |
    v
Patient billed for remaining balance
```

**Standards:** X12 837 (claim), X12 835 (remittance), X12 277 (claim status)

> **Analogy:** Claims billing is like **submitting an expense report to a strict reimbursement system**. The coder turns the visit into line items (ICD-10 diagnoses + CPT procedures), submits via 837, and waits for approval. The 835 is the response — "we paid X for line A, denied line B because of reason code Y." X12 itself is a fixed-width format from the 1980s, the kind of thing where every field is at a specific character position. Treat it like reading a mainframe wire transfer.

---

## 7. Care Coordination / Transitions of Care

```
Patient discharged from Hospital A
    |
    v
Discharge Summary (CCDA) generated
    |
    v (via HIE, Direct, Care Everywhere)
PCP's EHR receives notification + documents
    |
    v
PCP schedules follow-up visit
    |
    v
Care manager follows up with patient
```

**Tech:** ADT notifications via [[Health Information Exchange (HIE)|HIE]], CCDA documents, FHIR notifications

> **Analogy:** Care coordination is like **observability for a customer journey across multiple companies**. The PCP needs to know when "their" patient hits another vendor's product (the ER, a specialist, a hospital). ADT alerts are essentially webhooks: "patient X just appeared at Hospital Y." The CCDA is the attached context payload. Without this, every provider operates in a silo and the patient becomes the courier carrying their own history.

---

## 8. Public Health Reporting

```
Lab identifies reportable condition (e.g., COVID, TB)
    |
    v (Electronic Lab Report - ELR)
State/local public health department
    |
    v
Case investigation
```

**Reporting types:**
- **ELR** (Electronic Lab Reporting) — HL7v2 or FHIR
- **eCR** (Electronic Case Reporting) — FHIR-based, automated
- **Immunization registries** — HL7v2 VXU messages
- **Syndromic surveillance** — ADT data to monitor disease patterns

> **Analogy:** Public health reporting is **legally-mandated telemetry**. Like how a fintech must auto-report suspicious transactions to FinCEN, a lab must auto-report certain conditions (COVID, TB, measles) to the health department. You don't ask permission, you just emit the event. The state aggregates these into outbreak dashboards.

---

## Key Roles You'll Hear About

| Role | What They Do |
|------|-------------|
| **Provider / Clinician** | Doctor, NP, PA who sees patients and documents care |
| **Nurse** | Clinical staff managing patient care |
| **Coder** | Assigns ICD-10/CPT codes for billing |
| **Registrar** | Registers patients, enters demographics |
| **HIM** (Health Info Management) | Manages medical records, release of information |
| **Care Manager** | Coordinates care across settings, follows up with patients |
| **Analyst** | Configures and builds within the EHR |
| **Interface Analyst** | Builds and maintains HL7v2/FHIR integrations (this might be you) |

---

**See also:** [[HL7v2 Essentials]], [[Healthcare Data Standards]], [[Integration Patterns]]
