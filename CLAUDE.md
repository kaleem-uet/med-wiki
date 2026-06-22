# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal **Obsidian-style knowledge wiki** on Healthcare IT, written from a software engineer's perspective. It is plain Markdown notes — no code, no build system, no tests, not a git repo. The vault root is `/Users/kaleem/Documents/Obsidian Vault/`; these notes live in the `wiki/` subfolder. Edits to `.md` files are the only work that happens here.

Entry point / index: `Healthcare IT - Home.md`. All other notes are linked from there and should remain reachable from it when added.

## Cross-linking convention

Notes link to each other using Obsidian wikilink syntax: `[[Note Name]]` (no `.md` extension, exact filename minus `.md`, case-sensitive). When adding a new note, link it from `Healthcare IT - Home.md` under the appropriate section and from any topically-adjacent notes. When renaming a file, update every `[[...]]` reference to it across the wiki — there is no tooling that will catch a broken link.

## Audience and voice

Every note is written *for developers building healthcare integrations*, not for clinicians or compliance officers. This shapes content choices:

- Lead with the practical "what you need to know to ship," not history or policy background.
- Use web-dev analogies for unfamiliar healthcare concepts (e.g., "FHIR is like a government-mandated OpenAPI spec," "Resources are like Django models"). These analogies are a signature pattern of the wiki — preserve and extend them.
- Prefer concrete examples (sample HL7 messages, FHIR JSON, curl calls) over abstract description.
- Tables are heavily used for reference material (resource lists, acronyms, vendor comparisons). Match the existing table style when editing.
- Callouts use `>` blockquotes for "Analogy for web devs:" framing and notes.

## Structure of a typical note

Looking at `FHIR Deep Dive.md`, `HL7v2 Essentials.md`, etc., the convention is:

1. `# Title` matching the filename
2. A one-line `>` blockquote framing what it is and why a developer cares
3. "What is X" / "Why it matters" up front
4. Core concepts with tables and code blocks
5. Wikilinks to related notes inline where relevant, not just at the end

Keep this shape when authoring new notes so they feel consistent with the rest of the wiki.

## Glossary is canonical

`Healthcare Glossary.md` is the single source of truth for acronym expansions. When introducing an acronym in any note, the expansion should match what's in the glossary; if it's a new term, add it to the glossary at the same time.
