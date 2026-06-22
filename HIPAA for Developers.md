# HIPAA for Developers

> HIPAA (Health Insurance Portability and Accountability Act) is the US federal law governing health data privacy and security. As a healthcare developer, everything you build must comply with HIPAA. Violations can result in fines up to $1.9M per incident.

> **Beginner primer:** HIPAA is the law that turns "engineering decisions" into "legal liability." When you choose a logging provider, a queue, a hosting platform, or even a Slack channel for incident response — HIPAA constrains those choices. The good news: most modern cloud platforms (AWS, GCP, Azure) have HIPAA-compliant tiers. The bad news: you have to *opt in* to them and sign a BAA. Defaults are usually not HIPAA-safe. The single rule that will save you most pain: **never let PHI touch a service you don't have a BAA with**.

## What HIPAA Actually Requires (Developer Version)

### PHI — Protected Health Information
PHI is any health data that can identify a patient. If your system touches PHI, HIPAA applies.

> **Analogy:** Think of PHI like **PII on steroids**. In regular software, you worry about GDPR and PII (name, email, etc.). In healthcare, the rules are stricter, the penalties are higher ($1.9M per violation), and the data is more sensitive. If your database has a patient's name + diagnosis, that's PHI — and HIPAA governs everything about how you store, transmit, access, and log it.

**The 18 HIPAA Identifiers** (any of these + health data = PHI):
1. Name
2. Address (anything smaller than state)
3. Dates (except year) — DOB, admission date, discharge date, death date
4. Phone numbers
5. Fax numbers
6. Email addresses
7. SSN
8. Medical record numbers (MRN)
9. Health plan beneficiary numbers
10. Account numbers
11. Certificate/license numbers
12. Vehicle identifiers
13. Device identifiers and serial numbers
14. Web URLs
15. IP addresses
16. Biometric identifiers
17. Full-face photos
18. Any other unique identifier

**De-identified data** (all 18 removed) is NOT PHI and HIPAA doesn't apply to it.

---

## The Three HIPAA Rules That Affect You

### 1. Privacy Rule
- Defines WHO can access PHI and for WHAT purposes
- **Minimum Necessary** — Only access/expose the minimum PHI needed for the task
- Patients have rights: access their data, request corrections, get accounting of disclosures
- **Developer impact:** Build role-based access controls. Log who accessed what. Provide patient data export.

> **Analogy:** "Minimum Necessary" is the **principle of least privilege**, but for data instead of permissions. The same way you wouldn't give a `read-only-reports` user admin keys, you shouldn't fetch a patient's full chart when you only need their DOB. Translate that into your API design: scoped tokens, narrow SELECTs, response field filtering. "I'll just grab everything in case we need it" is a HIPAA violation waiting to happen.

### 2. Security Rule
Technical safeguards for electronic PHI (ePHI). Three categories:

> **Analogy:** The Security Rule is basically **SOC 2 with legal teeth**. Same kinds of controls — access logging, encryption, MFA, incident response — but instead of losing a customer when you fail an audit, the government can fine you millions per incident. If your company has a SOC 2 program, you're 80% of the way to HIPAA Security Rule compliance; the remaining 20% is paperwork and BAAs.

#### Administrative Safeguards
- Risk assessments
- Workforce training
- Security policies
- Incident response plan
- Business Associate Agreements (BAAs) — see below

#### Physical Safeguards
- Facility access controls
- Workstation security
- Device disposal procedures

#### Technical Safeguards (What You Build)
| Requirement | What It Means for You |
|------------|----------------------|
| **Access Control** | Unique user IDs, role-based access, auto-logoff |
| **Audit Controls** | Log all access to PHI — who, what, when |
| **Integrity Controls** | Prevent unauthorized modification of ePHI |
| **Transmission Security** | Encrypt data in transit (TLS 1.2+) |
| **Encryption at Rest** | Encrypt stored ePHI (AES-256) |
| **Authentication** | Verify user identity (MFA recommended) |

### 3. Breach Notification Rule
- If PHI is exposed, you must notify affected patients within 60 days
- Breaches affecting 500+ people must be reported to HHS and media
- **Developer impact:** Build systems to detect breaches. Have an incident response plan.

> **Analogy:** This is **GDPR's 72-hour breach notification rule, but in slow motion** — you get 60 days instead of 72 hours. The catch: if 500+ records are affected, the breach lands on a public-facing HHS website (the "wall of shame") and local news outlets get notified. From a developer standpoint, this means your detection has to actually work; "we didn't notice" is not a legal defense.

---

## BAA — Business Associate Agreement

