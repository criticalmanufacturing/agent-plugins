# Turning documentation answers into deliverables

Read this when the user wants a CM GPT answer as a file rather than a chat reply.

## Order of operations

1. **Search first.** Run the normal `search_documentation` workflow from SKILL.md, including the
   5-search limit. Never start building a file before the documentation is retrieved — the document
   must be built from retrieved chunks, not from general knowledge.
2. **Confirm scope if the request is broad.** A "training deck on Dispatch" could be 5 slides or 40.
   Ask about audience, depth, and length before building.
3. **Then read the output-format skill** (`docx`, `pptx`, `xlsx`, `pdf`) and build the file.
4. For any Critical Manufacturing PowerPoint, combine the `pptx` skill with the `cm-pptx` skill so
   the deck follows CM brand standards.

## Coverage rule

A deliverable must not exceed what the documentation supports. If the user asks for a section the
docs don't cover:

- Build the sections that are covered.
- List the gaps explicitly to the user in the chat reply — not silently, and not filled in with
  plausible-sounding invented detail.

This is the same no-fabrication rule as a chat answer. A gap in a document is harder for the reader
to spot than a gap in a chat reply, so it matters more here, not less.

## Carrying citations into each format

Citations survive the format change. Use the same
`[Title - DocSource (DocVersion)](SourceUrl)` reference, placed where the format allows:

| Format | Where citations go |
|---|---|
| Word (`.docx`) | Footnotes, or a "Sources" section at the end of each major section |
| PowerPoint (`.pptx`) | Speaker notes for the slide, plus a closing Sources slide |
| Excel (`.xlsx`) | A dedicated `Sources` column, or a separate `Sources` sheet keyed to row IDs |
| PDF | Same as Word |

## Version labelling

State the documentation version the deliverable was built from, on the title page or first slide
(for example, "Based on MES documentation v11.1"). MES behaviour differs across releases, and a
document that outlives its release without saying which one it describes becomes actively
misleading. If `docVersions` was left empty and the latest docs were searched, label it as such.

## What not to build

Do not produce documents that present MES capabilities as commitments — pricing, delivery timelines,
contractual guarantees, or roadmap promises. Documentation describes what the product does today.
Redirect those requests to the relevant CM team.
