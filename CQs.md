# LCAnet — Competency Questions and SPARQL Queries

This document lists the competency questions (CQs) defined for the LCAnet ontology network, together with a brief description and the corresponding SPARQL query evaluated over the StorAIge IC manufacturing knowledge graph.

## Prefix Declarations

All queries use the following prefix declarations:

```sparql
PREFIX core:  <https://w3id.org/lcanet/core#>
PREFIX act:   <https://w3id.org/lcanet/activity#>
PREFIX actor: <https://w3id.org/lcanet/actor#>
PREFIX prop:  <https://w3id.org/lcanet/property#>
PREFIX st:    <https://w3id.org/lcanet/st#>
PREFIX ia:    <https://w3id.org/lcanet/ia#>
PREFIX ic:    <https://w3id.org/lcanet/ICdata#>
PREFIX qudt:  <http://qudt.org/schema/qudt/>
PREFIX rdfs:  <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:   <http://www.w3.org/2001/XMLSchema#>
```

---

## Activity-Centered Questions

---

### CQ1 — Input and output flows of activities

**Question:** What are the input and output flows of all activities?

**Description:** Retrieves all activities in the knowledge graph together with their associated input and output flows, distinguishing the direction of each flow. Covers the Activity module.

```sparql
SELECT ?activity ?activityLabel ?flow ?flowType
WHERE {
  ?activity a act:Activity .
  OPTIONAL { ?activity rdfs:label ?activityLabel . }
  {
    ?activity act:hasInputProduct ?flow .
    BIND("input" AS ?flowType)
  }
  UNION
  {
    ?activity act:hasOutputFlow ?flow .
    BIND("output" AS ?flowType)
  }
}
ORDER BY ?flowType ?flow
```

---

### CQ2 — Activity dependency chain

**Question:** What is the dependency chain of activities in a system model?

**Description:** Retrieves the direct and recursive upstream dependency chain of a given activity by traversing flow connectivity relations. Demonstrates LCAnet's ability to navigate complex activity/flow diagrams using SPARQL property paths. Covers the Activity module.

**Part 1 — Direct dependency:**

```sparql
SELECT ?activity ?downstreamActivity ?inputFlow
WHERE {
  ?activity a act:Activity ;
            rdfs:label "Production of Mother Board" ;
            act:hasInputProduct ?inputFlow .
  ?inputFlow act:connected ?outputFlow .
  ?outputFlow act:isOutputProductOf ?downstreamActivity .
}
```

**Part 2 — Recursive dependency (full chain):**

```sparql
SELECT ?activity ?dependentActivity
WHERE {
  ?activity a act:Activity ;
            rdfs:label "Production of Mother Board" ;
            act:hasInputProduct ?inputFlow .
  ?inputFlow act:connected ?outputFlow .
  ?outputFlow act:isOutputProductOf ?intermediateActivity .
  ?intermediateActivity
    ( act:hasInputProduct /
      act:connected       /
      act:isOutputProductOf )* ?dependentActivity .
}
```

---

## Flow / Exchange Questions

---

### CQ3 — Flow dataset provenance

**Question:** Which flow datasets come from database X?

**Description:** Retrieves all flows whose provenance metadata (stored as a QualitativeProperty individual) indicates a specified data source. Covers the Activity and Property modules.

```sparql
SELECT ?flow ?flowLabel
WHERE {
  ?flow core:hasProperty ?x .
  ?x    rdfs:label           "Database_Source" ;
        prop:hasQualityValue "CODDE (v3.00.000)" .
  OPTIONAL { ?flow rdfs:label ?flowLabel . }
}
```

---

### CQ4 — Elementary and intermediate flows of an activity

**Question:** What are the elementary and intermediate flows associated with a given activity?

**Description:** Retrieves all flows associated with a specific activity that are classified as elementary or intermediate, based on their environment type or explicit typing. Covers the Activity module.

```sparql
SELECT ?flow ?flowLabel
WHERE {
  ?flow a act:Flow ;
        act:isOutputFlowOf ?activity .
  ?activity rdfs:label "Production of Mother Board" .
  OPTIONAL { ?flow rdfs:label ?flowLabel . }
  {
    ?flow act:hasEnvironmentType ?x .
    ?x a act:Biosphere .
  }
  UNION
  {
    ?flow a act:ElementaryFlow .
  }
}
```

---

