# Story T3.2: Validate NR Skipped Contract Compatibility

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a test architect,
I want the NR skipped contract shape to be validated,
so that an intentional NR skip is not confused with missing evidence.

## Acceptance Criteria

1. Given `run_nr` is false and NR is intentionally skipped, when `tea-nr-skipped.gate.json` is validated, then the artifact is accepted only when it includes `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate: SKIPPED`, and `reason`.
2. Given an NR skipped contract is accepted, when downstream summary or traceability join logic reads it, then `node` is `NR`, `gate` is `SKIPPED`, `reason` is a non-empty string, and the NR branch is treated as intentionally resolved.
3. Given `tea-nr-skipped.gate.json` is missing, malformed JSON, missing required fields, has `gate` other than `SKIPPED`, has `node` other than `NR`, has an empty `reason`, or has a mismatched `story_ref` or `round`, when validation runs, then it is not accepted as skipped evidence and the validation result is `ERROR` or an equivalent fail-closed invalid-contract result.
4. Given both real NR and skipped NR contracts are present in fixtures, when compatibility validation runs, then downstream consumers can distinguish `tea-nr.gate.json` from `tea-nr-skipped.gate.json` without parsing Markdown.
5. Given Archon owns the decision to skip NR, when this story is implemented, then BMAD-TEA defines and validates the expected skipped contract shape but does not add NR skip-decision logic to the `bmad-testarch-nfr` workflow.

## Tasks / Subtasks

- [ ] Define the NR skipped contract shape and validation behavior.
  - [ ] If workflow metadata needs to name the skipped artifact, add `nr_skipped_gate_output: "{test_artifacts}/tea-nr-skipped.gate.json"` as an expected validator input path.
  - [ ] Document that `nr_skipped_gate_output` is Archon-produced skipped evidence, not an artifact emitted by a normal `nfr-assess` run.
  - [ ] Reuse a shared TEA gate contract validator from Story T2.2 or Story T3.1 if one exists when implementation starts.
  - [ ] If no shared helper exists, create the smallest reusable local validator needed for skipped TEA gate envelopes.
  - [ ] Require `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate`, and `reason` for skipped contracts.
  - [ ] Require `node` to equal `NR` for `tea-nr-skipped.gate.json`.
  - [ ] Require `gate` to equal `SKIPPED` exactly.
  - [ ] Require `reason` to be a non-empty string.
  - [ ] Treat missing files, malformed JSON, missing fields, wrong node, wrong gate, empty reason, story mismatch, or round mismatch as invalid evidence.
  - [ ] Keep `ERROR` or an equivalent invalid result separate from `SKIPPED`.
- [ ] Add deterministic fixtures for NR skipped compatibility.
  - [ ] Add a valid fixture for `tea-nr-skipped.gate.json`.
  - [ ] Add invalid fixtures for missing `reason`, wrong `gate`, wrong `node`, missing `story_ref`, and story or round mismatch.
  - [ ] Include a missing-file case in tests so intentional skip cannot be confused with absent evidence.
  - [ ] Keep fixture values local and self-contained so implementation does not need to read external Archon planning files.
- [ ] Add automated validation coverage.
  - [ ] Add or extend a focused test such as `test/test-tea-gate-contracts.js`.
  - [ ] If a new test script is added, wire it into `package.json` and `npm test`.
  - [ ] Assert the valid NR skipped fixture is accepted as `SKIPPED`.
  - [ ] Assert malformed, missing, wrong-node, wrong-gate, empty-reason, story-mismatch, and round-mismatch cases fail closed.
  - [ ] Assert validation does not parse Markdown reports to decide skipped compatibility.
- [ ] Update NFR workflow checklist coverage when the validation surface is added.
  - [ ] Update `src/workflows/testarch/bmad-testarch-nfr/checklist.md` if NR route-contract validation is documented there.
  - [ ] Cover accepted `SKIPPED`, missing evidence distinction, story identity, `node: NR`, integer `round`, and non-empty `reason`.
  - [ ] Keep trace `WAIVED`, NFR report `CONCERNS`, NFR worker `CONCERN`, and missing evidence separate from `gate: SKIPPED`.
