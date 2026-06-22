# FHIR (Fast Healthcare Interoperability Resources)

> Pronounced "fire". The modern healthcare API standard. If you're building anything new in healthcare, you're almost certainly using FHIR.

> **Beginner primer:** If you've built a CRUD app with REST + JSON, you already understand 80% of FHIR. The other 20% is healthcare-specific: which resources exist, how they're searched, and which coding systems (SNOMED, LOINC, ICD-10) you use for values. Don't try to learn FHIR all at once — pick one resource (`Patient` is the easiest), get one EHR sandbox to talk to (HAPI public server or Epic sandbox), and read/write that one resource end-to-end. Add more resources from there.

## What Is FHIR?

FHIR is a **RESTful API specification** for healthcare data. Think: "What if healthcare data exchange worked like a modern web API?"

> **Analogy for web devs:** FHIR is like a **government-mandated OpenAPI spec** for healthcare. Imagine if every SaaS company was legally required to expose a standardized REST API with the same resource names, the same JSON structure, and the same search parameters. That's what FHIR does for hospitals.

- Created by **HL7 International** (same org behind HL7v2)
- Current production version: **R4** (Release 4) — this is what you target
- Uses **JSON** (or XML) over **HTTPS**
- Built around **Resources** (like database models/entities)

## Why It Matters (Legally)

The **21st Century Cures Act** and **CMS Interoperability Rules** require:
- Patients must access their health data via FHIR APIs
- Payers (insurance companies) must expose claims data via FHIR
- EHR vendors must provide FHIR endpoints
- **Information blocking** (refusing to share data) is now illegal

This is federal law, not a suggestion. That's why every major EHR now has FHIR APIs.

---

## Core Concepts

### Resources

Everything in FHIR is a **Resource**. If you've used Django models, Prisma schemas, or Mongoose models — Resources are the same concept. A `Patient` resource is like a `User` model, an `Observation` is like a `Measurement` record, etc.

Resources you'll work with most:

| Resource | What It Is | Example |
|----------|-----------|---------|
| `Patient` | A person receiving care | Name, DOB, MRN, address |
| `Practitioner` | A provider (doctor, nurse) | Name, NPI, specialty |
| `Encounter` | A visit or interaction | ER visit, office visit |
| `Condition` | A diagnosis | "Type 2 Diabetes" (ICD-10: E11) |
| `Observation` | A measurement | Lab result, blood pressure, heart rate |
| `MedicationRequest` | A prescription | "Metformin 500mg twice daily" |
| `AllergyIntolerance` | Known allergies | "Penicillin — causes rash" |
| `DocumentReference` | A clinical document | A CCDA, a PDF report |
| `DiagnosticReport` | Lab report | Groups multiple Observations |
| `Immunization` | Vaccination record | "COVID-19 vaccine dose 1" |
| `Procedure` | Something done to patient | Surgery, therapy |
| `Organization` | Hospital, clinic, lab | "Mayo Clinic" |
| `Coverage` | Insurance info | Plan, subscriber, group |
| `Bundle` | Collection of resources | Search results wrapper |

### RESTful API

> **Analogy:** If you've used the Stripe or GitHub REST APIs, FHIR is the same shape: nouns in URL paths, HTTP verbs do the work, JSON in/out. The difference is that *every* FHIR server in the world (Epic, Cerner, athena, your in-house one) uses the same resource names, the same field names, and the same search parameters — because they're all conforming to the same public spec.

Standard HTTP verbs, standard patterns:

```http
GET    /Patient/123                → Read one patient
GET    /Patient?name=smith         → Search patients
POST   /Patient                    → Create a patient
PUT    /Patient/123                → Update a patient
DELETE /Patient/123                → Delete a patient
GET    /Patient/123/_history       → Version history
```

### Search (This is where FHIR gets powerful)

> **Analogy:** FHIR search is like **GraphQL via query strings** — instead of `GET /users?name=smith`, you write `GET /Patient?family=Smith&_include=Patient:organization&_revinclude=Condition:patient`. The `_include` / `_revinclude` keywords let you join related resources in one request, the same way GraphQL fields traverse relationships. The price you pay: search parameter support is server-dependent — what works on HAPI may not work on Epic.

```http
# Basic search
GET /Patient?family=Smith&birthdate=1990-01-01

# Search with coding system (LOINC code for glucose)
GET /Observation?patient=123&code=http://loinc.org|2339-0

# Date ranges
GET /Observation?date=ge2024-01-01&date=le2024-12-31

# Pagination
GET /Patient?_count=10&_offset=0

# Include related resources in response
GET /Patient?_include=Patient:organization

# Reverse include
GET /Patient/123?_revinclude=Condition:patient
```

### Bundles (Response Wrapper)

> **Analogy:** A `Bundle` is like a **paginated API response wrapper** — the same `{ data: [], links: {...}, meta: {...} }` envelope you've seen in every REST API, just with FHIR-flavored naming. `entry` is your array of results, `link[rel=next]` is your next-page URL, `total` is the count. You almost never use a Bundle directly — you read it once to get the resources out, then work with the resources.

Search results come back in a `Bundle`:

```json
{
  "resourceType": "Bundle",
  "type": "searchset",
  "total": 1,
  "entry": [
    {
      "resource": {
        "resourceType": "Patient",
        "id": "123",
        "name": [{ "family": "Smith", "given": ["John"] }],
        "birthDate": "1990-01-01",
        "gender": "male"
      }
    }
  ]
}
```

