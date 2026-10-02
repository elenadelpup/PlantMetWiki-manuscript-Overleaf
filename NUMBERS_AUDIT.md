# PlantMetWiki manuscript — number audit

Every number in the bioRxiv preprint (doi:10.64898/2026.07.22.733699) checked against the
live SPARQL endpoint `https://plantmetwiki.bioinformatics.nl/sparql` and against internal
arithmetic. Endpoint state at time of audit: **31,818,800 triples**.

Status key: **[FIXED]** corrected in `main.tex` · **[FLAG]** needs your decision or a rebuild

---

## 1. The triple-count puzzle — root cause found  **[FLAG — most important]**

Three different totals appear: `~28.2M` (Fig. 1 caption), `6,500,299` (the seven named
graphs as listed in the Fig. 1 box), and `31,818,800` (live).

**Cause:** the deployed Virtuoso holds the **entire NCBITaxon ontology in the default graph**
`http://rdf-plantmetwiki.bioinformatics.nl/` — 25,294,994 triples containing **2,708,804 taxon
classes** (all of cellular life) — *in addition to* the 17,707-triple MIREOT subset in
`graph/ncbitaxon` (1,181 taxon classes).

```
6,500,299  (seven named graphs, as listed)
+ 21,719,262  (full NCBITaxon, as the paper states its size)
= 28,219,561  ≈ the "28.2 million" in the Fig. 1 caption
```

So the paper's own headline figure already includes the full ontology — while the Methods say
MIREOT subsetting was *"required for hosting this data in the Virtuoso triplestore"*, which implies
it is not hosted. It is. The live base graph is 25.3M rather than 21.7M, so a newer NCBITaxon
release is loaded than the one described.

This also matters for the FAIR table: an undeclared 25M-triple graph is absent from the VoID,
which is in tension with the R1.2 "detailed provenance" claim — and this is a FAIR-themed collection.

**Options**
1. **Drop** the full ontology from the default graph. Total becomes ~6.5M, the Methods narrative
   becomes true as written, and the MIREOT rationale stands. Check first that no example query or
   UI feature silently depends on the full ontology for label resolution.
2. **Keep** it, declare it as an eighth named graph with its own VoID entry, report ~31.8M, and
   reword the MIREOT sentence to "used to extract the lineage subset exposed in `graph/ncbitaxon`"
   rather than "required for hosting".

Either way, state one triple count for one defined scope, and say which.

---

## 2. Build drift: paper reports an older build than is deployed  **[FLAG]**

| Graph | Paper | Live | Δ |
|---|---|---|---|
| `graph/pathways` | 3,826,567 | 3,847,644 | +21,077 |
| `graph/gpml-properties-extra` | 2,617,839 | 2,617,839 | ✓ |
| `graph/gpml-taxonomy-extra` | 30,176 (Results) / 30,178 (Fig. 1) | **30,178** | see §3.4 |
| `graph/ncbitaxon` | 17,707 | 17,707 | ✓ |
| `graph/bgc-plantismash` | 6,030 | 5,965 | −65 |
| `graph/bgc-mibig` | 1,813 | 1,770 | −43 |
| BGC triples total | 7,843 | 7,735 | −108 |
| `void` | 165 | 213 | +48 |

**Solution:** regenerate every count from one tagged build and cite that build's Zenodo DOI in the
Methods. Until then the preprint values are retained in `main.tex` — they are self-consistent with
each other, just not with today's endpoint.

---

## 3. Errors corrected in `main.tex`

### 3.1 *A. thaliana* pathway coverage — "89%" **[FIXED]**
Preprint: *"direct annotations on 1,038 of 1,162 pathways (89%)"*.

Verified live: 1,038 is the count across **all 2,478 pathway instances** (PC + RC). Restricted to
the 1,162 PC pathways the figure is **679**.

```
679 / 1,162   = 58.4%   ← pathways
1,038 / 2,478 = 41.9%   ← pathway instances
1,038 / 1,162 = 89.3%   ← the published figure: PC+RC numerator over a PC-only denominator
```

Now reads: *"679 of the 1,162 PlantCyc pathways (58%) … counting the 1,316 standalone reaction
representations as well, 1,038 of the 2,478 pathway instances."* Please confirm which framing
you prefer — 58% is a weaker-sounding but defensible number.

