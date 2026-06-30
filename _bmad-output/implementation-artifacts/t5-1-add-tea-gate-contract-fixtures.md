# Story T5.1: Add TEA Gate Contract Fixtures

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a test architect,
I want deterministic fixtures for TEA gate contracts,
so that Archon can trust TEA route-facing outputs.

## Acceptance Criteria

1. Given passing RV, NR, and TR fixture contracts are validated, when the TEA gate contract suite runs, then each executed contract is accepted with `gate: PASS`, the correct `node`, the common envelope fields, the required gate-specific fields, a consistent `story_ref`, a consistent `round`, and report or artifact pointers where required.
2. Given blocking-finding RV, NR, and TR fixture contracts are validated, when the TEA gate contract suite runs, then each executed contract is accepted with `gate: FAIL`, `blocking_findings_count` greater than zero, and no conversion of route `FAIL` into route `ERROR`.
3. Given valid skipped fixture contracts for RV, NR, and TR are validated, when downstream consumers read them, then each artifact is accepted with `gate: SKIPPED`, the correct `node`, a non-empty `reason`, a consistent `story_ref`, and a consistent `round`.
4. Given valid ERROR fixture contracts for RV, NR, and TR are validated, when the TEA gate contract suite runs, then each artifact is accepted as a trusted `gate: ERROR` contract and remains distinct from `gate: FAIL`.
5. Given invalid or untrusted fixture inputs exist, including malformed JSON, missing required fields, wrong node, wrong gate, missing report pointers, story mismatch, round mismatch, invalid counts, or missing skipped reason, when validation runs, then the result is `ERROR` or an equivalent fail-closed invalid-contract result without uncaught exceptions.
6. Given an Archon-summary-like fixture consumes a mix of executed and skipped TEA contracts, when the fixture is validated, then downstream summary logic can classify resolved nodes, blocking findings, skipped nodes, and ERROR outcomes without parsing Markdown.
7. Given this story adds the aggregate fixture suite, when implementation is complete, then the focused suite is wired into `npm test`, fixture files are deterministic and local, no external Archon files are read, and `CHANGELOG.md` is not manually edited.

## Tasks / Subtasks

- [ ] Define the shared TEA gate contract validation surface.
  - [ ] Reuse an existing gate-contract validator from Stories T2.2, T3.2, T4.1, or T4.2 if one exists when implementation starts.
  - [ ] If no shared helper exists, create a focused CommonJS helper such as `tools/tea-gate-contract-validator.js`.
  - [ ] Export a deterministic API such as `validateTeaGateContract(contract, expectations)` and, if useful, `loadTeaGateContract(filePath, expectations)`.
  - [ ] Return result objects such as `{ ok: true, gate: "PASS", node: "RV", errors: [] }` for valid contracts and `{ ok: false, gate: "ERROR", errors: [...] }` for invalid inputs.
  - [ ] Do not throw uncaught exceptions for ordinary invalid fixture cases.
  - [ ] Require the common route envelope: `contract_version`, `workflow`, `story_ref`, `node`, `round`, and `gate`.
  - [ ] Require `workflow` to be a non-empty string.
  - [ ] Require `story_ref` to match the expected story reference when an expectation is provided.
  - [ ] Require `round` to be a positive integer and match the expected round when an expectation is provided.
  - [ ] Accept only route-facing gate values `PASS`, `FAIL`, `SKIPPED`, and `ERROR` for this v2 fixture suite.
  - [ ] Treat `CONCERNS`, `CONCERN`, `WAIVED`, skipped tests, pending tests, and fixme tests as non-route-gate signals that cannot satisfy `SKIPPED`.
- [ ] Add deterministic fixture organization under `test/fixtures/tea-gate-contracts/`.
  - [ ] Add a fixture manifest that lists every fixture file, expected node, expected gate, expected validity, expected story reference, expected round, and purpose.
  - [ ] Use a single stable route identity across fixtures unless a fixture intentionally tests mismatch.
  - [ ] Use stable timestamps such as `2026-06-30T00:00:00.000Z` instead of current time.
  - [ ] Keep all fixture data local and self-contained.
  - [ ] Do not write fixtures into `_bmad-output` because runtime output is ignored and user-specific.
  - [ ] Do not require generated Markdown reports to exist unless the validator explicitly tests report pointer existence with local fixture files.
