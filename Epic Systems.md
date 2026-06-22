# Epic Systems

> Epic is the **dominant EHR** (Electronic Health Record) in the US. It serves ~38% of the US population's health records. If you're doing healthcare integration, you will almost certainly work with Epic.

> **Beginner primer:** When healthcare folks say "the EHR," they usually mean Epic. It's the iPhone of hospital software — not because it's slick, but because it's everywhere and sets the de-facto standard. Most of your career in healthcare integration will involve Epic in some way: reading data from it, writing into it, or building apps that launch from inside it. If you only learn one EHR, learn this one.

## What Epic Is

Epic is a comprehensive healthcare software platform covering:
- **EHR** (electronic health records) — the clinical charting system doctors use
- **MyChart** — the patient portal (the app patients use to see results, message doctors)
- **Revenue cycle** — billing, claims, coding
- **Scheduling** — appointment management
- **Pharmacy** — medication management
- **Radiology, Lab, OR** — departmental modules

Epic is **on-premise** (installed at each hospital) though they offer a hosted option. Each hospital has its own Epic instance with its own configuration.

> **Analogy:** Epic is like **Salesforce for hospitals** — but instead of being cloud-native, imagine every customer ran their own Salesforce instance on their own servers, with heavy customization. That's why integrating with "Epic" really means integrating with "Hospital X's version of Epic," which may differ from Hospital Y's.

---

## Integrating with Epic (Developer Perspective)

### Epic's Integration Options

| Method | Use Case | Format |
|--------|----------|--------|
| **Epic FHIR APIs** | Patient apps, third-party apps, data access | FHIR R4 (and DSTU2 on older systems) |
| **Epic on FHIR (Open.Epic)** | Public-facing FHIR APIs for registered apps | FHIR R4 |
| **Bridges/BCA** | Backend clinical data access (custom) | Proprietary Epic web services |
| **HL7v2 interfaces** | Traditional integration (ADT, labs, orders) | HL7v2 over MLLP or TCP |
| **Care Everywhere** | Epic-to-Epic and Carequality data sharing | IHE profiles (XCA/XCPD) + FHIR |
| **Epic App Orchard / Showroom** | Marketplace for Epic-integrated apps | SMART on FHIR, APIs |
| **Interconnect** | Epic's middleware/integration layer | Various |

### Epic FHIR APIs (What You'll Use Most)

Epic provides FHIR R4 APIs. Two main types:

#### 1. Patient-Facing (MyChart)
- Patient launches your app from MyChart
- Uses SMART on FHIR launch flow
- Patient authenticates with their MyChart credentials
- Your app gets a scoped access token
- Can read patient's own data

#### 2. Backend/System (EHR)
- System-to-system integration
- Uses SMART Backend Services (client credentials with JWT)
- No user interaction needed
- Used for bulk data, population health, automated workflows

### Getting Access to Epic APIs

1. **Register on open.epic.com** — Create a developer account
2. **Create an app** — Define scopes, redirect URIs, FHIR resources needed
3. **Test in sandbox** — Epic provides a sandbox with synthetic data (fhir.epic.com)
4. **Get approved** — Submit for review. Epic reviews your app.
5. **Deploy** — Each customer (hospital) must individually approve your app connection

**Important:** Just because Epic approves your app doesn't mean hospitals will enable it. Each hospital's Epic admin must grant access. This is a sales/business process, not just a technical one.

> **Analogy:** Getting an app into Epic is like getting approved on the **App Store AND then needing each enterprise customer to individually whitelist your app** in their MDM. Apple (Epic) says your app is fine, but each company (hospital) still needs to approve it through their own IT governance. Expect months per hospital.

### Epic App Orchard / Showroom
Epic's marketplace for third-party apps. If you want broad distribution across Epic hospitals, you list your app here. Requires Epic review and certification.

> **Analogy:** App Orchard / Showroom is **the App Store, but for hospital apps**. Same model: developers publish, Epic reviews, hospitals browse the marketplace and install. The submission bar is high (security review, clinical safety review, technical certification), and approval is slow (months, not days). But once you're listed, you're discoverable across thousands of Epic customers.

