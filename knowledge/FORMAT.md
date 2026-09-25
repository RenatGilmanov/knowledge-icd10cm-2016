# FreeFact 3 — compact transport profile

Status: implemented optional encoding. Canonical facts and identity remain
[FreeFact 2](freefact_v2.md). A v2-only reader must reject `.ffl 3`. This profile
compresses representation; it does not reinterpret the standard or authorize
semantic deletion. Files use UTF-8 and LF-delimited directives/assertions.

## 1. Literal row and defaults

```text
.ffl 3
.release "ICD-10-CM:2026-04-01"
.cat classification/title
.s "cm:A00.0"
title|"Cholera due to Vibrio cholerae 01, biovar cholerae"|"e:example"
.end 1 <canonical-row-stream-SHA256>
```

The evidence identifier and footer digest above are illustrative. The footer
always contains a real 64-character lowercase digest in an emitted file.

Named row: `predicate|object|evidence[|qualifiers]`.

- `object` is one complete JSON value. Preserve strings and ordered arrays exactly.
- `evidence` is a nonempty JSON string (one ID), a nonempty array of nonempty
  strings, or `=` meaning the current subject identifier is the sole evidence ID.
  It never implies an unrecorded source. The encoder uses `=` only after exact
  equality with the original evidence set.
- `qualifiers` is a JSON object, default `{}`. Other JSON types are rejected.
- `.release` is required once. `.cat` and `.s` replace only their own default.
  Category and subject are required for a named assertion.
- Comments begin with `#`; blank lines and comments do not create facts.

Parse one JSON value at a time, then consume the next separator. Literal pipes
inside strings do not split a field. Duplicate JSON keys, nonfinite numbers,
invalid UTF-8, trailing content and unknown directives fail validation. JSON
values follow [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259); the existing
FreeFact serializer additionally preserves its numeric type/lexical distinctions
for canonical identity. Do not replace that algorithm with a different JSON
canonicalization scheme.

## 2. Exact templates and text references

```text
.t 0|["classification/title","title",{}]
.v 0|"An exact repeated title"
.s "cm:example"
@0|^0|"e:example"
```

`.t n|[category,predicate,qualifiers]` defines the complete category, predicate
and qualifiers for `@n` rows. Template rows have exactly three fields; a fourth
qualifier field is forbidden. No implicit category/qualifier merge occurs.
`.v n|JSON_string` defines exact text for an unquoted `^n` object reference.
Quoted `"^0"` remains literal text. Pooling never normalizes text or merges
different predicates, semantic qualifiers or code identifiers.

Both index spaces are zero-based contiguous nonnegative decimal integers without
leading zeros. Definitions must precede facts; duplicate definitions/values,
forward references, negative indices and unresolved references are errors.
Dictionary indices are transport addresses and do not participate in fact identity.
Subjects, evidence IDs and ordered object components are not silently rewritten.

For large containers, `.dict "sha256"` replaces inline definitions. The caller
must supply the exact dictionary bytes named in the manifest; the decoder checks
the SHA-256 before using them. A dictionary has exactly:

```json
{"version":1,"templates":[["classification/title","title",{}]],"values":[]}
```

It is compact canonical JSON followed by LF when emitted. The decoder checks the
hash of supplied bytes, independent of whitespace or key order, and then validates
the structure. Only string values may be pooled. Inline and external dictionaries
cannot mix; no URL, filesystem path, environment lookup or fallback dictionary is
implicitly resolved. A supplied but unused dictionary is an error.

## 3. Canonical expansion and completeness

Expand each row to exactly `{r,s,p,o,q,c,e}`. Evidence IDs form a sorted unique set.
Use the unchanged FreeFact 2 semantic ID: SHA-256 of canonical `[r,s,p,o,q]`.
Category assignments and evidence are verified in addition to semantic identity.
Different qualifiers, exact wordings, Unicode sequences, `false`/`0`, `1`/`1.0`,
`null`, and ordered arrays remain distinct. Duplicate semantic IDs within a file
are rejected.