- [ ] Add executed RV contract fixtures.
  - [ ] Add `PASS`, `FAIL`, and trusted `ERROR` RV fixtures for `tea-rv.gate.json`.
  - [ ] Require `node: RV`.
  - [ ] Require `quality_score`, `findings_count`, `blocking_findings_count`, and `report_file`.
  - [ ] Require `artifact_pointers.test_review_report` when artifact pointers are present.
  - [ ] Assert RV `PASS` fixtures have `blocking_findings_count: 0`.
  - [ ] Assert RV `FAIL` fixtures have `blocking_findings_count` greater than zero and include at least one blocking finding about test quality or reliability.
  - [ ] Do not copy the Story T2.1 sample inconsistency where `gate: PASS` appeared with `blocking_findings_count: 1`.
  - [ ] Keep RV semantics focused on reliability, assertions, fixtures, timing, flakiness, and test meaning, not coverage.
- [ ] Add executed NR contract fixtures.
  - [ ] Add `PASS`, `FAIL`, and trusted `ERROR` NR fixtures for `tea-nr.gate.json`.
  - [ ] Require `node: NR`.
  - [ ] Require `nfr_categories`, `findings_count`, `blocking_findings_count`, and `report_file`.
  - [ ] Require `nfr_categories` entries to include category name, status, risk level, findings count, blocking findings count, and evidence count.
  - [ ] Assert NR `PASS` fixtures have `blocking_findings_count: 0`.
  - [ ] Assert NR `FAIL` fixtures have `blocking_findings_count` greater than zero and include at least one missing or failing NFR evidence finding.
  - [ ] Keep NR semantics focused on NFR evidence and do not treat coverage gaps as NR failures.
- [ ] Add executed TR contract fixtures.
  - [ ] Add `PASS`, `FAIL`, and trusted `ERROR` TR fixtures for `tea-tr.gate.json`.
  - [ ] Require `node: TR`.
  - [ ] Require `coverage_gaps_count`, `findings_count`, `blocking_findings_count`, `trace_file`, and `report_file`.
  - [ ] Require `artifact_pointers.trace_summary` and `artifact_pointers.traceability_matrix` when artifact pointers are present.
  - [ ] Assert TR `PASS` fixtures have `blocking_findings_count: 0` and `coverage_gaps_count: 0`.
  - [ ] Assert TR `FAIL` fixtures have `blocking_findings_count` greater than zero and include at least one blocking traceability gap.
  - [ ] Keep trace-domain `WAIVED` and `CONCERNS` out of route-facing `tea-tr.gate.json` gates.
- [ ] Add skipped-contract fixtures for RV, NR, and TR.
  - [ ] Add valid `tea-rv-skipped.gate.json`, `tea-nr-skipped.gate.json`, and `tea-tr-skipped.gate.json` fixtures.
  - [ ] Require `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate: SKIPPED`, and `reason`.
  - [ ] Require `reason` to be a non-empty string after trimming whitespace.
  - [ ] Do not require executed-only fields such as `quality_score`, `nfr_categories`, `coverage_gaps_count`, `findings_count`, `blocking_findings_count`, `trace_file`, or `report_file` for skipped fixtures.
  - [ ] Assert skipped contracts are distinguishable from missing files.
  - [ ] Assert skipped contracts are distinguishable from trace `WAIVED`, trace `CONCERNS`, skipped tests, pending tests, and fixme tests.
- [ ] Add invalid fixture cases and fail-closed tests.
  - [ ] Add malformed JSON coverage.
  - [ ] Add missing required envelope field coverage.
  - [ ] Add wrong node coverage for each contract family.
  - [ ] Add wrong gate coverage for executed and skipped contracts.
  - [ ] Add story mismatch and round mismatch coverage.
  - [ ] Add invalid count coverage for negative counts, non-integer counts, and inconsistent PASS with blocking counts.
  - [ ] Add missing or empty skipped reason coverage.
  - [ ] Add missing report pointer coverage for executed contracts.
  - [ ] Add a missing-file test case that proves absence is `ERROR` or invalid, not `SKIPPED`.
