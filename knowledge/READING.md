# Reading the FFL knowledge

Start with [manifest.json](manifest.json) for the release, every category, file counts and hashes; [vocabulary.json](vocabulary.json) defines the predicates. The [complete format](FORMAT.md), [canonical identity](freefact_v2.md) and [standard profile](standard_profile.md) are included locally.

## Layout

There are 41 primary category shards, using FreeFact 3's `category-readable` / `ffl3-named` layout. Every fact occurs once in this knowledge set. The same subject can occur in several files because titles, hierarchy, instructions and status are different categories. Read all applicable categories before treating a code's context as complete.

Each file is plain UTF-8 with LF-delimited records:

- `.ffl 3` selects the grammar.
- `.release "ICD-10-CM:2026-04-01"` binds the edition.
- `.cat instruction/excludes1` selects the current primary category.
- `.s "cm:E11.9"` selects the current subject.
- `predicate|object|evidence[|qualifiers]` records one exact fact. `object` and optional `qualifiers` are JSON; qualifiers default to `{}`. Read one JSON value at a time: a literal `|` inside a quoted string is not a field separator.
- Evidence is a string, a nonempty array of strings, or `=` meaning the current subject is its sole evidence ID. It never implies an unrecorded source.
- `.end N H` verifies the fact count and SHA-256 of the expanded canonical row stream. A parser must exhaust and verify the file before trusting returned rows.

No `.dict`, `.t`, `.v`, numeric predicates or text aliases are needed in this handoff. The normative format describes those optional modes for other containers.

## Reasoning contract

1. Expand each fact to release, subject, predicate, object, qualifiers, category and evidence. Preserve exact codes, strings, numeric types, ordered arrays, `false`, `null` and absent object fields.
2. Keep each rule on its recorded subject. Follow `parent` and `seventh_character_of` for structural context, and read applicable guideline scopes separately. An ancestry lookup does not prove guideline applicability or a correct clinical decision.
3. Treat `code_for` qualifiers as part of the assertion. Poisoning circumstances and neoplasm behaviors are distinct table assignments. `null` with `status: "not_listed"` is an explicit absent mapping, not a missing extraction.
4. Preserve `see` versus `see_also`, Excludes1 versus Excludes2, parent versus seventh-character relationship, and reportable versus non-reportable entries.
5. Keep full index term paths and nonessential modifiers. A terminal word such as “other” is not an independent complete concept.
6. Do not expand dash expressions or ranges by guessing. Keep references absent from the supplied registry unresolved. Graph `literal_mention` edges are lexical locators into rule text, not additional clinical assertions.

The [selected-fact CSV](../graph/selected-facts.csv) provides the compact graph's 21,110 exact facts and their one-based FFL assertion ordinals. It is a browsing sample. For every fact on a subject, run `python3 tools/query.py cm:E11.9 --ancestors` from the repository root, or read all FFL categories. The complete fact set is held only in the FFL files.

## Identity and editions

Semantic identity is SHA-256 of canonical `[release, subject, predicate, object, qualifiers]`. Category and evidence do not enter that ID but are included in full canonical integrity checks. The manifest's canonical-set digest covers all seven expanded fields in semantic-ID order.

Do not mix facts from another edition into this release. A new edition changes release-bound fact IDs even when wording remains unchanged. Code identifiers can recur across editions; opaque index IDs must not be assumed stable without an explicit alignment. File hashes and canonical digests provide the comparison boundary.

## Scope of verification

All 809,979 facts and all 98,186 registry identifiers match the existing extracted knowledge exactly. [Validation](../validation.json) records the measured boundary. Complete source-meaning or autonomous clinical-reasoning equivalence is not certified. Evidence IDs are retained; resolving them to originals requires the separate source audit bridge.
