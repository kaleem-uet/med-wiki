# QHIN and TEFCA

## TEFCA — Trusted Exchange Framework and Common Agreement

### What It Is
TEFCA is a **national framework** created by the ONC (Office of the National Coordinator for Health IT) to enable nationwide health data exchange.

> **Analogy:** TEFCA is to health networks what **TCP/IP + peering agreements** are to the internet. Before the internet, networks (CompuServe, AOL, university nets) couldn't talk to each other. Internet protocols + peering rules connected them all. TEFCA does the same thing for health data networks — it's the set of rules that lets Carequality talk to CommonWell talk to Epic's network, etc.

### The Problem TEFCA Solves
Before TEFCA, health data networks were siloed:
- Epic Care Everywhere only connects Epic hospitals
- CommonWell connects non-Epic EHRs
- State HIEs serve their region
- Carequality connects some of these, but not all

**Result:** No universal way to query for a patient's records across all networks.

### How TEFCA Works

```
Organization A                              Organization B
(Hospital, Clinic, etc.)                    (Hospital, Clinic, etc.)
     |                                            |
     v                                            v
   QHIN 1  ←── TEFCA Common Agreement ──→  QHIN 2
     (e.g., Carequality)                   (e.g., eHealth Exchange)
```

TEFCA establishes:
1. **Common Agreement** — Legal/technical rules all participants follow
2. **Standard Operating Procedures (SOPs)** — How exchanges happen
3. **QHINs** — The designated networks that implement the framework

### TEFCA Exchange Purposes (What You Can Do)
| Purpose | Description |
|---------|-------------|
| **Treatment** | Query/retrieve records for patient care |
| **Individual Access Services (IAS)** | Patient requests their own data |
| **Payment/Operations** | Insurance and administrative uses |
| **Public Health** | Reporting to public health agencies |
| **Government Benefits Determination** | Eligibility for government programs |

---

## QHIN — Qualified Health Information Network

### What It Is
A QHIN is an organization that has been **approved/designated** under TEFCA to serve as a national on-ramp. QHINs connect to each other and route queries between their member organizations.

> **Analogy:** QHINs are like **Tier 1 ISPs** (AT&T, Comcast, Verizon at the backbone level). They peer with each other and provide connectivity to smaller networks (regional HIEs, hospitals). You don't connect directly to the internet backbone — you connect through an ISP. Similarly, you don't connect directly to TEFCA — you connect through a QHIN.

> **Real-world example:** A small rural clinic in Kansas can't afford to build connections to every hospital network in the country. Instead, they connect to KONZA (a QHIN), and through KONZA they can query patient records from any other QHIN — Epic Nexus, Carequality, CommonWell, etc. One connection, nationwide reach.

### Designated QHINs (as of 2025)
| QHIN | Background |
|------|-----------|
| **Carequality** | Sequoia Project. Already connects many HIEs and Epic. |
| **eHealth Exchange** | The Sequoia Project. One of the oldest national networks. |
| **CommonWell Health Alliance** | Founded by Cerner, Athenahealth, others. |
| **KONZA** | Kansas-based. Serves rural and critical access hospitals. |
| **Epic Nexus** | Epic's QHIN for Epic-connected organizations. |
| **MedAllies** | Focuses on Direct messaging and smaller practices. |
| **Health Gorilla** | Clinical data network, strong in lab/diagnostics. |
| **Kno2** | Document exchange platform (see [[Kno2]]). |

### How a QHIN Works

```
Your App / EHR
     |
     v
Your Organization (Participant)
     |
     v
Sub-QHIN (optional intermediary, like a state HIE)
     |
     v
QHIN (e.g., Carequality)
     |
     v  ← TEFCA Common Agreement governs this connection
Other QHINs
     |
     v
Other Organizations nationwide
```

### Developer Implications
- You typically **don't connect directly to a QHIN** — you connect through a **Participant** or **Sub-Participant** that is part of a QHIN
- QHINs handle identity proofing, patient matching across networks, and routing
- Data exchanged follows **USCDI** (United States Core Data for Interoperability) — a defined set of data elements
- Technically, exchanges use **IHE profiles** (XCPD for patient discovery, XCA for document query/retrieve) and increasingly **FHIR**

---

## USCDI — United States Core Data for Interoperability

The **minimum dataset** that must be exchangeable under TEFCA and ONC regulations.

> **Analogy:** USCDI is like a **database migration that the government mandates**. "Every system must have these columns in these tables." If your FHIR server can't return a patient's allergies, medications, and lab results in the standardized format, you're not compliant.

### USCDI v1 (baseline)
- Patient name, DOB, sex, race, ethnicity, address, phone, email
- Allergies and intolerances
- Assessment and plan of treatment
- Conditions (diagnoses)
- Immunizations
- Lab results
- Medications
- Procedures
- Vital signs
- Smoking status
- Clinical notes (consultation, discharge, history & physical, progress, etc.)

### USCDI v3/v4 (expanding)
Adds: health insurance, sexual orientation, gender identity, SDOH (social determinants of health), tribal affiliation, treatment interventions, and more.

**As a developer:** If you're building FHIR APIs, you need to support the USCDI data classes. US Core FHIR profiles map directly to USCDI.

---

## Key Acronyms in This Space

| Acronym | Full Name | What It Is |
|---------|-----------|-----------|
| TEFCA | Trusted Exchange Framework and Common Agreement | The national interoperability framework |
| QHIN | Qualified Health Information Network | Designated national exchange networks |
| RCE | Recognized Coordinating Entity | The Sequoia Project — administers TEFCA |
| USCDI | US Core Data for Interoperability | The minimum data standard |
| ONC | Office of the National Coordinator | Federal agency overseeing health IT |
| IAS | Individual Access Services | Patient right to retrieve their own data |

---

**See also:** [[Health Information Exchange (HIE)]], [[FHIR Deep Dive]], [[Healthcare Glossary]]
