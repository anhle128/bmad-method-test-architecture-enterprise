# Story T4.1: Emit TR Traceability Contract

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a test architect,
I want `TR` to emit a final traceability contract,
so that trace gaps are visible before PR preparation.

## Acceptance Criteria

1. Given `TR` runs, when `tea-tr.gate.json` is emitted, then it includes `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate`, `coverage_gaps_count`, `findings_count`, `blocking_findings_count`, `trace_file`, and `report_file`.
2. Given route-blocking traceability gaps exist, when `TR` emits its contract, then it produces `gate: FAIL` and `blocking_findings_count` is greater than zero.
3. Given traceability evidence is trusted, story identity matches, and no route-blocking trace gaps exist, when `TR` emits its contract, then it produces `gate: PASS` even if non-blocking trace concerns are represented in `findings_count`.
4. Given TR execution fails, Phase 1 coverage matrix output is missing or invalid, evidence is invalid, upstream RV or NR evidence is missing or mismatched for an Archon-routed run, `story_ref` mismatches, the report cannot be written, or the contract output is invalid, when the contract is emitted or validated, then it produces `gate: ERROR`.
5. Given the human traceability report exists, when downstream orchestration needs the route decision, then it uses `tea-tr.gate.json`, follows `trace_file` to machine-readable trace evidence, follows `report_file` to the human report, and does not parse Markdown for the route decision.
6. Given existing trace outputs may use `CONCERNS` or `WAIVED`, when `tea-tr.gate.json` is emitted, then the route-facing `gate` is only `PASS`, `FAIL`, or `ERROR`; Story T4.2 owns `SKIPPED` compatibility.

## Tasks / Subtasks

- [ ] Define the TR gate contract output path and workflow metadata.
  - [ ] Add `tr_gate_output: "{test_artifacts}/tea-tr.gate.json"` to `src/workflows/testarch/bmad-testarch-trace/workflow.yaml`.
  - [ ] Document the machine-readable TR contract fields near the existing `e2e_trace_summary_output` and `gate_decision_output` configuration.
  - [ ] Update `src/workflows/testarch/bmad-testarch-trace/instructions.md` if initialization should resolve `tr_gate_output`.
  - [ ] Update `src/workflows/testarch/bmad-testarch-trace/workflow-plan.md` so the outputs list includes `traceability-matrix.md`, `e2e-trace-summary.json`, `gate-decision.json` when gate-eligible, and `tea-tr.gate.json`.
- [ ] Emit `tea-tr.gate.json` from the final trace decision step.
  - [ ] Extend `src/workflows/testarch/bmad-testarch-trace/steps-c/step-05-gate-decision.md` after `e2eTraceSummary` is assembled and before the workflow exits.
  - [ ] Build the TR contract from `coverageMatrix`, `gateReport`, and `e2eTraceSummary`.
  - [ ] Set `workflow` to `bmad-testarch-trace`.
  - [ ] Set `node` to `TR`.
  - [ ] Resolve `story_ref` from the Archon route context or `coverageMatrix.trace_target.id`.
  - [ ] For Archon-routed runs, treat missing or mismatched `story_ref` as `ERROR`.
  - [ ] Resolve `round` from Archon loop metadata when supplied and use `1` only for standalone trace runs that have no loop metadata.
  - [ ] Write `{tr_gate_output}` using `JSON.stringify(contract, null, 2)`.
- [ ] Map trace evidence into route contract fields without parsing Markdown.
  - [ ] Set `trace_file` to `{e2e_trace_summary_output}`.
  - [ ] Set `report_file` to `{outputFile}` for `traceability-matrix.md`.
  - [ ] Include `artifact_pointers.trace_summary` pointing to `e2e-trace-summary.json`.
  - [ ] Include `artifact_pointers.traceability_matrix` pointing to `traceability-matrix.md`.
  - [ ] Include `artifact_pointers.gate_decision` only when `gate-decision.json` was emitted.
  - [ ] Derive `coverage_gaps_count` from unique `coverageMatrix.requirements` entries whose coverage is not `FULL`.
  - [ ] Derive `findings_count` from coverage gaps, heuristic gaps, and deduplicated test blockers that require attention.
  - [ ] Derive `blocking_findings_count` from route-blocking trace findings rather than from Markdown prose.
