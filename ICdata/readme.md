# LCAnet — IC Manufacturing Knowledge Graph (StorAIge Use Case)

Part of the **LCAnet ontology network** for Life Cycle Assessment.  
🔗 Network landing page: https://w3id.org/lcanet

## Overview
A populated LCAnet knowledge graph instantiating a real-world LCA 
study conducted on the StorAIge integrated circuit (IC) prototype 
within the SmartACV Carnot project. The study covers the research 
phase of the prototype lifecycle, focusing on raw material extraction 
and transformation/assembly stages, using the Environmental Footprint 
3.0 (EF 3.0) methodology.

## URI
https://w3id.org/lcanet/ICdata

## Contents
- 278 individuals instantiating all six LCAnet modules
- Inventory data sourced from CODDE (v3.00.000) and NegaOctet databases
- LCA study conducted using EIME software
- Inferred triples materialized using HermiT reasoner

## Modules Covered
| Module | Examples of instantiated classes |
|---|---|
| Core | LcaModel, Goal, Scope, LifeCycleStage |
| Activity | Activity, ComposedActivity, ProductFlow, EmissionFlow |
| Actor | Organization, Person, Role, Responsibility |
| Property | QuantitativeProperty, QualitativeProperty |
| Spatiotemporal | Location, AdministrativeRegion, SpatialScale |
| Impact Assessment | ImpactCategory, LcaResult, OverallImpactResult |

## Repository Structure
ICdata/
├── inputs/                       # Raw input data and Bill of Materials
├── ICdata-HermiT-inferred.rdf    # Populated knowledge graph with the inferred triples (RDF/XML)
└── ICdata.ttl                    # Asserted triples only (Turtle)
