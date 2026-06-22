# Healthcare Data Standards

> **Beginner primer:** Healthcare has roughly four "data languages," and each is used for a specific job. You don't pick one — you'll touch all of them depending on which system you're integrating with.
>
> | Standard | Job | One-Line Mental Model |
> |----------|-----|----------------------|
> | **[[HL7v2 Essentials\|HL7v2]]** | Internal hospital events | CSV-over-TCP from 1987 |
> | **[[FHIR Deep Dive\|FHIR]]** | Modern APIs | Government-mandated REST/JSON |
> | **[[CCDA]]** | Cross-org documents | XML doc with both narrative and structured data |
> | **X12 EDI** | Insurance billing | Fixed-width mainframe wire format |
>
> The rest of this page expands on each. If you're starting out, focus on FHIR first (where the industry is going), HL7v2 second (where the industry currently lives), and learn the others as needed.

## The Problem

Healthcare is deeply fragmented. A patient might have records across 10+ systems that don't natively communicate. Your job as a developer is to bridge these systems.

> **Analogy:** Imagine if every bank used a different file format, a different API, and a different definition of "account balance" — and there was no Plaid or Stripe to normalize it. That's healthcare. You're basically building the Stripe for medical data, one integration at a time.

**Why it's hard:**
- Legacy systems from the 90s/2000s are still running production
- Multiple competing standards exist simultaneously
- Vendor lock-in — Epic, Cerner, etc. each add proprietary layers
- Heavy regulation ([[HIPAA for Developers|HIPAA]])
- Medical data is messy — free text, inconsistent coding, missing fields

---

## The Standards You'll Actually Encounter

### Message-Based (Point-to-Point)

> **Analogy:** Think of HL7v2 like CSV files sent over a TCP socket — old-school, no schema validation, but every system speaks it. HL7v3 was like the SOAP/XML overhaul that nobody wanted to adopt.

| Standard | Format | Era | Status | Use Case |
|----------|--------|-----|--------|----------|
| **[[HL7v2 Essentials\|HL7 v2]]** | Pipe-delimited text | 1987 | Still dominant | Lab results, ADT (admit/discharge/transfer), orders |
| **HL7 v3 / RIM** | XML | 2005 | Mostly dead | Avoid if possible. Overly complex. |

### Document-Based (Clinical Documents)

| Standard | Format | Use Case |
|----------|--------|----------|
| **CDA** (Clinical Document Architecture) | XML | Generic clinical documents |
| **[[CCDA]]** (Consolidated CDA) | XML (constrained CDA) | Care summaries, discharge notes, referral notes. The standard for document exchange in the US. |

### API-Based (Modern)

> **Analogy:** FHIR is to healthcare what GraphQL/REST was to web development — a clean, modern API layer designed to replace the legacy mess. Imagine if the government mandated that every backend must expose a REST API following a specific OpenAPI spec. That's FHIR.

| Standard | Format | Use Case |
|----------|--------|----------|
| **[[FHIR Deep Dive\|FHIR]]** (R4) | JSON/XML over REST | The modern standard. Federally mandated for patient access. |

### Billing / Administrative

> **Analogy:** X12 EDI is like a mainframe-era wire transfer format — fixed-width, positional, cryptic codes. Imagine building a payment system where every field is at a specific character position in a flat file. Welcome to healthcare billing.

| Standard | Format | Use Case |
|----------|--------|----------|
| **X12 EDI** | Fixed-width positional | Insurance claims (837), eligibility (270/271), remittance (835) |
| **NCPDP** | Specialized | Pharmacy claims |

---

## How They Relate

```
Patient Visit
    |
    v
EHR System (Epic, Cerner)
    |
    ├── HL7v2 messages  → Internal systems (labs, pharmacy, radiology)
    ├── CCDA documents  → Other organizations via HIE or Direct
    ├── FHIR API        → Patient apps, third-party apps, payer systems
    └── X12 EDI         → Insurance companies for billing
```

## The Trend

Everything is slowly moving toward **FHIR**. Federal regulation (21st Century Cures Act) is accelerating this. But HL7v2 isn't going anywhere for internal hospital messaging — it's too entrenched.

**As a developer, prioritize learning:**
1. **FHIR** — for anything new you're building
2. **HL7v2** — for integrating with existing hospital systems
3. **CCDA** — for document exchange between organizations

---

**See also:** [[FHIR Deep Dive]], [[HL7v2 Essentials]], [[Integration Patterns]]