- [ ] Preserve trace workflow gate semantics while adding the route contract.
  - [ ] Keep existing `e2e-trace-summary.json` output and schema behavior intact.
  - [ ] Keep existing `gate-decision.json` behavior intact for gate-eligible runs.
  - [ ] Keep the human `traceability-matrix.md` report intact.
  - [ ] Do not emit `CONCERNS` as a route-facing `tea-tr.gate.json` gate value.
  - [ ] Do not emit `WAIVED` as a route-facing `tea-tr.gate.json` gate value.
  - [ ] Do not emit `SKIPPED` in this story; Story T4.2 owns `tea-tr-skipped.gate.json`.
  - [ ] Treat non-gate-eligible trace collection as `ERROR` for the TR route contract unless Archon intentionally skipped TR through the T4.2 skipped contract.
- [ ] Validate upstream route evidence when it is supplied.
  - [ ] Accept executed or skipped RV and NR branch contracts as upstream evidence when provided by the route context.
  - [ ] Require upstream branch contracts to share the same expected `story_ref` and `round` for Archon-routed runs.
  - [ ] Treat missing, malformed, wrong-node, wrong-story, wrong-round, or untrusted upstream contracts as `ERROR` for Archon-routed TR.
  - [ ] Do not traverse out of this repository for Archon files; use local fixtures or route-context inputs.
- [ ] Link and validate the contract from trace validation surfaces.
  - [ ] Update `src/workflows/testarch/bmad-testarch-trace/checklist.md` so validation covers `tea-tr.gate.json`, the full envelope, PASS, FAIL, ERROR behavior, counts, `trace_file`, `report_file`, artifact pointers, and no Markdown parsing.
  - [ ] Inspect `src/workflows/testarch/bmad-testarch-trace/steps-v/step-01-validate.md` and update it only if validate mode should load or schema-check `tea-tr.gate.json`.
  - [ ] Update `src/workflows/testarch/bmad-testarch-trace/trace-template.md` only if the human report should point readers to the route contract.
- [ ] Update public docs and command metadata if the artifact is documented as user-facing or integrator-facing output.
  - [ ] Update `docs/how-to/workflows/run-trace.md` to prefer `tea-tr.gate.json` for route decisions and keep `traceability-matrix.md` as the human report.
  - [ ] Update `docs/reference/commands.md` and `docs/reference/configuration.md` so `tea-tr.gate.json` appears in the trace output list.
  - [ ] Update `docs/explanation/tea-overview.md` and `src/module-help.csv` only if the public command catalog should advertise the contract.
  - [ ] Do not manually edit `CHANGELOG.md`; follow the maintainer release process if a changelog entry is required.