### What It Is
If your company handles PHI on behalf of a healthcare organization (a "Covered Entity"), you are a **Business Associate** and must sign a BAA.

> **Analogy:** A BAA is like a **Data Processing Agreement (DPA)** under GDPR, but for healthcare. It's a legal contract that says: "We handle your patient data. Here's what we promise to do to protect it. If we screw up, here's our liability." Every vendor in the chain that touches PHI needs one.

> **Real-world example:** You build a SaaS app for clinics. Your stack: React frontend, Node.js backend on AWS, PostgreSQL on RDS, logs in Datadog. You need BAAs with: AWS (they have one), your database hosting (covered by AWS BAA), and Datadog (check if they offer one — if not, you can't send PHI to Datadog logs). If you use Vercel for hosting? No BAA available — you can't put PHI there.

### When You Need One
- Your SaaS product stores patient data → **BAA required** with each customer
- You use AWS/GCP/Azure → **BAA required** with cloud provider
- You use a database hosting service → **BAA required** if it stores PHI
- You use an analytics service that sees PHI → **BAA required**

### Cloud Providers with BAAs

> **Analogy:** Think of BAAs as **"this vendor has been HIPAA-certified" checkboxes**. Same dynamic as picking a SOC 2-compliant subprocessor for SaaS — only certain providers qualify, you have to explicitly opt into the compliant tier, and the rest are off-limits for PHI. The list below is small for a reason.
| Provider | BAA Available | Notes |
|----------|--------------|-------|
| AWS | Yes | Must enable specific HIPAA-eligible services |
| Google Cloud | Yes | Must accept BAA in console |
| Azure | Yes | HIPAA compliance built into enterprise agreements |
| Heroku | No (mostly) | Not recommended for PHI |
| Vercel | No | Not HIPAA compliant |
| MongoDB Atlas | Yes | Dedicated clusters only |

---

## Practical Developer Checklist

### Data in Transit
- [ ] TLS 1.2+ for all connections
- [ ] No PHI in URLs/query parameters (use POST body or headers)
- [ ] No PHI in logs (or encrypt/mask it)
- [ ] Encrypted email/messaging if sending PHI ([[Direct Messaging Protocol]])

### Data at Rest
- [ ] AES-256 encryption for databases storing PHI
- [ ] Encrypted backups
- [ ] Encrypted file storage
- [ ] Disk encryption on any device that processes PHI

### Access Control
- [ ] Unique user accounts (no shared logins)
- [ ] Role-based access control (RBAC)
- [ ] Minimum necessary access (don't expose all patient data to all users)
- [ ] Auto-logout/session timeout
- [ ] MFA for accessing systems with PHI

### Audit Logging
- [ ] Log all PHI access: who, what record, when, from where
- [ ] Log all modifications to PHI
- [ ] Logs must be tamper-resistant
- [ ] Retain logs for 6 years (HIPAA requirement)
- [ ] Regularly review audit logs

### Application Security
- [ ] Input validation (prevent injection attacks)
- [ ] OWASP Top 10 protections
- [ ] Regular security assessments / pen testing
- [ ] Secure development lifecycle
- [ ] No PHI in error messages shown to users
- [ ] No PHI in client-side storage (localStorage, cookies) unless encrypted

### Environment
- [ ] Separate dev/staging/prod environments
- [ ] Never use real PHI in dev/test — use synthetic data (Synthea)
- [ ] Secure configuration management (no secrets in code)
- [ ] Vulnerability scanning and patching

---

## Common Mistakes

1. **Logging PHI** — Your application logs should never contain patient names, MRNs, SSNs. Mask them.
   > *Example:* Instead of `logger.info("Fetching records for John Smith MRN-12345")`, write `logger.info("Fetching records for patient [REDACTED]")` or use opaque IDs: `logger.info("Fetching records for patient_uuid=abc-def-123")`
2. **PHI in URLs** — `GET /patient?name=JohnSmith` puts PHI in server logs, browser history, etc.
   > *Fix:* Use `POST /patient/search` with the name in the request body, or use opaque IDs: `GET /patient/abc-def-123`
3. **No audit trail** — You must log who accessed what PHI and when.
4. **Using non-BAA services** — Sending PHI through a service without a BAA is a violation.
5. **Screenshots and screen recordings** — PHI visible in screenshots is a breach if shared.
6. **Slack/email** — Don't discuss specific patient data in Slack, email, or tickets.
7. **Copy/paste to AI tools** — Don't paste PHI into ChatGPT, Claude, etc. (unless you have a BAA with the AI provider).

---

**See also:** [[Healthcare Authentication]], [[Healthcare Glossary]]