### CodeableConcept (Important Pattern)

> **Analogy:** Imagine you have a `status` field that could be represented as an HTTP status code (404), a human-readable string ("Not Found"), AND a custom company code ("ERR_MISSING") — all at the same time. That's CodeableConcept. A single medical concept can be coded in multiple systems simultaneously.

Medical data uses coded values, not plain text. FHIR represents these as `CodeableConcept`:

```json
{
  "code": {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "73211009",
        "display": "Diabetes mellitus"
      },
      {
        "system": "http://hl7.org/fhir/sid/icd-10-cm",
        "code": "E11",
        "display": "Type 2 diabetes mellitus"
      }
    ],
    "text": "Type 2 Diabetes"
  }
}
```

A single concept can have codes from multiple systems (SNOMED, ICD-10, etc.).

---

## Profiles and Implementation Guides

Base FHIR is intentionally generic. **Profiles** constrain resources for specific contexts.

> **Analogy:** Base FHIR is like TypeScript's `any` type — it allows everything. A Profile is like defining a strict interface: "A US Core Patient MUST have a name, gender, and birthdate." Profiles turn loose FHIR into strict, validated contracts.

### US Core (Most Important for US Development)
Defines the minimum data EHRs must support. If you're building in the US, you build against US Core.

- US Core Patient — requires name, gender, birthdate, identifier
- US Core Observation — defines vitals, labs, social history
- US Core Condition — how to represent diagnoses

### Other Key IGs (Implementation Guides)
| IG | Purpose |
|----|---------|
| **SMART App Launch** | OAuth2 for FHIR apps — see [[Healthcare Authentication]] |
| **Bulk Data Access** | Async export of large datasets (population health, analytics) |
| **Da Vinci** | Payer-provider data exchange (prior auth, coverage, etc.) |
| **CARIN Blue Button** | Patient access to insurance claims |
| **International Patient Summary** | Cross-border patient summaries |

---

## FHIR Versions

> **Analogy:** FHIR versions are like **HTTP versions** — multiple are in the wild at once, mostly compatible, but with real gotchas. **R4** is the HTTP/1.1 of FHIR: the universal default, the version every tutorial assumes, and what every regulator points to. **DSTU2/STU3** are HTTP/1.0 territory — only seen in old Epic installs and slowly being retired. **R5** is HTTP/2 — published, technically better, but adoption is slow because the ecosystem is hard to move. **Target R4 unless someone tells you otherwise.**

| Version | Status | Notes |
|---------|--------|-------|
| DSTU2 | Legacy | Some older Epic installs still use this |
| STU3 | Legacy | Being phased out |
| **R4** | **Production standard** | Target this. It's normative (stable API). |
| R4B | Minor update to R4 | Small additions |
| R5 | Published | Limited production adoption so far |

---

## Developer Practical Guide

### Libraries

| Language | Library | Notes |
|----------|---------|-------|
| Python | `fhir.resources`, `fhirclient` | `fhir.resources` has Pydantic models |
| JavaScript/TS | `fhir-kit-client` | Good for Node.js backends |
| Java | **HAPI FHIR** | Gold standard. Also a full FHIR server. |
| .NET | `Hl7.Fhir.R4` | Official .NET SDK |

### Sandbox / Testing Environments

| Tool | What It Is |
|------|-----------|
| **HAPI FHIR Server** (hapi.fhir.org) | Public test server with synthetic data |
| **Inferno** | ONC's official FHIR conformance testing tool |
| **Epic Sandbox** (fhir.epic.com) | Test against Epic's FHIR implementation |
| **Cerner Sandbox** (fhir.cerner.com) | Test against Cerner's FHIR implementation |
| **Synthea** | Generates realistic synthetic patient data |
| **Postman** | Most vendors provide Postman collections |

### Common Gotchas

1. **FHIR is a spec, not a database** — Each server implements it differently. Search parameter support varies.
2. **`id` vs `identifier`** — `id` is the server's internal ID (changes per server). `identifier` is the business ID (like MRN — stays consistent). Think: `id` is like a database auto-increment primary key, `identifier` is like a user's email — it's the same everywhere.
3. **Extensions are everywhere** — FHIR allows custom extensions. Expect vendor-specific ones.
4. **Pagination is mandatory** — Large result sets are paged. Always handle `Bundle.link` with `relation: "next"`.
5. **Dates are tricky** — FHIR dates can be partial: `2024`, `2024-01`, `2024-01-15`. Handle all formats.
6. **References** — Resources reference each other: `"subject": {"reference": "Patient/123"}`. You often need to resolve these.
7. **Contained resources** — Sometimes resources are embedded inside others instead of referenced. Check for these.

---

## How FHIR Connects to Everything Else

```
HL7v2 messages ──→ Integration Engine (Mirth) ──→ FHIR Resources
CCDA documents ──→ FHIR Document conversion     ──→ FHIR Resources
X12 Claims     ──→ Da Vinci / CARIN IGs         ──→ FHIR Resources
```

FHIR is increasingly the **common API layer** that older standards get converted into.

---

**See also:** [[Healthcare Authentication]], [[Epic Systems]], [[HL7v2 Essentials]], [[Healthcare Glossary]]
