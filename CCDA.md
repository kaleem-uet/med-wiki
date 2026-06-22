# CCDA (Consolidated Clinical Document Architecture)

> CCDA is the XML format used to exchange **clinical documents** between healthcare organizations in the US. If you're sending a discharge summary, referral note, or care summary across an organizational boundary, it's almost certainly a CCDA.

> **Beginner primer:** When two hospitals need to share a patient's chart and a real-time API isn't available, they exchange a CCDA. It's the format you use when "send the whole discharge summary as one file" makes more sense than "let the other system query my data piece by piece." If FHIR is the live API, CCDA is the snapshot — a frozen, signed document representing what one provider knew about the patient at one moment in time.

## What It Is

CCDA is a **constrained version of HL7 CDA** — an XML schema for self-contained clinical documents. A single CCDA file represents a whole document (e.g., one discharge summary for one patient on one date), not an event or an API response.

> **Analogy for web devs:** Think of a CCDA as an **MDX file for medical records**. Each document carries both a human-readable narrative (renders to a styled HTML page via an XSL stylesheet) *and* machine-parseable structured data (coded entries, dates, identifiers) inside the same XML envelope. The narrative is the "source of truth" for clinicians; the entries are the "source of truth" for systems. The two are supposed to agree, but in practice they often drift.

