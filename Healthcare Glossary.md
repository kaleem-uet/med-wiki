# Healthcare IT Glossary

> Quick reference for acronyms and terms. Linked to detailed notes where available.

> **Beginner tip:** Healthcare has more acronyms than any other industry you've worked in. Don't try to memorize this whole page — just bookmark it. After a few weeks on the job, the ones you actually need will stick. The rest you can look up here.

## The 10 You'll Hear Daily

If you only learn ten, learn these — you'll hear them in every meeting, ticket, and Slack thread:

| Term | One-Line Mental Model |
|------|----------------------|
| **EHR** | The hospital's main database + UI (Epic, Cerner). Like Salesforce, but for clinical care. |
| **PHI** | Patient data + identifier. Like PII, but stricter penalties (up to $1.9M per violation). |
| **HIPAA** | The privacy law. Like GDPR for health data. |
| **HL7v2** | Old pipe-delimited message format. Like CSV-over-TCP from 1987. |
| **FHIR** | Modern REST/JSON healthcare API. Like a federally-mandated OpenAPI spec. |
| **CCDA** | XML clinical document. Like an MDX file — has both human narrative and structured data. |
| **MRN** | Medical Record Number — the hospital's patient ID. Like a `user.id` but per hospital. |
| **ADT** | "Admit, Discharge, Transfer" messages. Like `user.created`/`user.deleted` webhooks for patients. |
| **HIE** | Health Information Exchange — regional network sharing patient data. Like a CDN for records. |
| **SMART on FHIR** | OAuth2 for healthcare apps. Like "Login with GitHub" but for EHRs. |

---

## A

| Term | Full Name | Definition |
|------|-----------|-----------|
| **ACO** | Accountable Care Organization | Group of providers who share financial risk for a patient population |
| **ADT** | Admit, Discharge, Transfer | HL7v2 message category tracking patient movement |
| **ARRA** | American Recovery and Reinvestment Act | 2009 law that funded EHR adoption (HITECH Act) |

## B

| Term | Full Name | Definition |
|------|-----------|-----------|
| **BAA** | Business Associate Agreement | Legal contract required when a vendor handles PHI on behalf of a covered entity |
| **BPA** | Best Practice Alert | Clinical decision support alert in an EHR |

## C

| Term | Full Name | Definition |
|------|-----------|-----------|
| **CCDA** | Consolidated Clinical Document Architecture | XML standard for clinical documents (care summaries, discharge notes) |
| **CDA** | Clinical Document Architecture | HL7 standard for clinical documents. CCDA is a constrained version. |
| **CDR** | Clinical Data Repository | Database storing clinical data from multiple sources |
| **CDS** | Clinical Decision Support | Rules/logic that assist clinicians (drug interaction alerts, order recommendations) |
| **Clearinghouse** | — | Intermediary that routes claims between providers and payers (e.g., Change Healthcare, Availity) |
| **CMS** | Centers for Medicare & Medicaid Services | Federal agency running Medicare/Medicaid. Sets payment rules and some interoperability requirements. |
| **CPOE** | Computerized Provider Order Entry | EHR function where doctors enter orders electronically |
| **CPT** | Current Procedural Terminology | Code system for medical procedures/services (used in billing). Owned by AMA. |

## D

| Term | Full Name | Definition |
|------|-----------|-----------|
| **DICOM** | Digital Imaging and Communications in Medicine | Standard for medical images (X-rays, MRIs, CTs) |
| **Direct** | Direct Messaging Protocol | Encrypted messaging protocol for sending clinical documents between providers |
| **DQ** | Document Query | IHE XCA operation: list documents for a known patient on a remote network. Step 2 of PD → DQ → DR. |
| **DR** | Document Retrieve | IHE XCA operation: fetch a specific document by ID. Step 3 of PD → DQ → DR. |

## E

| Term | Full Name | Definition |
|------|-----------|-----------|
| **EDI** | Electronic Data Interchange | Umbrella term for electronic business transactions (X12 is the healthcare EDI standard) |
| **EHR** | Electronic Health Record | The core clinical software system (Epic, Cerner, etc.) |
| **EMR** | Electronic Medical Record | Often used interchangeably with EHR, technically just one provider's records |
| **ePHI** | Electronic Protected Health Information | PHI in electronic form — subject to HIPAA Security Rule |
| **ERA** | Electronic Remittance Advice | X12 835 — payment explanation from payer to provider |
| **EOB** | Explanation of Benefits | Statement from insurance to patient about what was covered |

## F

| Term | Full Name | Definition |
|------|-----------|-----------|
| **FHIR** | Fast Healthcare Interoperability Resources | Modern RESTful API standard for healthcare data. Pronounced "fire". See [[FHIR Deep Dive]]. |

## H