### 3.2 BridgeDb InChIKey agreement — self-contradiction **[FIXED]**
Preprint states agreement *"for 2,904 of the 2,904 metabolites carrying both (100%)"* and then
describes 4 stereo-variants and 3 structurally unrelated cofactors. Both cannot hold.
Corrected to **2,897 of 2,904 (99.8%)** exact. The "0.14% stereo-variants (4/2,904)" figure is
unaffected and correct.

### 3.3 BGC crosslinks: 199 vs 225 **[FIXED in text; figure needs regenerating]**
Live confirms the Results: **225** crosslinks, 44 BGCs, 58 genes, 132 pathways — exact.
The "199" in the Fig. 1 box is stale.

### 3.4 Taxonomy graph: 30,176 vs 30,178 **[FIXED]**
Live = **30,178** (27,700 `wp:organism` + 2,478 `foaf:page`). Figure 1 was right, Results wrong.

### 3.5 `wp:DirectedInteraction` described as a fallback **[FIXED]**
Preprint: *"translated to wp:DirectedInteraction (44,934, 82%) as a fallback type when no more
specific class applies."*

It is the **parent class**, not a fallback. Of 44,934, **44,917 (99.96%) also carry a specific
subtype**; only **17** are DirectedInteraction-only. The genuinely untyped residue is **9,796
interactions (17.9%)**.

The internal tell: the subtype percentages sum to 82.2%, and adding DirectedInteraction's 82%
gives 164% — impossible for mutually exclusive categories.

### 3.6 Typo **[FIXED]**
*"covers the remaining ing eight of 15 reactions"* → "the remaining eight".

---

## 4. Not errors, but a reader checking the endpoint will think they are

### 4.1 GPML-file counts vs RDF-graph counts  **[clarified in text]**
Three numbers are counted across all 2,478 GPML files, while neighbouring numbers are unique IRIs
in the graph:

| Preprint | Means | Graph-level equivalent |
|---|---|---|
| "33 of 23,449 nodes" | GPML node entries | 11,155 unique DataNodes |
| "10,583 → 10,585 annotated Protein nodes" | GPML annotation entries | 3,750 Protein IRIs (3,712 annotated) |
| "18,762 species-annotated DataNodes" | GPML annotation entries | 6,377 nodes / 27,700 `wp:organism` triples |

Qualifiers ("GPML-level", "entries across all GPML files") have been added. Worth stating the
convention once in the Methods instead.

### 4.2 Case-study reaction counts need the dedup rule stated **[clarified in text]**
Species counts match the endpoint exactly. Conversion counts do not, because the paper collapses
GPML branch-point duplicates — stated only parenthetically for capsaicin.

| Pathway | Paper reactions | Raw `wp:Conversion` | Species (paper = live) |
|---|---|---|---|
| Capsaicin PWY-5710 (PC629) | 7 | 19 | 9 ✓ |
| Hyoscyamine PWY-7341 (PC953) | 16 | 44 | 13 ✓ |
| Avenacin PWY-7476 (PC241) | 13 | 30 | 15 ✓ |
| Monolignol PWY-361 (PC504) | 15 | 45 | 29 ✓ |

A sentence stating the rule once now opens that subsection. **Better still:** publish the
deduplication query alongside the notebooks so the 7/16/13/15 are reproducible.

### 4.3 "4,085 literature references" **[clarified in text]**
Correct for references attached to **pathway nodes**. The graph holds **5,722** distinct PubMed
references in total. Now reads "…on pathway nodes".

### 4.4 439 vs 424 taxa  **[FIXED in caption; figure artwork still says 439]**
Both are right: **439** source taxa in PlantCyc, **424** reaching the RDF taxonomy graph (live:
424 including Viridiplantae, 423 species-level). The Fig. 1 caption said the NCBITaxon graph covers
*"all 439 NCBI taxa"* while the Fig. 1 box says *"424 seed taxa"*. Live MIREOT graph = 1,181 classes
= 424 seeds + ancestors.

The caption now reads "the 424 taxa represented in the RDF, plus their ancestors up to the ontology
root (1,181 taxon classes in total)". **The artwork in `figure1.png` still shows 439 in the caption
area and 199 BGC crosslinks** — both need redrawing (see §8).

