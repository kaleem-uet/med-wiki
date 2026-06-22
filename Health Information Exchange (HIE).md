# Health Information Exchange (HIE)

> **Beginner primer:** Picture every hospital, clinic, and lab as a separate company with its own database — none of them talk to each other by default. An HIE is the **shared switchboard** that lets them share patient records on demand. There are dozens of HIEs (some state-run, some private, some Epic-only), and the big federal push is to make them all interconnect via [[QHIN and TEFCA|TEFCA]] so a doctor in Florida can pull records from a hospital in Oregon without thinking about it.

## What Is an HIE?

An HIE (Health Information Exchange) is both:
1. **The act** of electronically sharing health data between organizations
2. **The organization/network** that facilitates that sharing

Think of an HIE as a **regional network hub** that connects hospitals, clinics, labs, pharmacies, and payers so they can share patient data.

> **Analogy:** An HIE is like a **CDN for patient records**. Just as Cloudflare sits between servers and users to cache/route content efficiently, an HIE sits between healthcare organizations to route and share patient data. Without it, every hospital would need a direct VPN to every other hospital.

---

## Why HIEs Exist

Without an HIE:
```
Hospital A ←→ Hospital B      (custom interface)
Hospital A ←→ Lab C            (another custom interface)
Hospital A ←→ Clinic D         (yet another)
Hospital B ←→ Lab C            (and another)
... N*(N-1)/2 point-to-point connections
```

With an HIE:
```
Hospital A ──→ HIE ←── Hospital B
Lab C      ──→ HIE ←── Clinic D
Pharmacy E ──→ HIE ←── Payer F

Everyone connects once to the HIE. The HIE routes data.
```

---

## Three Models of HIE

### 1. Centralized (Data Warehouse)
- HIE stores a **copy** of all patient data in a central repository
- Query the HIE directly to find patient records
- **Pro:** Fast queries, data always available
- **Con:** Data freshness issues, large storage costs, security concerns
- Example: Some state HIEs

> **Analogy:** Like Google indexing the web — they crawl and copy everything into their own servers, so searches are fast. But the copy might be stale.

### 2. Federated (Query-Based)
- Data stays at the source. HIE routes **queries** to member systems
- You ask: "Do you have records for Patient X?" and each system responds
- **Pro:** Data stays current, organizations keep control
- **Con:** Slower, depends on all systems being online
- Example: Carequality network, eHealth Exchange

> **Analogy:** Like a DNS lookup or a microservices gateway — the HIE doesn't store the data, it just knows where to ask. Each request fans out to source systems in real-time.

### 3. Hybrid
- Mix of both. Common demographics in central index, clinical data stays federated
- Most modern HIEs use this approach

---

## What Data Flows Through an HIE

> **Analogy:** Think of an HIE as **a shared message bus that hospitals subscribe to**. Hospitals publish events (admissions, lab results, document updates) and other hospitals subscribe to the ones relevant to their patients. Some payloads are events (ADT messages), some are documents (CCDAs), some are query/response (FHIR APIs). The HIE handles routing, identity matching, and access control — like a Kafka cluster with consent management bolted on.

| Data Type | Format | Example |
|-----------|--------|---------|
| Patient demographics | ADT messages (HL7v2) | Name, DOB, MRN, address |
| Clinical summaries | CCDA documents | Care summary, discharge summary |
| Lab results | HL7v2 ORU, FHIR Observations | Blood work, pathology |
| Imaging reports | HL7v2, FHIR DiagnosticReport | Radiology reads |
| Notifications | ADT events | "Your patient was admitted to Hospital X" |
| Public health data | HL7v2, FHIR | Immunizations, reportable conditions |

---

## Key HIE Concepts

### Master Patient Index (MPI)
The HIE's system for matching patients across organizations. Patient "John Smith DOB 1990-01-01" at Hospital A needs to be matched to the same person at Clinic B — even if they have different MRNs.

MPI matching uses algorithms based on: name, DOB, gender, SSN (last 4), address, phone.

> **Analogy:** It's the same problem as **user deduplication** in a CRM. "J. Smith" at one source, "John A. Smith" at another, "Jonathan Smith" at a third. Are they the same person? MPI algorithms use fuzzy matching + scoring (like Levenshtein distance + rule-based weights) to decide. Getting this wrong has real consequences — merging two different patients' records can cause medication errors.

### Consent Management
Patients may need to opt-in or opt-out of HIE participation. Rules vary by state:
- **Opt-in states:** Patient must explicitly agree to share data
- **Opt-out states:** Data is shared by default, patient can opt out
- This is a common source of complexity

> **Analogy:** Consent management is **GDPR consent banners, but with 50 different state-level cookie laws layered on top**. The same patient's data may be shareable in one state and locked down in another. Every query has to check the consent rules in the patient's state of origin. Engineers underestimate this constantly; it's where most "we shipped to production and got sued" stories come from.

### Event Notifications (ADT Alerts)
One of the most valuable HIE services. When your patient shows up at another facility, you get notified. Used heavily for care coordination.

> **Real-world example:** A primary care doctor has 2,000 patients. One of them, Maria, goes to the ER at 2am. The HIE sends an ADT alert to Maria's PCP's system: "Your patient was admitted to St. Mary's Hospital." The next morning, the care coordinator sees the alert and calls Maria to schedule a follow-up. Without this, the PCP would never know Maria was in the ER until her next visit months later.

---

## Major HIE Networks and Frameworks

