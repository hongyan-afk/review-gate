---
template:
  id: https://hongyan.github.io/review-gate/method/dashboard
  name: "Gate Dashboard"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Gate Dashboard

The five questions the method has to answer, asked of the reasoned model. What the answers mean is written up in ANALYSIS.md.

## 1. Error classes no checkpoint guards, and checkpoints that guard nothing

```table
PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
PREFIX oml: <http://opencaesar.io/oml#>

SELECT ?Element ?Gap ?Deployment
WHERE {
  {
    ?Element a gate:ErrorClass .
    FILTER NOT EXISTS { ?link a gate:Guards ; oml:hasTarget ?Element }
    BIND("guarded by no checkpoint" AS ?Gap)
  }
  UNION
  {
    ?Element a gate:Checkpoint ;
             gate:isDeployedIn ?Deployment .
    FILTER NOT EXISTS { ?link a gate:Guards ; oml:hasSource ?Element }
    BIND("guards nothing" AS ?Gap)
  }
}
ORDER BY ?Gap ?Element
```

## 2. Verdicts with no verification record

A record that exists but lacks a part is reported by validation, so only the missing records are listed here.

```table
PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>

SELECT ?Activity ?Id ?Verdict ?Checkpoint
WHERE {
  ?Activity a gate:ReviewActivity ;
            gate:hasId ?Id ;
            gate:hasVerdict ?Verdict ;
            gate:performedAt ?Checkpoint .
  FILTER NOT EXISTS { ?record gate:documents ?Activity }
}
ORDER BY ?Id
```

## 3. Independent coverage on each provenance path

The number of different gauges that guard each error class among the checkpoints a policy requires. Fewer than two means no independent second check under the working definition in ANALYSIS.md; 0 means the class is not checked on that path at all.

```matrix
---
rowColumnLabel: Policy / Error class
stylesheet:
  - selector: cell [Number(value) >= 2]
    style:
      background-color: lightgreen
---
PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
PREFIX oml: <http://opencaesar.io/oml#>

SELECT ?row ?column (COALESCE(?n, 0) AS ?value)
WHERE {
  ?row a gate:ReleasePolicy .
  ?column a gate:ErrorClass .
  OPTIONAL {
    SELECT ?row ?column (COUNT(DISTINCT ?gauge) AS ?n)
    WHERE {
      ?row gate:requiresCheckpoint ?checkpoint .
      ?link a gate:Guards ;
            oml:hasSource ?checkpoint ;
            oml:hasTarget ?column .
      ?checkpoint gate:isOperatedBy ?gauge .
    }
    GROUP BY ?row ?column
  }
}
ORDER BY ?row ?column
```

## 4. AI gauges running a version no qualification covers

```table
PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>

SELECT ?Gauge ?RunningVersion ?Deployment
WHERE {
  ?Gauge a gate:AIGauge ;
         gate:runsVersion ?RunningVersion ;
         gate:isDeployedIn ?Deployment .
  FILTER NOT EXISTS {
    ?Gauge gate:holdsQualification ?qualification .
    ?qualification gate:certifiesVersion ?RunningVersion .
  }
}
ORDER BY ?Gauge
```

## 5. Verdicts not traced to a qualification covering the error class checked

```table
PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>

SELECT DISTINCT ?Activity ?Id ?ErrorClass
WHERE {
  ?Activity a gate:ReviewActivity ;
            gate:hasId ?Id ;
            gate:performedBy ?gauge ;
            gate:checksFor ?ErrorClass .
  FILTER NOT EXISTS {
    ?Activity gate:citesQualification ?qualification .
    ?gauge gate:holdsQualification ?qualification .
    ?qualification gate:qualifiesFor ?ErrorClass .
  }
}
ORDER BY ?Id ?ErrorClass
```

## Evidence chains

Each verdict linked to the gauge that issued it, the record behind it and the qualification it cites. A verdict with no record or no qualification shows up with that link missing.

