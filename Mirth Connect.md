# Mirth Connect (NextGen Connect)

> Mirth Connect is the most widely used **open-source integration engine** in healthcare. If you're moving HL7v2 messages, FHIR resources, or CCDA documents between systems, you'll likely use Mirth.

> **Beginner primer:** Mirth is the tool you reach for when System A speaks HL7v2 and System B speaks FHIR (or files, or a database, or a different HL7v2 dialect). It's a Java-based server with a desktop UI where you drag-and-drop "channels" that listen, transform, and forward messages. Most healthcare IT shops have a Mirth (or a paid competitor like Rhapsody) somewhere in the data path. If you've used Logstash pipelines, n8n workflows, or Apache NiFi flows, you'll feel at home — same idea, more healthcare connectors, JavaScript instead of YAML for transformations.

## What It Is

Mirth Connect (now officially "NextGen Connect Integration Engine") is middleware that:
- **Receives** messages from source systems
- **Transforms** data from one format to another
- **Routes** messages to destination systems
- **Logs** everything for troubleshooting

Think of it as a healthcare-specific message broker + ETL tool.

> **Analogy:** Mirth is like **Zapier/n8n for healthcare data**, but running on your own servers. Or if you prefer: it's like an **API gateway + message queue + ETL pipeline** rolled into one GUI-based tool. Source receives data in Format A, transformer converts it to Format B, destination sends it to System C. You configure this visually and write JavaScript for the transformations.

---

## Core Concepts

### Channels
A **channel** is a processing pipeline — like a single route handler in Express, but for messages instead of HTTP requests. Each channel has:

```
Source (listener)  →  Filter  →  Transformer  →  Destination(s)
                                                    ├── Destination 1
                                                    ├── Destination 2
                                                    └── Destination 3
```

**Example channel:** "Lab Results Router"
- Source: Listen on TCP port 6661 for HL7v2 messages
- Filter: Only process ORU^R01 messages
- Transformer: Map fields, add/remove segments
- Destination 1: Forward to EHR on port 7771
- Destination 2: Write a copy to database for analytics

### Connectors (Source and Destination Types)

> **Analogy:** Connectors are **the "Trigger" and "Action" blocks in Zapier**, but for healthcare protocols instead of SaaS apps. Each connector is just a configured I/O adapter — listen on a TCP port, poll a folder, hit an HTTP endpoint. Source connectors bring data in; destination connectors push it out.

| Connector | Protocol | Use Case |
|-----------|----------|----------|
| **TCP/MLLP Listener** | HL7v2 over MLLP | Receive HL7v2 messages |
| **TCP/MLLP Sender** | HL7v2 over MLLP | Send HL7v2 messages |
| **HTTP Listener** | REST/SOAP | Receive FHIR, webhooks, API calls |
| **HTTP Sender** | REST/SOAP | Call external APIs, send FHIR |
| **File Reader** | File system | Poll directory for files (CSV, HL7, XML) |
| **File Writer** | File system | Write output files |
| **Database Reader** | JDBC | Query database on schedule |
| **Database Writer** | JDBC | Insert/update database records |
| **SMTP/POP3** | Email | Send/receive email (rarely used) |
| **DICOM Listener/Sender** | DICOM | Medical imaging (radiology) |
| **JavaScript** | Custom | Custom code execution |

### Transformers
Transformers map data from one format to another. You write these in **JavaScript** (or Python/Velocity, but JS is most common).

```javascript
// Example: Map HL7v2 PID segment to variables
var firstName = msg['PID']['PID.5']['PID.5.2'].toString();  // Given name
var lastName = msg['PID']['PID.5']['PID.5.1'].toString();   // Family name
var dob = msg['PID']['PID.7']['PID.7.1'].toString();        // Date of birth
var mrn = msg['PID']['PID.3']['PID.3.1'].toString();        // MRN

// Set channel map variables (available to destinations)
channelMap.put('patientFirstName', firstName);
channelMap.put('patientLastName', lastName);

// Modify outbound message
tmp['PID']['PID.5']['PID.5.1'] = lastName.toUpperCase();
```

### Filters
Filters decide whether a message should be processed or dropped:

```javascript
// Only process ADT^A01 (admit) messages
if (msg['MSH']['MSH.9']['MSH.9.1'].toString() == 'ADT' &&
    msg['MSH']['MSH.9']['MSH.9.2'].toString() == 'A01') {
    return true;  // Process this message
}
return false;  // Drop it
```

