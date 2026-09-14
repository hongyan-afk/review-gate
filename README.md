# Review Gate

An OML model of a human-AI review gate process, built for SIE 502. One vocabulary, two
deployments of the same process: code review, still at the design stage, and AI scoring of
course syllabi, which is running.

## What the system is

Artifacts pass through checkpoints before they are released. A code change is an artifact in
one deployment, a course syllabus in the other. Each checkpoint is staffed by exactly one
gauge, and a gauge is a reviewer treated as a measuring instrument: it holds qualification
records saying which error classes it is qualified for, at what tolerance, and for which
configuration version. A release policy says which checkpoints an artifact has to clear on
each provenance path. Every review activity yields a verdict and a five-part verification
record.

Human reviewers and AI evaluators are one concept here. Both are qualified the same way. What
differs is only what the qualification is pinned to: a calibration batch for a person, a
configuration version for a model.

## How to read this repository

```
src/method/oml/hongyan.github.io/review-gate/method/
    trace.oml    what a record is, and how records trace to one another
    gate.oml     the review gate vocabulary, start here
    bundle.oml   the vocabulary bundle

src/model/oml/hongyan.github.io/review-gate/model/
    common.oml       what both deployments share: error classes and configuration versions
    grading.oml      the syllabus scoring deployment
    code-review.oml  the code review deployment
    bundle.oml       the description bundle
```

Read `gate.oml` first, then `common.oml`, then either deployment. At the end of `gate.oml`
are five rules. They derive who performed an activity, which error classes it checked for,
which deployment an activity and a record belong to, and which policy governs an artifact.
`AiGeneratedArtifact` is defined with `=` rather than declared, so the reasoner classifies
artifacts into it.

## How to build it

```bash
npm install -g @oml/cli
oml whoami
oml start
oml lint
oml reason
```

`oml reason` writes entailments to `build/owl`. Last run 2026-09-13 on CLI 0.26.1: lint
reports no errors and the model reasons consistent.

## The questions it has to answer

These are the business questions from the scope statement, and the acceptance criteria I
build against.

1. Which error classes does no checkpoint guard, and which checkpoint guards nothing?
2. Which issued verdicts still have no verification record with all five parts filled in?
3. On which provenance paths is an error class left without a second, independent checkpoint?
4. Which AI gauges run a version that none of their own qualification records covers?
5. Which verdicts cannot be traced back to a qualification covering the error class checked?
