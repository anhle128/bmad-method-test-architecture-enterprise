# Story T4.2: Validate TR Skipped Contract Compatibility

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a test architect,
I want the TR skipped contract shape to be validated,
so that a blocked release-gate evaluation can still produce a resolved TR role contract.

## Acceptance Criteria

1. Given `run_tr` is false because release-gate evaluation is already blocked and TR is intentionally skipped, when `tea-tr-skipped.gate.json` is validated, then the artifact is accepted only when it includes `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate: SKIPPED`, and `reason`.
2. Given a TR skipped contract is accepted, when downstream summary logic reads it, then `node` is `TR`, `gate` is `SKIPPED`, `reason` is a non-empty string, and the TR role is treated as intentionally resolved without trace execution.
3. Given `tea-tr-skipped.gate.json` is missing, malformed JSON, missing required fields, has `gate` other than `SKIPPED`, has `node` other than `TR`, has an empty `reason`, or has a mismatched `story_ref` or `round`, when validation runs, then it is not accepted as skipped evidence and the validation result is `ERROR` or an equivalent fail-closed invalid-contract result.
4. Given both executed TR and skipped TR contracts are present in fixtures, when compatibility validation runs, then downstream consumers can distinguish `tea-tr.gate.json` from `tea-tr-skipped.gate.json` without parsing Markdown.
5. Given Archon owns the decision to skip TR, when this story is implemented, then BMAD-TEA defines and validates the expected skipped contract shape but does not add TR skip-decision logic to the `bmad-testarch-trace` workflow.
6. Given current trace outputs may use `WAIVED`, `CONCERNS`, skipped test cases, pending test cases, or fixme test cases, when TR skipped compatibility is validated, then those trace-domain signals remain separate from route-level `gate: SKIPPED`.

## Tasks / Subtasks

- [ ] Define the TR skipped contract shape and validation behavior.
  - [ ] If workflow metadata needs to name the skipped artifact, add `tr_skipped_gate_output: "{test_artifacts}/tea-tr-skipped.gate.json"` as an expected validator input path.
  - [ ] Document that `tr_skipped_gate_output` is Archon-produced skipped evidence, not an artifact emitted by a normal `trace` run.
  - [ ] Reuse a shared TEA gate contract validator from Story T2.2, Story T3.2, or Story T4.1 if one exists when implementation starts.
  - [ ] If no shared helper exists, create the smallest reusable local validator needed for skipped TEA gate envelopes.
  - [ ] Require `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate`, and `reason` for skipped contracts.
  - [ ] Require `node` to equal `TR` for `tea-tr-skipped.gate.json`.
  - [ ] Require `gate` to equal `SKIPPED` exactly.
  - [ ] Require `reason` to be a non-empty string.
  - [ ] Require `story_ref` to match the expected story reference when the validator receives one.
  - [ ] Require `round` to match the expected route-loop round when the validator receives one.
  - [ ] Treat missing files, malformed JSON, missing fields, wrong node, wrong gate, empty reason, story mismatch, or round mismatch as invalid evidence.
  - [ ] Keep `ERROR` or an equivalent invalid result separate from `SKIPPED`.
- [ ] Add deterministic fixtures for TR skipped compatibility.
  - [ ] Add a valid fixture for `tea-tr-skipped.gate.json`.
  - [ ] Add invalid fixtures for missing `reason`, empty `reason`, wrong `gate`, wrong `node`, missing `story_ref`, story mismatch, and round mismatch.
  - [ ] Add a malformed JSON fixture or direct test case.
  - [ ] Include a missing-file case in tests so intentional skip cannot be confused with absent evidence.
  - [ ] Include an executed TR fixture for `tea-tr.gate.json` so compatibility tests prove executed and skipped contracts are distinguishable.
  - [ ] Include a trace-domain fixture or sample object with `WAIVED`, `CONCERNS`, and skipped test inventory so those signals are not accepted as route-level skipped evidence.
  - [ ] Keep fixture values local and self-contained so implementation does not need to read external Archon planning files.
- [ ] Add automated validation coverage.
  - [ ] Add or extend a focused test such as `test/test-tea-gate-contracts.js`.
  - [ ] If a new test script is added, wire it into `package.json` and `npm test`.
  - [ ] Assert the valid TR skipped fixture is accepted as `SKIPPED`.
  - [ ] Assert malformed, missing, wrong-node, wrong-gate, empty-reason, story-mismatch, and round-mismatch cases fail closed.
  - [ ] Assert validation returns `ERROR` or an equivalent invalid-contract result for ordinary invalid fixture cases without throwing uncaught exceptions.
  - [ ] Assert validation does not parse Markdown reports to decide skipped compatibility.
  - [ ] Assert `tea-tr.gate.json` and `tea-tr-skipped.gate.json` can be distinguished by contract file name and contract fields.
  - [ ] Assert `WAIVED`, `CONCERNS`, skipped tests, pending tests, and fixme tests are not accepted as `gate: SKIPPED`.