---

## Message Flow in Mirth

```
1. Source receives raw message
2. Source transformer processes it (optional)
3. Source filter decides: process or skip?
4. Message enters channel
5. For each destination:
   a. Destination filter: should this destination get it?
   b. Destination transformer: modify for this destination
   c. Destination connector: send it out
6. Response sent back to source (ACK/NACK)
```

### Message Statuses

> **Analogy:** Message statuses are like **job states in Sidekiq, BullMQ, or Celery**. A message moves from `RECEIVED → TRANSFORMED → SENT` on the happy path, or sidetracks into `QUEUED` (retrying), `ERROR` (failed), or `FILTERED` (intentionally dropped). The Mirth dashboard is your job queue UI — you'll spend more time there than you expect.

| Status | Meaning |
|--------|---------|
| **RECEIVED** | Source got the message |
| **TRANSFORMED** | Transformers ran successfully |
| **SENT** | Delivered to destination |
| **QUEUED** | Destination unavailable, queued for retry |
| **ERROR** | Something went wrong |
| **FILTERED** | Message was intentionally dropped by filter |

---

## Variable Scopes (Important!)

> **Analogy:** Think of these like variable scopes in a web framework: `channelMap` is like `req.locals` (per-request), `globalChannelMap` is like app-level middleware state, and `globalMap` is like a Redis cache shared across all services.

Mirth has different variable maps for passing data around:

| Map | Scope | Use |
|-----|-------|-----|
| `channelMap` | Current message in current channel | Pass data between transformer and destinations |
| `globalChannelMap` | All messages in current channel | Channel-wide config, counters |
| `globalMap` | All channels | Cross-channel shared data |
| `sourceMap` | Source connector | Source-specific data |
| `responseMap` | Destinations | Collect responses to send back |
| `connectorMap` | Current connector | Connector-specific temp data |

---

## Common Use Cases

### 1. HL7v2 Interface (Most Common)
```
Hospital System A  --HL7v2/MLLP-->  Mirth  --HL7v2/MLLP-->  Hospital System B
                                      |
                                  Transform:
                                  - Remap patient IDs
                                  - Convert codes
                                  - Add/remove segments
```

### 2. HL7v2 to FHIR Conversion

> **Real-world example:** A hospital lab system from 2005 only speaks HL7v2. Your new patient portal needs FHIR. Mirth sits in the middle, receiving HL7v2 lab results and converting them to FHIR Observation resources in real-time. The lab system doesn't change. Your app gets clean JSON.

```
Lab System  --HL7v2 ORU-->  Mirth  --FHIR POST-->  FHIR Server
                              |
                          Transform:
                          - Parse HL7v2 OBX segments
                          - Build FHIR Observation resources
                          - POST to FHIR endpoint
```

### 3. File Processing
```
SFTP Drop Folder  --CSV files-->  Mirth  --HL7v2-->  EHR
                                    |
                                Parse CSV,
                                build HL7v2 messages
```

### 4. Database Integration
```
Database (poll every 5 min)  -->  Mirth  --FHIR/HL7v2-->  Destination
                                    |
                                Query new records,
                                transform to messages
```

---

## Developer Tips

1. **Channel naming convention** — Use prefixes: `LAB_ResultsToEHR`, `ADT_FeedToAnalytics`
2. **Always log** — Use `logger.info()` in transformers. You'll thank yourself when debugging.
3. **Use channel tags** — Organize channels by system, type, or environment.
4. **Test with message templates** — Save sample messages and replay them.
5. **Version control your channels** — Export channels as XML and commit to git. Mirth doesn't have built-in version control.
6. **Error handling** — Set up error queues and alerts. Messages will fail.
7. **Performance** — For high-volume channels, tune thread counts and queue sizes.
8. **Environments** — Keep separate Mirth instances for dev/test/prod.

### Common Debugging Steps
1. Check the **Dashboard** — see message counts and errors
2. Open **Message Browser** — view individual messages and their processing steps
3. Look at **Raw**, **Transformed**, **Encoded**, and **Sent** tabs for each message
4. Check **Errors** tab for stack traces
5. Add `logger.info()` statements to transformers

---

**See also:** [[HL7v2 Essentials]], [[Integration Patterns]], [[Healthcare Data Standards]]