| Term | Full Name | Definition |
|------|-----------|-----------|
| **HAPI** | HL7 Application Programming Interface | Popular open-source Java library for HL7v2 and FHIR |
| **HIE** | Health Information Exchange | Organization/network for sharing health data. See [[Health Information Exchange (HIE)]]. |
| **HIPAA** | Health Insurance Portability and Accountability Act | Federal law governing health data privacy/security. See [[HIPAA for Developers]]. |
| **HISP** | Health Information Service Provider | Organization operating Direct messaging infrastructure |
| **HITECH** | Health Information Technology for Economic and Clinical Health | 2009 act that funded EHR adoption and strengthened HIPAA |
| **HL7** | Health Level Seven | Standards organization and the standards they publish (v2, v3, CDA, FHIR) |

## I

| Term | Full Name | Definition |
|------|-----------|-----------|
| **ICD-10** | International Classification of Diseases, 10th Revision | Diagnosis code system. ~70,000 codes. Used worldwide. |
| **IHE** | Integrating the Healthcare Enterprise | Organization that defines integration profiles (XDS, XCA, PIX, etc.) |

## L

| Term | Full Name | Definition |
|------|-----------|-----------|
| **LIS** | Laboratory Information System | Software that manages lab operations and results |
| **LOINC** | Logical Observation Identifiers Names and Codes | Code system for lab tests and clinical observations |

## M

| Term | Full Name | Definition |
|------|-----------|-----------|
| **MFA** | Multi-Factor Authentication | Required for systems accessing PHI |
| **MLLP** | Minimal Lower Layer Protocol | TCP wrapper protocol for HL7v2 messages |
| **MPI** | Master Patient Index | System for matching/linking patient identities across systems |
| **MRN** | Medical Record Number | Hospital-assigned patient identifier (different at each hospital) |
| **Mirth** | Mirth Connect / NextGen Connect | Open-source healthcare integration engine. See [[Mirth Connect]]. |

## N

| Term | Full Name | Definition |
|------|-----------|-----------|
| **NCPDP** | National Council for Prescription Drug Programs | Standard for pharmacy transactions (e-prescribing) |
| **NDC** | National Drug Code | Unique identifier for medications in the US |
| **NPI** | National Provider Identifier | Unique 10-digit ID for every healthcare provider in the US |

## O

| Term | Full Name | Definition |
|------|-----------|-----------|
| **ONC** | Office of the National Coordinator for Health IT | Federal agency overseeing health IT policy, TEFCA, and certification |

## P

| Term | Full Name | Definition |
|------|-----------|-----------|
| **PACS** | Picture Archiving and Communication System | System for storing/viewing medical images (DICOM) |
| **PBM** | Pharmacy Benefit Manager | Company managing prescription drug benefits for insurers |
| **PD** | Patient Discovery | IHE XCPD operation: "does this patient exist on the network?" Step 1 of PD → DQ → DR. |
| **PHI** | Protected Health Information | Any health data + patient identifier. Protected by HIPAA. |
| **PHR** | Personal Health Record | Patient-controlled health record (e.g., Apple Health, Google Health) |

## Q

| Term | Full Name | Definition |
|------|-----------|-----------|
| **QHIN** | Qualified Health Information Network | Designated national network under TEFCA. See [[QHIN and TEFCA]]. |
| **QRDA** | Quality Reporting Document Architecture | CDA-based format for quality measure reporting |

## R

| Term | Full Name | Definition |
|------|-----------|-----------|
| **RCM** | Revenue Cycle Management | The billing/collections process in healthcare |
| **RIS** | Radiology Information System | Software managing radiology workflow |

## S

| Term | Full Name | Definition |
|------|-----------|-----------|
| **SDOH** | Social Determinants of Health | Non-clinical factors affecting health (housing, food, transportation) |
| **SMART** | Substitutable Medical Applications, Reusable Technologies | OAuth2-based app framework for FHIR. See [[Healthcare Authentication]]. |
| **SNOMED CT** | Systematized Nomenclature of Medicine - Clinical Terms | Comprehensive clinical terminology system (~350,000 concepts) |
| **SOA** | Service-Oriented Architecture | Common architecture pattern in health IT |
| **Surescripts** | — | Dominant e-prescribing network in the US |

## T

| Term | Full Name | Definition |
|------|-----------|-----------|
| **TEFCA** | Trusted Exchange Framework and Common Agreement | National framework for health data interoperability. See [[QHIN and TEFCA]]. |

## U

| Term | Full Name | Definition |
|------|-----------|-----------|
| **USCDI** | United States Core Data for Interoperability | Minimum dataset that must be exchangeable. Defined by ONC. |
| **US Core** | US Core FHIR Profiles | FHIR profiles implementing USCDI requirements |

## V

| Term | Full Name | Definition |
|------|-----------|-----------|
| **VPN** | Virtual Private Network | Common for site-to-site healthcare connections |

## X

| Term | Full Name | Definition |
|------|-----------|-----------|
| **X12** | ASC X12 | EDI standard for healthcare billing (837 claims, 835 remittance, 270/271 eligibility) |
| **XCA** | Cross-Community Access | IHE profile for querying documents across organizations |
| **XCPD** | Cross-Community Patient Discovery | IHE profile for finding patients across organizations |
| **XDS** | Cross-Enterprise Document Sharing | IHE profile for document registries and repositories |

---

**See also:** [[Healthcare IT - Home]]
