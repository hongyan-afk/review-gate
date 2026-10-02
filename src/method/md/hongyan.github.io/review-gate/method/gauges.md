---
template:
  id: https://hongyan.github.io/review-gate/method/gauges
  name: "Gauges"
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
# Gauges

The first table lists every gauge in the deployment, human or AI, with the fields they share. The second lists the AI gauges alone, because only they have to record a running version, and only they are checked for running one that no qualification record covers. A human role that is designed but not yet staffed is legal data here, not a violation.

The first table targets the two gauge kinds by name as well as the parent, because a table only sees the types an instance was written with, not the ones the reasoner would add. The second table shows the version an AI gauge's qualification records certify next to the version it runs.

```table-editor
---
target: ${target}
columns: { this: { label: "Gauge" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix gate: <https://hongyan.github.io/review-gate/method/gate#> .

gate:GaugeShape
    a sh:NodeShape ;
    sh:targetClass gate:Gauge ;
    sh:targetClass gate:HumanGauge ;
    sh:targetClass gate:AIGauge ;
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
        sh:path gate:holdsQualification ;
        sh:name "Qualification records" ;
        sh:class gate:QualificationRecord ;
        sh:order 4 ;
    ] ;
    sh:sparql [
        sh:message "A gauge has to be either a human gauge or an AI gauge. Left untyped, it is counted as neither, and every path it sits on looks as if it had one gauge kind fewer." ;
        sh:select """
            PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
            SELECT $this WHERE {
                $this a gate:Gauge .
                FILTER NOT EXISTS { $this a gate:HumanGauge }
                FILTER NOT EXISTS { $this a gate:AIGauge }
            }
        """ ;
    ] ;
    .
```

## AI gauges

```table-editor
---
target: ${target}
columns: { this: { label: "AI gauge" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix gate: <https://hongyan.github.io/review-gate/method/gate#> .

gate:AIGaugeShape
    a sh:NodeShape ;
    sh:targetClass gate:AIGauge ;
    sh:property [
        sh:path gate:runsVersion ;
        sh:name "Running version" ;
        sh:class gate:ConfigVersion ;
        sh:minCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path gate:holdsQualification ;
        sh:name "Qualification records" ;
        sh:class gate:QualificationRecord ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path gate:certifiedVersion ;
        sh:name "Versions its records certify" ;
        dash:readOnly true ;
        sh:order 3 ;
    ] ;
    sh:rule [
        a sh:SPARQLRule ;
        sh:construct """
            PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
            CONSTRUCT { $this gate:certifiedVersion ?v }
            WHERE {
                $this gate:holdsQualification ?q .
                ?q gate:certifiesVersion ?v .
            }
        """ ;
    ] ;
    sh:sparql [
        sh:severity sh:Warning ;
        sh:message "This AI gauge runs a version that none of its own qualification records certifies. Requalify it at the running version, or record the qualification that already covers it." ;
        sh:select """
            PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
            SELECT $this WHERE {
                $this gate:runsVersion ?v .
                FILTER NOT EXISTS {
                    $this gate:holdsQualification ?q .
                    ?q gate:certifiesVersion ?v .
                }
            }
        """ ;
    ] ;
    .
```
