---
ontology: https://hongyan.github.io/review-gate/model/grading
---

# Deployment: AI syllabus scoring

The operating deployment. Scores are what it records; each checkpoint states how a score reads as a verdict. The provenance labels and the three policies are illustrative until syllabi carry a declared path. The two later scoring runs and their records, RA-G5, RA-G6, VR-G5 and VR-G6, were added once these tables were in place.

```compose
template: https://hongyan.github.io/review-gate/method/checkpoints
target: https://hongyan.github.io/review-gate/model/grading
```

```compose
template: https://hongyan.github.io/review-gate/method/gauges
target: https://hongyan.github.io/review-gate/model/grading
```

```compose
template: https://hongyan.github.io/review-gate/method/release-policies
target: https://hongyan.github.io/review-gate/model/grading
```

```compose
template: https://hongyan.github.io/review-gate/method/activities
target: https://hongyan.github.io/review-gate/model/grading
```

```compose
template: https://hongyan.github.io/review-gate/method/records
target: https://hongyan.github.io/review-gate/model/grading
```