- [ ] Add an Archon-summary-like compatibility fixture.
  - [ ] Add a local fixture that represents downstream summary input using a mix of executed and skipped TEA contracts.
  - [ ] Include at least one all-resolved path, one blocking `FAIL` path, one skipped branch path, and one `ERROR` path.
  - [ ] Assert summary classification uses JSON fields and file names, not Markdown report contents.
  - [ ] Treat Archon as an external consumer assumption and do not read files outside this repository.
- [ ] Wire the focused test suite into repository commands.
  - [ ] Add `test/test-tea-gate-contracts.js` using the existing plain Node test style unless an implemented local helper already uses another test harness.
  - [ ] Add `test:tea-gate-contracts` to `package.json`.
  - [ ] Add `npm run test:tea-gate-contracts` to the aggregate `npm test` command.
  - [ ] Update `package-lock.json` only if `package.json` changes.
  - [ ] Do not add Ajv only because it is present transitively through ESLint.
  - [ ] If JSON Schema is introduced, add Ajv as a direct dependency or devDependency and update `package-lock.json`.
  - [ ] If a hand-written validator is simpler, avoid adding new dependencies.
- [ ] Update documentation only where the fixture contract becomes visible.
  - [ ] Add or update `test/fixtures/tea-gate-contracts/README.md` if needed to explain the fixture manifest and expected outcomes.
  - [ ] Update `test/README.md` if the test suite inventory should mention TEA gate contract validation.
  - [ ] Update public docs only if the implemented fixture suite becomes a documented integrator contract.
  - [ ] Do not manually edit `CHANGELOG.md`.
- [ ] Run validation.
  - [ ] Run `npm run test:tea-gate-contracts`.
  - [ ] Run `npm run test:install`.
  - [ ] Run `npm run test:tea-workflow-descriptions`.
  - [ ] Run `npm run test:release-metadata`.
  - [ ] Run `npm run lint`.
  - [ ] Run `npm run lint:md`.
  - [ ] Run `npm run format:check`.
  - [ ] Run `npm run docs:validate-links` if public docs or test docs are changed.
  - [ ] Run `npm test` when the focused suite passes.

## Dev Notes

### Source Planning Context

Epic T5 says BMAD-TEA proves route-facing gate contracts are stable and compatible with Archon.
Story T5.1 is the aggregate validation story for RV, NR, TR, skipped contracts, and ERROR cases.

PRD requirement `T-FR-8` requires deterministic tests or fixtures for PASS, FAIL, SKIPPED compatibility, and ERROR outcomes.
The PRD acceptance criteria explicitly cover RV, NR, and TR passing fixtures, blocking-finding FAIL fixtures, skipped-contract fixtures, and invalid or untrusted ERROR outcomes.

Architecture decision `T-AD-6` says `ERROR` is separate from `FAIL`.
This fixture suite must preserve that distinction because `FAIL` means development can fix findings while `ERROR` means TEA cannot trust execution, evidence, contract shape, story identity, or output.

Architecture validation rules require RV, NR, and TR PASS, FAIL, and ERROR fixtures, skipped fixtures for RV, NR, and TR, story identity mismatch ERROR coverage, report links, and Archon summary compatibility.

Treat Archon dependencies as local fixture assumptions.
Do not traverse out of this repository to read Archon story files or parent workspace planning documents.

### Contract Families

Executed RV fixtures target `tea-rv.gate.json`.
They must use the common envelope plus `quality_score`, `findings_count`, `blocking_findings_count`, and `report_file`.

Executed NR fixtures target `tea-nr.gate.json`.
They must use the common envelope plus `nfr_categories`, `findings_count`, `blocking_findings_count`, and `report_file`.

Executed TR fixtures target `tea-tr.gate.json`.
They must use the common envelope plus `coverage_gaps_count`, `findings_count`, `blocking_findings_count`, `trace_file`, and `report_file`.

