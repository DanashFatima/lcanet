# LCAnet

LCAnet is a modular ontology for representing Life Cycle Assessment (LCA) knowledge, including activities, flows, actors, spatiotemporal aspects, properties, and impact assessment results.

## Repository Structure

- `core/` — core ontology concepts and upper-level relations
- `activity/` — activities, processes, and flow relations
- `actor/` — stakeholders and organizational entities
- `st/` — spatiotemporal concepts and scales
- `property/` — properties and quality values
- `ia/` — impact assessment concepts and results
- `ICdata/` — populated use case and example data
- `LCAnet-architecture.svg` — ontology architecture diagram
- `CQs.md` — competency questions and SPARQL validation queries

## Competency Questions

The competency questions (CQs) used to validate the ontology and their corresponding SPARQL queries are available in:

- [`CQs.md`](./CQs.md)

## Usage

The ontology modules can be imported into RDF/OWL tools such as:

- Protégé
- GraphDB

SPARQL queries can be executed on the populated dataset provided in `ICdata/`.

## License

This work is licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).
