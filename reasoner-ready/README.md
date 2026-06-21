# AMT Ontology — reasoner-ready bundle

Two self-contained OWL/RDF artifacts that any OWL reasoner can load with
**no network access** and immediately reason over:

| File | Pick this if your reasoner is | Trade-off |
|---|---|---|
| `amt-ontology-full.owl` | **Pellet** (recommended), ELK | Faithful to source: `xsd:date` and `rdf:langString` preserved |
| `amt-ontology-full-hermit.owl` | **HermiT** | Lossy: `xsd:date` → `xsd:dateTime` (midnight UTC), `rdf:langString` → `xsd:string` (language tags dropped) |

Same triples otherwise. Distinct ontology IRIs
(`http://ontology.amt.org/amt-ontology-full` vs `…-hermit`) so a downstream
tool can tell which one it loaded.

## What's inside (both variants)

Built by `publish/build_reasoner_ready.py`, each file is the union of:

- All authored AMT schema TTLs (`amtmeta`, `pc`, `im`, `ind`).
- All internal CodeSets (`ind_codeset`, `visitor_pc_codeset`, `exhibitor_pc_codeset`, `lead_pc_codeset`, `ex_raw_pc_codeset`, `imts_regclass_codeset`, `jobfn_codeset`, `acqrole_codeset`, `empcount_codeset`).
- Both external CodeSets (`pc_jimtof_cs`, `ind_jimtof_cs`).
- The 6 pseudonymized fixture TTLs (`shows`, `companies`, `imts_2024_visitor_records`, `imts_2024_exhibitor_records`, `imts_2024_lead_records`, `jimtof_2024_exhibitor_records`).
- gUFO (vendored at `ontology/imports/gufo.ttl`).
- The 6 OMG Commons modules vendored at `ontology/imports/cmns-*.rdf` (AnnotationVocabulary, TextDatatype, Collections, Designators, CodesAndCodeSets, Classifiers).

All `owl:imports` triples have been stripped — the reasoner has every
triple it needs locally.

## Not included

- SHACL shapes (`shapes/codeset-shapes.ttl`). SHACL is for validation, not
  DL reasoning, and importing the SHACL vocabulary historically tripped
  HermiT on a strict-anyURI literal. Use the standalone shapes file with
  pyshacl or a SHACL processor.
- The lean `amt-ontology.owl` companion artifact (still published next to
  this directory in dist-repo). That file keeps `owl:imports` to the OMG
  Commons URIs and expects the reasoner to fetch them at load time.

## Reasoner choice

**Pellet** is the recommended OWL DL reasoner — load `amt-ontology-full.owl`.
The ontology stays inside OWL 2 DL (a few `min 0` cardinality constraints
from OMG Commons sit just outside OWL 2 RL — Pellet handles them, and so
does the broader OWL 2 DL spec). Pellet accepts `xsd:date` and
`rdf:langString` natively, so no semantic rewriting is required.

**HermiT** — load `amt-ontology-full-hermit.owl`. HermiT 1.4.x has a
hardcoded `DatatypeRegistry` that omits `xsd:date` and `rdf:langString`
(both in OWL 2 Full / W3C-standard but excluded from OWL 2 DL's default
datatype map). The `-hermit` variant losslessly rewrites those datatypes
to ones HermiT does support:

- `"YYYY-MM-DD"^^xsd:date` → `"YYYY-MM-DDT00:00:00Z"^^xsd:dateTime`
- `"text"@en` → `"text"^^xsd:string` (language tag dropped)
- Schema axioms (`rdfs:range xsd:date`, `owl:onDatatype rdf:langString`)
  are rewritten in parallel so HermiT's preprocessing doesn't trip on
  them either.

The midnight-UTC time component and the language-tag-loss are
*conventions of the bundle*, not facts recovered from the source. If
your analysis needs date-without-time semantics or locale-aware labels,
load `amt-ontology-full.owl` with Pellet, or query against the lean
`amt-ontology.owl` (which preserves authored lexical forms).

Separately, every `xsd:dateTime` literal that the source TTLs emit
without a timezone designator gets `Z` (UTC) appended at bundle-build
time (both variants), so HermiT accepts those too. Opt out via
`--no-normalize-datetimes`.

**ELK** — load `amt-ontology-full.owl`. ELK does TBox-only reasoning (it
ignores ABox individuals and several DL-specific axioms); useful for
fast subsumption queries but won't answer "who's interested in additive"
style questions.

## Loading

### Python / `owlready2`

```python
from owlready2 import get_ontology, sync_reasoner_pellet, sync_reasoner_hermit

# Pellet (recommended) -- faithful semantics
onto = get_ontology("amt-ontology-full.owl").load()
with onto:
    sync_reasoner_pellet(infer_property_values=True)

# HermiT -- use the -hermit variant
onto = get_ontology("amt-ontology-full-hermit.owl").load()
with onto:
    sync_reasoner_hermit()
```

### Protégé

Open `amt-ontology-full.owl` for Pellet, or `amt-ontology-full-hermit.owl`
for HermiT. Reasoner menu → choose the matching reasoner → Start
Reasoner. No catalog file is needed.

### HermiT CLI

```
java -jar HermiT.jar --consistency file:///path/to/amt-ontology-full-hermit.owl
```

Expected output: `owl:Thing is satisfiable.` (HermiT's positive answer
to `--consistency`).

### rdflib + pyshacl (validation, no DL inference)

```python
import rdflib, pyshacl
data = rdflib.Graph(); data.parse("amt-ontology-full.owl")
shapes = rdflib.Graph(); shapes.parse("amt-ontology-shapes.ttl")
conforms, _, report = pyshacl.validate(data, shacl_graph=shapes)
```

## Provenance

`merged version`: max of per-file `owl:versionInfo`. Source file count
and triple count are baked into the bundle's `rdfs:comment`.

The bundle ontology IRI is `http://ontology.amt.org/amt-ontology-full`
— distinct from `http://ontology.amt.org/amt-ontology` (the lean dist
artifact) so a downstream tool can disambiguate.