---

## Key Epic Concepts for Developers

### FHIR IDs and Identifiers
- Epic's internal patient ID format: `T1234` or `eABC123` (varies by instance)
- **FHIR ID** — Epic-assigned, unique within that Epic instance
- **MRN** (Medical Record Number) — Hospital-assigned, found in `Patient.identifier`
- **FHIR ID is NOT portable** — Patient 123 at Hospital A's Epic is completely different from Patient 123 at Hospital B's Epic

> **Analogy:** It's like how a user's `id` in your PostgreSQL database is meaningless to another company's database. User #42 in your system is not User #42 in theirs. The MRN is closer to an email address — a more meaningful identifier, but even MRNs differ per hospital.

### MyChart vs EHR Access
- **MyChart APIs** = patient-authorized, limited scope, patient sees/consents
- **EHR APIs** = clinical/backend, broader access, needs hospital IT approval
- Different OAuth flows, different token scopes, different data access

> **Analogy:** Think of it as **"Sign in with Google" (personal Gmail) vs "Sign in with Google Workspace" (company-managed account)**. MyChart access is the consumer flow — the patient logs in themselves and consents to share their own data. EHR access is the enterprise flow — the hospital's IT department configures and approves system-level access, just like a Workspace admin would approve an integration. Different identity providers, different consent flows, different scopes.

### Epic's Proprietary Elements
- **Epic Web Services** — SOAP-based APIs, older but still used
- **Chronicles** — Epic's proprietary database (Intersystems Caché/IRIS)
- **BestPractice Alerts (BPA)** — Clinical decision support alerts in the EHR
- **SmartData Elements (SDE)** — Epic's discrete data capture fields
- **MyChart Bedside** — Inpatient patient-facing tablet app

### Care Everywhere

> **Analogy:** Care Everywhere is essentially **iMessage between Epic hospitals**. If both ends are on Epic, records flow seamlessly with high fidelity — same protocols, same patient-matching, same UI. If you reach off-network to a non-Epic system, it works but feels like falling back to SMS: lower fidelity, more friction. Epic uses Carequality and TEFCA as the bridges to "non-Epic SMS."

Epic's network for sharing records between Epic hospitals (and non-Epic systems via Carequality).

- When a patient arrives, the hospital can query Care Everywhere to pull records from other facilities
- Uses IHE profiles (XCPD for patient discovery, XCA for document retrieve)
- Also connects to national networks via Carequality and now TEFCA
- As a developer, you typically don't build Care Everywhere interfaces directly — it's built into Epic

---

## Other Major EHR Vendors (For Context)

| Vendor | Market | Notes |
|--------|--------|-------|
| **Epic** | Large hospitals, academic medical centers | Dominant. ~38% market share by patient records |
| **Oracle Health (Cerner)** | Large hospitals, VA | Acquired by Oracle. Second largest. FHIR APIs available. |
| **Meditech** | Community hospitals | Common in smaller hospitals. FHIR support growing. |
| **Athenahealth** | Ambulatory/outpatient | Cloud-based. Strong in physician practices. |
| **Allscripts/Veradigm** | Mixed | Being split/restructured |
| **eClinicalWorks** | Ambulatory | Large ambulatory install base |
| **NextGen Healthcare** | Ambulatory | Specialty practices |

Each has its own API approach, but FHIR R4 is becoming the common denominator thanks to federal mandates.

---

## Developer Tips

1. **Start at open.epic.com** — Register, explore the documentation, spin up the sandbox
2. **Understand the two-step access model** — Epic approval + individual hospital approval
3. **Test with synthetic data** — Never use real patient data in development
4. **Epic's FHIR is US Core compliant** — But they have extensions. Read their resource documentation carefully.
5. **Version matters** — Hospitals upgrade Epic at different times. One might be on Feb 2024 version, another on Aug 2024. API availability can differ.
6. **Expect long timelines** — Getting integrated with Epic at a real hospital can take months (security reviews, legal, IT approvals)

---

**See also:** [[FHIR Deep Dive]], [[Healthcare Authentication]], [[EHR Landscape]]
