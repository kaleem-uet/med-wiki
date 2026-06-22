# EHR Landscape

> Overview of Electronic Health Record systems you'll encounter as a developer.

> **Beginner primer:** "EHR" is the umbrella term for the mega-application a hospital actually runs on. If you've ever worked at a company where one giant Salesforce/SAP/Workday install runs the whole business — that's the energy. Doctors live in the EHR all day; integrations are how everything else (your app, the lab system, the billing engine) talks to it. The market is dominated by a handful of vendors; you'll likely only deal with 2-3 of them in your career.

## What Is an EHR?

An EHR (Electronic Health Record) is the core software system that hospitals and clinics use to:
- Chart patient encounters (notes, diagnoses, orders)
- Manage medications and prescriptions
- View lab results and imaging
- Schedule appointments
- Handle billing and claims
- Communicate with patients (patient portals)

The EHR is the **system of record** for clinical data. Almost every integration you build connects to or from an EHR.

> **Analogy:** The EHR is like **the monolithic database** at the center of everything. In a typical tech company, your main PostgreSQL/MySQL database is the source of truth and everything reads from or writes to it. In a hospital, the EHR is that database — except it also has a massive UI, business logic, workflows, and a 30-year head start. You don't replace it; you integrate with it.

---

## Major EHR Vendors

### Tier 1: Large Hospital Market

| Vendor | Market Share | Key Facts |
|--------|-------------|-----------|
| **[[Epic Systems\|Epic]]** | ~38% of US patient records | Dominant in large hospitals and academic medical centers. Private company (Verona, WI). Not cloud-native. |
| **Oracle Health (Cerner)** | ~25% | Acquired by Oracle in 2022. Second largest. Powers the VA (US Veterans Affairs). Moving to Oracle Cloud. |
| **Meditech** | ~15% community hospitals | Strong in community and rural hospitals. Newer "Expanse" platform has FHIR. |

### Tier 2: Ambulatory / Outpatient

| Vendor | Market | Key Facts |
|--------|--------|-----------|
| **Athenahealth** | Physician practices | Cloud-based. Strong in billing/RCM. Good API platform. |
| **eClinicalWorks** | Ambulatory | Large install base. Controversial (past DOJ settlement). |
| **NextGen Healthcare** | Specialty practices | Strong in specialty-specific workflows. |
| **Veradigm (Allscripts)** | Mixed | Being restructured. Practice Fusion (free EHR) is a subsidiary. |
| **DrChrono** | Small practices | iPad-first EHR. Acquired by EverHealth. |

### Specialty / Niche

| Vendor | Specialty |
|--------|-----------|
| **Greenway Health** | Small practices |
| **ModMed** | Dermatology, ophthalmology, orthopedics |
| **AdvancedMD** | Independent practices |
| **CureMD** | Multi-specialty |

---

## Developer Considerations by Vendor

| Vendor | FHIR Support | Integration Approach | Developer Portal |
|--------|-------------|---------------------|-----------------|
| **Epic** | Strong (R4, US Core) | FHIR + proprietary APIs. App Orchard/Showroom marketplace. | open.epic.com |
| **Oracle Health** | Good (R4) | FHIR + Millennium APIs. Code Console. | fhir.cerner.com |
| **Meditech** | Growing (Expanse) | FHIR for Expanse. Older platforms need interfaces. | meditech.com |
| **Athenahealth** | Yes | REST APIs (proprietary + FHIR). Good developer experience. | developer.athenahealth.com |
| **eClinicalWorks** | Basic | FHIR available but limited. HL7v2 still primary. | — |

---

## Key EHR Concepts

### Modules

> **Analogy:** EHR modules are like the **product suites at a company like Atlassian** — Jira, Confluence, Bitbucket are separate products with separate licenses, but they share users, permissions, and integrate tightly. Same with Epic: a hospital might license Inpatient + ED + OR but skip Pharmacy. Each module is a separate sub-purchase, separately built and configured, but they live in the same database and user model.

EHRs are modular. A hospital might use:
- **Ambulatory** (outpatient/office visits)
- **Inpatient** (hospital stays)
- **ED** (emergency department)
- **OR** (operating room/surgical)
- **Pharmacy**
- **Radiology**
- **Lab**
- **Revenue Cycle / Billing**
- **Patient Portal** (MyChart for Epic, HealtheLife for Cerner)

### Build vs Buy

> **Analogy:** Think of it like **Salesforce admins vs developers**. Hospitals don't write the core EHR code (the vendor does), but they have an internal team of "builders" — really, configurators — who set up workflows, custom fields, alerts, and reports. The same EHR version at two hospitals can feel like two different products because the build teams configured them differently. This is why "we integrate with Epic" really means "we integrate with each hospital's specific Epic build."

Hospitals extensively customize their EHR. "Build" teams configure:
- Order sets (templates for common orders)
- Documentation templates
- Clinical decision support rules
- Workflows and navigation
- Reports and dashboards

This means **every hospital's EHR instance is different**, even if they use the same vendor.

### The "Go-Live"
Implementing a new EHR at a hospital is a massive project (often 1-3 years, hundreds of millions of dollars for large systems). Your integrations need to be part of this planning.

> **Analogy:** A Go-Live is **launching a rewrite of the company's most critical system on day one for every employee**. Imagine if Amazon migrated every warehouse, every fulfillment system, and every backend off the old stack onto a new one on a single Saturday morning — that's the scale. Your integration either works on Go-Live day or it gets cut from the launch. Hospitals do this once every 10-15 years; getting on the project plan early matters.

---

**See also:** [[Epic Systems]], [[FHIR Deep Dive]], [[Healthcare Data Standards]]
