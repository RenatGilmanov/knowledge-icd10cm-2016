# Token effectiveness · measured, reproducible

The delivered FFL represents **809,979 exact canonical facts** in **23,719,770 tokens** across 41 category files. It uses **54.74% fewer tokens than equivalent canonical JSONL**, saving **28,688,756 tokens**, and **6.76% fewer than the previous FFL 2**, saving **1,718,604 tokens**.

## Figures

| View | Vector | Raster |
|---|---|---|
| Whole-set token efficiency | [SVG](../visualizations/06-token-efficiency.svg) | [PNG](../visualizations/06-token-efficiency.png) |
| Token cost by category | [SVG](../visualizations/07-token-profile.svg) | [PNG](../visualizations/07-token-profile.png) |
| Selected context costs | [SVG](../visualizations/08-context-token-cost.svg) | [PNG](../visualizations/08-context-token-cost.png) |

The standalone SVGs contain selectable text and embedded data bound to the report hash. PNGs are exported at twice the vector dimensions (5,280–5,400 pixels wide). All category and context rows are shown, including negative savings.

## Whole-set comparison

| Representation | Exact tokens | Tokens / fact | Facts / 1,000 tokens |
|---|---:|---:|---:|
| Canonical JSONL | 52,408,526 | 64.70 | 15.46 |
| Previous FFL 2 | 25,438,374 | 31.41 | 31.84 |
| Delivered FFL 3 | 23,719,770 | 29.28 | 34.15 |

**Tokenizer:** `cl100k_base`, tiktoken 0.14.0; ordinary-text mode. The report includes a SHA-256 fingerprint of the token ranks, regular expression and special-token definitions. Token counts are measured locally, not estimated from bytes or characters. A different tokenizer can give different results.

**Boundaries:** each complete category file is tokenized independently, then counts are summed. Included: all FFL headers, comments, category/subject defaults, evidence identifiers, qualifiers, integrity footers and final LF endings. Excluded: schema/reader instructions, graph inspection copies, images, documentation, original source binaries, resolved source evidence text and model request/response wrappers. The figures do not estimate API billing or model answer quality.

**Equivalent baselines:** the previous FFL 2 files come from the exact source manifest bound by the delivered knowledge manifest. JSONL is regenerated from their same seven expanded fields (`c,e,o,p,q,r,s`), sorted keys, compact JSON separators, UTF-8 and one LF per fact. It uses one-letter keys and excludes derived hash IDs. Each category retains its fact order. This is an equivalent-fact serialization comparison, not a comparison against the original PDF/ZIP package or an assertion of optimal compression.

**Formulas:** savings = `100 × (baseline_tokens − FFL3_tokens) / baseline_tokens`; tokens/fact = `sum(tokens) / sum(facts)`; facts/1,000 tokens = `1,000 × sum(facts) / sum(tokens)`. Whole-set rates use weighted totals. Negative savings mean FFL framing uses more tokens in that slice; they are not clipped. Seven categories show that result: `coding/caution`, `index/book`, `index/caution`, `index/synonym`, `instruction/neoplasm_table`, `release/edition`, `table/visual_impairment`.

## Selected contexts

| Selection | Facts | JSONL tokens | FFL 3 tokens | Saved vs JSONL |
|---|---:|---:|---:|---:|
| E11.9 · Diabetes context | 44 | 3,439 | 1,907 | 44.55% |
| S06.1X7A · Seventh-character exception | 80 | 6,015 | 3,226 | 46.37% |
| H54.8 · Visual-acuity criteria | 49 | 4,199 | 2,493 | 40.63% |
| Aspirin · Circumstance table | 23 | 1,854 | 972 | 47.57% |
| Guideline I.A.12.a · Excludes1 | 11 | 972 | 673 | 30.76% |
| POA Y · Present on admission | 2 | 106 | 117 | -10.38% |

Each context includes all facts on its exact subject and every recursive `parent` / `seventh_character_of` ancestor, across all categories. It does not expand children, reverse index links or clinical guideline applicability. The aspirin selection is the top-level “Aspirin (aluminum) (soluble)” index subject, not every aspirin descendant. These six fixed examples can overlap and do not describe a representative retrieval workload. The 8,192-token allowance is illustrative, not a model limit or recommendation.

To reproduce a selection, use its `subject`, `subjects` and `fact_ids` in [token-effectiveness.json](token-effectiveness.json). Union all recorded facts, order by `(s,c,p,id)`, and serialize with named FFL 3, including the exact `ffl_comments` and integrity footer. The equivalent FFL 2 uses the same comments; JSONL contains the same seven canonical fields without FFL framing. The report gives exact hashes, bytes and tokens for all three representations. Contexts are measurement views; the 41 primary FFLs remain the only delivered fact set.

## Machine-readable data and verification

| File | Coverage |
|---|---|
| [token-effectiveness.json](token-effectiveness.json) | Complete methodology, tokenizer fingerprint, totals, category/family counts, baselines, context fact IDs and all hashes. |
| [token-formats.csv](token-formats.csv) | Three whole-set representations. |
| [token-categories.csv](token-categories.csv) | All 41 primary categories, raw counts, rates and exact file/baseline hashes. |
| [token-families.csv](token-families.csv) | Seven category families, using weighted counts. |
| [token-contexts.csv](token-contexts.csv) | Six subject/ancestor selections and their measured costs. |
| [token-validation.json](token-validation.json) | Independent full recount, fact equivalence, selection reconstruction and figure/data checks. |

The independent verifier uses public `encode_ordinary` token-ID lists instead of the producer's buffer counter. It freshly compares all 809,979 delivered facts to the original FFL, checks all 98,186 registry identifiers and both canonical digests, recounts 123 complete category representations, and reconstructs each context from the actual FFL ancestry assertions. It verifies every CSV cell, 192 tagged numerical labels, 56 proportional bars, SVG metadata and PNG dimensions/hashes. All 374 token-figure text lines passed canvas and collision checks, followed by raster review.

The [original knowledge validation boundary](../validation/extracted-knowledge.json) remains in effect. These statistics verify representation efficiency and preservation of the extracted facts; they do not certify source completeness or clinical reasoning accuracy. Use [the root checksums](../CHECKSUMS.sha256) to detect changes to the delivered files.
