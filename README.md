# Cigars Ontology

A SKOS/OWL semantic model of the cigar domain — 283 defined concepts covering cigar
anatomy, tobacco origins, cultivation, production, aging and enjoyment — built with
[Lakeloom](#acknowledgements) following the Ontology Pipeline method, and published here
as versioned, checksum-verified RDF artifacts.

| | |
|---|---|
| Concepts | 283, all with definitions |
| Taxonomy | 10 thematic top-level categories, 273 broader/narrower links |
| Thesaurus | 27 associative (related) links between concept pairs, 116 alternative labels |
| Attributes | 6 datatype properties (ring gauge, length, smoking time, box code, aging, capacity) |
| Axioms | 6 — 5 exact-cardinality restrictions + 1 class disjointness (Parejo ⊥ Figurado) |
| External mappings | 18 links to Wikidata (16 `skos:exactMatch`, 1 `closeMatch`, 1 `relatedMatch`) |
| Shapes | 283 SHACL node shapes, 6 property shapes |
| Governance notes | scope and history notes on 4 concepts |
| Current release | **v2.1.0** |

## What's in this repository

```
ontology.ttl      the model as Turtle (concepts, taxonomy, thesaurus, attributes, axioms, mappings)
ontology.jsonld   the same as JSON-LD
ontology.owl      the same as RDF/XML (OWL)
skos.ttl          the SKOS view: everything above plus the governance annotations
                  (scope/history notes) that live in the pipeline's proposal log
shapes.ttl        SHACL shapes for every concept and attribute
context.jsonld    a JSON-LD @context for API/RAG consumers
deliverables/     human-readable exports of each pipeline stage:
                  glossary.md/csv · taxonomy.md/mmd/csv · synonyms.csv/txt · deprecations.csv
CHANGELOG.md      the full, governed release history
manifest.yaml     release provenance
SHA256SUMS        sha256 of every artifact in this release
LICENSE           CC BY 4.0
```

The base IRI of the model is `http://localhost:8000/ontology/cigars/` (the Lakeloom
instance it was built on); concept IRIs use the concept name, e.g.
`http://localhost:8000/ontology/cigars/Vitola`.

## How it was built

This model was not hand-written as RDF. It was grown stage by stage through
[Lakeloom](https://github.com/elcioabrahao/lakeloom), a semantic-modeling layer over
[OpenMetadata](https://open-metadata.org/) (v2.1.0-rc1) that implements the Ontology
Pipeline method — and, crucially, **governs every change**: nothing enters the model
outside a proposal, no proposal is applied without a human review, and every applied
state is captured by a semantic-versioned release.

### The pipeline stages

1. **Vocabulary.** 258 terms were harvested from a domain glossary into OpenMetadata as a
   glossary, then every term was given a definition; a curation round added 15 missing
   concepts, ending at 283 defined concepts.
2. **Taxonomy.** Ten thematic top-level categories (Cigar Anatomy, Cigar Components,
   Cigars, Cigar Production, Aging and Storage, Cigar Flavors and Aromas, Cigar Pairings,
   Cigar Accessories, Smoking, Tobaccos and Origins, Tobacco Cultivation) were created and
   all 273 non-root terms were re-parented under them.
3. **Thesaurus.** 27 associative (related-term) links were curated pairwise — e.g.
   Cameroon ↔ Wrapper tooth, Vuelta Abajo ↔ Wrapper — on top of the 273 hierarchical
   links, plus alternative labels (116 in the current release).
4. **Ontology.** Domain structure was added as measurable attributes (with units, and an
   identifier flag on box codes), exact-cardinality axioms, a disjointness axiom
   (a cigar shape is either a Parejo or a Figurado, never both), and external mappings
   linking origins, seed strains and the tobacco species to Wikidata entities
   (e.g. Nicotiana Tabacum → [Q181095](http://www.wikidata.org/entity/Q181095)).

Each stage produced a deliverable (`deliverables/`) and, at the thesaurus, a first
release. Every change — term, note, synonym, attribute, axiom, mapping — went through
the same loop:

```
draft proposal → submit → human review (approve/reject) → apply → release (semver, tagged)
```

The AI agent doing the modeling work could propose anything, but could never approve its
own proposals: reviews were signed by a human principal, and releases are computed,
tagged and checksummed by the pipeline, not by hand.

### Release history

| Version | What it captured |
|---|---|
| v1.0.0 | Stages 1–4: vocabulary, taxonomy, thesaurus |
| v1.1.0 | First ontology wave: 5 attributes, 5 axioms |
| v1.2.0 | Second wave: 18 Wikidata mappings, `Humidor.capacity`; first release carrying the disjointness axiom |
| v2.0.0 | A synonym update accidentally dropped one Wikidata mapping (a tool bug, since fixed) — the diff correctly forced a major bump |
| **v2.1.0** | Mapping restored via a governed proposal, with a history note recording the incident |

The full story, with the change set of every release, is in [CHANGELOG.md](CHANGELOG.md).

## Using the model

**Load it into a triple store** (any SPARQL 1.1 store with Graph Store support works;
this example uses Apache Jena Fuseki):

```bash
# into the default graph of a dataset called "cigars"
curl -u admin:admin -X POST "http://localhost:3030/cigars/data" \
     -H "Content-Type: text/turtle" --data-binary @ontology.ttl

# or as a named graph
curl -u admin:admin -X PUT "http://localhost:3030/cigars/data?graph=http://example.org/cigars" \
     -H "Content-Type: text/turtle" --data-binary @ontology.ttl
```

**Query it** — everything that is both a wrapper origin and maps to Wikidata:

```sparql
PREFIX cigars: <http://localhost:8000/ontology/cigars/>
PREFIX skos:   <http://www.w3.org/2004/02/skos/core#>

SELECT ?concept ?label ?wikidata WHERE {
  ?concept skos:broader cigars:Tobaccos%20and%20Origins ;
           skos:prefLabel ?label ;
           skos:exactMatch ?wikidata .
}
```

**Validate data against the shapes** with [pySHACL](https://github.com/RDFLib/pySHACL):

```bash
pip install pyshacl
pyshacl --data-file your-cigar-data.ttl --shapes-file shapes.ttl
```

**Use it from JSON** — `context.jsonld` is a ready-made `@context` so API and RAG
consumers can compact JSON-LD documents that use the model's terms.

**Read it as a human** — `deliverables/glossary.md` is the full annotated glossary,
`deliverables/taxonomy.md` (and `taxonomy.mmd`, a Mermaid diagram) show the hierarchy,
`deliverables/synonyms.csv` exports the thesaurus for search engines.

## Integrity

Every artifact of this release is checksummed in `SHA256SUMS`:

```bash
shasum -a 256 -c SHA256SUMS
```

`manifest.yaml` records the release provenance (model, version, generator).

## Acknowledgements

Built with [Lakeloom](https://github.com/elcioabrahao/lakeloom) on
[OpenMetadata](https://open-metadata.org/), following the method of Jessica Talisman's
book *Ontology Pipeline* ("The Ontology Pipeline" is her registered trademark). External
concept mappings link this model to [Wikidata](https://www.wikidata.org/).

## License

The model and all artifacts in this repository are licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — see [LICENSE](LICENSE).
