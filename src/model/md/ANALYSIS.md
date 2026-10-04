# Analysis

Each question from the scope statement, the evidence for it on the dashboard, and what that evidence means. The run behind this page is from 2026-10-04 over the description bundle.

The dataset carries planted gaps on purpose, at least one for each question, and METHOD.md lists the ones validation reports. When a check finds a planted gap, that shows the check works. The findings marked as not planted are things the analysis turned up.

## 1. Which error classes does no checkpoint guard, and which checkpoint guards nothing?

Evidence: dashboard section 1.

Finding: ecIrrelevantFeature is guarded by no checkpoint in either deployment, and cpFormatCheck in the code review deployment guards nothing. Both are planted. The question is answered by the query as it stands.

## 2. Which issued verdicts still have no verification record with all five parts filled in?

Evidence: dashboard section 2, and validation of the records table.

Finding: RA-G2 and RA-G3 have no record, both planted. No record is missing a part, because every part is a required column of the records table and validation reports any record that lacks one. So the question is answered by a query for the missing records and by validation for the incomplete ones.

## 3. On which provenance paths is an error class left without a second, independent checkpoint?

Evidence: dashboard section 3, and the allowed escape table at the end of the dashboard.

Finding: of the 30 combinations of policy and error class, 6 are guarded by two different gauges, 12 by one, and 12 by none. Six of the empty ones belong to ecIrrelevantFeature, which is planted. The others are not planted: in both deployments the AI-generated path has no checkpoint for bias, the AI-generated code path has none for unsupported claims either, and the grading deployment checks no path for inconsistency. The AI-generated path is also the only one where no error class has a second check, which runs against the premise that AI-generated work needs the closest checking.

Diagnosed gap, vocabulary: the vocabulary has no notion of independence. The dashboard counts different gauges as a working definition, but under that definition two AI gauges built on the same base model would count as independent, and the method should not accept that. Counting checkpoints or gauge kinds instead gives the same cells on the current data, so the data cannot tell the definitions apart either. Closing the gap needs a term in the vocabulary for what two gauges share, such as a base model or a calibration batch.

## 4. Which AI gauges run a version that none of their own qualification records covers?

Evidence: dashboard section 4.

Finding: gCodeAI runs v2 while its only qualification certifies v1. Planted, and answered by the query.

## 5. Which verdicts cannot be traced back to a qualification covering the error class checked?

Evidence: dashboard section 5 and the evidence chain graph.

Finding: RA-G4 cites no qualification, so none of the three error classes its checkpoint guards is covered. Planted. In the graph, RA-G4 has no qualification link, and RA-G2 and RA-G3 have no record linked to them.

## Gaps in each pattern

Evidence: the last table on the dashboard, one missing-relationship check per pattern.

| Pattern | Check | Result |
| --- | --- | --- |
| Checkpoints | guards no error class | cpFormatCheck, planted |
| Gauges | holds no qualification record | gStudentReviewer, not planted |
| Release policies | governs no artifact yet | pCodeAiAssisted and pGradeAiGenerated, not planted |
| Review activities | has no verification record | RA-G2 and RA-G3, planted |
| Verification records | addresses an error class its activity does not check | VR-G5, not planted |

The human reviewer in the code review deployment has never been qualified. It is a design-stage deployment, but the reviewer already staffs a checkpoint, so this is a process gap to close before the first real review.

Two of the six policies have never governed an artifact, so their checkpoint lists have not been exercised. That is expected at this stage and is recorded here so it is not mistaken for coverage.

VR-G5 records a check for inconsistency on a scoring run, but the AI scorer checkpoint does not guard inconsistency. The work is being done without a checkpoint that owns it. Nothing in the records pattern ties a record's error class to what its checkpoint guards, so this is a pattern gap. A rule on the records table could report it while the record is being written.

## Computed analysis

The script at the end of the dashboard multiplies, for each policy and error class, the miss rates the guards' tolerances allow. Under independence that product is the escape rate the tolerances allow on that path.

Finding: the paths with two checkpoints come out between 0.0017 and 0.005, the cells with one checkpoint between 0.05 and 0.1, except bias on two grading paths, which comes out 0. The AI-generated paths are among the one-checkpoint cells. The product rests on the independence the vocabulary cannot yet express, so for gauges that share a base model it understates the escape. One result is misleading on its own: the human reference's tolerance for bias is zero misses in 60, which makes the product 0. Sixty samples with no miss are still consistent with a miss rate near 5 percent, so the zero says the tolerance is optimistic, not that bias cannot escape.

## Outside the model

Whether a tolerance is the right one, and whether a gauge really performs within it, are measured in the study the model supports, not in the model. The model can show which tolerance a verdict relied on, but not whether that tolerance was earned.