- [ ] Align the skipped shape with the NR gate contract story without depending on its implementation being complete.
  - [ ] Reuse the common envelope from Story T3.1.
  - [ ] Do not require `nfr_categories`, `findings_count`, `blocking_findings_count`, or `report_file` for `tea-nr-skipped.gate.json` because no NR report exists when NR is intentionally skipped.
  - [ ] Do not make `bmad-testarch-nfr` emit `gate: SKIPPED`; Story T3.1 owns executed NR outcomes and Archon owns intentional skip emission.
  - [ ] If a shared validator is introduced for route gate stories, design it so later TR stories can reuse it without duplicating field checks.
- [ ] Update docs only where the compatibility contract becomes visible to users or downstream integrators.
  - [ ] Document that `tea-nr-skipped.gate.json` is an accepted route-facing skipped artifact when Archon intentionally skips NR.
  - [ ] State that missing skipped evidence is not the same as `gate: SKIPPED`.
  - [ ] Keep NR NFR decisions out of trace coverage semantics.
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
  "node": "NR",
  "round": 1,
  "gate": "SKIPPED",
  "generated_at": "2026-06-30T00:00:00.000Z",
  "reason": "run_nr was false because no NR-triggering NFR evidence signal was present",
  "artifact_pointers": {}
}
```

`workflow` identifies the producer of the skipped contract.
Because Archon owns intentional skip emission, validation should require a non-empty workflow string but should not hard-code a BMAD-TEA NFR workflow id unless the external Archon contract has already standardized that value.

`story_ref` must identify the story that the route loop is evaluating.
If a validator receives an expected story reference, mismatch must fail closed.

`round` must identify the route-loop round.
If a validator receives an expected round, mismatch must fail closed.

`reason` must be present and non-empty.
A skipped artifact without a reason is not acceptable evidence because downstream summary cannot explain why NR was resolved without execution.

Skipped NR contracts should not include NR execution fields such as `nfr_categories`, `findings_count`, `blocking_findings_count`, or `report_file`.
Those fields belong to executed NR contracts from Story T3.1.

### Validation Result Guidance

Prefer a deterministic result object over implicit truthy checks.
An implementation shape such as `{ ok: true, gate: "SKIPPED", node: "NR", errors: [] }` for valid contracts and `{ ok: false, gate: "ERROR", errors: [...] }` for invalid contracts is sufficient.

The exact helper API can follow repository style, but tests must prove missing evidence and malformed skipped evidence do not return `ok: true`.
Do not throw uncaught exceptions for ordinary invalid fixture cases.

### Implementation Boundary

BMAD-TEA owns the expected skipped-contract shape and local compatibility validation.
Archon owns `run_nr`, branch orchestration, skipped node execution, and writing `tea-nr-skipped.gate.json`.

This story must not add conditional skip behavior to `src/workflows/testarch/bmad-testarch-nfr/`.
That workflow runs NR when invoked.

`tea-nr-skipped.gate.json` is not an artifact emitted by a normal `nfr-assess` run.
It is route-facing skipped evidence supplied by Archon when Archon intentionally does not invoke NR.

The existing `trace` workflow is useful as a precedent for machine-readable gate artifacts, but its `WAIVED` decision is not the same as `SKIPPED`.
Do not map NR skipped contracts to `WAIVED`, `CONCERNS`, `CONCERN`, `PASS`, or `FAIL`.

### Current Repository State

There is no current `tea-nr-skipped.gate.json` fixture or validator in the repository.
There is no current TEA gate contract fixture set under `test/fixtures`.
There is no current automated test that accepts `SKIPPED` as a TEA route-facing contract outcome.

The closest existing validation style is plain Node scripts under `test/` and `tools/`.
Use CommonJS unless the surrounding file already uses another module format.

`test/test-installation-components.js` validates workflow structure, YAML, step anchors, and knowledge indexes.
It is not currently a route-contract validator.

`src/workflows/testarch/bmad-testarch-trace/workflow.yaml` and `steps-c/step-05-gate-decision.md` show how this repo documents and emits machine-readable artifacts.
Use them as style references, not as a skipped-gate enum source.

Trace currently uses `WAIVED` as a gate decision and also tracks skipped test cases.
Neither is equivalent to a route-facing `gate: SKIPPED` contract.

### Previous Story Intelligence

Story T3.1 defines the executed NR contract as `tea-nr.gate.json` with the common route envelope plus NR execution fields.
Story T3.1 explicitly leaves `gate: SKIPPED` to this story.

Story T3.1 is `ready-for-dev` at creation time, not complete.
If no shared contract helper exists yet when T3.2 implementation starts, create the smallest reusable helper needed for skipped validation and keep it compatible with the T3.1 envelope.

Story T2.2 is the closest skipped-contract precedent.
If Story T2.2 has already added reusable skipped validation by the time T3.2 starts, extend that helper instead of creating an NR-only validator.

Story T1.1 is also `ready-for-dev` at creation time.
Do not depend on implemented TA evidence when validating a skipped NR artifact.

### NFR Workflow Notes

`src/workflows/testarch/bmad-testarch-nfr/workflow.yaml` currently declares only `{test_artifacts}/nfr-assessment.md` as the default output.
It does not declare `nr_gate_output` or `nr_skipped_gate_output`.

`src/workflows/testarch/bmad-testarch-nfr/workflow-plan.md` currently lists only `nfr-assessment.md` as a workflow output.
Do not add skipped-output behavior there unless it is documented as external Archon input for validation.

`src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04e-aggregate-nfr.md` is the executed NR aggregation point from Story T3.1.
T3.2 should not make that step emit `gate: SKIPPED`.

`src/workflows/testarch/bmad-testarch-nfr/steps-c/step-05-generate-report.md` writes the human report.
Downstream route consumers must use JSON contracts instead of parsing that Markdown report for skipped status.

The current NFR worker domains are security, performance, reliability, and scalability.
Skipped validation does not need worker domain output because NR did not run.

### Testing Notes

Prefer fixture-driven tests under `test/fixtures/tea-gate-contracts/`.
Use focused valid and invalid JSON fixtures rather than generating fixture objects only inside test code.

If a JSON schema is introduced, choose and document the schema draft explicitly.
The repository currently has `yaml` and `js-yaml` as direct dependencies, while `ajv` is only present transitively through ESLint.
Do not rely on transitive `ajv` availability.

JSON Schema Draft 2020-12 is the current specification family listed by JSON Schema.
Ajv supports multiple JSON Schema drafts, but the implementation must choose the draft deliberately if Ajv is added directly.

The package engine is currently `node >=20.0.0`.
Implementation can use modern Node file system APIs that are available in Node 20 and later.

As of 2026-06-30, Node 20 is listed as EOL by Node.js.
Changing the package engine is outside this story.

### Project Structure Notes

Canonical implementation content lives under `src/`, and automated checks live under `test/` or `tools/`.
Generated or installed skill copies under `.agents/` and `.claude/` must not be treated as canonical implementation source.

Public documentation lives under `docs/`.
Only update docs when the skipped compatibility artifact becomes part of the documented user-facing or integrator-facing contract.

Release metadata must stay synchronized, but this story should not require package version changes.
Run `npm run test:release-metadata` after implementation to prove no accidental drift was introduced.

### References

- [Source: `_bmad-output/planning-artifacts/epics.md` Story T3.2]
- [Source: `_bmad-output/planning-artifacts/prd.md` T-FR-5]
- [Source: `_bmad-output/planning-artifacts/architecture.md` T-AD-5, T-AD-6, contract envelope, skipped contracts, and validation rules]
- [Source: `_bmad-output/planning-artifacts/implementation-readiness-report-2026-06-30.md` skipped-contract and external Archon assumptions]
- [Source: `_bmad-output/implementation-artifacts/t3-1-emit-nr-gate-contract.md` previous story context]
- [Source: `_bmad-output/implementation-artifacts/t2-2-validate-rv-skipped-contract-compatibility.md` skipped-contract precedent]
- [Source: `src/workflows/testarch/bmad-testarch-nfr/workflow.yaml`]
- [Source: `src/workflows/testarch/bmad-testarch-nfr/workflow-plan.md`]
- [Source: `src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04e-aggregate-nfr.md`]
- [Source: `src/workflows/testarch/bmad-testarch-nfr/steps-c/step-05-generate-report.md`]
- [Source: `src/workflows/testarch/bmad-testarch-trace/workflow.yaml`]
- [Source: `src/workflows/testarch/bmad-testarch-trace/steps-c/step-05-gate-decision.md`]
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
