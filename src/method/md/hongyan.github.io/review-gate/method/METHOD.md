# Review Gate Method

The vocabulary in `gate.oml` says what can be stated about a review gate process. This page says how a deployment is written down with it: what to create, in what order, where each fact lives, and which mistakes the editors flag.

## What a deployment is made of

A deployment is one installation of the process. It has checkpoints, each staffed by one gauge and each guarding some error classes at a stated tolerance. It has release policies, one per provenance path, saying which checkpoints an artifact on that path has to clear. Once it runs, it accumulates review activities, one per execution of a checkpoint on an artifact, and behind each verdict a five-part verification record. Gauges hold qualification records that pin them to a configuration version; when the running version and the certified version drift apart, the gauge is due for requalification.

## Order of work

Create things in the order the editors can accept them.

1. The deployment itself, in OML, since it is one line and nothing else can reference it before it exists.
2. Error classes and configuration versions. Both are shared across deployments, so they are not part of a deployment page; they live in `error-classes.oml` and `versions.oml` and are edited there.
3. Gauges, on the Gauges page. Type each one as human or AI at creation; the page flags an untyped gauge. For an AI gauge, record the version it runs.
4. Qualification records, in OML for now. Each certifies one version and covers some error classes. There is no editor for them yet, because a record's tolerance is free text whose form has not settled.
5. Checkpoints, on the Checkpoints page: which gauge staffs it and how its raw output reads as a verdict. Then, in the second table on the same page, what it guards and at what tolerance.
6. Release policies, on the Release Policies page: one per provenance path, naming the checkpoints of this deployment only.
7. Artifacts, in OML, with a declared provenance path. Which policy governs them follows by rule.
8. Review activities and their verification records, on the last two pages, as the deployment runs.

Steps 3 to 6 are the design of a deployment and change rarely. Step 8 is its operation and grows every day. That difference in cadence is why activities and records could live in their own file; today they share the deployment file because one person, the quality owner, writes all of it.

## The five patterns and their tables

| Pattern | Page | Tables | Construction rule enforced |
|---|---|---|---|
| Checkpoint and what it guards | `checkpoints.md` | checkpoints; guards with tolerances | a checkpoint names its deployment and exactly one gauge |
| Gauges, human and AI | `gauges.md` | all gauges; AI gauges | a gauge is typed human or AI; an AI gauge records a running version |
| Release policy | `release-policies.md` | policies | a policy covers one path and requires checkpoints of its own deployment only |
| Review activity | `activities.md` | activities | an activity names its checkpoint, its artifact and its verdict |
| Verification record | `records.md` | records | all five parts are filled in |

One pattern can need two tables. The checkpoint pattern's second table edits the guarding links themselves, because the tolerance sits on the link and not on either end. The guarding link could have been turned into a concept of its own, which would have given it an ordinary table; it stays a relation entity because that keeps `guards` as a plain relation, which the `ChecksFor` rule in `gate.oml` uses. The running example shows its connections, the nearest kind of link, only as a diagram and leaves them to OML; here the tolerances on each link are what the quality owner edits, so the links get a table. The gauge pattern's second table exists because only AI gauges have to record a running version.

## Where facts live

The folders follow the running example. What the method prescribes sits under `src/method`: the vocabulary in `oml`, these pages in `md`. What a project states sits under `src/model`: its descriptions in `oml`, its pages in `md`. A project page holds only its context and a call to each method page, so a rule fixed on a method page is fixed for both deployments at once.

Within `src/model`, files are split by who changes them and how often, not by class.

- `error-classes.oml`: the quality owner's list of what a review can get wrong. Shared by every deployment. Changes when the process learns a new failure mode, which is rare.
- `versions.oml`: configuration versions and calibration batches. Shared, but owned by whoever maintains the model or runs the calibration, and changed every time one is upgraded. It is a separate file from the error classes for that reason alone.
- `grading.oml`, `code-review.oml`: one file per deployment, owned by that deployment's quality owner.
- `bundle.oml`: the description bundle, which is what the reasoner reads.

## What the reasoner does and what validation does

The vocabulary's restrictions say what a thing is. A checkpoint is staffed by exactly one gauge; an AI gauge runs at least one version; a verification record has exactly one of each of its five parts. The reasoner enforces the "at most" half of those: two gauges on one checkpoint, or two values for a functional property, is a contradiction it reports. It cannot enforce the "at least" half, because under the open world assumption a missing value is unknown, not absent.

The tables enforce the "at least" half. Every required field above is a `sh:minCount 1` in a shape, and `oml validate` reports each instance that lacks one. With the 0.26 OML tools, the two kinds of constraint see different graphs. A count such as `sh:minCount` covers only what the edited description writes down, so a fact the rules derive, such as who performed an activity, cannot be a required column. A `sh:sparql` rule runs over the whole reasoned model, derived facts included. So a check on a derived fact is written as a `sh:sparql` rule, and the five rules in `gate.oml` stay the only place those facts are defined. The table still shows the derived value read-only while editing. Validation reasons for itself; it does not need `oml reason` to have been run first.

Reasoning results are reported with three things: the model version, the bundle they were scoped to, and the identity assumption in force. The unique names assumption is a per-run choice of the CLI, on by default. The same files can be consistent with it off and inconsistent with it on.

## Two kinds of finding

Validation distinguishes malformed data from a gap the process has.

A violation is malformed data: a gauge with no type, a record with four parts, a policy requiring a checkpoint from another deployment. The dataset has none, and a deployment page that shows one is not finished.

A warning is a gap in the process itself: a checkpoint that guards nothing, an AI gauge running a version no qualification covers. The dataset carries two on purpose, `cpFormatCheck` and `gCodeAI` in the code review deployment, as positive controls for the first and fourth questions. They are warnings so that the dataset still validates clean while the gaps stay visible in the table where they can be fixed. `oml validate` exits zero on warnings and non-zero on violations, so a build can stop on malformed data without stopping on a known gap.

## What goes in prose

A page's prose carries what a query cannot produce: why a checkpoint exists, why a tolerance was set where it was, what was tried and rejected, what is still open. Facts stay in the model and are shown by the tables.

## Known limits

- Qualification records have no editor. Their tolerance is free text and the form of a qualification record is still being settled in the underlying study.
- Saving from an editor rewrites the whole description file and drops any `//` comments in it, so nothing that matters is kept in OML comments.
- A check that needs a count across several instances, such as whether a path has a second independent checkpoint, is left to a query rather than a table rule. A check on one instance's derived facts, such as whether the gauge that issued a verdict holds the qualification it cites, is a table rule.
- A gap that has been accepted for now is only a warning today. Whether it should be recorded in the model instead, naming who accepted it and the version it was accepted for so that it lapses when the version changes, is still open.
