# HL7v2 Essentials

> HL7 version 2 is a messaging standard from 1987 that is still the **most widely used** standard for clinical data exchange inside hospitals. You will encounter it constantly.

> **Beginner primer:** HL7v2 is the format hospitals have used to talk to each other internally for nearly four decades. It looks weird (pipes everywhere, three-letter codes, no JSON) but the underlying idea is simple: when something happens (patient admitted, lab result ready), one system fires a text message to another. Once you can read one ADT message line-by-line, the rest of HL7v2 clicks. Don't be intimidated by the syntax — it's just badly-formatted CSV, and you'll use a library to parse it anyway.

## What It Is

HL7v2 is a **pipe-delimited text message format** for sending clinical events between systems. When a patient is admitted, a lab result comes back, or a medication is ordered — an HL7v2 message is probably being sent.

It is **not** an API. It's a message format typically sent over TCP (using MLLP — Minimal Lower Layer Protocol) or sometimes file drops.

> **Analogy:** If FHIR is a REST API (request/response, JSON, HTTP), then HL7v2 is like **a raw TCP socket that sends pipe-delimited CSV lines as fire-and-forget events**. Imagine Kafka, but from 1987, with no schema registry, and every producer formats messages slightly differently. That's HL7v2.

---

## Message Structure

An HL7v2 message is plain text with segments, fields, and components separated by delimiters:

```
MSH|^~\&|SendingApp|SendingFac|ReceivingApp|ReceivingFac|20240115120000||ADT^A01|MSG00001|P|2.5
EVN|A01|20240115120000
PID|1||MRN12345^^^Hospital^MR||Smith^John^A||19900101|M|||123 Main St^^Springfield^IL^62704
PV1|1|I|ICU^101^A|||AttendingDoc^Jane^Dr|||MED||||||||VN12345|||||||||||||||||||||||||20240115
```

### Anatomy

> **Analogy:** Think of it like nested CSV. Fields are separated by `|` (like columns). Within a field, components are separated by `^` (like a nested object). Within a component, subcomponents use `&`. It's basically JSON before JSON existed — but flat text.
>
> ```
> JSON:    { "name": { "family": "Smith", "given": "John" } }
> HL7v2:   Smith^John     (^ separates components within a field)
> ```

```
MSH|^~\&|...    ← Message Header (always first)
 |   |
 |   └── Encoding characters: ^ component, ~ repeat, \ escape, & subcomponent
 └── Field separator (pipe)

Segments: Each line is a "segment" starting with a 3-letter code
Fields:   Separated by | (pipe)
Components: Separated by ^ (caret) within a field
```

### Key Segments

| Segment | Name | What It Contains |
|---------|------|-----------------|
| **MSH** | Message Header | Sender, receiver, message type, timestamp, version |
| **PID** | Patient Identification | Name, DOB, MRN, gender, address, phone |
| **PV1** | Patient Visit | Visit type (inpatient/outpatient), attending doctor, location |
| **OBR** | Observation Request | Order info (what was ordered — lab test, imaging) |
| **OBX** | Observation Result | Actual results (lab values, vital signs) |
| **ORC** | Common Order | Order control (new, cancel, update) |
| **EVN** | Event Type | What triggered this message |
| **NK1** | Next of Kin | Emergency contacts |
| **IN1** | Insurance | Insurance plan details |
| **DG1** | Diagnosis | Diagnosis codes |
| **AL1** | Allergy | Patient allergies |

---

## Message Types (What You'll See Daily)

> **Analogy:** HL7v2 message types are like **event names in a pub-sub system**. `ADT^A01` is "patient.admitted", `ORM^O01` is "order.created", `ORU^R01` is "result.published", `SIU^S12` is "appointment.scheduled". Internally, a single hospital might be emitting hundreds of these per minute, and dozens of downstream systems are subscribing to the ones they care about.

### ADT — Admit, Discharge, Transfer
The most common message category. Tracks patient movement.

> **Real-world scenario:** A patient walks into the ER. The registration desk enters their info, triggering an ADT^A04 (register). They're admitted to the ICU — ADT^A01 (admit). Moved to a regular room — ADT^A02 (transfer). Finally sent home — ADT^A03 (discharge). Every downstream system (billing, lab, pharmacy, bed management) receives each of these messages to stay in sync.

| Type | Trigger | When |
|------|---------|------|
| **ADT^A01** | Admit | Patient is admitted to hospital |
| **ADT^A02** | Transfer | Patient moves to new unit/bed |
| **ADT^A03** | Discharge | Patient leaves |
| **ADT^A04** | Register | Outpatient registration |
| **ADT^A08** | Update | Patient demographics changed |
| **ADT^A28** | Add person | New person record created |
| **ADT^A31** | Update person | Person demographics updated |
| **ADT^A40** | Merge | Two patient records merged into one |

### ORM — Orders
| Type | When |
|------|------|
| **ORM^O01** | New order (lab, radiology, medication) |

### ORU — Results
| Type | When |
|------|------|
| **ORU^R01** | Unsolicited observation result (lab results coming back) |