---

## 5. Citation errors inherited from the preprint  **[FLAG]**

Three references are each used correctly once and incorrectly once — the signature of a
reference-manager renumbering slip.

| Ref | Actually is | Correct use | **Incorrect use** |
|---|---|---|---|
| 32 | Alexander, VoID vocabulary | VoID metadata (l. 427) | atropine's naming (l. 501) |
| 33 | Leveau, oat C-21β oxidase | avenacin A-1 (l. 525) | "identifiers, such as InChIKeys" (l. 643) |
| 35 | Huang, caffeine convergent evolution | methylxanthine evolution (l. 738) | PubChem (l. 645) |

**All three are now resolved in `main.tex`:**
- atropine — citation dropped (the sentence is a statement of etymology and needs no reference)
- InChIKey — now cites `heller2015` (Heller *et al.* 2015, *J Cheminform*, the InChI/InChIKey paper)
- PubChem — now cites `kim2025` (Kim *et al.*, *PubChem 2025 update*, *Nucleic Acids Res*)

Seven entries have been **added** to `references.bib`: the five the preprint cites but the bib was
missing (`owen2025`, `brickley_foaf`, `martens2020`, `smith2007`, `huangr2016`) plus the two new
ones above. **All 58 cited keys resolve, with no unused entries.**

Please sanity-check `heller2015` and `kim2025` against your reference manager before submission —
I chose the canonical paper for each resource, but you may prefer a different PubChem year.

---

## 6. Author order  **[FLAG — your decision]**

Author list: Del Pup, Muller, **Martens, Willighagen**, Medema, Slenter, van der Hooft.
Contributions paragraph: … **Willighagen, Martens** …

The two orders disagree. The contributions paragraph is currently left as in the preprint.

---

## 7. Verified exact — no action needed

Checked against the live endpoint and matching to the digit:

- 1,162 PC pathways · 1,316 RC reactions · 2,478 pathway instances
- 2,758 GeneProduct · 3,750 Protein · 4,577 Metabolite · 70 Complex · 11,155 unique DataNodes
- 54,800 Interactions and **every** subtype: 19,927 / 12,359 / 8,280 / 3,486 / 865 / 70,
  and every percentage derived from them
- 225 BGC crosslinks · 44 BGCs · 58 genes · 132 pathways
- 2,656/2,758 gene-product and 3,712/3,750 enzyme species coverage
- 4,111 InChIKey-annotated metabolites
- All BridgeDb per-database counts: 3,059 / 2,926 / 2,805 / 2,317 / 1,120 / 975 / 510, and 2,968 bdbInChIKey
- *A. thaliana*: 1,121 gene products, 1,325 enzymes (exact); catalysis chain within 0.1%
  (live 4,727 / 2,318 / 1,351 vs published 4,732 / 2,323 / 1,354 — the small difference is the
  gene→protein `wp:TranscriptionTranslation` hop; worth pinning the definition in the notebook)
- Every percentage in the annotation matrix (29,061 = 7,012 + 13,922 + 8,127; 63.1% / 36.9%;
  21.0 / 29.8 / 49.2 → 50.8%) is internally consistent
- Every percentage in the Wikidata analysis (4,111 = 2,876 + 814 + 421; the pathway-tier and
  species-tier breakdowns) is internally consistent


---

## 8. Figures  **[DONE — with two caveats]**

All six main figures are now included and the document references them correctly.

| Manuscript | Source | Format |
|---|---|---|
| Figure 1 | `figure1.png` (as supplied) | PNG |
| Figure 2 | `figure2.png` (as supplied) | PNG |
| Figure 3 | `figure3.png` (as supplied) | PNG |
| Figure 4 | `figure4a.pdf` + `figure4b.pdf` — composite, panels A–B above C–E | **vector PDF** |
| Figure 5 | `figure5.pdf` ← `bridgedb_identifier_complementation.pdf` | **vector PDF** |
| Figure 6 | `figure6.pdf` ← `wikidata_coverage_overlap.pdf` | **vector PDF** |

