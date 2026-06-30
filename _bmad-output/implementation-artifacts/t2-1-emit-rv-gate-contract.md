# Story T2.1: Emit RV Gate Contract

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a test architect,
I want `RV` to emit a test quality gate contract,
so that weak or unreliable tests can route back to development through Archon's single loop.

## Acceptance Criteria

1. Given `RV` runs, when `tea-rv.gate.json` is emitted, then it includes `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate`, `quality_score`, `findings_count`, `blocking_findings_count`, and `report_file`.
2. Given weak tests, flaky behavior, unreliable fixtures, timing-sensitive behavior, async or network-heavy instability, UI fragility, or weak assertions exist, when `RV` emits its contract, then blocking findings produce `gate: FAIL`.
3. Given RV execution fails, evidence is missing or invalid, worker summaries are untrusted, story identity cannot be trusted, the report cannot be written, or the contract output is invalid, when the contract is emitted or validated, then the contract produces `gate: ERROR`.
4. Given the human review report exists, when downstream orchestration needs a route decision, then it uses `tea-rv.gate.json` and the contract points to the report through `report_file`.
5. Given coverage-looking evidence appears during review, when the RV gate is decided, then coverage is not treated as the gate decision because coverage gates remain owned by `trace`.

## Tasks / Subtasks

- [ ] Define the RV gate contract output path and workflow metadata.
  - [ ] Add `rv_gate_output: "{test_artifacts}/tea-rv.gate.json"` to `src/workflows/testarch/bmad-testarch-test-review/workflow.yaml`.
  - [ ] Document the machine-readable contract fields near the output configuration, following the `trace` workflow style.
  - [ ] Update `workflow-plan.md` so the workflow outputs list includes both `test-review.md` and `tea-rv.gate.json`.
- [ ] Emit `tea-rv.gate.json` from score aggregation.
  - [ ] Extend `steps-c/step-03f-aggregate-scores.md` after `reviewSummary` is built.
  - [ ] Map `reviewSummary.overall_score` to `quality_score`.
  - [ ] Map `reviewSummary.all_violations.length` to `findings_count`.
  - [ ] Derive `blocking_findings_count` from route-blocking RV findings, with high-severity findings blocking by default.
  - [ ] Treat weak assertions, flaky behavior, unreliable fixtures, timing-sensitive waits, async or network instability, and UI fragility as blocking when the finding indicates route risk.
  - [ ] Emit `gate: FAIL` only for a trusted RV run with at least one blocking finding.
  - [ ] Emit `gate: PASS` only for a trusted RV run with zero blocking findings.
  - [ ] Emit `gate: ERROR` for execution, evidence, identity, write, validation, or contract-shape failures.
  - [ ] Keep `ERROR` separate from `FAIL`; do not convert infrastructure or trust failures into quality failures.
  - [ ] Do not emit `gate: SKIPPED` in this story; Story T2.2 owns skipped-contract compatibility.
- [ ] Preserve the existing RV scoring model and boundaries.
  - [ ] Keep the four dimensions as determinism, isolation, maintainability, and performance.
  - [ ] Keep the current weights of `0.30`, `0.30`, `0.25`, and `0.15`.
  - [ ] Keep the existing worker JSON contracts unless the implementation needs additive fields for blocking classification.
  - [ ] Keep coverage decisions out of RV and route them to `bmad-testarch-trace`.
- [ ] Link and validate the contract from the human report path.
  - [ ] Update `steps-c/step-04-generate-report.md` so it verifies `tea-rv.gate.json` exists and is readable before completion.
  - [ ] Update `test-review-template.md` to point readers to `tea-rv.gate.json` for route decisions.
  - [ ] Update `checklist.md` so validation covers the full envelope, PASS, FAIL, ERROR, report pointer, and coverage exclusion.
- [ ] Update public docs and command metadata if the artifact is documented as a user-facing output.
  - [ ] Update `docs/how-to/workflows/run-test-review.md` to prefer reading `tea-rv.gate.json` over parsing Markdown for quality gates.
  - [ ] Update `docs/reference/commands.md` and `docs/reference/configuration.md` so `tea-rv.gate.json` appears in the RV output list.
  - [ ] Update `docs/explanation/tea-overview.md` and `src/module-help.csv` only if the public command catalog should advertise the contract.
  - [ ] Do not manually edit `CHANGELOG.md`; follow the maintainer release process if a changelog entry is required.
- [ ] Add focused automated coverage.
  - [ ] Extend `test/test-installation-components.js` or add a focused test file for the RV contract workflow metadata.
  - [ ] Assert `workflow.yaml` declares `rv_gate_output` as `{test_artifacts}/tea-rv.gate.json`.
  - [ ] Assert Step 3F writes `{rv_gate_output}` and uses `JSON.stringify(..., null, 2)`.
  - [ ] Assert the implementation maps score, findings count, and blocking findings count from `reviewSummary`.
  - [ ] Assert checklist text covers the full envelope and PASS, FAIL, ERROR behavior.
  - [ ] Add fixture-level validation for PASS, FAIL, and ERROR if a reusable validator is introduced.
- [ ] Run validation.
  - [ ] Run `npm run test:install`.
  - [ ] Run `npm run test:tea-workflow-descriptions`.
  - [ ] Run `npm run lint`.
  - [ ] Run `npm run lint:md`.
  - [ ] Run `npm run format:check`.
  - [ ] Run `npm run test:release-metadata`.
  - [ ] Run `npm run docs:validate-links` if docs are changed.
  - [ ] Run `npm test` when the focused checks pass locally.

## Dev Notes

### Contract Shape

