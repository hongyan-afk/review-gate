---
ontology: https://hongyan.github.io/review-gate/model/code-review
---

# Deployment: code review

The design-stage deployment. Nothing here has run yet; the instances are the design, written down so the same five questions can be asked of it before the first real review. The second and third reviews and their records, RA-C2, RA-C3, VR-C2, VR-C2B and VR-C3, and the bias guard on the human review were added once these tables were in place.

```compose
template: https://hongyan.github.io/review-gate/method/checkpoints
target: https://hongyan.github.io/review-gate/model/code-review
```

```compose
template: https://hongyan.github.io/review-gate/method/gauges
target: https://hongyan.github.io/review-gate/model/code-review
```

```compose
template: https://hongyan.github.io/review-gate/method/release-policies
target: https://hongyan.github.io/review-gate/model/code-review
```

```compose
template: https://hongyan.github.io/review-gate/method/activities
target: https://hongyan.github.io/review-gate/model/code-review
```

```compose
template: https://hongyan.github.io/review-gate/method/records
target: https://hongyan.github.io/review-gate/model/code-review
```
