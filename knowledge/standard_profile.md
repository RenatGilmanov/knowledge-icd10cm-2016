# FreeFact 2 — standard knowledge profile

Status: implemented ICD-10-CM domain projection, version 1. The primary knowledge
container is **`output/standard/ffl/`**. `output/ffl/` is the separate extraction
and preservation audit ledger. Both use the [FreeFact 2 grammar](freefact_v2.md).

## 1. Membership rule

A primary fact describes the standard: a diagnosis identifier, classification,
meaning, index term, coding instruction, table classification, guideline,
convention, release change, or unresolved clinical reference. A fact does not
describe how an extractor encountered or serialized that information.

The namespace is the supplied **ICD-10-CM April 2026 update**. It is distinct from
WHO ICD-10 and ICD-10-PCS. The guideline period is April 1–September 30, 2026.
Shared technical documentation mentioning PCS does not add PCS codes or claims
of PCS coverage to this container.

The projection preserves all source-supported domain content it recognizes,
retains clinical uncertainty, and records every inclusion, replacement, merge,
or audit-only exclusion in a separate disposition ledger. It does not certify
complete clinical interpretation merely because every input assertion has a
disposition.

## 2. Compact fact contract

The canonical fields remain `r,s,p,o,q,c,e`. Release, subject, and category use FFL
directives; predicates use domain terms in `snake_case`; objects carry the
smallest complete assertion. Evidence IDs are opaque references. Their physical
locations and transformation history belong to the audit container.

```ffl
.ffl 2
.release "ICD-10-CM:2026-04-01"
.cat classification/title
.s "cm:A00.0"
title|"Cholera due to Vibrio cholerae 01, biovar cholerae"|{"e":["e:example"]}
.cat classification/hierarchy
parent|"cm:A00"|{"e":["e:example"]}
.cat instruction/inclusion_terms
inclusion_terms|["Classical cholera"]|{"e":["e:example"]}
```

The example evidence ID is illustrative. Production IDs resolve through the
projection audit map to the original evidence. Identity is still the exact hash
of `[release,subject,predicate,object,qualifiers]`; category and evidence remain
outside identity. Equal assertions union their evidence. A projection transforms
the assertion explicitly before identity is computed; it never changes the
FreeFact identity algorithm.

Identifiers, punctuation, ranges, incomplete codes, nonessential modifiers, and
seventh characters retain their meanings. A transformed representation can omit
indentation once an explicit semantic parent or term path carries the hierarchy.
The original notation remains in the audit ledger. Unknown code references remain
unknown; no substitute or current validity is invented.

## 3. Domain vocabulary

The generated `output/standard/vocabulary.json` enumerates the actual predicates
and categories. The following contracts define their meaning.

| Category family | Domain assertions |
|---|---|
| `classification/title`, `classification/short_title` | Full and abbreviated code meanings. |
| `classification/identifier`, `classification/hierarchy` | Exact identifier and semantic parent. |
| `classification/status` | `classification_entry` with reportability, code without decimal, and tabular sequence. Sequence is classification order, not an extraction row number. |
| `classification/sections` | Section identifier, title, first category, and last category. |
| `classification/notation` | Placeholder requirement. |
| `classification/seventh_character` | Base relationship and selected character for a complete code. |
| `instruction/*` | `includes`, `inclusion_terms`, `excludes1`, `excludes2`, `code_first`, `use_additional_code`, `code_also`, `notes`, `convention`, `seventh_character_applicability`, and `seventh_characters`. |
| `instruction/neoplasm_table` | `code_selection`: the three ordered introductory paragraphs governing neoplasm code selection. |
| `index/*` | Term title and parent, nonessential modifier, code reference, manifestation code, distinct `see`/`see_also` links, category/subcategory references, and synonyms. |
| `index/book` | Explicitly published disease-index and external-cause-index titles. All four index books also have a semantic parent link. |
| `index/notation`, `index/reference`, `index/caution` | Exact ambiguous clinical notation, referenced identifiers without inferred roles, and unresolved-code status. |
| `table/drug`, `table/neoplasm` | Classification dimensions and code assignment by named drug circumstance or neoplasm behavior. Numeric column positions are excluded. |
| `table/visual_impairment` | Visual-acuity categories with clinical thresholds and special conditions. |
| `guideline/section` | Clause title and parent clause. |
| `guideline/rule` | `states`: an ordered array of complete source paragraphs within one guideline scope. |
| `guideline/poa` | Present-on-admission indicator labels and definitions. |
| `coding/convention` | Reportability, code length, tabular precedence, inclusive range membership, and release-dependent classification order. |
| `coding/caution` | An exact clinical notation and its unresolved delimiter issue. |
| `release/edition`, `release/summary`, `release/change` | Edition, guideline period, code-set counts, unchanged standard components, and ordered historical changes. |