- [ ] Update trace workflow validation surfaces when the compatibility validator is added.
  - [ ] Update `src/workflows/testarch/bmad-testarch-trace/checklist.md` if TR route-contract validation is documented there.
  - [ ] Cover accepted `SKIPPED`, missing evidence distinction, story identity, `node: TR`, integer `round`, and non-empty `reason`.
  - [ ] Keep `traceability-matrix.md`, `e2e-trace-summary.json`, and `gate-decision.json` validation intact.
  - [ ] Keep trace `WAIVED`, trace `CONCERNS`, skipped test cases, pending test cases, fixme test cases, and missing evidence separate from `gate: SKIPPED`.
- [ ] Align the skipped shape with the executed TR gate contract story without depending on its implementation being complete.
  - [ ] Reuse the common envelope from Story T4.1.
  - [ ] Do not require `coverage_gaps_count`, `findings_count`, `blocking_findings_count`, `trace_file`, or `report_file` for `tea-tr-skipped.gate.json` because TR did not run and no trace report exists.
  - [ ] Do not make `bmad-testarch-trace` emit `gate: SKIPPED`; Story T4.1 owns executed TR outcomes and Archon owns intentional skip emission.
  - [ ] If a shared route gate validator is introduced, design it so Story T5.1 fixtures can reuse it without duplicating field checks.
  - [ ] Accept executed or skipped RV and NR branch contracts as upstream context only when route validation supplies them.
  - [ ] Do not require upstream RV or NR files to validate the standalone TR skipped contract shape.
- [ ] Update docs only where the compatibility contract becomes visible to users or downstream integrators.
  - [ ] Document that `tea-tr-skipped.gate.json` is an accepted route-facing skipped artifact when Archon intentionally skips TR because the release-gate evaluation is already blocked.
  - [ ] State that missing skipped evidence is not the same as `gate: SKIPPED`.
  - [ ] State that `tea-tr-skipped.gate.json` is not emitted by normal `bmad-testarch-trace` execution.
  - [ ] State that trace `WAIVED`, trace `CONCERNS`, and skipped test inventory are not route-level `SKIPPED`.
  - [ ] Do not manually edit `CHANGELOG.md`; follow the maintainer release process if a changelog entry is required.
- [ ] Run validation.
  - [ ] Run the focused skipped-contract test.
  - [ ] Run `npm run test:install`.
  - [ ] Run `npm run test:tea-workflow-descriptions`.
  - [ ] Run `npm run lint`.
  - [ ] Run `npm run lint:md`.
  - [ ] Run `npm run format:check`.
  - [ ] Run `npm run test:release-metadata`.
  - [ ] Run `npm run docs:validate-links` if docs are changed.
  - [ ] Run `npm test` when the focused checks pass locally.

## Dev Notes

### Skipped Contract Shape

Use this local fixture shape unless a later project contract version already exists when development starts.

```json
{
  "contract_version": "1.0",
  "workflow": "archon-quality-route-loop",
  "story_ref": "story-id-or-key",
  "node": "TR",
  "round": 1,
  "gate": "SKIPPED",
  "generated_at": "2026-06-30T00:00:00.000Z",
  "reason": "run_tr was false because release-gate evaluation was already blocked before final traceability review",
  "artifact_pointers": {}
}
```

`workflow` identifies the producer of the skipped contract.
Because Archon owns intentional skip emission, validation should require a non-empty workflow string but should not hard-code a BMAD-TEA trace workflow id unless the external Archon contract has already standardized that value.

`story_ref` must identify the story that the route loop is evaluating.
If a validator receives an expected story reference, mismatch must fail closed.

`round` must identify the route-loop round.
If a validator receives an expected round, mismatch must fail closed.

`reason` must be present and non-empty.
A skipped artifact without a reason is not acceptable evidence because downstream summary cannot explain why TR was resolved without execution.

Skipped TR contracts should not include TR execution fields such as `coverage_gaps_count`, `findings_count`, `blocking_findings_count`, `trace_file`, or `report_file`.
Those fields belong to executed TR contracts from Story T4.1.

### Validation Result Guidance

Prefer a deterministic result object over implicit truthy checks.
An implementation shape such as `{ ok: true, gate: "SKIPPED", node: "TR", errors: [] }` for valid contracts and `{ ok: false, gate: "ERROR", errors: [...] }` for invalid contracts is sufficient.

The exact helper API can follow repository style, but tests must prove missing evidence and malformed skipped evidence do not return `ok: true`.
Do not throw uncaught exceptions for ordinary invalid fixture cases.

