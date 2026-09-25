# Visual atlas

Nine standalone SVG/PNG pairs. The current compact graph is shown first. Every PNG is at least 5,000 pixels wide; vectors have selectable text and embedded scope/data. No website or external asset is required to view them.

[![Compact category explorer](00-category-explorer.png)](00-category-explorer.svg)

| View | Vector | Raster |
|---|---|---|
| Compact category explorer | [SVG](00-category-explorer.svg) | [PNG](00-category-explorer.png) |
| Source-wide knowledge map | [SVG](01-knowledge-map.svg) | [PNG](01-knowledge-map.png) |
| Cross-chapter rule connections | [SVG](02-rule-connections.svg) | [PNG](02-rule-connections.png) |
| E11.9 neighborhood | [SVG](03-diabetes-neighborhood.svg) | [PNG](03-diabetes-neighborhood.png) |
| Aspirin circumstances | [SVG](04-drug-table.svg) | [PNG](04-drug-table.png) |
| Visual-acuity criteria | [SVG](05-visual-acuity.svg) | [PNG](05-visual-acuity.png) |
| Token efficiency | [SVG](06-token-efficiency.svg) | [PNG](06-token-efficiency.png) |
| Token cost by category | [SVG](07-token-profile.svg) | [PNG](07-token-profile.png) |
| Selected context costs | [SVG](08-context-token-cost.svg) | [PNG](08-context-token-cost.png) |

## Scope

- **00:** current category hierarchy and all 21,110 selected fact marks. Counts show selected / total facts. All 41 subcategories are visible; 72 predicate groups are retained in the CSV graph.
- **01–05:** retained research views of the source-wide knowledge and specific clinical structures. Their original graph counts describe the analysis used to create those views. The full entity graph is not shipped. Matching small view nodes/links/details remain in [data/](data/).
- **06–08:** equivalent-fact token comparisons, all-category costs and six illustrative subject/ancestor selections. See [measurements](../metrics/README.md).

The compact overview was rendered with D3 7.9 and Sharp/libvips. Its text and 21,110 fact marks passed measured bounds and collision checks and the PNG was reviewed visually. Each subcategory has a 50-column dot grid; each dot represents one exact selected fact. [Compact figure quality](compact-figure-quality.json), [retained knowledge figure quality](figure-quality.json) and [token figure quality](token-figure-quality.json) preserve their exact hashes. [Repository validation](../validation.json) checks all nine SVG/PNG pairs and compact count labels/fact marks.

For original view data, each file stem in `data/` has `.nodes.csv`, `.links.csv` and `.details.json`. These are small explanatory projections, separate from the category browsing graph.