- Created by **HL7 International**, based on the older CDA R2 standard
- Current version: **C-CDA R2.1** (you'll also see R1.1 in older systems)
- Always XML — there is no JSON CCDA
- Self-contained: a CCDA includes patient demographics, provider info, document type, and clinical sections all in one file
- Validated against published **template IDs** (OIDs) that constrain what each section must contain

## Why It Matters (Regulatory)

CCDA is the document format that nearly every US compliance regime points to:

- **ONC Certification (formerly Meaningful Use / Promoting Interoperability)** — EHRs must generate and consume CCDAs in specified formats
- **Transitions of Care** — federally required document for patient handoffs
- **[[Direct Messaging Protocol|Direct]] / [[Health Information Exchange (HIE)|HIE]] / Carequality** — CCDA is the default payload
- **TEFCA / [[QHIN and TEFCA|QHINs]]** — supports CCDA exchange alongside FHIR

In short: if you ship anything that exchanges clinical documents with another organization, you handle CCDAs.

---

## CDA vs CCDA (Clearing Up the Confusion)

| Term | What It Is |
|------|-----------|
| **CDA** | The base HL7 standard. Very permissive — defines XML structure for any clinical document. |
| **CCDA** | A US-specific **constraint** on CDA. Tightens the rules, mandates specific sections and templates. "Consolidated" = consolidates many older document templates into one library. |
| **C-CDA** | Same thing as CCDA. The hyphen is the official spelling; "CCDA" is the common spelling. |

If someone says "CDA" in a US healthcare context, they almost always mean CCDA.

---

## Document Types You'll See

CCDA is a library of document templates. Each has its own `templateId` and LOINC code identifying what kind of document it is.

| Document | LOINC | When It's Used |
|----------|-------|---------------|
| **CCD** (Continuity of Care Document) | 34133-9 | The generic "summary of care" document. Default for HIE pulls. |
| **Discharge Summary** | 18842-5 | Generated when a patient leaves a hospital. |
| **Referral Note** | 57133-1 | PCP → specialist handoff. |
| **Consultation Note** | 11488-4 | Specialist's reply to a referral. |
| **History and Physical** | 34117-2 | New patient or admission note. |
| **Progress Note** | 11506-3 | Ongoing visit/encounter note. |
| **Care Plan** | 52521-2 | Ongoing care goals, problems, interventions. |
| **Transfer Summary** | 18761-7 | Patient moving between facilities. |

Most integrations only need to handle CCD + Discharge Summary + Referral Note. The rest are situational.

---

## Anatomy of a CCDA

Every CCDA has the same skeleton: a **header** (who, what, when), then a **body** of **sections**, each of which has **narrative text** and **structured entries**.

```xml
<ClinicalDocument xmlns="urn:hl7-org:v3">
  <!-- Template IDs identify this as a US Realm CCD -->
  <templateId root="2.16.840.1.113883.10.20.22.1.1"/>   <!-- US Realm Header -->
  <templateId root="2.16.840.1.113883.10.20.22.1.2"/>   <!-- CCD -->

  <!-- LOINC code identifying the document type -->
  <code code="34133-9" codeSystem="2.16.840.1.113883.6.1"
        displayName="Summarization of Episode Note"/>
  <title>Continuity of Care Document</title>
  <effectiveTime value="20240115143022"/>

  <recordTarget>
    <patientRole>
      <id extension="MRN12345" root="2.16.840.1.113883.19.5"/>
      <patient>
        <name><given>Jane</given><family>Doe</family></name>
        <birthTime value="19850315"/>
        <administrativeGenderCode code="F"/>
      </patient>
    </patientRole>
  </recordTarget>

  <component>
    <structuredBody>
      <component>
        <section>
          <templateId root="2.16.840.1.113883.10.20.22.2.6.1"/> <!-- Allergies -->
          <code code="48765-2" codeSystem="2.16.840.1.113883.6.1"/>
          <title>Allergies</title>

          <!-- Human-readable narrative -->
          <text>
            <table>
              <tr><th>Substance</th><th>Reaction</th></tr>
              <tr><td>Penicillin</td><td>Hives</td></tr>
            </table>
          </text>

          <!-- Machine-readable structured entry -->
          <entry>
            <act classCode="ACT" moodCode="EVN">
              <code code="CONC" codeSystem="2.16.840.1.113883.5.6"/>
              <!-- ...nested observation with SNOMED code for penicillin allergy... -->
            </act>
          </entry>
        </section>
      </component>
    </structuredBody>
  </component>
</ClinicalDocument>
```

### Key Pieces

| Piece | What It Does |
|-------|-------------|
| `templateId` (OID) | Identifies which CCDA template applies. A document or section can carry multiple — most specific wins. |
| `code` (LOINC) | Identifies what *kind* of document or section this is. |
| `recordTarget` | The patient the document is about. |
| `author`, `custodian`, `documentationOf` | Who wrote it, who owns it, what encounter it covers. |
| `structuredBody` → `section` → `text` | The narrative — what humans read. Renders via stylesheet. |
| `structuredBody` → `section` → `entry` | The structured data — what systems parse. Uses SNOMED / RxNorm / LOINC codes. |

---

## The Narrative-vs-Entries Duality (The Most Important Thing to Understand)

A CCDA section contains *both*:

1. A human-readable **narrative** block (free-form HTML-like markup), and
2. Zero or more **structured entries** (coded clinical facts)

These are supposed to be **redundant representations of the same information**. The narrative is what a clinician reads; the entries are what your code parses.

> **Real talk:** They drift. Constantly. A discharge summary might say "Patient discharged on Lisinopril 10mg daily" in the narrative, while the structured entry codes it as Lisinopril 20mg or omits it entirely. Some sending systems generate entries from the narrative; others generate narrative from entries; some produce one and stub the other. **Never trust that narrative and entries agree.** If you must pick one, the narrative is what the clinician signed.

---

## How CCDAs Travel

CCDAs are payloads, not protocols. They ride on top of other transports:

```
CCDA XML  ──→  [[Direct Messaging Protocol|Direct]]      ──→  Provider mailbox (S/MIME email)
          ──→  [[Health Information Exchange (HIE)|HIE]]   ──→  Query/Retrieve (IHE XDS.b, XCA)
          ──→  Carequality / TEFCA                          ──→  Document query across QHINs
          ──→  [[Kno2]] API                                 ──→  Hides Direct/HIE plumbing
          ──→  FHIR `DocumentReference.content[].attachment` ──→  Embedded as base64 or URL
```

The transport you choose mostly depends on which networks the receiving organization is on. Most modern apps use [[Kno2]] or a similar abstraction to avoid managing Direct/HIE plumbing directly.

---

## CCDA ↔ FHIR

The industry is migrating from documents (CCDA) to resources ([[FHIR Deep Dive|FHIR]]), but both will coexist for a long time.

- **CCDA → FHIR**: Most FHIR servers expose a `$convert` operation (or you use a library like Microsoft FHIR Converter) to turn a CCDA into a FHIR `Bundle` of resources. **This is lossy** — narrative is preserved, but template-specific structure flattens to generic resources.
- **FHIR → CCDA**: Going the other way means assembling a CCDA from FHIR resources. Several libraries exist, but the output rarely passes strict CCDA validation without hand-tuning.
- **Wrapping**: Often easier to wrap a CCDA inside a FHIR `DocumentReference` (with the CCDA XML as a base64 attachment) than to convert it. Preserves fidelity, defers parsing.

---

## Developer Tips

1. **Use a parser.** Never parse CCDA with regex or string operations. Libraries handle namespaces, templateId lookup, and section traversal correctly:
   - Python: `ccda-parser`, `bbox-py`, or generic `lxml` with XPath
   - JavaScript/Node: `bluebutton.js`, `@finta/ccda-parser`
   - Java: HAPI (the CDA flavor, not FHIR), Lantana's `CDA Validator`
   - .NET: `Everest Framework`

2. **Validate with the ONC validators.** The official validators (e.g., ETT, Lantana's Trifolia, the SITE validator) catch templateId mismatches and missing required entries. Pass-but-warn is the realistic bar; pass-clean is rare.

3. **OIDs are everywhere and you will memorize none of them.** Keep a reference list. Common ones:
   - `2.16.840.1.113883.10.20.22.1.2` — CCD document
   - `2.16.840.1.113883.6.1` — LOINC
   - `2.16.840.1.113883.6.96` — SNOMED CT
   - `2.16.840.1.113883.6.88` — RxNorm

4. **Render the narrative for humans.** Use the official HL7 CDA stylesheet (`CDA.xsl`) to transform CCDA XML into displayable HTML. Don't try to render entries directly — that path leads to madness.

5. **Trust the narrative over entries** when they disagree (see duality section above).

6. **Z-segments don't exist here, but vendor extensions do.** Epic, Cerner, and others embed proprietary `templateId`s under their own root OIDs. Ignore them if you don't know them.

7. **Patient matching is the hard part.** Two CCDAs from different organizations rarely share an identifier — you'll match on name + DOB + gender + address and accept ambiguity.

8. **Timestamps use HL7 format** (`YYYYMMDDHHMMSS[+TZ]`), same as [[HL7v2 Essentials|HL7v2]].

---

**See also:** [[Healthcare Data Standards]], [[FHIR Deep Dive]], [[Direct Messaging Protocol]], [[Health Information Exchange (HIE)]], [[Kno2]], [[Integration Patterns]], [[Common Healthcare Workflows]]
