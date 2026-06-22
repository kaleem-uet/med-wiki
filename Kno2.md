# Kno2

> Kno2 is a cloud-based **health data exchange platform** focused on document exchange, Direct messaging, and interoperability. Think of it as a managed service for connecting your healthcare application to the broader health data network.

> **Beginner primer:** If you've heard "Twilio for SMS" or "Stripe for payments," Kno2 is the same idea for clinical document exchange. Behind their REST API is a mess of Direct certificates, HIE connections, fax gateways, and TEFCA paperwork — they hide all of it. You don't need to deeply understand the underlying protocols to ship; you just need to know which endpoint to hit and what payload to send. This page is mostly about *what they handle for you* and *when you'd choose them vs. doing it yourself*.

## What Kno2 Does

Kno2 provides:
1. **Direct Messaging** — Send and receive clinical documents via the [[Direct Messaging Protocol]]
2. **Document Exchange** — Route CCDA documents, PDFs, and other clinical documents
3. **Fax Digitization** — Convert incoming faxes to structured digital data (yes, healthcare still uses fax heavily)
4. **FHIR APIs** — Modern API access for sending/retrieving data
5. **Network Connectivity** — Connected to national networks including Carequality, CommonWell, and TEFCA (Kno2 is a designated QHIN)

---

## Why You'd Use Kno2

### The Problem
You're building a healthcare app and need to:
- Send clinical documents to other providers
- Receive referrals and records from external organizations
- Connect to national health data networks
- Handle fax (unfortunately still a thing)

### Without Kno2
You'd need to:
- Set up your own Direct messaging infrastructure (HISP — Health Information Service Provider)
- Build and maintain Carequality/CommonWell connections
- Handle certificate management, trust anchors, etc.
- Build fax infrastructure

### With Kno2
You integrate with Kno2's API, and they handle the network connectivity, certificate management, and routing.

> **Analogy:** Kno2 is to healthcare document exchange what **Twilio is to SMS/voice**. You don't build your own SMS infrastructure — you call Twilio's API and they handle carrier connections, phone number provisioning, and delivery. Similarly, you call Kno2's API and they handle Direct messaging certs, TEFCA compliance, Carequality connections, and even fax.

---

## Kno2 as a QHIN

> **Analogy:** Becoming a QHIN is like **becoming a Tier 1 internet backbone provider**. There are only ~8 in the country, the bar is high (capital, governance, technical certification), and once you're in, you peer with all the others. Smaller organizations connect to a QHIN the way a small ISP connects to a backbone — one upstream connection, global reach.

Kno2 is a designated **QHIN** (Qualified Health Information Network) under [[QHIN and TEFCA|TEFCA]]. This means:
- Organizations connecting through Kno2 can exchange data with other QHINs nationwide
- Kno2 handles the TEFCA compliance, legal agreements, and technical requirements
- Smaller organizations that can't become QHINs themselves can participate in TEFCA through Kno2

---

## Key Concepts

### Direct Address
A Direct address looks like an email: `doctor.smith@hospital.direct.kno2fy.com`

But it's not regular email — it uses the [[Direct Messaging Protocol|Direct]] protocol with encryption, certificates, and trust verification.

> **Real-world example:** Your app needs to send a patient's referral letter to a specialist at another hospital. You POST a CCDA document to Kno2's API with the specialist's Direct address. Kno2 encrypts it, routes it through the Direct network, and the specialist's EHR receives it — all without you managing certificates or SMTP servers.

### Document Types Kno2 Handles

> **Analogy:** Kno2 normalizes document types the way **Cloudinary normalizes images** — you POST whatever you have (CCDA XML, scanned PDF, an actual fax), and they hand you a clean representation on the other side. The fax-to-data feature is the surprising one: yes, faxes still flow through healthcare, and yes, Kno2 will OCR them into structured data for you.

| Type | Format | Example |
|------|--------|---------|
| CCDA | XML | Continuity of Care Document, Referral Note |
| PDF | Binary | Scanned documents, reports |
| Fax | Image → data | Incoming faxes converted to structured data |
| FHIR | JSON | Modern resource-based exchange |