- [ ] Add focused automated coverage.
  - [ ] Extend `test/test-installation-components.js` or add a focused test file for TR contract workflow metadata.
  - [ ] Assert `workflow.yaml` declares `tr_gate_output` as `{test_artifacts}/tea-tr.gate.json`.
  - [ ] Assert Step 5 writes `{tr_gate_output}` and uses `JSON.stringify(..., null, 2)`.
  - [ ] Assert Step 5 maps `coverage_gaps_count`, `findings_count`, `blocking_findings_count`, `trace_file`, and `report_file`.
  - [ ] Assert Step 5 does not copy `CONCERNS`, `WAIVED`, or `SKIPPED` into the route-facing `gate`.
  - [ ] Assert checklist text covers the full envelope and PASS, FAIL, ERROR behavior.
  - [ ] Add fixture-level validation for PASS, FAIL, story mismatch ERROR, and invalid evidence ERROR if a reusable TEA gate validator is introduced.
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
  "workflow": "bmad-testarch-trace",
  "story_ref": "resolved-story-ref-or-null",
  "node": "TR",
  "round": 1,
  "gate": "PASS",
  "generated_at": "2026-06-30T00:00:00.000Z",
  "coverage_gaps_count": 0,
  "findings_count": 0,
  "blocking_findings_count": 0,
  "trace_file": "e2e-trace-summary.json",
  "report_file": "traceability-matrix.md",
  "artifact_pointers": {
    "trace_summary": "e2e-trace-summary.json",
    "traceability_matrix": "traceability-matrix.md",
    "gate_decision": "gate-decision.json"
  },
  "coverage_summary": {
    "total_requirements": 12,
    "fully_covered": 12,
    "overall_coverage_percentage": 100,
    "priority_breakdown": {
      "P0": { "total": 3, "covered": 3, "percentage": 100 },
      "P1": { "total": 4, "covered": 4, "percentage": 100 },
      "P2": { "total": 3, "covered": 3, "percentage": 100 },
      "P3": { "total": 2, "covered": 2, "percentage": 100 }
    }
  },
  "blocking_findings": []
}
```

`story_ref` must be the resolved story target when TR is running for a story.
If a standalone legacy invocation has no story target, keep the key present and use `null` rather than fabricating an identity.
For Archon-routed invocations, a missing or mismatched story identity is an `ERROR`.

`round` must be the Archon loop round when it is supplied.
Use `1` only for standalone runs that have no loop metadata.

`trace_file` must point to machine-readable trace evidence.
Use `e2e-trace-summary.json` as the default because downstream route consumers must not parse `traceability-matrix.md`.

`report_file` must point to the human traceability report.
Use `traceability-matrix.md` as the default report pointer.

`coverage_gaps_count` should count unique oracle or requirement items that are not `FULL`.
A practical derivation is `coverageMatrix.requirements.filter((item) => item.coverage !== "FULL")`, deduplicated by `id`.

`findings_count` should count all trace findings that require attention.
A practical default is unique coverage gaps plus heuristic gaps plus deduplicated skipped, fixme, or pending test blockers from `coverageMatrix.test_inventory`.

`blocking_findings_count` should count only findings that block route progress.
A practical default is P0 coverage gaps, P1 coverage below the hard 80 percent minimum, overall coverage below 80 percent, and high-severity test blockers that affect mapped coverage.

### Gate Mapping

`PASS` means the TR run was trusted, the contract is valid, story identity matches, upstream branch evidence is resolved, and no route-blocking trace findings are open.

`FAIL` means the TR run was trusted and development work is needed to resolve blocking traceability gaps.

`ERROR` means TEA cannot trust execution, Phase 1 evidence, contract shape, story identity, upstream branch evidence, or output.
`ERROR` must not be converted into `FAIL`.

Do not emit `CONCERNS` as the route contract gate value in this v2 story.
The existing trace summary and gate decision outputs may keep `CONCERNS`, but `tea-tr.gate.json` must emit `PASS`, `FAIL`, or `ERROR` for executed TR.

Do not emit `WAIVED` as the route contract gate value.
If the existing trace gate records a waiver, preserve it in `e2e-trace-summary.json` or `gate-decision.json` and derive `tea-tr.gate.json` from the underlying blocking findings.
If the underlying blocking status cannot be trusted, emit `ERROR`.

Do not emit `SKIPPED` in this story.
Story T4.2 owns `tea-tr-skipped.gate.json` compatibility.

### Implementation Guidance

The best emission point is `src/workflows/testarch/bmad-testarch-trace/steps-c/step-05-gate-decision.md`.
That step already reads the Phase 1 coverage matrix, applies deterministic gate logic, builds `gateReport`, assembles `e2eTraceSummary`, writes `{e2e_trace_summary_output}`, and optionally writes `{gate_decision_output}`.

Build `tea-tr.gate.json` from the same in-memory objects instead of reading the generated Markdown report.
The route contract should be emitted after the summary object is stable and before the final workflow completion message.

Keep `e2e-trace-summary.json` as the richer automation summary.
`tea-tr.gate.json` is a smaller route-facing contract for Archon and should not replace the existing summary.

Keep `gate-decision.json` as the optional CI-friendly signal emitted only when gate-eligible.
`tea-tr.gate.json` should still be emitted for TR route runs, using `ERROR` when TR cannot evaluate trusted traceability evidence.

Treat Archon Story A3.3 as an external integration assumption.
This story should be implementable and testable locally with fixtures and without pulling external Archon project files.

Stories T2.1, T2.2, T3.1, and T3.2 are `ready-for-dev` at story creation time, not complete.
Do not assume their implementations exist.
If shared gate validators or fixtures exist when T4.1 is implemented, reuse them.
If they do not exist, create the smallest reusable helper needed for TR and keep it compatible with RV, NR, skipped contracts, and later T5.1 fixture aggregation.

### Current Trace Workflow State

`src/workflows/testarch/bmad-testarch-trace/workflow.yaml` currently declares `default_output_file`, `e2e_trace_summary_output`, and `gate_decision_output`.
It does not declare `tr_gate_output`.

`src/workflows/testarch/bmad-testarch-trace/workflow-plan.md` currently lists `traceability-matrix.md`, `e2e-trace-summary.json`, and `gate-decision.json` when gate-eligible.
It does not list `tea-tr.gate.json`.

`src/workflows/testarch/bmad-testarch-trace/steps-c/step-04-analyze-gaps.md` writes a complete Phase 1 coverage matrix to a temp JSON file and records `tempCoverageMatrixPath` in the progress document.
Do not bypass that mechanism.

`src/workflows/testarch/bmad-testarch-trace/steps-c/step-05-gate-decision.md` halts when `tempCoverageMatrixPath` is missing.
For the route contract, that failure path should become `gate: ERROR` or an invalid-contract result when the implementation can safely write the contract.

Step 5 currently writes `e2e-trace-summary.json` always.
It writes `gate-decision.json` only when gate evaluation was performed and produced `PASS`, `CONCERNS`, `FAIL`, or `WAIVED`.

The trace checklist already validates `e2e-trace-summary.json` and `gate-decision.json`.
It does not yet validate `tea-tr.gate.json`.

### Existing Trace Data To Reuse

`coverageMatrix.coverage_statistics` includes `total_requirements`, `fully_covered`, `partially_covered`, `uncovered`, `overall_coverage_percentage`, and priority breakdowns for P0 through P3.

`coverageMatrix.gap_analysis` includes `critical_gaps`, `high_gaps`, `medium_gaps`, `low_gaps`, `partial_coverage_items`, and `unit_only_items`.

`coverageMatrix.coverage_heuristics.counts` includes endpoint, auth negative-path, happy-path-only, UI journey, and UI state gap counts.

`coverageMatrix.test_inventory.summary` contains deduplicated test counts by file, case, skipped, fixme, pending, and level.

`coverageMatrix.test_inventory.blockers` contains deduplicated skipped, fixme, or pending test blockers.

`coverageMatrix.trace_target` is the closest existing local source for target type, target id, and label.
Use it for `story_ref` only when the target is the story under evaluation.

### Upstream Evidence Boundary

TR is the final traceability gate after resolved RV and NR branches.
When Archon supplies RV and NR contracts, TR should validate that they are resolved contracts for the same story and round.

Valid upstream branch evidence can be either an executed contract such as `tea-rv.gate.json` or `tea-nr.gate.json`, or a skipped contract such as `tea-rv-skipped.gate.json` or `tea-nr-skipped.gate.json`.

This story does not require implementing Archon branch orchestration.
Archon owns `run_rv`, `run_nr`, `run_tr`, `when:` expressions, and skipped node execution.

This story does require BMAD-TEA to fail closed when Archon-routed TR receives missing, malformed, mismatched, or untrusted upstream evidence.
Use local fixtures to prove the behavior.

### Testing Notes

Prefer plain Node tests that inspect workflow files and fixture output.
The repository already uses Node-based validation scripts and does not need a new dependency for this story.

Prefer fixture-driven contract tests under `test/fixtures/tea-gate-contracts/` if a reusable validator is introduced.
Recommended T4.1 fixture cases are PASS, FAIL with blocking trace gaps, story mismatch ERROR, invalid Phase 1 evidence ERROR, non-gate-eligible ERROR, and WAIVED or CONCERNS source output that does not leak into the route-facing `gate`.

If a JSON schema file is added, use an explicitly declared validator that supports the selected schema draft.
Do not rely on transitive `ajv` availability.

JSON Schema Draft 2020-12 is the current specification family listed by JSON Schema.
Ajv supports multiple JSON Schema drafts, but the implementation must choose the draft deliberately if Ajv is added directly.

The repository currently has `yaml` and `js-yaml` as direct dependencies.
Use built-in JSON APIs for writing the contract unless a schema validator is intentionally added as a direct dependency.

The package engine is currently `node >=20.0.0`.
Implementation can use modern Node file system APIs that are available in Node 20 and later.

As of 2026-06-30, Node 20 is listed as EOL by Node.js.
Changing the package engine is outside this story.

### Project Structure Notes

Canonical workflow source lives under `src/workflows/testarch/bmad-testarch-trace/`.
Generated or installed skill copies under `.agents/` and `.claude/` must not be treated as the implementation source.

Public documentation lives under `docs/`.
Only update docs when the implementation changes the documented user-facing output surface.

Release metadata must stay synchronized, but this story should not require package version changes.
Run `npm run test:release-metadata` after implementation to prove no accidental drift was introduced.

### References

- [Source: `_bmad-output/planning-artifacts/epics.md` Story T4.1]
- [Source: `_bmad-output/planning-artifacts/prd.md` T-FR-6]
- [Source: `_bmad-output/planning-artifacts/architecture.md` T-AD-4, T-AD-6, contract envelope, and `tea-tr.gate.json`]
- [Source: `_bmad-output/planning-artifacts/implementation-readiness-report-2026-06-30.md` story quality findings]
- [Source: `_bmad-output/implementation-artifacts/t1-1-emit-structured-ta-evidence.md` dependency context]
- [Source: `_bmad-output/implementation-artifacts/t2-1-emit-rv-gate-contract.md` common gate contract precedent]
- [Source: `_bmad-output/implementation-artifacts/t2-2-validate-rv-skipped-contract-compatibility.md` skipped-contract precedent]
- [Source: `_bmad-output/implementation-artifacts/t3-1-emit-nr-gate-contract.md` common gate contract precedent]
- [Source: `_bmad-output/implementation-artifacts/t3-2-validate-nr-skipped-contract-compatibility.md` skipped-contract precedent]
- [Source: `src/workflows/testarch/bmad-testarch-trace/workflow.yaml`]
- [Source: `src/workflows/testarch/bmad-testarch-trace/workflow-plan.md`]
- [Source: `src/workflows/testarch/bmad-testarch-trace/instructions.md`]
- [Source: `src/workflows/testarch/bmad-testarch-trace/checklist.md`]
- [Source: `src/workflows/testarch/bmad-testarch-trace/steps-c/step-04-analyze-gaps.md`]
- [Source: `src/workflows/testarch/bmad-testarch-trace/steps-c/step-05-gate-decision.md`]
- [Source: `src/workflows/testarch/bmad-testarch-trace/steps-v/step-01-validate.md`]
- [Source: `docs/how-to/workflows/run-trace.md`]
- [Source: `docs/reference/commands.md`]
- [Source: `docs/reference/configuration.md`]
- [Source: `src/module-help.csv`]
- [Source: `test/test-installation-components.js`]
- [Source: `package.json` engines and scripts]
- [Source: JSON Schema specifications at `https://json-schema.org/specification`]
- [Source: Ajv documentation at `https://ajv.js.org/`]
- [Source: Node.js previous releases at `https://nodejs.org/en/about/previous-releases`]

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

Ultimate context engine analysis completed - comprehensive developer guide created.

### File List