Skipped fixtures target `tea-rv-skipped.gate.json`, `tea-nr-skipped.gate.json`, and `tea-tr-skipped.gate.json`.
They must use the skipped envelope with `gate: SKIPPED` and `reason`.

Trusted ERROR fixtures are valid contracts whose `gate` is `ERROR`.
Invalid fixture inputs are malformed or mismatched data that should make validation return `ERROR` or an invalid-contract result.
Keep these two categories distinct in the manifest so the test names stay clear.

TA evidence from Story T1.1 is planning evidence, not a route quality gate.
This story may include a TA evidence fixture only when it helps downstream summary or route-planning compatibility, but T5.1 acceptance is about RV, NR, TR, skipped contracts, and ERROR route outcomes.

### Required Invariants

All fixtures must use deterministic JSON.
Avoid current time, machine-specific absolute paths, random IDs, network calls, or generated Markdown parsing.

Use a stable story reference such as `story-fixture-route-001` for valid fixtures.
Use a stable route round such as `1` for valid fixtures.

A contract with a mismatched `story_ref` is not acceptable evidence for the current route evaluation.
A contract with a mismatched `round` is not acceptable evidence for the current route evaluation.

`PASS` fixtures must not include route-blocking findings.
For this suite, `blocking_findings_count` must be `0` for `PASS`.

`FAIL` fixtures must include route-blocking findings.
For this suite, `blocking_findings_count` must be greater than `0` for `FAIL`.

`SKIPPED` fixtures must not be substituted by missing files.
Missing evidence must fail closed.

`ERROR` fixtures and invalid-contract validation results must not be converted into `FAIL`.

`CONCERNS`, `CONCERN`, and `WAIVED` are not accepted route gates for this v2 fixture suite.
They may appear only in nested trace-domain or NFR-domain sample data that the validator explicitly keeps separate from the route `gate`.

### Current Repository State

There is no current `test/fixtures/tea-gate-contracts/` fixture set.
There is no current `test/test-tea-gate-contracts.js`.
There is no current `test:tea-gate-contracts` package script.

The existing automated tests are plain Node scripts.
`test/test-installation-components.js` is the closest structural pattern, but it validates workflow source files rather than route contracts.

`package.json` currently defines `npm test` as a sequence of schema, install, knowledge, release metadata, TEA workflow description, validation, lint, markdown lint, and format checks.
T5.1 should insert the focused TEA gate contract test into that sequence after it exists.

The repository currently has no direct `ajv` dependency.
`package-lock.json` contains transitive Ajv entries through other tooling, but implementations must not rely on those as public project dependencies.

The local runtime used during story creation is Node `v22.22.3`, while `package.json` declares `node >=20.0.0`.
Do not require Node 24-only APIs unless the package engine is intentionally changed in a separate story.

`test/fixtures/**` is ignored by Prettier and markdownlint.
That makes it suitable for intentionally malformed fixture files, but developers still need deterministic formatting for valid JSON fixtures.

`_bmad-output` is ignored by git and should not be the canonical home for committed fixture data.

### Implementation Guidance

Prefer one validator that understands the common envelope and dispatches gate-specific checks by `node` and expected mode.
Do not create RV-only, NR-only, and TR-only validators unless earlier stories already established that pattern and refactoring would create risk.

Keep file-name conventions aligned with contract semantics.
For example, executed fixture names should identify node and expected gate, while skipped fixture names should include `skipped`.

Use the fixture manifest as the source of truth for expected results.
The test runner should iterate the manifest rather than hard-coding every expected result inside the test body.

If JSON Schema is used, select and document a schema dialect such as draft 2020-12.
Ajv supports modern JSON Schema dialects, but draft 2020-12 has different handling than older drafts, so do not mix dialect assumptions casually.

If a small hand-written validator is used, keep the checks explicit and table-driven.
This story is mainly contract compatibility validation, not a general-purpose schema framework.

Keep the Archon summary fixture small and local.
It should prove BMAD-TEA contracts are consumable without making BMAD-TEA own Archon orchestration.

### Previous Story Intelligence

