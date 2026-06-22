# Healthcare Integration Patterns

> Common architectural patterns you'll encounter when connecting healthcare systems.

> **Beginner primer:** Almost every "healthcare integration" job is one of the seven patterns below. They're not healthcare-specific — they're the same patterns you'd see in any enterprise architecture book (hub-and-spoke, point-to-point, pub-sub, API gateway, facade). The healthcare twist is *what flows through them* (HL7v2, FHIR, CCDA) and *the regulatory constraints* (HIPAA, BAAs). If you understand the patterns, you can read any architecture diagram in a healthcare RFP.

## Pattern 1: Interface Engine Hub-and-Spoke

> **Analogy:** This is the **API Gateway / Message Broker pattern** from microservices — but for healthcare. Instead of each service calling every other service directly, everything goes through a central hub that handles routing, transformation, and logging.

The most common pattern. An integration engine ([[Mirth Connect]], Rhapsody, etc.) sits in the middle.

```
System A ──HL7v2──→ ┌──────────────┐ ──HL7v2──→ System D
System B ──HL7v2──→ │ Integration  │ ──FHIR───→ System E
System C ──CSV────→ │   Engine     │ ──DB─────→ Database
                    │  (Mirth)     │ ──File───→ SFTP
                    └──────────────┘
```

**When:** You have many internal systems that need to exchange data.
**Benefit:** One connection per system instead of N*(N-1) point-to-point.

---

## Pattern 2: Point-to-Point Interface

Direct connection between two systems. Like hardcoding an API URL in your service — works fine for one connection, nightmare at scale.

```
Lab System ──HL7v2/MLLP──→ EHR
```

**When:** Simple, low-volume, single-purpose connection.
**Drawback:** Doesn't scale. 10 systems = 45 possible connections.

> **Real-world example:** A small clinic has one lab vendor. They set up a direct HL7v2 TCP connection between the lab system and their EHR. Simple, works great. But a large hospital with 30+ systems doing point-to-point would have spaghetti architecture — any change requires touching multiple connections.

---

## Pattern 3: FHIR API Gateway

> **Analogy:** This is the classic **BFF (Backend for Frontend) + API Gateway** pattern. Multiple frontends (mobile, web, analytics) hit a single gateway that fans out to back-end systems. Adding SMART on FHIR auth and audit logging at the gateway gives you HIPAA-grade controls in one place instead of bolting them onto every backend.

Modern pattern using FHIR as the common API layer.

```
Mobile App ───→ ┌──────────────┐ ──→ EHR (Epic FHIR)
Web Portal ───→ │  FHIR API    │ ──→ Lab System
Analytics  ───→ │  Gateway     │ ──→ Claims DB
                └──────────────┘
                   Auth (SMART on FHIR)
                   Rate limiting
                   Audit logging
```

**When:** Building modern applications on top of existing EHRs.

---

## Pattern 4: Event-Driven / Pub-Sub

> **Analogy:** Exactly like Kafka/RabbitMQ in a microservices architecture. The EHR publishes an event ("patient admitted"), and any interested service subscribes without the EHR knowing or caring who's listening.

Systems publish events, others subscribe.

```
EHR publishes ADT event
    |
    v
Event Bus / Message Queue
    |
    ├──→ Analytics (subscribes to all ADT)
    ├──→ Care Management (subscribes to ADT^A01 admits)
    └──→ Billing (subscribes to ADT^A03 discharges)
```

**When:** Multiple downstream systems need the same events.
**Tech:** Kafka, RabbitMQ, or cloud pub/sub. Some integration engines support this natively.

---

## Pattern 5: Document Exchange

> **Analogy:** Document exchange is like **emailing a PDF attachment, but with cryptographic identity, audit logging, and tenant isolation built in**. You're moving a whole document (a CCDA) from one organization to another. Direct/HIE/Carequality is the postal service; the CCDA is the envelope's contents.

Sharing clinical documents (CCDAs) between organizations.

```
Sending Org → CCDA Document → Direct/HIE/Carequality → Receiving Org
```