```graph
---
layout:
  mode: force
  fit: true
group:
  byPredicate: true
---
PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>

CONSTRUCT {
  ?activity gate:performedBy ?gauge .
  ?record gate:documents ?activity .
  ?activity gate:citesQualification ?qualification .
  ?gauge gate:holdsQualification ?held .
}
WHERE {
  ?activity a gate:ReviewActivity ;
            gate:performedBy ?gauge .
  OPTIONAL { ?record gate:documents ?activity }
  OPTIONAL { ?activity gate:citesQualification ?qualification }
  OPTIONAL { ?gauge gate:holdsQualification ?held }
}
```

## Gaps in each pattern

One missing-relationship check per authoring pattern.

```table
PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
PREFIX oml: <http://opencaesar.io/oml#>

SELECT ?Pattern ?Element ?Gap
WHERE {
  {
    ?Element a gate:Checkpoint .
    FILTER NOT EXISTS { ?link a gate:Guards ; oml:hasSource ?Element }
    BIND("Checkpoints" AS ?Pattern)
    BIND("guards no error class" AS ?Gap)
  }
  UNION
  {
    ?Element a gate:Gauge .
    FILTER NOT EXISTS { ?Element gate:holdsQualification ?qualification }
    BIND("Gauges" AS ?Pattern)
    BIND("holds no qualification record" AS ?Gap)
  }
  UNION
  {
    ?Element a gate:ReleasePolicy .
    FILTER NOT EXISTS { ?artifact gate:isGovernedBy ?Element }
    BIND("Release policies" AS ?Pattern)
    BIND("governs no artifact yet" AS ?Gap)
  }
  UNION
  {
    ?Element a gate:ReviewActivity .
    FILTER NOT EXISTS { ?record gate:documents ?Element }
    BIND("Review activities" AS ?Pattern)
    BIND("has no verification record" AS ?Gap)
  }
  UNION
  {
    ?Element a gate:VerificationRecord ;
             gate:documents ?activity ;
             gate:addresses ?errorClass .
    FILTER NOT EXISTS { ?activity gate:checksFor ?errorClass }
    BIND("Verification records" AS ?Pattern)
    BIND("addresses an error class its activity does not check" AS ?Gap)
  }
}
ORDER BY ?Pattern ?Element
```

## Allowed escape on each path

For each policy and error class, the script multiplies the miss rates the guards' tolerances allow, maximum missed over sample size. The product is the escape rate the tolerances allow if the checkpoints miss independently. An error class no required checkpoint guards escapes with rate 1.

```python
result = await query("""
PREFIX gate: <https://hongyan.github.io/review-gate/method/gate#>
PREFIX oml: <http://opencaesar.io/oml#>
SELECT ?policy ?errorClass ?checkpoint ?missed ?sample
WHERE {
  ?policy a gate:ReleasePolicy .
  ?errorClass a gate:ErrorClass .
  OPTIONAL {
    ?policy gate:requiresCheckpoint ?checkpoint .
    ?link a gate:Guards ;
          oml:hasSource ?checkpoint ;
          oml:hasTarget ?errorClass ;
          gate:toleranceMaxMissed ?missed ;
          gate:toleranceSampleSize ?sample .
  }
}
""")

cells = {}
for r in result["rows"]:
    key = (r["policy"].split("#")[-1], r["errorClass"].split("#")[-1])
    rates = cells.setdefault(key, [])
    if r.get("checkpoint") and float(r["sample"]) > 0:
        rates.append(float(r["missed"]) / float(r["sample"]))

rows = []
for (policy, error_class), rates in cells.items():
    escape = 1.0
    for rate in rates:
        escape *= rate
    rows.append((escape, policy, error_class, len(rates)))
rows.sort(key=lambda x: (-x[0], x[1], x[2]))

body = "".join(
    f"<tr><td>{p}</td><td>{e}</td><td>{n}</td><td>{x:.4f}</td></tr>"
    for x, p, e, n in rows
)
display(
    '<table class="oml-md-table">'
    '<thead><tr><th>Policy</th><th>Error class</th><th>Checkpoints</th><th>Allowed escape</th></tr></thead>'
    f'<tbody>{body}</tbody></table>'
)
```