> **Analogy:** The HIE landscape is **the email/instant-messaging landscape circa 2000**. Lots of independent networks (AOL, Yahoo, MSN, ICQ), each with their own membership and protocols, slowly being forced to interoperate. TEFCA is the federal push to make them all talk to each other. As a developer, the practical question is "which network is my customer hospital on?" — that decides who you have to integrate with first.

| Network | Scope | Notes |
|---------|-------|-------|
| **State/Regional HIEs** | State or metro area | See [detail section below](#major-stateregional-hies-in-detail). Examples: Healthix (NY), CRISP (MD), IHIE (IN). |
| **Carequality** | National framework | Enables HIE-to-HIE connections. Not a QHIN itself — a separate Sequoia Project framework. |
| **CommonWell Health Alliance** | National network | Founded by EHR vendors (not Epic). Partners with Carequality. |
| **Epic Care Everywhere** | Epic-to-Epic | Epic's own network. Connects all Epic hospitals. Massive. |
| **eHealth Exchange** | National | One of the oldest networks. Federal agencies participate. |
| **[[QHIN and TEFCA\|TEFCA / QHINs]]** | National framework | The new federal framework to connect all networks together |

### How They Interconnect

```
TEFCA (National Framework)
  └── QHINs (Qualified Health Information Networks)
        ├── eHealth Exchange ←→ connects to many HIEs and federal agencies
        │     ├── State HIE 1
        │     ├── State HIE 2
        │     └── Carequality participants (via eHealth Exchange)
        ├── Epic Nexus  ←→ all Epic hospitals
        ├── CommonWell  ←→ non-Epic EHRs (Cerner/athena/etc.)
        ├── KONZA, MedAllies, Health Gorilla, Kno2, Oracle Health
        └── (8 designated QHINs total — see [[QHIN and TEFCA]])
```

---

## Major State/Regional HIEs (in Detail)

Beyond the national QHINs, dozens of state and regional HIEs operate as sub-national networks. Most connect into TEFCA through a QHIN rather than running their own. The handful below are the ones a US developer is most likely to encounter:

### 1. Healthix (New York)
- **Region:** New York City, Long Island, and lower NY state. The largest single-region HIE in the US (~22M patient records).
- **Example:** A Manhattan ED queries Healthix on patient arrival and instantly sees prior visits across multiple unrelated hospital systems in the city — without setting up a single point-to-point integration.

### 2. CRISP (Maryland, DC, West Virginia)
- **Region:** State-mandated HIE for Maryland; expanded into DC and West Virginia. Often cited as the model US state HIE.
- **Example:** A Maryland hospital admits a patient — within seconds, CRISP fires an ADT notification to the patient's PCP and surfaces their controlled-substance prescription history (CRISP runs the state PDMP).

### 3. Indiana Health Information Exchange (IHIE)
- **Region:** Indiana statewide. One of the oldest HIEs in the US (founded in the 1990s), now serving most major Indiana health systems.
- **Example:** A small clinic in Bloomington pulls a patient's discharge summary from a hospital in Indianapolis via IHIE — no bilateral interface required.

### 4. Manifest MedEx (California)
- **Region:** California's largest nonprofit HIE, formed from the merger of Cal INDEX and Inland Empire HIE.
- **Example:** A telehealth visit in Los Angeles fetches the patient's recent Northern California lab results through Manifest MedEx in real time during intake.

### 5. MiHIN (Michigan Health Information Network)
- **Region:** Michigan statewide. State-designated entity, strong public-health and care-coordination integrations.
- **Example:** A patient discharged from a Detroit hospital triggers a MiHIN ADT notification that fans out to their PCP, care manager, and home-health agency statewide.

### How State/Regional HIEs Connect to TEFCA

Most regional HIEs don't run their own QHIN — they connect to TEFCA through one of the 8 designated QHINs. Common pathways:

- A state HIE joins **eHealth Exchange** as a Participant — by far the most common route.
- Some partner with **KONZA** (which itself started as the Kansas state HIE) as a sub-QHIN.
- Epic-heavy regions effectively reach TEFCA through **Epic Nexus** instead of a separate HIE connection.

> **Note:** Many state HIEs that existed in 2014–2018 have shut down or consolidated due to funding pressure (the federal HITECH grants that launched most of them expired). The list above reflects the consolidated 2025 landscape.

---

## Developer Perspective

### What You'll Build
- **Interfaces** to send/receive data from HIEs (usually HL7v2 or FHIR)
- **Patient matching** logic or MPI integrations
- **Consent management** workflows
- **Notification handlers** for ADT alerts
- **Document retrieval** services (query HIE for patient records)

### Common Integration Patterns
1. **Push:** Your system sends ADT/lab/documents to the HIE automatically
2. **Pull/Query:** Your system queries the HIE for patient records on demand
3. **Subscribe:** Register for notifications about specific patients or events

### Technologies You'll Use
- [[Mirth Connect]] or similar integration engine to handle HL7v2/FHIR transformations
- [[Direct Messaging Protocol]] for secure document delivery
- FHIR APIs for modern HIE connections
- CCDA for document exchange
- IHE profiles (XDS, XCA, XCPD) for document sharing — federated HIE queries use the **PD → DQ → DR** sequence (Patient Discovery → Document Query → Document Retrieve). See [[Kno2]] for a worked PD/DQ/DR example.

---

**See also:** [[QHIN and TEFCA]], [[Integration Patterns]], [[Direct Messaging Protocol]]