### CQ5 — Flow classification

**Question:** Which flows are classified as resources, emissions, or products?

**Description:** Retrieves all flows in the knowledge graph and identifies their classification type (elementary, intermediate, emission, or resource), together with their associated environment compartment where applicable. Covers the Activity module.

```sparql
SELECT DISTINCT ?flow ?flowName ?flowType ?source
WHERE {
  ?flow a act:Flow .
  OPTIONAL { ?flow rdfs:label ?flowName . }

  OPTIONAL {
    ?flow a act:IntermediateFlow ;
          act:hasEnvironmentType ?intEnv .
    BIND("Intermediate" AS ?flowType)
    BIND(?intEnv AS ?source)
  }
  OPTIONAL {
    ?flow a act:ElementaryFlow .
    BIND("Elementary" AS ?flowType)
  }
  OPTIONAL {
    ?flow a act:EmissionFlow ;
          act:hasEnvironmentType ?emEnv .
    BIND(CONCAT("Emission to ", STR(?emEnv)) AS ?source)
  }
  OPTIONAL {
    ?flow a act:ResourceFlow ;
          act:hasEnvironmentType ?resEnv .
    BIND(CONCAT("Resource from ", STR(?resEnv)) AS ?source)
  }
}
ORDER BY ?flow
```

---

## System Model & Scenario Questions

---

### CQ6 — LCA study metadata

**Question:** What is the functional unit, scope, system boundary, spatiotemporal scales, and responsible agent of the LCA model?

**Description:** Retrieves the full metadata description of an LCA study, including its goal, life cycle stages, functional unit, reference flow, system boundary, temporal and spatial scales, and assumptions. Covers the Core module.

```sparql
SELECT ?lcaModel
       (SAMPLE(?goal)                                    AS ?goal)
       (GROUP_CONCAT(DISTINCT ?lcStage; separator=", ") AS ?lcStages)
       (SAMPLE(?functionalUnit)                          AS ?functionalUnit)
       (SAMPLE(?referenceFlow)                           AS ?referenceFlow)
       (GROUP_CONCAT(DISTINCT ?ass;    separator=" | ") AS ?assumptions)
       (SAMPLE(?systemBoundary)                          AS ?systemBoundary)
       (SAMPLE(?temporalScale)                           AS ?temporalScale)
       (SAMPLE(?spatialScale)                            AS ?spatialScale)
WHERE {
  ?lcaModel a core:LcaModel ;
            core:lcaModelConcernsLcStage ?lcStage ;
            core:lcaModelHasScope        ?scope .
  ?scope core:scopeHasFunctionalUnit  ?functionalUnit ;
         core:scopeHasReferenceFlow   ?referenceFlow .
  OPTIONAL {
    ?scope core:scopeHasTemporalScale   ?temporalScale ;
           core:scopeHasSpatialScale    ?spatialScale ;
           core:scopeHasSystemBoundary  ?systemBoundary ;
           core:scopeHasAssumption      ?ass .
  }
  OPTIONAL { ?lcaModel core:lcaModelHasGoal ?goal . }
}
GROUP BY ?lcaModel
```

---

## Impact Assessment Questions

---

### CQ7 — Impact results per activity or flow

**Question:** What is the impact result of a given activity or flow across impact categories?

**Description:** Retrieves impact result values associated with activities or flows, together with their corresponding impact categories. Covers the Impact Assessment and Core modules.

**For flows:**

```sparql
SELECT ?impactResultValue ?flow ?impactCategory
WHERE {
  ?impactResult a ia:ImpactResult ;
                qudt:numericValue                 ?impactResultValue ;
                core:impactResultConcernsFlow     ?flow ;
                ia:correspondsToIc                ?impactCategory .
}
```

**For activities:**

```sparql
SELECT ?impactResultValue ?activity ?impactCategory
WHERE {
  ?impactResult a ia:ImpactResult ;
                qudt:numericValue                  ?impactResultValue ;
                core:impactResultConcernsActivity  ?activity .
  OPTIONAL {
    ?impactResult ia:correspondsToIc ?impactCategory .
  }
}
```

---

### CQ8 — Activities contributing above threshold to an impact category

**Question:** Which activities contribute more than 2% to a given impact category?

**Description:** Computes the relative contribution of each activity to a specified impact category by comparing individual impact results against the overall impact result. Uses cross-module relations between the Activity, Impact Assessment, and Core modules. The `containedInOIR` relation is partially inferred via SWRL rules.