**When:** Cross-organizational data sharing (referrals, transitions of care).
**Tech:** [[Direct Messaging Protocol]], [[Health Information Exchange (HIE)]], [[Kno2]].

---

## Pattern 6: Bulk Data / ETL Pipeline

> **Analogy:** Bulk Data is **`COPY TO` for FHIR servers**. Instead of querying one resource at a time, you ask the server to dump all matching resources as NDJSON files you can download. Same idea as exporting from a data warehouse to S3 — async kickoff, polling for completion, download links when ready. The official spec is HL7's "Bulk Data Access IG."

Large-scale data extraction for analytics, research, or reporting.

```
EHR (FHIR Bulk Data) → NDJSON files → ETL Pipeline → Data Warehouse
                                            |
                                    Transform/clean
                                    Deduplicate
                                    Standardize codes
```

**When:** Population health, research, quality reporting, machine learning.
**Tech:** FHIR Bulk Data Access IG, Spark/Airflow, cloud data pipelines.

---

## Pattern 7: Facade / Adapter

> **Analogy:** Same as the **Strangler Fig pattern from Martin Fowler** — you can't replace the legacy system in one go, so you put a modern facade in front of it. The new app talks clean FHIR; the facade translates underneath. Over time, you can swap out the backend without anything in front of the facade noticing.

Wrap a legacy system's proprietary interface with a modern API.

```
Modern App → FHIR API → Adapter/Facade → Legacy System (proprietary DB or HL7v2)
```

**When:** You need to modernize access to a system you can't modify.

> **Real-world example:** A hospital has a 20-year-old lab system with a proprietary database (no API). You build a FHIR facade: a small service that queries the legacy DB, maps the results to FHIR Observation resources, and exposes them via a REST endpoint. Your new patient portal talks to clean FHIR; it never knows there's a legacy system behind the curtain. This is extremely common in healthcare.

---

## Common Integration Challenges

| Challenge | Typical Solution |
|-----------|-----------------|
| **Patient matching** | MPI (Master Patient Index), probabilistic matching algorithms. Like user dedup in a CRM — fuzzy name matching, scoring, thresholds. |
| **Code translation** | Mapping tables (ICD-10 ↔ SNOMED, local codes ↔ standard codes) |
| **Data format conversion** | Integration engine transformers (HL7v2 → FHIR, CSV → HL7v2) |
| **Reliability** | Message queuing, retry logic, dead letter queues |
| **Ordering/sequencing** | Message sequence numbers, timestamp-based ordering |
| **Duplicate detection** | Message control IDs, idempotency checks |
| **Error handling** | ACK/NACK in HL7v2, HTTP status codes in FHIR, alerting |
| **Monitoring** | Dashboard for message volumes, error rates, queue depths |

---

## IHE Profiles (Integration Standards)

> **Analogy:** IHE profiles are like **RFCs for healthcare integration workflows**. Each one specifies "if you implement patient identifier cross-referencing (PIX), here's exactly which messages, which APIs, and which error codes you must support." If two systems both claim "PIX-compliant," they can interoperate without bilateral discussions. Like HTTP RFCs, the profiles are dense, abbreviated, and a pain to read — but they're the lingua franca of HIE-to-HIE exchange.

**IHE** (Integrating the Healthcare Enterprise) defines integration profiles — standardized workflows:

| Profile | What It Does |
|---------|-------------|
| **PIX/PDQ** | Patient Identifier Cross-referencing / Patient Demographics Query |
| **XDS** | Cross-Enterprise Document Sharing (document registries/repositories) |
| **XCA** | Cross-Community Access (query documents across organizations) |
| **XCPD** | Cross-Community Patient Discovery (find patients across orgs) |
| **MHD** | Mobile access to Health Documents (FHIR-based XDS) |
| **ATNA** | Audit Trail and Node Authentication |

You'll encounter IHE profiles when working with [[Health Information Exchange (HIE)|HIEs]] and national networks.

---

**See also:** [[Mirth Connect]], [[Health Information Exchange (HIE)]], [[Healthcare Data Standards]]