Use this shape as the implementation target unless a newer project contract version already exists when development starts.

```json
{
  "contract_version": "1.0",
  "workflow": "bmad-testarch-test-review",
  "story_ref": "resolved-story-ref-or-null",
  "node": "RV",
  "round": 1,
  "gate": "PASS",
  "generated_at": "2026-06-30T00:00:00.000Z",
  "quality_score": 87,
  "findings_count": 3,
  "blocking_findings_count": 1,
  "report_file": "test-review.md",
  "artifact_pointers": {
    "test_review_report": "test-review.md"
  },
  "dimension_scores": {
    "determinism": 90,
    "isolation": 85,
    "maintainability": 82,
    "performance": 88
  },
  "findings_summary": {
    "HIGH": 1,
    "MEDIUM": 2,
    "LOW": 0
  },
  "blocking_findings": []
}
```

`story_ref` must be the resolved story target when RV is running for a story.
If a standalone legacy invocation has no story target, keep the key present and use `null` rather than fabricating an identity.
For Archon-routed invocations, a missing or mismatched story identity is an `ERROR`.

`round` must be the Archon loop round when it is supplied.
Use `1` only for standalone runs that have no loop metadata.

`quality_score` is the weighted score from `reviewSummary.overall_score`.
It is route evidence, but `FAIL` is driven by blocking findings rather than a Markdown grade.

`report_file` must point to the human `test-review.md` report.
Downstream orchestration must not parse Markdown for the route decision.

### Implementation Guidance

The best emission point is `src/workflows/testarch/bmad-testarch-test-review/steps-c/step-03f-aggregate-scores.md`.
That step already reads the four worker JSON files, computes the weighted score, aggregates violations, and writes `/tmp/tea-test-review-summary-${timestamp}.json`.

Use `src/workflows/testarch/bmad-testarch-test-review/steps-c/step-04-generate-report.md` to validate that the JSON exists and to link it from the human report.
Do not derive the route contract from Markdown.

The existing `trace` workflow is the closest local precedent.
It declares machine-readable outputs in `workflow.yaml` and writes route-facing JSON from the decision step.

Keep coverage out of RV.
The RV workflow evaluates test reliability, assertion value, fixture quality, timing stability, maintainability, and performance.
Coverage gates are owned by `bmad-testarch-trace`.

Do not edit `src/agents/bmad-tea/customize.toml`.
That file is generated and marked as not manually editable.

Treat Archon stories A3.1 and A3.2 as external integration assumptions.
This story should be implementable and testable locally with fixtures and without pulling external Archon project files.

Story T1.1 is only `ready-for-dev` at story creation time.
Do not assume `tea-ta.evidence.json` has already been implemented.
If RV needs TA evidence during development, use the local T1.1 contract shape as an optional fixture or compatibility assumption.

### Current RV Workflow State

`src/workflows/testarch/bmad-testarch-test-review/workflow.yaml` currently declares only `{test_artifacts}/test-review.md` as the default output.
It does not declare `rv_gate_output`.

`steps-c/step-03f-aggregate-scores.md` currently builds `reviewSummary` with `overall_score`, `overall_grade`, `quality_assessment`, dimension scores, violation summaries, all violations, high-severity violations, and top recommendations.
Preserve those fields and add contract emission around them.

`steps-c/step-04-generate-report.md` currently writes the human report and validates it with the checklist.
Keep that responsibility and add JSON existence or shape validation rather than moving score aggregation into Step 4.

### Testing Notes

Prefer plain Node tests that inspect workflow files and fixture output.
The repository already uses Node-based validation scripts and does not need a new dependency for this story.

If a JSON schema file is added, use an explicitly declared validator that supports the selected schema draft.
Do not rely on transitive `ajv` availability.

The current package engine is `node >=20.0.0`.
Implementation can use modern Node file system APIs that are available in Node 20 and later.

### Project Structure Notes

Canonical workflow source lives under `src/workflows/testarch/bmad-testarch-test-review/`.
Generated or installed skill copies under `.agents/` and `.claude/` must not be treated as the implementation source.

Public documentation lives under `docs/`.
Only update docs when the implementation changes the documented user-facing output surface.

Release metadata must stay synchronized, but this story should not require package version changes.
Run `npm run test:release-metadata` after implementation to prove no accidental drift was introduced.

### References

- [Source: `_bmad-output/planning-artifacts/epics.md` Story T2.1]
- [Source: `_bmad-output/planning-artifacts/prd.md` T-FR-2]
- [Source: `_bmad-output/planning-artifacts/architecture.md` T-AD-2, T-AD-6, and contract envelope]
- [Source: `_bmad-output/planning-artifacts/implementation-readiness-report-2026-06-30.md` story quality findings]
- [Source: `_bmad-output/implementation-artifacts/t1-1-emit-structured-ta-evidence.md` previous story context]
- [Source: `src/workflows/testarch/bmad-testarch-test-review/workflow.yaml`]
- [Source: `src/workflows/testarch/bmad-testarch-test-review/steps-c/step-03f-aggregate-scores.md`]
- [Source: `src/workflows/testarch/bmad-testarch-test-review/steps-c/step-04-generate-report.md`]
- [Source: `src/workflows/testarch/bmad-testarch-trace/workflow.yaml`]
- [Source: `src/workflows/testarch/bmad-testarch-trace/steps-c/step-05-gate-decision.md`]
- [Source: `package.json` engines and scripts]
- [Source: JSON Schema specifications at `https://json-schema.org/specification`]
- [Source: Ajv documentation at `https://ajv.js.org/`]
- [Source: Node.js previous releases at `https://nodejs.org/en/about/previous-releases`]

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

### File List
