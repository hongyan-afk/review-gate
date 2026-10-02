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

One row per execution of a checkpoint on an artifact. The checkpoint, the artifact and the verdict are required because a verdict missing any of them drops out of every question that would otherwise report it. The gauge is not typed in; it follows from the checkpoint by rule, so it is shown read-only. It cannot be a required column, because a count only sees what this description writes down, so the first rule below checks it over the reasoned model instead. The other two warn when a verdict has no record behind it, or does not rest on a qualification of the performing gauge that covers what its checkpoint guards.

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
        sh:message "No verification record documents this verdict. Add one in the records table." ;
        sh:select """
            PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
            SELECT $this WHERE {
                $this a gate:ReviewActivity .
                FILTER NOT EXISTS { ?record gate:documents $this }
            }
        """ ;
    ] ;
    sh:sparql [
        sh:severity sh:Warning ;
        sh:message "This verdict does not cite a qualification record of the performing gauge that covers every error class its checkpoint guards. Cite the record that does." ;
        sh:select """
            PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
            SELECT DISTINCT $this WHERE {
                $this gate:checksFor ?ec ;
                      gate:performedBy ?g .
                FILTER NOT EXISTS {
                    $this gate:citesQualification ?q .
                    ?g gate:holdsQualification ?q .
                    ?q gate:qualifiesFor ?ec .
                }
            }
        """ ;
    ] ;
    .
```
