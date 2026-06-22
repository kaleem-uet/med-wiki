# Healthcare IT - Developer's Guide

A practical reference for software engineers working in healthcare. Written from a developer's perspective — focused on what you need to know to build, integrate, and ship.

> **New to healthcare IT? Start here.** Healthcare software is a world of its own — different acronyms, different protocols, different rules. But under the hood, the concepts map cleanly to things you already know:
> - **EHR** = the monolithic database every hospital is built around (Epic, Cerner, etc.)
> - **HL7v2** = pipe-delimited messages, like CSV-over-TCP, used inside hospitals
> - **FHIR** = a government-mandated REST/JSON API for healthcare
> - **CCDA** = clinical documents in XML, used between organizations
> - **HIPAA** = GDPR's stricter American cousin, just for health data
>
> Read the notes below in order if you're brand new. If you're hunting for a specific topic, the section headers below are the entry points.

---

## Core Concepts
- [[Healthcare Data Standards]] — HL7v2, FHIR, CDA, X12 — the languages systems speak
- [[FHIR Deep Dive]] — The modern API standard you'll use most
- [[HL7v2 Essentials]] — The legacy standard you'll encounter everywhere
- [[CCDA]] — The XML document format for clinical document exchange

## Key Systems & Vendors
- [[Epic Systems]] — The dominant EHR and how to integrate with it
- [[EHR Landscape]] — Epic, Cerner (Oracle Health), Athenahealth, and others
- [[Mirth Connect]] — The integration engine you'll likely work with
- [[Kno2]] — Document exchange and Direct messaging

## Integration & Interoperability
- [[Health Information Exchange (HIE)]] — How organizations share patient data
- [[QHIN and TEFCA]] — The national framework for health data exchange
- [[Integration Patterns]] — Common patterns for connecting healthcare systems
- [[Direct Messaging Protocol]] — Secure email-like messaging for healthcare

## Compliance & Security
- [[HIPAA for Developers]] — What you must know to stay compliant
- [[Healthcare Authentication]] — SMART on FHIR, OAuth, and identity

## Terminology & Reference
- [[Healthcare Glossary]] — Quick-reference for acronyms and terms
- [[Common Healthcare Workflows]] — Admit, discharge, lab orders, referrals, etc.

---

> These notes are written for developers. If something is unclear or missing, update it.
