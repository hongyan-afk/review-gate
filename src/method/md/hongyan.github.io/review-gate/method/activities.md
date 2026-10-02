---
template:
  id: https://hongyan.github.io/review-gate/method/activities
  name: "Review Activities"
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
# Review Activities

One row per execution of a checkpoint on an artifact. The checkpoint, the artifact and the verdict are required because a verdict missing any of them drops out of every question that would otherwise report it. The gauge is not typed in; it follows from the checkpoint by rule, so it is shown read-only. It cannot be a required column, because a count only sees what this description writes down, so the two rules below check it over the reasoned model instead. The second one checks that a verdict rests on the performing gauge's own qualification.

```table-editor
---
target: ${target}
columns: { this: { label: "Activity" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix gate: <https://hongyan.github.io/review-gate/method/gate#> .

gate:ActivityShape
    a sh:NodeShape ;
    sh:targetClass gate:ReviewActivity ;
    sh:property [
        sh:path gate:hasId ;
        sh:name "Id" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path gate:performedAt ;
        sh:name "Checkpoint" ;
        sh:class gate:Checkpoint ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path gate:performedBy ;
        sh:name "Gauge (derived)" ;
        sh:class gate:Gauge ;
        dash:readOnly true ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path gate:evaluates ;
        sh:name "Artifact" ;
        sh:class gate:Artifact ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path gate:hasVerdict ;
        sh:name "Verdict" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 5 ;
    ] ;
    sh:property [
        sh:path gate:rawVerdictLabel ;
        sh:name "Raw verdict label" ;
        sh:maxCount 1 ;
        sh:order 6 ;
    ] ;
    sh:property [
        sh:path gate:citesQualification ;
        sh:name "Cited qualification" ;
        sh:class gate:QualificationRecord ;
        sh:order 7 ;
    ] ;
    sh:sparql [
        sh:message "No gauge can be derived for this activity, because its checkpoint is missing or has no operator. The verdict cannot be attributed to anyone." ;
        sh:select """
            PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
            SELECT $this WHERE {
                $this a gate:ReviewActivity .
                FILTER NOT EXISTS { $this gate:performedBy ?g }
            }
        """ ;
    ] ;
    sh:sparql [
        sh:severity sh:Warning ;
        sh:message "This activity cites a qualification record that the gauge who performed it does not hold. Cite the performing gauge's own record." ;
        sh:select """
            PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
            SELECT $this WHERE {
                $this gate:performedBy ?g ;
                      gate:citesQualification ?q .
                FILTER NOT EXISTS { ?g gate:holdsQualification ?q }
            }
        """ ;
    ] ;
    .
```