Sources traced to the notebooks in `gpml-to-rdf/notebooks/figures/output/figures/`:
Figure 4 = `metabolic_space_estimate` (nb 04, panels A–B) stacked above
`inferable_gap_species_contribution` (nb 04, panels C–E), matching the preprint's two-page layout.

Two files in the same directory are **supplementary**, not main figures, and are cited as such in
the text: `metabolite_identifier_coverage` → Fig. S18, `absence_by_species_breadth` → Fig. S19.

Filenames were normalised (`Figure 1.png` → `figure1.png`): spaces in `\includegraphics` paths are
a common cause of Overleaf build failures.

**Caveat 1 — stale artwork.** `figure1.png` still shows **199** BGC crosslinks (should be 225) and
**439** taxa for the NCBITaxon graph (should be 424), and its triple counts are from the older build
(§2). The caption is now correct but the image is not. It needs redrawing.

**Caveat 2 — not compiled.** There is no LaTeX toolchain on this machine, so the document has been
validated statically only: balanced braces and environments, no bare `<`/`>` outside `\texttt`/math,
every `\ref` resolving to a `\label`, every `\cite` resolving to a bib entry, and every
`\includegraphics` target existing on disk. All pass. Please run one compile in Overleaf.

Figure 4 stacks two full-width panels in one float; if it overflows the page, add
`[p]` to that `figure` environment or trim the `\\[1.2em]` spacer.

---

## 9. Commands for the two things that need your decision

### 9.1 Confirm the full-ontology situation (§1)

Verify the default graph is what this audit says it is — paste into the SNORQL UI:

```sparql
SELECT ?g (COUNT(*) AS ?triples) WHERE { GRAPH ?g { ?s ?p ?o } }
GROUP BY ?g ORDER BY DESC(?triples)
```

Expected: `http://rdf-plantmetwiki.bioinformatics.nl/` ≈ 25.3M. Then count its taxon classes:

```sparql
SELECT (COUNT(DISTINCT ?t) AS ?taxa) WHERE {
  GRAPH <http://rdf-plantmetwiki.bioinformatics.nl/> { ?t a <http://www.w3.org/2002/07/owl#Class> }
  FILTER(STRSTARTS(STR(?t), "http://purl.obolibrary.org/obo/NCBITaxon_"))
}
```

Expected: 2,708,804 — i.e. all of cellular life, not the 1,181 in `graph/ncbitaxon`.

**Before dropping it**, check nothing depends on it. If this returns labels, something resolves
through the default graph rather than `graph/ncbitaxon`:

```sparql
SELECT ?label WHERE {
  GRAPH <http://rdf-plantmetwiki.bioinformatics.nl/> {
    <http://purl.obolibrary.org/obo/NCBITaxon_3702> <http://www.w3.org/2000/01/rdf-schema#label> ?label }
} LIMIT 1
```

To drop (Virtuoso `isql`, **irreversible — take a backup first**):

```sql
SPARQL CLEAR GRAPH <http://rdf-plantmetwiki.bioinformatics.nl/>;
```

Then re-run the per-graph count; the total should land near 6.5M. Re-run the Snorql-UI example
queries and the taxonomy tutorial page afterwards to confirm labels still resolve.

If you keep it instead: add it to the VoID as its own `void:Dataset` with the NCBITaxon release
version and license, and change the Methods sentence from "required for hosting this data in the
Virtuoso triplestore" to something like "used to extract the lineage subset exposed in
`graph/ncbitaxon` for tree navigation".

### 9.2 Regenerate the counts from one build (§2)

`gpml-to-rdf` already has the right script:

```bash
cd gpml-to-rdf
conda run -n plantmetwiki-rdf python scripts/export_named_graph_metadata.py
```

That writes the per-graph triple counts and provenance table. Reconcile it against
`notebooks/figures/output/named_graphs_metadata.csv`, then update Supplementary Table S6, the
Figure 1 box, and the Results paragraph from that one table — and cite that build's Zenodo DOI
in the Methods so the numbers are pinned to something a reader can fetch.

### 9.3 Author order (§6)

One-line decision. If the author list is right, swap the two sentences in the Author contributions
paragraph so Martens precedes Willighagen; if the contributions paragraph is right, swap the names
in the author list and renumber the affiliation superscripts.