Stories T2.1, T3.1, and T4.1 define the executed RV, NR, and TR contract shapes.
Stories T2.2, T3.2, and T4.2 define skipped contract compatibility for RV, NR, and TR.

All prior route-gate stories were `ready-for-dev` when this story was created.
Do not assume their implementations already exist.
If their implementation introduced a shared helper by the time T5.1 starts, extend that helper instead of duplicating it.

Story T4.2 explicitly says Story T5.1 fixtures should reuse any shared route gate validator without duplicating field checks.
Honor that handoff.

The skipped-contract stories consistently require missing, malformed, wrong-node, wrong-gate, empty-reason, story-mismatch, and round-mismatch cases to fail closed.
T5.1 should aggregate those cases across RV, NR, and TR.

The executed TR story keeps `CONCERNS`, `WAIVED`, and `SKIPPED` out of `tea-tr.gate.json`.
T5.1 should prevent those trace-domain values from leaking into route-gate validation.

### Git Intelligence Summary

Recent commits are repository configuration and handoff commits, not TEA gate implementation commits.
Do not infer existing contract validator implementation from git history.

The current tracked worktree already has sprint-status changes from story creation work.
The T5.1 story file itself is under ignored `_bmad-output`, while `sprint-status.yaml` is tracked.

### Latest Technical Notes

JSON Schema draft 2020-12 is the current JSON Schema specification family.
If schema files are added, make the selected dialect explicit instead of relying on defaults.

Ajv documentation distinguishes draft 2020-12 usage from earlier drafts.
If Ajv is introduced, add it as a direct dependency and instantiate the correct validator for the chosen draft.

Node.js 20 is listed as EOL on the official Node previous releases page as of this story creation date.
Changing the repository engine is outside T5.1, so write the validator for the declared `node >=20.0.0` floor unless a separate engine-change story is created.

### Project Structure Notes

Canonical implementation content lives under `src/`, `tools/`, and `test/`.
Generated or installed skill copies under `.agents/` and `.claude/` must not be treated as the implementation source.

Public documentation lives under `docs/`.
Only update public docs when the fixture suite becomes part of the documented integrator-facing contract.

Release metadata must stay synchronized across `package.json`, `package-lock.json`, and `.claude-plugin/marketplace.json`.
This story should not require a version bump.

Do not manually edit `CHANGELOG.md`.
This repository's AGENTS guidance forbids manual changelog edits even though the general repository changelog discipline mentions release notes.

## References

- [Source: `_bmad-output/planning-artifacts/epics.md` Story T5.1]
- [Source: `_bmad-output/planning-artifacts/prd.md` T-FR-8]
- [Source: `_bmad-output/planning-artifacts/architecture.md` Contract Envelope and Validation Rules]
- [Source: `_bmad-output/planning-artifacts/implementation-readiness-report-2026-06-30.md` Epic Structure Validation]
- [Source: `_bmad-output/implementation-artifacts/t2-1-emit-rv-gate-contract.md` RV contract shape]
- [Source: `_bmad-output/implementation-artifacts/t2-2-validate-rv-skipped-contract-compatibility.md` RV skipped contract shape]
- [Source: `_bmad-output/implementation-artifacts/t3-1-emit-nr-gate-contract.md` NR contract shape]
- [Source: `_bmad-output/implementation-artifacts/t3-2-validate-nr-skipped-contract-compatibility.md` NR skipped contract shape]
- [Source: `_bmad-output/implementation-artifacts/t4-1-emit-tr-traceability-contract.md` TR contract shape]
- [Source: `_bmad-output/implementation-artifacts/t4-2-validate-tr-skipped-contract-compatibility.md` TR skipped contract shape]
- [Source: `package.json` scripts and dependencies]
- [Source: `test/test-installation-components.js` existing CommonJS test style]
- JSON Schema specification: [json-schema.org/specification](https://json-schema.org/specification)
- Ajv JSON Schema validator: [ajv.js.org](https://ajv.js.org/)
- Node.js previous releases: [nodejs.org/en/about/previous-releases](https://nodejs.org/en/about/previous-releases)

## Dev Agent Record

### Agent Model Used

TBD by dev agent.

### Debug Log References

### Completion Notes List

### File List
