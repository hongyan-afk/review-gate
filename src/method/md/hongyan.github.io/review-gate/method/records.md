---
template:
  id: https://hongyan.github.io/review-gate/method/records
  name: "Verification Records"
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
# Verification Records

A record is complete when all five parts are filled in, and this table flags one that is not. The reasoner cannot do this: a record with a missing part is not a contradiction, only an unknown. A verdict with no record at all is not a row here, since there is nothing to target; that is the second question in the README.

```table-editor
---
target: ${target}
columns: { this: { label: "Record" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix gate: <https://hongyan.github.io/review-gate/method/gate#> .

gate:VerificationRecordShape
    a sh:NodeShape ;
    sh:targetClass gate:VerificationRecord ;
    sh:property [
        sh:path gate:hasId ;
        sh:name "Id" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 0 ;
    ] ;
    sh:property [
        sh:path gate:documents ;
        sh:name "Documents activity" ;
        sh:class gate:ReviewActivity ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path gate:claimText ;
        sh:name "1. Claim" ;
        dash:editor dash:TextAreaEditor ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path gate:addresses ;
        sh:name "2. Error class" ;
        sh:class gate:ErrorClass ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path gate:detectionRoute ;
        sh:name "3. Detection route" ;
        dash:editor dash:TextAreaEditor ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path gate:responsibleRole ;
        sh:name "4. Responsible role" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 5 ;
    ] ;
    sh:property [
        sh:path gate:knownResidual ;
        sh:name "5. Known residual" ;
        dash:editor dash:TextAreaEditor ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 6 ;
    ] ;
    .
```
