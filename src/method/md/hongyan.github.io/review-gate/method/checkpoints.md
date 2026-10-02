---
template:
  id: https://hongyan.github.io/review-gate/method/checkpoints
  name: "Checkpoints"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
    - id: target
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Checkpoints

A checkpoint belongs to one deployment and is staffed by one gauge. What it guards, and at what tolerance, sits on the link between the checkpoint and the error class rather than on the checkpoint, so the second table edits those links directly. A checkpoint that guards nothing is a gap in the process rather than malformed data, so the rule below only warns.

```table-editor
---
target: ${target}
columns: { this: { label: "Checkpoint" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix gate: <https://hongyan.github.io/review-gate/method/gate#> .

gate:CheckpointShape
    a sh:NodeShape ;
    sh:targetClass gate:Checkpoint ;
    sh:property [
        sh:path gate:hasId ;
        sh:name "Id" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path gate:label ;
        sh:name "Label" ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path gate:isDeployedIn ;
        sh:name "Deployment" ;
        sh:class gate:Deployment ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path gate:isOperatedBy ;
        sh:name "Operated by" ;
        sh:class gate:Gauge ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path gate:verdictRule ;
        sh:name "Verdict rule" ;
        sh:maxCount 1 ;
        sh:order 5 ;
    ] ;
    sh:sparql [
        sh:severity sh:Warning ;
        sh:message "This checkpoint guards no error class." ;
        sh:select """
            PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
            PREFIX oml: <http://opencaesar.io/oml#>
            SELECT $this WHERE {
                $this a gate:Checkpoint .
                FILTER NOT EXISTS { ?link a gate:Guards ; oml:hasSource $this }
            }
        """ ;
    ] ;
    .
```

## What each checkpoint guards

```table-editor
---
target: ${target}
columns: { this: { label: "Guard" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix oml: <http://opencaesar.io/oml#> .
@prefix gate: <https://hongyan.github.io/review-gate/method/gate#> .

gate:GuardsShape
    a sh:NodeShape ;
    sh:targetClass gate:Guards ;
    sh:property [
        sh:path oml:hasSource ;
        sh:name "Checkpoint" ;
        sh:class gate:Checkpoint ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path oml:hasTarget ;
        sh:name "Error class" ;
        sh:class gate:ErrorClass ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path gate:toleranceLabel ;
        sh:name "Tolerance" ;
        sh:datatype xsd:string ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path gate:toleranceMaxMissed ;
        sh:name "Max missed" ;
        sh:datatype xsd:integer ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path gate:toleranceSampleSize ;
        sh:name "Sample size" ;
        sh:datatype xsd:integer ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 5 ;
    ] ;
    .
```