The required final non-comment line is `.end N H`:

- `N` is the number of emitted assertions, a positive integer.
- `H` is SHA-256 of each expanded seven-field canonical JSON object followed by
  LF, concatenated in physical assertion order. Keys use the FreeFact 2 sorted-key
  serializer. Each row includes category and the normalized evidence set.

A missing/mismatched footer, trailing assertion, altered type, dropped row or
changed evidence causes failure. The streaming parser's result is provisional
until its iterator is exhausted; `decode_verified` buffers the result and only
returns after verification. File conversion writes atomically after successful
verification. A footer detects corruption, not authenticity; container manifests
and comparison with the trusted source FFL provide the external binding.

## 4. Container layouts

The manifest declares `layout`. `subject-compact` is the default for existing
manifests without that field. `category-readable` is an alternative for a clear,
self-contained handoff. Both layouts preserve the same canonical facts and use
the same assertion grammar and integrity footer.

### Subject-compact layout

The manifest records release, collection role, exact category counts, source
manifest/hash records, canonical-set digests, chosen encoding, complete dictionary
hash, and every shard's hash/count/category subtotals and subject range. A shard
contains whole subject groups in subject/category/predicate/ID order. Objects keep
their internal order. A nominal 50,000-fact boundary is extended to finish the
current subject. Shard filenames are their content hashes, supporting reuse of
unchanged content without inventing release-delta savings.

The build compares full subject-grouped FFL 2, named FFL 3, templated FFL 3 and
templated/text-pooled FFL 3, charging complete dictionary/framing costs. The smallest
measured `cl100k_base` representation wins; the manifest declares the actual
version. Primary standard and source-preservation audit containers remain separate.

Canonical-set digest: sort by full semantic ID, serialize the expanded seven-field
record with sorted keys, append LF to every record, hash concatenation. The semantic-
set digest hashes sorted full IDs, each followed by LF. A byte equality claim about
the original physical FFL is not made; canonical facts must compare exactly.

### Category-readable layout

Use one self-contained named-predicate FFL 3 shard per primary category. The
manifest records `layout: "category-readable"`, `encoding: "ffl3-named"`, release,
exact category counts, every shard's filename/category/count/byte size/hash, and
the same canonical-set and semantic-set digests defined above. Human-readable
filenames identify categories; file hashes, not filenames, establish integrity.
External dictionaries and numeric predicate aliases are absent. Every shard has
explicit release/category/subject defaults and its own count/digest footer.

Subjects can occur in several category shards. A complete subject lookup must
union facts across those shards; a category file is not a complete code context.
Ordered objects, qualifiers, evidence and canonical IDs must remain unchanged.
Readers and graph exports must not treat structural projections as extra facts.

This layout intentionally selects readable predicate names and independent files.
It does not claim to be the smallest encoding. Token measurements charge every
shipped FFL header, comment and footer, identify the tokenizer, and distinguish
knowledge tokens from the separate graph and visualization artifacts.

## 5. Query delivery and budgets

Query packs union canonical assertions and supporting evidence from all selected
subjects/ancestors. Conflicting primary category assignments or releases fail
explicitly; this transport profile does not discard alternate labels or invent a
reconciliation. Existing packs from one domain collection have consistent categories.
The standard's current scope/inheritance rules and source-replacement limits remain.

`auto` compares whole self-contained candidates, including inline dictionaries,
reader instructions, clinical comments and integrity footer, and falls back to
FFL 2 when smaller. A token ceiling rejects an oversized complete pack; it never
truncates facts to fit. Decode aliases before reasoning if reference resolution
would burden the consumer. A compact byte or token count does not establish
improved model answer accuracy.

The canonical FFL 2 package remains available. New strict readers are implemented
in `scripts/freefact_compact.py`; independent archive verification is separate.
