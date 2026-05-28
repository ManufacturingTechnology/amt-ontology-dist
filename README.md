# AMT Ontology -- Distribution

Published artifacts for the AMT Ontology Suite. Browse the ontology
interactively [here.](https://manufacturingtechnology.github.io/amt-ontology-dist/)
## Contents

- `index.html`, `browser.js`, `browser.css` -- the interactive
  ontology browser (information model, product categories, industries)
- `ontology/amt-ontology.owl` -- merged OWL distribution (RDF/XML)
- `ontology/ttl/*.ttl` -- authored Turtle source modules:
  amtmeta, pc, im, ind, and the three AMT codeset vocabularies
  (`ind_codeset`, `visitor_pc_codeset`, `exhibitor_pc_codeset`)
- `shapes/amt-ontology-shapes.ttl` -- SHACL shapes for validating
  consumer data against the AMT model
- `taxonomies/*.xlsx` -- Excel exports of the IMTS Product Interest,
  Exhibitor Product, and Industry Sector taxonomies

## Provenance

This repository is generated. Source-of-truth, engineering history,
and the build pipeline live in a private companion repository owned
by AMT. Issues + corrections should be filed there; this repository
does not accept pull requests directly against generated artifacts.