Category and predicate names are retrieval aids. Clinical wording can legitimately
discuss code structure, the Alphabetic Index, or the Tabular List. Removing such
domain conventions because they contain words such as “format” would lose
standard knowledge.

## 4. Scope, hierarchy, and repeated context

Code instructions remain at their published owner. Retrieve ancestor instructions
with the code; do not repeat every inherited instruction as a new explicit fact.
Keep Excludes1 distinct from Excludes2, sequencing distinct from association, and
all exceptions with the rule they limit. A code or index match alone is not a
complete code-assignment decision.

`seventh_characters` preserves the published sequence of character definitions
and eight intervening notes. A definition is `{"character":"A","meaning":...}`;
a note is `{"note":...}` at its original position. Adjacency alone does not attach
a note exclusively to the preceding character. In particular, W08's “Fall from
stool” after the S definition does not establish a sequela-only limitation.

Index identity depends on its semantic ancestry, term, and required disambiguating
content. Alphabet separator nodes are navigation and are removed from the domain
hierarchy. Term identifiers use a 12-character URL-safe base64 encoding of a
72-bit SHA-256 prefix, with explicit collision checks against distinct semantic
identities. These compact entity identifiers do not replace FreeFact's full
256-bit assertion identity. Numeric occurrence addresses remain in the audit
mapping. Equal words under different clinical parents do not become synonyms.
All four index books retain an explicit parent link to the standard, so term
retrieval can reach book-level conventions and shared release knowledge.

Table dimensions use clinical labels. Every explicit code or unlisted cell must
retain its associated behavior/circumstance. An unlisted cell does not license an
inferred code. The audit retains distinctions among absent, empty, and printed
placeholder forms even where the domain projection represents the same outcome.

## 5. Guideline prose

`states` preserves paragraph order within each guideline subject. Ordered arrays
retain list introductions, code lists, accompanying descriptions, conditions,
exceptions, and examples together. They are prose assertions, not a claim that
all conditions have been compiled into executable logic.

Repeated `scope_path` labels are replaced by explicit guideline title/parent
facts. A consumer must retrieve the parent chain to interpret the scope. The
subject already names the owning guideline, so a redundant scope qualifier is
unnecessary. Present-on-admission indicators retain their own domain identities.

For this package, 967 original paragraph facts are represented in 439 guideline
scopes. Twenty-nine adjacent-page continuations are joined only after both
evidence positions resolve uniquely and exact concatenation reproduces the joined
text. All retained words and their scope order are independently compared with
the original ordered paragraph artifact. The result has 934 paragraphs.

Printed code-to-guidance rows must remain adjacent within their owning scope.
Separate code and description columns are interleaved in printed row order before
paragraph grouping; the original character-conservation check still applies.
Do not require a reader to infer row associations from detached lists. Heading
levels must follow the printed hierarchy: alphabetic `(c)`, `(d)`, `(i)`, `(l)`
and `(m)` are not nested Roman subsections merely because they can be Roman digits.
The direct source audit independently verifies numbered guideline headings from
their original-page positions and reports unverified narrative alignments.

One typography explanation and two deduplicated bullet-only facts are audit-only.
The guideline publication heading becomes its effective period. Navigation
contents and page furniture remain in the audit. No other paragraph is removed
because a lexical classifier did not recognize it as a rule.

## 6. Changes and release summaries

`changes` is an ordered array of objects:

```json
{"action":"add","text":"pancytopenia (acquired) D61.818","term_path":["myelodysplastic D46.9","with"]}
```

Each change has `action` and `text`; `rule` identifies a coding instruction when
present. `term_path` holds intermediate clinical index terms when needed. The
fact's subject identifies its target, and `q.effective` gives the effective date.
Index-change targets also have a title, so an opaque identifier cannot hide the
root term. Array order replaces incidental source-order fields. Index indentation
is removed after explicit ancestry is retained.

All 122 operations are preserved across 38 targets. Individual operations and
paired revisions map to the same ordered change assertions, with evidence union.
Unchanged context labels are retained through the target/title/term path where
semantically needed. Unrelated earlier tabular headings are excluded from scope.

