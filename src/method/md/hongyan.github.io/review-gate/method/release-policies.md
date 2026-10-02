---
template:
  id: https://hongyan.github.io/review-gate/method/release-policies
  name: "Release Policies"
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
# Release Policies

One policy per provenance path per deployment. It names the checkpoints an artifact on that path has to clear, and nothing else; which artifacts it governs is derived from their declared path, never written here. A policy that reaches into another deployment's checkpoints is malformed, not a gap, so that rule is a violation.

```table-editor
---
target: ${target}
columns: { this: { label: "Policy" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix gate: <https://hongyan.github.io/review-gate/method/gate#> .

gate:ReleasePolicyShape
    a sh:NodeShape ;
    sh:targetClass gate:ReleasePolicy ;
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
        sh:path gate:coversPath ;
        sh:name "Provenance path" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path gate:requiresCheckpoint ;
        sh:name "Required checkpoints" ;
        sh:class gate:Checkpoint ;
        sh:minCount 1 ;
        sh:order 5 ;
    ] ;
    sh:sparql [
        sh:message "A release policy may only require checkpoints of its own deployment. This one requires a checkpoint that belongs to another deployment." ;
        sh:select """
            PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
            SELECT $this WHERE {
                $this gate:isDeployedIn ?d ;
                      gate:requiresCheckpoint ?cp .
                ?cp gate:isDeployedIn ?other .
                FILTER (?other != ?d)
            }
        """ ;
    ] ;
    .
```
