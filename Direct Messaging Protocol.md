# Direct Messaging Protocol

> Direct is a **secure, encrypted messaging protocol** for sending clinical documents between healthcare organizations. Think of it as "encrypted email for healthcare" — but it's not regular email.

> **Beginner primer:** Before Direct existed, hospitals literally **faxed** clinical documents between offices — because regular email isn't secure enough for HIPAA. Direct is what the federal government built to replace the fax machine. It's still email-shaped (uses SMTP under the hood and addresses look like email), but every message is encrypted and every sender's identity is verified by a certificate authority. As a developer, you almost never touch the raw protocol — you call a HISP's API (like Kno2's) and it handles the rest.

## What It Is

- Developed by ONC as part of the Direct Project
- Uses **S/MIME encryption** over SMTP
- Each participant has a **Direct address** (looks like email: `dr.smith@hospital.direct.example.com`)
- Messages are encrypted end-to-end using **X.509 certificates**
- Primarily used to send **CCDA documents** and other clinical attachments

> **Analogy:** Direct is like **PGP-encrypted email, but managed and healthcare-specific**. If you've ever used GPG/PGP to send encrypted emails — same concept. Each provider has a keypair, messages are encrypted with the recipient's public key and signed with the sender's private key. The difference: Direct has a managed trust infrastructure (DirectTrust) so you don't have to do key exchange manually like PGP's web of trust.

> **Why not just use regular email?** Because email isn't encrypted at rest, anyone can spoof a sender address, and there's no guaranteed identity verification. Direct solves all of these with certificates + trust bundles. HIPAA requires this level of protection for PHI.

---

## How It Works

```
Sender's System
    |
    v
Sender's HISP (Health Information Service Provider)
    |  encrypts with recipient's public cert
    |  signs with sender's private cert
    v
Internet (SMTP/TLS)
    |
    v
Recipient's HISP
    |  verifies sender's signature
    |  decrypts with recipient's private key
    v
Recipient's System
```

### Key Components

| Component | What It Is |
|-----------|-----------|
| **Direct Address** | An address like `provider@hospital.direct.example.com` |
| **HISP** | Health Information Service Provider — manages certs, routing, encryption. Think of it like a managed email provider (SendGrid) but for Direct. |
| **Trust Anchor** | Root certificate that establishes trust between HISPs |
| **Trust Bundle** | Collection of trust anchors (DirectTrust maintains the main bundle) |
| **Certificate** | X.509 cert for encryption and signing |

---

> **Analogy for the moving parts:** Think of it like **sending a courier package internationally**. The HISP is your courier company (FedEx). The certificate is the tamper-evident seal proving who packed it. The Trust Bundle is the list of courier companies that the destination country lets through customs. DirectTrust is the international agency that decides which couriers qualify. You hand FedEx the package + an address; they handle customs, encryption, and delivery.

## When Direct Is Used

| Use Case | Example |
|----------|---------|
| **Referrals** | PCP sends referral note + CCDA to specialist |
| **Transitions of Care** | Hospital sends discharge summary to follow-up provider |
| **Lab Results** | Lab sends results to ordering provider |
| **Public Health Reporting** | Provider sends immunization data to state registry |
| **Patient Access** | Patient requests records sent to their Direct address (via PHR) |

---

## Direct vs Regular Email

| Feature | Direct | Regular Email |
|---------|--------|--------------|
| Encryption | S/MIME (mandatory) | Optional (TLS in transit only) |
| Authentication | X.509 certificates | Username/password |
| Trust | Trust bundles, verified identities | Anyone can send |
| HIPAA Compliant | Yes (by design) | No (without additional measures) |
| Attachments | CCDA, PDFs, structured data | Anything |
| Delivery Confirmation | MDN (Message Disposition Notification) | Read receipts (unreliable) |

---

> **Analogy:** The Direct-vs-email distinction is the same as **WhatsApp vs SMS**. Both look like text messages to the user, but one is end-to-end encrypted with verified identities and the other isn't. You'd send your bank password over WhatsApp before SMS, for the same reasons hospitals send referrals over Direct instead of regular email.

## Developer Perspective

### You Probably Won't Build Direct from Scratch
Most developers use a **HISP service** that handles the protocol:
- **[[Kno2]]** — Managed Direct messaging + document exchange
- **EMR Direct** — HISP services
- **MaxMD** — Direct messaging provider
- **MedAllies** — HISP and health data exchange

### Integration Pattern

```
Your App → HISP API (REST) → HISP handles Direct protocol → Recipient
```

### What You'll Actually Build
1. **Compose outbound messages** — Create CCDA documents, attach to Direct message via HISP API
2. **Process inbound messages** — Receive webhook/poll from HISP, parse CCDA, file in your system
3. **Address lookup** — Query provider directories for Direct addresses
4. **Status tracking** — Monitor MDNs (delivery confirmations)

---

## DirectTrust

**DirectTrust** is the organization that:
- Manages the trust framework for Direct messaging
- Accredits HISPs
- Maintains the trust bundle (collection of trusted certificates)
- Operates the provider directory

If a HISP is DirectTrust-accredited, it can exchange messages with any other accredited HISP.

---

**See also:** [[Kno2]], [[Health Information Exchange (HIE)]], [[Healthcare Data Standards]]