Additions, deletions, and revisions describe history. Deleted wording is never
promoted to a current instruction. `code_set_changes` describes the reportable
code/nonreportable-heading inventories and title-change counts; zero code-set
changes do **not** mean zero instruction or index changes. Unchanged drug,
external-cause, and neoplasm components remain explicit release assertions.

## 7. Known clinical notation issues

| Audited issue | Primary representation |
|---|---|
| Three unbalanced clinical references | Preserve the instruction text and emit `notation_caution`. |
| Three index occurrences referencing Z15.08 or Q99.814 | Preserve the reference and emit `unresolved_code` with `status:"absent_from_release"`. |
| Four compound synonym/code expressions | Preserve the full printed line as `index_notation`, mark it `ambiguous`, and retain the listed identifiers as `referenced_codes`. Do not assign etiology/manifestation roles by guesswork. |
| SCFE and an anatomical bracketed synonym | Preserve as synonyms, not diagnosis codes. |
| Four table discrepancies affecting sixteen cells | Preserve the supported clinical cell results; reconciliation mechanics remain in the audit. |
| One empty note | Preserve in the audit; it asserts no additional clinical rule. |

The profile uses domain cautions rather than embedding extractor dispositions,
transport field names, or repair narratives in facts intended for reasoning.

## 8. Audit-only material and traceability

Excluded from primary facts: source-file manifests, archives, checksums, parser
schemas, namespaces, element/attribute trees, document blocks, coordinates,
typography, field widths/offsets, filenames, and extraction diagnostics. The
original `output/ffl/`, preserved inputs, evidence, and reports retain these.

Reviewed clinical meanings found only in document passages still enter the
primary container. Three neoplasm introduction paragraphs form one scoped
`code_selection` assertion under `index:neoplasm`. Eight introductory Tabular List
passages exactly reconcile with seven existing conventions and add evidence to
those assertions. The raw block identities and reconciliation mechanics remain
in the audit. This targeted boundary review does not certify independent semantic
review of every preserved document block.

Some technical documentation states genuine domain conventions. Those meanings
are retained explicitly: 3–7 code characters excluding the decimal point,
reportable codes versus headings, tabular order versus alphanumeric order,
inclusive ranges by tabular order, and order changes between releases. Physical
file layouts are excluded.

`output/reports/standard_projection.sqlite` records each legacy assertion's
disposition, old-to-domain assertion links, subject mappings, and the bridge from
opaque domain evidence IDs to original evidence. A transformation can map several
old assertions to one domain assertion, or one old assertion to several domain
assertions. An audit-only exclusion has a reason and no invented domain fact.

## 9. Verification and use

The producer fails on unmapped structures, unrecognized document changes, missing
paragraph positions, unproved continuation joins, and uncovered change
operations. Verify the primary container against its own manifest and vocabulary.
Verify interpretation coverage separately from audit preservation coverage.

Relevant implementations:

- `scripts/project_standard.py`: complete primary projection and trace ledger.
- `scripts/standard_documents.py`: guideline, change, release, and caution mapping.
- `scripts/standard_supplements.py`: reviewed neoplasm guidance and duplicate
  tabular-convention evidence.
- `scripts/test_standard_documents.py`: independent ordered-prose conservation,
  change coverage, scope/title retention, release counts, and ambiguous references.
- `scripts/validate_standard.py`: container and projection checks.
- `scripts/freefact.py`: unchanged FFL parser and exact identity contract.
- `scripts/measure_standard_tokens.py`: equivalent-fact token comparison and
  seventeen primary context examples.

Token measurements must identify whether they cover primary domain facts or the
audit ledger and compare equivalent expanded facts. PDF/ZIP bytes are not a token
baseline. Compactness does not establish clinical reasoning quality or certify
complete guideline applicability.

### Measured package

The validated primary package contains **809,979 facts in 41 shards**. Current
`cl100k_base` counts are recorded in `output/reports/standard_token_metrics.json`
and compare its FFL with the same
facts in compact canonical JSONL. The same report measures all seventeen context
examples in aggregate; small packs can have higher FFL overhead. See the
the historical source-workspace report `output/reports/standard_token_metrics.md` (not part of this release; current package measurements are in `manifest.json`)
for per-shard counts, hashes, gzip comparisons, and reproducible methodology.