### Kno2 Integration Methods
| Method | Use Case |
|--------|----------|
| **REST API** | Programmatic send/receive of documents |
| **FHIR API** | FHIR-native document exchange |
| **Direct** | Send/receive via Direct addresses |
| **Kno2 Portal** | Web UI for manual document management |
| **EHR Integrations** | Pre-built connectors for some EHR systems |

---

## Developer Integration

### Typical Flow: Sending a Document

```
Your App  →  Kno2 API (POST document)  →  Kno2 Platform  →  Recipient
                                              |
                                              Routes via:
                                              - Direct Protocol
                                              - Carequality
                                              - TEFCA/QHIN network
```

### Typical Flow: Receiving a Document

```
External Provider → Direct/Carequality/TEFCA → Kno2 Platform → Webhook to Your App
                                                                   |
                                                              Parse CCDA/PDF,
                                                              extract data,
                                                              file in your system
```

### Typical Flow: Querying for a Patient's Records (PD → DQ → DR)

When you want to **pull** a patient's records from other organizations (instead of waiting for a push), you run the IHE-standard three-step query workflow. Every QHIN and HIE — including Kno2 — exposes some version of this.

| Step | Acronym | IHE Profile | What It Does | Returns |
|------|---------|-------------|--------------|---------|
| 1 | **PD** | XCPD (Cross-Community Patient Discovery) | "Does anyone on the network have this patient?" | Match candidates + a remote patient ID per source |
| 2 | **DQ** | XCA Query | "List all documents you have for that patient" | Metadata only (doc IDs, types, dates) — not the docs |
| 3 | **DR** | XCA Retrieve | "Send me that specific document" | The actual CCDA / PDF |

> **Analogy:** Think of it like **interacting with a remote git repo you've never cloned**: PD is `git ls-remote` (does this exist?), DQ is `git log` (what's in there?), DR is `git fetch <ref>` (give me the actual content). You can't skip a step — each one returns the handle the next one needs.

#### How to do PD/DQ/DR through Kno2

Kno2 wraps all three IHE profiles behind a REST API, so you call clean HTTPS endpoints instead of speaking SOAP/XCPD directly. The actual endpoint names depend on Kno2's API version — the shape below is illustrative:

```http
# 1. Patient Discovery — find the patient across the network
POST /patient-discovery
{
  "givenName": "Jane",
  "familyName": "Doe",
  "birthDate": "1985-03-15",
  "gender": "F",
  "address": { "postalCode": "10001" }
}
→ 200 OK
{
  "matches": [
    { "remotePatientId": "abc-123", "source": "HealthixHIE", "score": 0.92 }
  ]
}

# 2. Document Query — list docs for the matched patient
GET /document-query?patientId=abc-123&source=HealthixHIE
→ 200 OK
{
  "documents": [
    { "id": "doc-1", "type": "CCD",               "date": "2025-04-12" },
    { "id": "doc-2", "type": "Discharge Summary", "date": "2025-05-20" }
  ]
}

# 3. Document Retrieve — pull the actual document content
GET /document-retrieve?documentId=doc-1&source=HealthixHIE
→ 200 OK  (application/xml)
<ClinicalDocument ...>...</ClinicalDocument>
```

> **Gotcha — patient matching is fuzzy.** PD may return zero, one, or several candidates. Most platforms surface a confidence score; pick a threshold (e.g. 0.9+) before auto-using a match, otherwise surface ambiguity to a human. Acting on a wrong match means you've pulled the wrong person's chart — a HIPAA incident.

---

## When to Use Kno2 vs Alternatives

| Need | Kno2 | Alternative |
|------|------|-------------|
| Send/receive clinical documents | Good fit | Build own HISP (complex) |
| Connect to national networks | Good fit (QHIN) | Join Carequality/CommonWell directly (expensive, complex) |
| FHIR-to-FHIR exchange | Can do it | Direct FHIR APIs between systems |
| High-volume HL7v2 messaging | Not primary use case | [[Mirth Connect]], Rhapsody |
| Fax replacement | Good fit | Direct only covers digital-to-digital |

---

**See also:** [[Direct Messaging Protocol]], [[QHIN and TEFCA]], [[Health Information Exchange (HIE)]]
