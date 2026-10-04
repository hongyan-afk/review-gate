# Review Gate

An OML model of a human-AI review gate process. One vocabulary, two deployments of the same
process: code review, still at the design stage, and AI scoring of course syllabi, which is
running. It also has a small method: five authoring patterns, each with its own editor, and
a `METHOD.md`.

## What the system is

Artifacts pass through checkpoints before they are released. A code change is an artifact in
one deployment, a course syllabus in the other. Each checkpoint is staffed by exactly one
gauge, and a gauge is a reviewer treated as a measuring instrument: it holds qualification
records saying which error classes it is qualified for, at what tolerance, and for which
configuration version. A release policy says which checkpoints an artifact has to clear on
each provenance path. Every review activity issues a verdict, which a five-part
verification record is supposed to back up.

Human reviewers and AI evaluators are one concept here. Both are qualified the same way. What
differs is only what the qualification is pinned to: a calibration batch for a person, a
configuration version for a model.

## How to read this repository

```
src/method/oml/hongyan.github.io/review-gate/method/
    trace.oml            what a record is, and how records trace to one another
    gate.oml             the review gate vocabulary, start here
    bundle.oml           the vocabulary bundle

src/method/md/hongyan.github.io/review-gate/method/
    METHOD.md            what the method prescribes, in what order, and why
    checkpoints.md       a checkpoint and what it guards, at what tolerance
    gauges.md            gauges, human and AI, and the version an AI gauge runs
    release-policies.md  which checkpoints each provenance path has to clear
    activities.md        one execution of a checkpoint on an artifact
    records.md           the five-part record behind a verdict
    dashboard.md         the five questions asked of the reasoned model

src/model/oml/hongyan.github.io/review-gate/model/
    error-classes.oml    the kinds of review failure, shared by both deployments
    versions.oml         configuration versions and calibration batches
    grading.oml          the syllabus scoring deployment
    code-review.oml      the code review deployment
    bundle.oml           the description bundle

src/model/md/
    index.md             start page
    ANALYSIS.md          what the dashboard shows, question by question
    Review Gate/         an overview, one page per deployment, and the dashboard
```

Read `gate.oml` first, then `METHOD.md`, then either deployment page. The method pages are
compose templates and each deployment page uses all five, so its tables edit that
deployment's file. The dashboard is a compose template too, owned by the method, and
`ANALYSIS.md` records what it found.

## How to build it

```bash
npm install -g @oml/cli
oml whoami
oml start
oml lint
oml validate
oml reason
oml render
```

`oml reason` writes entailments to `build/owl` and `oml render` writes the pages to
`build/web`. Last run 2026-10-04 on CLI 0.26.5, description bundle, unique names on (the default): lint
reports no errors, validation reports six warnings and no errors, and the model reasons
consistent. Five warnings are left in on purpose and one is a real finding; `METHOD.md` and
`src/model/md/ANALYSIS.md` say which.

## The questions it has to answer

These are the business questions from the scope statement, and the acceptance criteria I
build against.

1. Which error classes does no checkpoint guard, and which checkpoint guards nothing?
2. Which issued verdicts still have no verification record with all five parts filled in?
3. On which provenance paths is an error class left without a second, independent checkpoint?
4. Which AI gauges run a version that none of their own qualification records covers?
5. Which verdicts cannot be traced back to a qualification covering the error class checked?