### SIU — Scheduling
| Type | When |
|------|------|
| **SIU^S12** | New appointment scheduled |
| **SIU^S14** | Appointment modified |
| **SIU^S15** | Appointment cancelled |

### MDM — Medical Documents
| Type | When |
|------|------|
| **MDM^T02** | New document with content |

---

## Reading an HL7v2 Message (Worked Example)

```
MSH|^~\&|LabSystem|MainHospital|EHR|MainHospital|20240115143022||ORU^R01|12345|P|2.5
PID|1||MRN98765^^^MainHospital^MR||Doe^Jane||19850315|F
OBR|1|ORD001|LAB001|CBC^Complete Blood Count^L|||20240115140000
OBX|1|NM|WBC^White Blood Cell Count^L||7.5|10*3/uL|4.5-11.0|N|||F
OBX|2|NM|HGB^Hemoglobin^L||13.2|g/dL|12.0-16.0|N|||F
OBX|3|NM|PLT^Platelet Count^L||250|10*3/uL|150-400|N|||F
```

**Translation:**
- Lab system is sending an ORU^R01 (lab result) to the EHR
- Patient: Jane Doe, MRN 98765, Female, born 1985-03-15
- Order: Complete Blood Count (CBC)
- Results:
  - WBC: 7.5 (normal range 4.5-11.0) — **N**ormal
  - Hemoglobin: 13.2 (normal range 12.0-16.0) — **N**ormal
  - Platelets: 250 (normal range 150-400) — **N**ormal
- `F` = Final result

### OBX Data Types
| Code | Type | Example |
|------|------|---------|
| NM | Numeric | `7.5` |
| ST | String | `Positive` |
| CE | Coded Entry | `260373001^Detected^SCT` |
| TX | Text | Free-text narrative |
| DT | Date | `20240115` |

### OBX Abnormal Flags
| Flag | Meaning |
|------|---------|
| N | Normal |
| H | High |
| L | Low |
| HH | Critical High |
| LL | Critical Low |
| A | Abnormal |

---

## MLLP (How HL7v2 Messages Travel)

> **Analogy:** MLLP is like a super-simple version of HTTP framing. Where HTTP uses `Content-Length` headers or chunked encoding to tell you where a message starts and ends, MLLP just wraps the message in a start byte and end bytes. That's literally all it does.

HL7v2 messages are typically sent over **MLLP** (Minimal Lower Layer Protocol) — a thin wrapper over TCP:

```
<VT> (0x0B) — Start of message
... HL7 message content ...
<FS><CR> (0x1C 0x0D) — End of message
```

The receiving system sends back an **ACK** (acknowledgment):

```
MSH|^~\&|ReceivingApp|ReceivingFac|SendingApp|SendingFac|20240115143023||ACK^R01|54321|P|2.5
MSA|AA|12345
```

`MSA|AA` = Application Accept (message received OK)
`MSA|AE` = Application Error
`MSA|AR` = Application Reject

> **Analogy:** ACKs are **HTTP status codes for HL7v2**. `AA` is `200 OK`, `AE` is `500 Internal Server Error` (the receiver got it but couldn't process), and `AR` is `400 Bad Request` (the message was malformed and won't be retried). Unlike HTTP, ACKs travel back as their own full HL7v2 message — there's no built-in request/response framing, just two messages going opposite directions on the same MLLP connection.

---

## HL7v2 Versions

| Version | Notes |
|---------|-------|
| 2.3 | Still common in legacy systems |
| 2.3.1 | Common |
| **2.5** | **Most widely deployed** |
| 2.5.1 | Common, especially for lab reporting |
| 2.7 | Newer but less common |

In practice, you'll see messages claiming to be one version but using fields from another. HL7v2 compliance is... loose.

> **Real talk:** HL7v2 is the JavaScript of healthcare — technically there's a spec, but in practice everyone does it differently. A hospital might send a "v2.5" message that uses v2.3 date formats and includes custom Z-segments nobody documented. Your code needs to be defensive.

---

## Developer Tips

1. **Use a parser** — Never parse HL7v2 by hand with string splits. Use a library.
   - Python: `hl7apy`, `python-hl7`
   - Java: `HAPI` (not HAPI FHIR — different project, same org)
   - JavaScript: `hl7-standard`, `simple-hl7`
   - C#: `NHapi`

2. **Field positions are 1-indexed** — PID-3 is the third field (patient identifier), PID-5 is patient name.

3. **Repeating fields** — Some fields repeat using `~`: `PID-3` might have `MRN123~SSN456`

4. **Z-segments** — Custom segments starting with Z (like `ZPD`, `ZLB`) are vendor-specific. Expect them.

5. **Empty fields are common** — `||` means the field is empty. This is normal.

6. **Timestamps** — Format is `YYYYMMDDHHMMSS` (no separators): `20240115143022`

7. **Testing** — Use tools like:
   - **Mirth Connect** to inspect and transform messages
   - **7Edit** or **HL7 Inspector** for visual message viewing
   - Most integration engines have HL7v2 message viewers built in

---

**See also:** [[Mirth Connect]], [[Integration Patterns]], [[Healthcare Data Standards]]