```sparql
SELECT ?activity ?impactResultValue ?percentage
WHERE {
  ?impactCategory a ia:ImpactCategory ;
                  rdfs:label "Climate change, total" .
  ?overallIR a ia:OverallImpactResult ;
               ia:correspondsToIc  ?impactCategory ;
               qudt:numericValue   ?overallImpactValue .
  ?impactResult a ia:ImpactResult ;
                  core:impactResultConcernsActivity  ?activity ;
                  ia:correspondsToIc                 ?impactCategory ;
                  qudt:numericValue                  ?impactResultValue ;
                  ia:containedInOIR                  ?overallIR .
  BIND(
    xsd:decimal(REPLACE(STR(?impactResultValue), ",", ".")) /
    xsd:decimal(REPLACE(STR(?overallImpactValue), ",", "."))
    AS ?percentage
  )
  FILTER(?percentage > 0.02)
  FILTER(?overallImpactValue != 0)
}
ORDER BY DESC(?percentage)
```

---

### CQ9 — Top contributing flows to an impact category for a given activity

**Question:** What are the top contributing flows to an impact category for a given activity?

**Description:** Retrieves the flows that are inputs to a given activity and ranks them by their relative contribution to a specified impact category, computed against the activity-level impact result. Covers the Activity, Impact Assessment, and Core modules.

```sparql
SELECT ?flow ?activity ?flowImpactResult ?flowImpactValue ?percentage
WHERE {
  ?impactCategory a ia:ImpactCategory ;
                  rdfs:label "Climate change, total" .
  ?activityImpactResult a ia:ImpactResult ;
                          core:impactResultConcernsActivity ?activity ;
                          ia:correspondsToIc                ?impactCategory ;
                          qudt:numericValue                 ?activityImpactValue .
  ?activity rdfs:label "Production of Daughter Board" .
  ?flow act:isInputFlowOf ?activity .
  OPTIONAL {
    ?flow core:flowHasImpactResult ?flowImpactResult .
    ?flowImpactResult qudt:numericValue ?flowImpactValue .
  }
  BIND(
    IF(BOUND(?flowImpactValue),
       xsd:decimal(REPLACE(STR(?flowImpactValue),        ",", ".")) /
       xsd:decimal(REPLACE(STR(?activityImpactValue), ",", ".")),
       0
    ) AS ?percentage
  )
}
ORDER BY DESC(?percentage)
```

---

## Data Quality / Metadata Questions

---

### CQ10 — Flows associated with a given methodology

**Question:** Which flows are associated with a given assessment methodology?

**Description:** Retrieves all flows linked to a specified impact assessment methodology, enabling data quality checks and traceability of inventory data to their methodological context. Covers the Impact Assessment and Activity modules.

```sparql
SELECT ?flow ?flowLabel ?methodology
WHERE {
  ?methodology a ia:Methodology ;
               rdfs:label "EF 3.0" .
  ?impactResult a ia:ImpactResult ;
                  ia:usesMethodology               ?methodology ;
                  core:impactResultConcernsFlow    ?flow .
  OPTIONAL { ?flow rdfs:label ?flowLabel . }
}
```

---

## Summary Table

| ID | Theme | Competency Question | Modules |
|---|---|---|---|
| CQ1 | Activity | What are the input and output flows of all activities? | act |
| CQ2 | Activity | What is the dependency chain of activities in a system model? | act |
| CQ3 | Flow | Which flow datasets come from database X? | act, prop |
| CQ4 | Flow | What are the elementary and intermediate flows associated with an activity? | act |
| CQ5 | Flow | Which flows are classified as resources, emissions, or products? | act |
| CQ6 | System model | What is the functional unit, scope, system boundary, and spatiotemporal scales of the LCA model? | core |
| CQ7 | Impact Assessment | What is the impact result of a given activity or flow across impact categories? | ia, core |
| CQ8 | Impact Assessment | Which activities contribute more than 2% to a given impact category? | ia, core, act |
| CQ9 | Impact Assessment | What are the top contributing flows to an impact category for a given activity? | ia, act, core |
| CQ10 | Data quality | Which flows are associated with a given assessment methodology? | ia, act |

---

*LCAnet ontology network — https://w3id.org/lcanet*