If JSON Schema is used, pin the schema draft explicitly and use a direct dependency such as Ajv instead of relying on a transitive package.
If a small hand-written validator is simpler, keep the checks declarative and fixture-driven.

### Implementation Boundary

BMAD-TEA owns the expected skipped-contract shape and local compatibility validation.
Archon owns `run_tr`, branch orchestration, skipped node execution, and writing `tea-tr-skipped.gate.json`.

This story must not add conditional skip behavior to `src/workflows/testarch/bmad-testarch-trace/`.
That workflow runs TR when invoked.

`tea-tr-skipped.gate.json` is not an artifact emitted by a normal `trace` run.
It is route-facing skipped evidence supplied by Archon when Archon intentionally does not invoke TR.

Story T4.1 owns the executed TR contract `tea-tr.gate.json`.
Story T4.2 owns compatibility validation for `tea-tr-skipped.gate.json`.

Stories T2.2 and T3.2 are the closest templates for skipped-contract validation.
Mirror their fail-closed behavior, fixture strategy, and boundary between intentional skip and missing evidence.

Treat Archon Story A3.3 as an external integration assumption.
This story should be implementable and testable locally with fixtures and without pulling external Archon project files.

### Route Gate Semantics

`SKIPPED` means Archon intentionally resolved the TR route role without running TR because release-gate evaluation was already blocked.
It does not mean the trace workflow produced a waiver.
It does not mean tests inside the trace evidence were skipped.
It does not mean trace evidence was missing.

`ERROR` means TEA cannot trust execution, evidence, contract shape, story identity, route round identity, or output.
`ERROR` must not be converted into `FAIL`.
Missing skipped evidence must produce `ERROR` or an equivalent invalid-contract result, not `SKIPPED`.

`FAIL` and `PASS` belong to executed TR contracts from `tea-tr.gate.json`.
Do not map TR skipped contracts to `WAIVED`, `CONCERNS`, `PASS`, or `FAIL`.

Existing `e2e-trace-summary.json` and `gate-decision.json` semantics may continue to use trace-domain values such as `PASS`, `CONCERNS`, `FAIL`, and `WAIVED`.
Those files are not sufficient to prove route-level skipped compatibility.

### Current Repository State

There is no current `tea-tr-skipped.gate.json` fixture or validator in the repository.
There is no current TEA gate contract fixture set under `test/fixtures`.
There is no current automated test that accepts `SKIPPED` as a TR route-facing contract outcome.

`src/workflows/testarch/bmad-testarch-trace/workflow.yaml` currently declares `default_output_file`, `e2e_trace_summary_output`, and `gate_decision_output`.
It does not declare `tr_gate_output` or `tr_skipped_gate_output`.

`src/workflows/testarch/bmad-testarch-trace/checklist.md` validates the traceability matrix, `e2e-trace-summary.json`, and `gate-decision.json`.
It does not yet validate `tea-tr.gate.json` or `tea-tr-skipped.gate.json`.

`src/workflows/testarch/bmad-testarch-trace/steps-c/step-05-gate-decision.md` currently emits trace-domain gate decisions.
It treats `WAIVED`, `CONCERNS`, and skipped test inventory as trace evidence semantics, not route-level skipped contract semantics.

The closest existing validation style is plain Node scripts under `test/` and `tools/`.
Use CommonJS unless the surrounding file already uses another module format.

### Testing Notes

Create local fixtures that prove all required fields are present and typed correctly.
At minimum, validate `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate`, and `reason`.

Validate `round` as a route-loop identity value.
An integer is the safest initial constraint unless an existing project validator already supports a stricter route-round type.

Validate `reason` after trimming whitespace.
Whitespace-only reasons must fail closed.

Validate both identity and compatibility.
A syntactically valid skipped contract with the wrong `story_ref`, wrong `round`, or wrong `node` is not acceptable evidence for this route role.

Keep the test fixtures deterministic.
Do not depend on live Archon files, current time, generated Markdown, or network access.

### References

- PRD requirement: `_bmad-output/planning-artifacts/prd.md`, `T-FR-7`
- Epic requirement: `_bmad-output/planning-artifacts/epics.md`, `Story T4.2`
- Architecture decisions: `_bmad-output/planning-artifacts/architecture.md`, `T-AD-4`, `T-AD-5`, and `T-AD-6`
- Executed TR contract story: `_bmad-output/implementation-artifacts/t4-1-emit-tr-traceability-contract.md`
- Skipped RV precedent: `_bmad-output/implementation-artifacts/t2-2-validate-rv-skipped-contract-compatibility.md`
- Skipped NR precedent: `_bmad-output/implementation-artifacts/t3-2-validate-nr-skipped-contract-compatibility.md`
- JSON Schema specification: [json-schema.org/specification](https://json-schema.org/specification)
- Ajv JSON Schema validator: [ajv.js.org](https://ajv.js.org/)
