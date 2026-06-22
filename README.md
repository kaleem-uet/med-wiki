# med-wiki

A practical reference for software engineers working in healthcare. Written from a developer's perspective — focused on what you need to know to build, integrate, and ship.

If you're new to healthcare IT, read the notes below **in order**. Each section builds on the previous one. If you're hunting for a specific topic, skip to the relevant entry.

---

## Table of Contents — Learning Order

### 1. Orient yourself
Start here to learn the vocabulary and the day-to-day reality of healthcare software.

1. [Healthcare IT - Home](./Healthcare%20IT%20-%20Home.md) — Big-picture index and tech analogies for every core concept
2. [Healthcare Glossary](./Healthcare%20Glossary.md) — Acronyms and terms. Bookmark this; you'll come back constantly
3. [Common Healthcare Workflows](./Common%20Healthcare%20Workflows.md) — Admit/discharge, lab orders, referrals, billing, etc.
4. [EHR Landscape](./EHR%20Landscape.md) — The major Electronic Health Record vendors you'll integrate with

### 2. The languages systems speak
The four data formats you'll see in nearly every healthcare integration.

5. [Healthcare Data Standards](./Healthcare%20Data%20Standards.md) — Overview tying HL7v2, FHIR, CCDA, and X12 together
6. [HL7v2 Essentials](./HL7v2%20Essentials.md) — The legacy pipe-delimited message format used inside hospitals
7. [FHIR Deep Dive](./FHIR%20Deep%20Dive.md) — The modern REST/JSON API standard
8. [CCDA](./CCDA.md) — The XML document format for clinical document exchange

### 3. Security, privacy, and access
You cannot ship in healthcare without understanding these.

9. [HIPAA for Developers](./HIPAA%20for%20Developers.md) — The privacy law, BAAs, and what you must build to stay compliant
10. [Healthcare Authentication](./Healthcare%20Authentication.md) — SMART on FHIR (OAuth2 for healthcare apps)

### 4. Building integrations
Tools, vendors, and architectural patterns for connecting systems.

11. [Mirth Connect](./Mirth%20Connect.md) — The most common open-source integration engine
12. [Integration Patterns](./Integration%20Patterns.md) — Hub-and-spoke, pub-sub, facade, bulk data, and more
13. [Epic Systems](./Epic%20Systems.md) — The dominant US EHR and how to integrate with it

### 5. Cross-organizational data exchange
How healthcare data moves between separate organizations.

14. [Direct Messaging Protocol](./Direct%20Messaging%20Protocol.md) — Encrypted "email" for sending clinical documents
15. [Health Information Exchange (HIE)](./Health%20Information%20Exchange%20%28HIE%29.md) — Regional networks for sharing patient data
16. [QHIN and TEFCA](./QHIN%20and%20TEFCA.md) — The federal framework connecting all health networks
17. [Kno2](./Kno2.md) — A managed platform that abstracts Direct, HIE, and TEFCA behind a single API

---

## How to read these notes

- Every note targets developers, not clinicians. Expect web-dev analogies (Stripe, OAuth, Kafka, GitHub Apps, etc.) for unfamiliar healthcare concepts.
- Cross-references use Obsidian-style `[[wikilinks]]`. On GitHub these don't render as clickable links — open the vault in [Obsidian](https://obsidian.md) for full navigation, or follow the table of contents above.
- Notes are practical and code-first: real HL7 messages, real FHIR JSON, real curl examples. If something is wrong or missing, open a PR.
