# med-wiki

A practical reference for software engineers working in healthcare. Written from a developer's perspective — focused on what you need to know to build, integrate, and ship.

The notes below are organized like a book: a Preface, five thematic Parts of numbered Chapters that build on one another, and an Appendix for reference material. Read in order if you're new to healthcare IT. Jump to a specific chapter if you're hunting for a topic.

---

## Table of Contents

### Preface
- [About this wiki](#about-this-wiki)
- [How to read these notes](#how-to-read-these-notes)

### Part I — Foundations
*Orient yourself in the healthcare software landscape.*

1. [Healthcare IT - Home](./Healthcare%20IT%20-%20Home.md) — Big-picture index with tech analogies for every core concept
2. [Common Healthcare Workflows](./Common%20Healthcare%20Workflows.md) — Admit/discharge, lab orders, referrals, billing, and the events behind them
3. [EHR Landscape](./EHR%20Landscape.md) — The major Electronic Health Record vendors

### Part II — The Data Languages
*The four formats every healthcare integration speaks.*

4. [Healthcare Data Standards](./Healthcare%20Data%20Standards.md) — Overview tying HL7v2, FHIR, CCDA, and X12 together
5. [HL7v2 Essentials](./HL7v2%20Essentials.md) — The legacy pipe-delimited messages used inside hospitals
6. [FHIR Deep Dive](./FHIR%20Deep%20Dive.md) — The modern REST/JSON API standard
7. [CCDA](./CCDA.md) — The XML document format for clinical document exchange

### Part III — Security & Access
*You cannot ship in healthcare without these.*

8. [HIPAA for Developers](./HIPAA%20for%20Developers.md) — The privacy law, BAAs, and what you must build to stay compliant
9. [Healthcare Authentication](./Healthcare%20Authentication.md) — SMART on FHIR (OAuth2 for healthcare apps)

### Part IV — Building Integrations
*Tools, vendors, and architectural patterns for wiring systems together.*

10. [Mirth Connect](./Mirth%20Connect.md) — The most common open-source integration engine
11. [Integration Patterns](./Integration%20Patterns.md) — Hub-and-spoke, pub-sub, facade, bulk data, and more
12. [Epic Systems](./Epic%20Systems.md) — The dominant US EHR and how to integrate with it

### Part V — Cross-Organizational Exchange
*How data moves between separate organizations.*

13. [Direct Messaging Protocol](./Direct%20Messaging%20Protocol.md) — Encrypted "email" for clinical documents
14. [Health Information Exchange (HIE)](./Health%20Information%20Exchange%20%28HIE%29.md) — Regional networks for sharing patient data, including state/regional HIEs in detail
15. [QHIN and TEFCA](./QHIN%20and%20TEFCA.md) — The federal framework and the 8 designated national QHINs
16. [Kno2](./Kno2.md) — Managed API for Direct, HIE, TEFCA, and the PD/DQ/DR query workflow

### Appendix
- **A.** [Healthcare Glossary](./Healthcare%20Glossary.md) — Acronyms and terms. Bookmark this; you'll come back constantly.

---

## About this wiki

These are working developer notes — practical, opinionated, and code-first. Every chapter targets engineers building healthcare software, not clinicians or compliance officers. Expect:

- Real HL7 messages, real FHIR JSON, real curl calls
- Web-dev analogies (Stripe, OAuth, Kafka, GitHub Apps, etc.) for unfamiliar healthcare concepts
- "Beginner primer" callouts at the top of every chapter for fast orientation
- Cross-references to related chapters whenever concepts overlap

## How to read these notes

- **Cross-references use Obsidian-style `[[wikilinks]]`.** On GitHub these don't render as clickable links — open the vault in [Obsidian](https://obsidian.md) for full navigation, or use the table of contents above.
- **Read in order if new.** Each Part builds on the previous one; chapters within a Part can be read sequentially.
- **Jump around if experienced.** Every chapter is self-contained enough to read alone.
- **If something is wrong or missing, open a PR.**
