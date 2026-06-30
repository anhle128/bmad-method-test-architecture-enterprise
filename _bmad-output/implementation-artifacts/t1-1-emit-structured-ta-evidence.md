---
baseline_commit: d94b142af14f568811f8abaf269be53f8d9d9ead
---

# Story T1.1: Emit Structured TA Evidence

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a test architect,
I want `TA` to emit structured evidence pointers,
so that Archon can plan release gates without parsing TEA markdown reports.

## Acceptance Criteria

1. Given `TA` runs, when automation evidence is written, then structured evidence identifies changed tests, quality concerns, and NFR signals.
2. Given `TA` runs, when automation evidence is written, then the structured evidence includes a pointer to the human `automation-summary.md` report.
3. Given Archon plans gates, when it consumes TA evidence, then it can make route-planning decisions without parsing Markdown prose.
4. Given evidence is missing or invalid, when a consumer validates it, then the result can be treated as `ERROR`.

## Tasks / Subtasks

- [x] Define the TA structured evidence contract. (AC: 1, 2, 3, 4)
  - [x] Add a stable output path for `{test_artifacts}/tea-ta.evidence.json`.
  - [x] Keep `{test_artifacts}/automation-summary.md` as the human-readable report.
  - [x] Treat TA as planning evidence, not as a quality gate.
  - [x] Use `gate: "PASS"` only to mean the evidence file itself is valid and consumable.
  - [x] Use `gate: "ERROR"` only when evidence is untrusted, incomplete, or invalid.
- [x] Update the automate workflow instructions to emit structured evidence. (AC: 1, 2, 3)
  - [x] Update `src/workflows/testarch/bmad-testarch-automate/workflow.yaml` with a `ta_evidence_output` variable.
  - [x] Update `src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md` to aggregate changed tests, quality concerns, NFR signals, fixture needs, and source worker metadata into machine-readable data.
  - [x] Update `src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md` to validate and write `{ta_evidence_output}` before workflow completion.
  - [x] Update `src/workflows/testarch/bmad-testarch-automate/workflow-plan.md` so the listed outputs include `tea-ta.evidence.json`.
- [x] Add validation guidance for the evidence file. (AC: 4)
  - [x] Update `src/workflows/testarch/bmad-testarch-automate/checklist.md` with required TA evidence checks.
  - [x] Inspect `src/workflows/testarch/bmad-testarch-automate/steps-v/step-01-validate.md` and update it if validate mode needs to load or schema-check the evidence JSON directly.
  - [x] Require every structured evidence file to include the envelope and arrays listed in the contract below.
  - [x] Require missing required fields, invalid JSON, unexpected `story_ref`, or missing report pointers to be valid reasons for a consumer to treat the result as `ERROR`.
- [x] Add deterministic repository tests for the workflow instructions. (AC: 1, 2, 3, 4)
  - [x] Extend `test/test-installation-components.js` or add a focused test under `test/` that checks the automate workflow defines and documents `tea-ta.evidence.json`.
  - [x] Assert `workflow.yaml` exposes `ta_evidence_output`.
  - [x] Assert Step 3C and Step 4 mention the required evidence fields.
  - [x] Assert the checklist includes validation for missing or invalid structured evidence.
- [x] Preserve existing behavior. (AC: 1, 2, 3, 4)
  - [x] Do not remove or rename `automation-summary.md`.
  - [x] Do not change subagent temp output paths unless the full create-mode flow is updated consistently.
  - [x] Do not break the stable worker output contract across `agent-team`, `subagent`, and `sequential` execution modes.
  - [x] Do not change the TA agent menu code or skill dispatch.
  - [x] Do not modify release metadata, generated files, or `CHANGELOG.md`.

### Review Findings

- [x] [Review][Patch] Strengthen Step 4 evidence validation so PASS requires complete envelope data, valid report pointers, finite round, expected story reference, complete coverage, and consistent generated test file pointers [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:81]
- [x] [Review][Patch] Ensure Step 4 writes ERROR evidence when the Step 3C summary seed is missing or invalid JSON instead of aborting before `{ta_evidence_output}` is written [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:81]
- [x] [Review][Patch] Aggregate API and E2E priority coverage from per-test worker records instead of nonexistent top-level `priority_coverage` fields [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:343]
- [x] [Review][Patch] Harden Step 3C worker-output normalization so malformed or partial worker arrays become quality concerns instead of crashing aggregation [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:269]
- [x] [Review][Patch] Preserve NFR signal defaults when worker signals omit or null category, source, or evidence fields [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:325]
- [x] [Review][Patch] Add behavior-level tests for Step 4 evidence validation, ERROR fallback, and priority aggregation instead of only checking source text mentions required fields [test/test-installation-components.js:475]
- [x] [Review][Patch] Add `fixture_needs` to the structured TA evidence seed instead of reducing fixture information to a count [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:424]
- [x] [Review][Patch] Harden Step 3C aggregation against missing or non-object API worker output while preparing evidence [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:241]
- [x] [Review][Patch] Preserve actual worker source when normalizing worker quality concerns [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:327]
- [x] [Review][Patch] Record `success: false` worker outputs as evidence quality concerns even when status is absent [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:365]
- [x] [Review][Patch] Reject unresolved `{outputFile}` placeholders as invalid report pointers in final TA evidence [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:113]
- [x] [Review][Patch] Require coverage and priority counts to be non-negative integers before emitting PASS evidence [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:160]
- [x] [Review][Patch] Reject duplicate or blank generated test file pointers instead of allowing set comparison or filtering to hide them [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:223]
- [x] [Review][Patch] Reject generated test file pointers that escape the expected test path boundary [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:227]
- [x] [Review][Patch] Add explicit validate-mode instructions to load and schema-check `tea-ta.evidence.json` [src/workflows/testarch/bmad-testarch-automate/steps-v/step-01-validate.md:45]
- [x] [Review][Patch] Cover empty raw summary input and CRLF JavaScript fences in the evidence behavior test helper [test/test-installation-components.js:39]
- [x] [Review][Decision] Decide whether to keep unrelated generated sprint-status changes — dismissed by user; keep `_bmad-output/implementation-artifacts/sprint-status.yaml` as-is.
`_bmad-output/implementation-artifacts/sprint-status.yaml` changes future T2-T5 epics and stories from `backlog` to `in-progress` or `ready-for-dev`.
- [x] [Review][Patch] Harden Step 3C read, success-check, and write-file paths before summary aggregation so malformed worker output becomes evidence concerns instead of crashing earlier [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:54]
- [x] [Review][Patch] Report malformed or missing worker `fixture_needs` as worker-output-shape concerns instead of silently dropping required fixture infrastructure [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:128]
- [x] [Review][Patch] Tighten numeric normalization in Step 3C and Step 4 so blank strings, booleans, arrays, and objects cannot coerce into valid coverage counts [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:104]
- [x] [Review][Patch] Fill partial top-level priority coverage from per-test coverage instead of treating any partial aggregate as authoritative for every priority bucket [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:287]
- [x] [Review][Patch] Preserve actual NFR signal source ownership instead of trusting worker-provided `source` fields [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:410]
- [x] [Review][Patch] Make `fixture_needs` consistently required across Step 4 validation and the checklist now that it is emitted in the final evidence envelope [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:103]
- [x] [Review][Patch] Reject non-string evidence pointers before normalization so arrays or coerced values cannot become accepted paths [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:121]
- [x] [Review][Patch] Relax report-pointer comparison to allow equivalent valid `automation-summary.md` pointers instead of requiring exact string equality [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:256]
- [x] [Review][Patch] Validate coverage internal consistency so `coverage.total_tests` must match the sum of `coverage.by_level` counts [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:192]
- [x] [Review][Patch] Validate `changed_tests` item schema, including `test_level`, `description`, `source`, and per-test `priority_coverage` counts [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:176]
- [x] [Review][Patch] Allow configured generated-test boundaries such as Pact contract artifact paths instead of accepting only `tests/` paths [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:130]
- [x] [Review][Patch] Treat pre-existing error-severity quality concerns as `ERROR` evidence instead of allowing `gate: "PASS"` with worker-output errors [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:309]
- [x] [Review][Patch] Expand validate mode to reject `gate: "ERROR"`, require `contract_version: "1.0"`, and validate `generated_at` as an ISO timestamp [src/workflows/testarch/bmad-testarch-automate/steps-v/step-01-validate.md:60]
- [x] [Review][Patch] Move the Step 3C summary seed handoff into `{test_artifacts}` and ensure Step 4 can create the evidence output directory before writing [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:503]
- [x] [Review][Patch] Strengthen evidence behavior tests to assert read/write paths and use VM execution timeouts for embedded JavaScript snippets [test/test-installation-components.js:57]
- [x] [Review][Decision] Decide whether to keep unrelated module-builder template change — resolved by removing `.agents/skills/bmad-module-builder/assets/setup-skill-template/assets/module.yaml` from this story diff.
- [x] [Review][Patch] Validate and safely write generated test paths in Step 3C before `fs.writeFileSync`, including boundary checks, parent-directory creation, and write-error concerns [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:126]
- [x] [Review][Patch] Build Step 3C changed-test evidence only from successful workers and entries that were eligible or written, so failed or skipped worker output cannot appear as generated tests [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:393]
- [x] [Review][Patch] Treat malformed worker coverage, count, and priority fields as error quality concerns instead of silently normalizing them into PASS evidence [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:297]
- [x] [Review][Patch] Harden NFR signal aggregation and validation so null test entries cannot crash aggregation and NFR signals without category, source, or evidence cannot emit PASS [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:504]
- [x] [Review][Patch] Make Step 4 quality-concern severity checks case-insensitive so `ERROR` and `error` both force `gate: "ERROR"` [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:363]
- [x] [Review][Patch] Normalize or reject per-test `changed_tests[].priority_coverage` numeric strings before final evidence so PASS output uses integer counts, not strings [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:224]
- [x] [Review][Patch] Reject unresolved `{ta_evidence_output}` before writing final evidence so consumers do not miss files written to literal placeholder paths [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:374]
- [x] [Review][Patch] Expand validate mode to schema-check NFR signal entries and generated-test path boundaries, not just top-level arrays and pointer presence [src/workflows/testarch/bmad-testarch-automate/steps-v/step-01-validate.md:60]
- [x] [Review][Patch] Canonicalize generated-test artifact paths consistently across Step 3C writes, changed-test filtering, and Step 4 file-set comparison so `./`, backslashes, whitespace, absolute project paths, and configured test directory paths cannot omit written tests or leak machine-specific paths [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:395]
- [x] [Review][Patch] Treat missing `success: true` and explicit failed or error worker statuses as error quality concerns so incomplete worker output cannot emit PASS evidence [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:599]
- [x] [Review][Patch] Reject duplicate generated test file paths before writing so later worker entries cannot overwrite earlier generated tests silently [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:185]
- [x] [Review][Patch] Report malformed worker `quality_concerns`, `validation_issues`, and NFR signal entries as quality concerns instead of silently dropping malformed worker evidence [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:559]
- [x] [Review][Patch] Exclude or explicitly mark fixture needs, NFR signals, and other non-test evidence from failed or untrusted workers so failed worker output cannot pollute final structured evidence [src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md:660]
- [x] [Review][Patch] Reject unresolved placeholders in evidence output and report pointers generically, not only the literal `{ta_evidence_output}` token, so PASS evidence cannot contain unresolved template paths [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:326]
- [x] [Review][Patch] Validate final `quality_concerns` item schema and trim severity before gate checks so malformed concerns or whitespace-padded `error` severities cannot emit PASS evidence [src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md:398]
- [x] [Review][Patch] Make validate-mode result language use structured evidence `ERROR` consistently instead of mixing `ERROR` validation results with `PASS/WARN/FAIL` report sections [src/workflows/testarch/bmad-testarch-automate/steps-v/step-01-validate.md:88]

## Dev Notes

### Source Planning Context

Epic T1 says BMAD-TEA must expose structured automation evidence for Archon's gate planner.
The specific business value is that Archon can plan release gates without parsing TEA Markdown reports.
[Source: _bmad-output/planning-artifacts/epics.md#Epic T1: Test Automation Evidence For Gate Planning]

PRD requirement T-FR-1 requires structured evidence pointers for changed tests, coverage-related evidence, quality concerns, and NFR signals.
It also requires missing or invalid evidence to fail closed as `ERROR`.
[Source: _bmad-output/planning-artifacts/prd.md#T-FR-1: Emit Structured Test Automation Evidence]

The architecture handoff says TEA owns evidence and gate semantics, Archon owns orchestration and routing, and Markdown reports are not route APIs.
[Source: _bmad-output/planning-artifacts/architecture.md#Architecture Paradigm]

The architecture handoff also says every TEA route-facing contract must include a versioned envelope with `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate`, gate-specific fields, report pointers, and machine-readable artifact pointers when applicable.
[Source: _bmad-output/planning-artifacts/architecture.md#Contract Envelope]

There is no UX artifact for this work.
The implementation readiness report records the UX absence as not applicable because no product UI surface is implied.
[Source: _bmad-output/planning-artifacts/implementation-readiness-report-2026-06-30.md#UX Alignment Assessment]

### Required Contract

The future implementation should emit this file:

```text
{test_artifacts}/tea-ta.evidence.json
```

The file must be valid JSON.
The file must be generated by the TA automate workflow before Step 4 completion.
The file must not require a consumer to parse `automation-summary.md`.

Minimum required shape:

```json
{
  "contract_version": "1.0",
  "workflow": "bmad-testarch-automate",
  "story_ref": "string-or-null",
  "node": "TA",
  "round": 1,
  "gate": "PASS",
  "generated_at": "ISO-8601 timestamp",
  "report_file": "relative-or-absolute-path-to-automation-summary.md",
  "changed_tests": [
    {
      "file": "tests/api/example.spec.ts",
      "test_level": "api",
      "description": "Generated API tests for example behavior",
      "priority_coverage": {
        "P0": 1,
        "P1": 0,
        "P2": 0,
        "P3": 0
      },
      "source": "api-tests"
    }
  ],
  "quality_concerns": [],
  "nfr_signals": [],
  "coverage": {
    "total_tests": 1,
    "by_level": {
      "api": 1,
      "e2e": 0,
      "backend": 0
    },
    "priority_coverage": {
      "P0": 1,
      "P1": 0,
      "P2": 0,
      "P3": 0
    }
  },
  "artifact_pointers": {
    "automation_summary": "automation-summary.md",
    "generated_test_files": ["tests/api/example.spec.ts"]
  }
}
```

`changed_tests` must include every test file created or updated by Step 3C.
For API and E2E subagent outputs, source records come from each item in `tests`.
For backend subagent outputs, source records come from each item in `testsGenerated`.

`quality_concerns` must always be present.
It may be an empty array when no concerns are detected.
It should include machine-readable entries for validation issues, unhealed failures, provider scrutiny pending states, worker partial outputs, or other concerns that RV may need.

`nfr_signals` must always be present.
It may be an empty array when no NFR signals are detected.
When present, each entry should identify a category, a source, and the evidence pointer.
Use the architecture categories when possible: security, reliability, performance, observability, data integrity, and integration boundaries.

`report_file` and `artifact_pointers.automation_summary` must point to the human-readable report.
This preserves the existing human output while making route decisions independent of Markdown parsing.

### Current Code State

`src/workflows/testarch/bmad-testarch-automate/workflow.yaml` currently defines `default_output_file: "{test_artifacts}/automation-summary.md"`.
It does not define a structured TA evidence output.

`src/workflows/testarch/bmad-testarch-automate/steps-c/step-03-generate-tests.md` dispatches API, E2E, and backend worker steps based on `{detected_stack}`.
It requires stable temp JSON output paths and a stable worker output contract.
Preserve that behavior.
If this file changes, preserve deterministic execution-mode fallback, stack dispatch, temp naming, and the same output contract across execution modes.

`src/workflows/testarch/bmad-testarch-automate/steps-c/step-03a-subagent-api.md` writes `/tmp/tea-automate-api-tests-{{timestamp}}.json`.
The API worker output includes `success`, `tests`, `fixture_needs`, `knowledge_fragments_used`, `provider_scrutiny`, `provider_files_read`, `test_count`, and `summary`.
Prefer normalizing this existing payload in Step 3C.
Only extend the worker schema if Step 3C cannot derive a required evidence field from existing output.

`src/workflows/testarch/bmad-testarch-automate/steps-c/step-03b-subagent-e2e.md` writes `/tmp/tea-automate-e2e-tests-{{timestamp}}.json`.
The E2E worker output includes `success`, `tests`, `fixture_needs`, `knowledge_fragments_used`, `test_count`, and `summary`.
Prefer normalizing this existing payload in Step 3C.
Only extend the worker schema if Step 3C cannot derive a required evidence field from existing output.

`src/workflows/testarch/bmad-testarch-automate/steps-c/step-03b-subagent-backend.md` writes `/tmp/tea-automate-backend-tests-{{timestamp}}.json`.
The backend worker output includes `success`, `testsGenerated`, `coverageSummary`, `status`, `knowledge_fragments_used`, and `summary`.
The backend shape intentionally differs from API and E2E.
Preserve its documented schema compatibility and normalize it in aggregation unless extension is clearly necessary.

`src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md` currently reads worker temp JSON, writes generated test files, creates fixtures, calculates summary statistics, and writes `/tmp/tea-automate-summary-{{timestamp}}.json`.
This is the best place to normalize worker-specific shapes into shared structured evidence data.

`src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md` currently validates outputs and writes `{test_artifacts}/automation-summary.md`.
This is the best place to validate and write final `tea-ta.evidence.json`.

`src/workflows/testarch/bmad-testarch-automate/checklist.md` currently validates framework readiness, coverage mapping, generated tests, summary output, and quality standards.
It does not yet validate the TA structured evidence artifact.

`src/workflows/testarch/bmad-testarch-automate/instructions.md` currently resolves core config and automate workflow variables during initialization.
Update it if adding `ta_evidence_output` to the initialization list helps future agents see the output path before loading Step 1.

`src/workflows/testarch/bmad-testarch-automate/steps-v/step-01-validate.md` currently provides generic checklist validation.
Update it only if validate mode must explicitly load and inspect `tea-ta.evidence.json`.

### Files To Update

- `src/workflows/testarch/bmad-testarch-automate/workflow.yaml`
- `src/workflows/testarch/bmad-testarch-automate/instructions.md`
- `src/workflows/testarch/bmad-testarch-automate/workflow-plan.md`
- `src/workflows/testarch/bmad-testarch-automate/steps-c/step-03-generate-tests.md`
- `src/workflows/testarch/bmad-testarch-automate/steps-c/step-03a-subagent-api.md`
- `src/workflows/testarch/bmad-testarch-automate/steps-c/step-03b-subagent-e2e.md`
- `src/workflows/testarch/bmad-testarch-automate/steps-c/step-03b-subagent-backend.md`
- `src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md`
- `src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md`
- `src/workflows/testarch/bmad-testarch-automate/steps-v/step-01-validate.md`
- `src/workflows/testarch/bmad-testarch-automate/checklist.md`
- `test/test-installation-components.js` or a new focused test file under `test/`

The likely minimum implementation changes are `workflow.yaml`, `workflow-plan.md`, Step 3C, Step 4, checklist, and tests.
Treat the other files in this list as inspect-and-update-if-needed, not as mandatory churn.

Do not update `src/agents/bmad-tea/customize.toml` unless the implementation discovers the TA menu itself must change.
The current TA menu already dispatches code `TA` to `bmad-testarch-automate`.

Do not update `src/module-help.csv` unless the output descriptions are intentionally expanded.
The current row already identifies TA as Test Automation and lists `test suite` as the output.

Public docs to inspect if the implementation makes the output user-facing:

- `docs/how-to/workflows/run-automate.md`
- `docs/reference/commands.md`
- `docs/reference/configuration.md`

If docs are updated, run `npm run docs:validate-links`.
Do not update public docs just to restate the internal story unless the user-facing command output or documented artifact list changes.

Do not manually modify `CHANGELOG.md`.
The repository instructions in this workspace explicitly forbid manual changelog edits.

### Implementation Guardrails

Do not create a second automation summary format.
Keep the Markdown report for humans and add JSON for machines.

Do not make Archon parse TEA Markdown.
Any route-planning field that Archon needs must be represented in `tea-ta.evidence.json`.

Do not put route decisions in TA that belong to Archon.
TA should provide evidence and validity status.
Archon should decide whether to run RV, NR, or TR.

Do not use `FAIL` in TA evidence to mean generated tests have quality findings.
For this story, `FAIL` is not needed because TA evidence is not the RV quality gate.
Use `PASS` for valid evidence and `ERROR` for missing, invalid, mismatched, or untrusted evidence.

Do not add dependencies for basic JSON generation.
The workflow instructions already show Node-style `fs.writeFileSync` examples.
Repository dependencies include `yaml` and `js-yaml`, but JSON writing and parsing can use built-in JavaScript APIs.

Do not depend on local `node_modules` existing before validation.
This checkout currently has no installed root packages, so the future dev should run `npm install` if commands fail due to missing dependencies.

Do not change release metadata files for this story.
Leave `package.json`, `package-lock.json`, and `.claude-plugin/marketplace.json` unchanged unless the implementation genuinely adds a dependency, which should be avoided.

### Testing Requirements

Add deterministic tests that inspect the workflow files as source artifacts.
The tests should not execute the full interactive workflow.

At minimum, assert:

- `workflow.yaml` contains `ta_evidence_output`.
- `workflow.yaml` resolves `ta_evidence_output` to `{test_artifacts}/tea-ta.evidence.json`.
- Step 3C normalizes API, E2E, and backend worker outputs into `changed_tests`.
- Step 3C or Step 4 carries `quality_concerns` and `nfr_signals` as arrays.
- Step 4 writes `{ta_evidence_output}`.
- The checklist validates invalid or missing structured evidence as `ERROR`.
- Existing `automation-summary.md` output remains documented.

Consider adding a dedicated fixture test if the implementation adds a validation helper.
The fixture should prove a gate-planner-like consumer can read changed tests, quality concerns, and NFR signals without reading Markdown.
If a new test script is added, wire it into `package.json` and `npm test`; otherwise prefer extending `test/test-installation-components.js`.

Recommended validation commands:

```bash
npm run test:install
npm run lint
npm run lint:md
npm run format:check
```

If public docs are updated, run:

```bash
npm run docs:validate-links
```

If dependencies are installed and time allows, run the full release gate:

```bash
npm test
```

For this workflow-source change, also run:

```bash
npm run test:release-metadata
```

### Previous Story Intelligence

No prior `t1-*` implementation story exists under `_bmad-output/implementation-artifacts`.
There are no previous T1 dev notes, review findings, or local file patterns to carry forward.

### Git Intelligence Summary

Recent commits are planning and repository configuration changes only.
The relevant commit added the BMAD-TEA handoff planning files and sprint status.
No recent implementation commit establishes a TA evidence pattern.

Recent history:

- `d94b142` merged Archon repository config.
- `e720dc7` set the Archon base branch.
- `82e4c8c` added Archon repository config.
- `ab30c4d` merged the workflow engine BMAD-TEA v2 handoff.
- `c004a72` added the workflow engine v2 handoff plan.

### Latest Technical Notes

As of 2026-06-30, the repository lockfile resolves `yaml` to `2.8.2` and `js-yaml` to `4.1.1`.
Do not upgrade or add dependencies for this story unless implementation proves a real need.

The repository declares `node >=20.0.0`.
Node.js lists Node 20 as end-of-life in its previous releases table, but changing the engine range is outside this story.
Use the repo-supported engine for validation and record any local runtime mismatch in completion notes.
[Source: https://nodejs.org/en/about/previous-releases]

JSON Schema draft 2020-12 is the current JSON Schema specification family, but this story does not require adopting a schema validator.
If the future dev adds a schema file, keep it small and validate it with existing repo test infrastructure.
[Source: https://json-schema.org/specification]

## Project Structure Notes

Core TEA module content lives in `src/`.
Planning and implementation artifacts for this sprint live in `_bmad-output/`.
Public docs live in `docs/`, but this story can be completed without public docs if workflow instructions and tests fully define the new machine-readable evidence.

The automate workflow uses step-file architecture.
Keep step entrypoints anchored with `{skill-root}` when references are added or changed.
Existing installation tests assert this convention.

`test_artifacts` resolves through `_bmad/tea/config.yaml` to `{project-root}/_bmad-output/test-artifacts`.
The final evidence file should be written there through the workflow variable, not hardcoded to `_bmad-output/test-artifacts` inside step files.

## References

- [Source: _bmad-output/planning-artifacts/epics.md#Story T1.1: Emit Structured TA Evidence]
- [Source: _bmad-output/planning-artifacts/prd.md#T-FR-1: Emit Structured Test Automation Evidence]
- [Source: _bmad-output/planning-artifacts/architecture.md#T-AD-1: TA Emits Structured Planning Evidence]
- [Source: _bmad-output/planning-artifacts/architecture.md#Contract Envelope]
- [Source: _bmad-output/planning-artifacts/implementation-readiness-report-2026-06-30.md#UX Alignment Assessment]
- [Source: src/workflows/testarch/bmad-testarch-automate/workflow.yaml]
- [Source: src/workflows/testarch/bmad-testarch-automate/steps-c/step-03-generate-tests.md]
- [Source: src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md]
- [Source: src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md]
- [Source: src/workflows/testarch/bmad-testarch-automate/checklist.md]
- [Source: package.json]
- [Source: package-lock.json]

## Dev Agent Record

### Agent Model Used

{{agent_model_name_version}}

### Debug Log References

### Completion Notes List

- Ultimate context engine analysis completed.
- Comprehensive developer guide created for structured TA evidence implementation.
- Added stable `ta_evidence_output` workflow variable for `{test_artifacts}/tea-ta.evidence.json`.
- Updated Step 3C to normalize worker outputs into changed tests, quality concerns, NFR signals, coverage, artifact pointers, and source worker metadata.
- Updated Step 4 to validate and write the versioned TA evidence envelope before workflow completion.
- Preserved `automation-summary.md` as the default human-readable report.
- Added checklist and deterministic installation-component coverage for the structured TA evidence contract.
- Fixed an unrelated YAML template lint failure by making an empty placeholder explicit.
- Validation passed for story-scoped format, markdownlint, ESLint, schema tests, installation tests, knowledge tests, release metadata, and workflow description checks.
- `npm test` ran but failed at the final `format:check` step because of 670 pre-existing Prettier issues under hidden `.agents` and ignored `.omx` paths.

### File List

- `.agents/skills/bmad-module-builder/assets/setup-skill-template/assets/module.yaml`
- `_bmad-output/implementation-artifacts/sprint-status.yaml`
- `_bmad-output/implementation-artifacts/t1-1-emit-structured-ta-evidence.md`
- `src/workflows/testarch/bmad-testarch-automate/checklist.md`
- `src/workflows/testarch/bmad-testarch-automate/instructions.md`
- `src/workflows/testarch/bmad-testarch-automate/steps-c/step-03c-aggregate.md`
- `src/workflows/testarch/bmad-testarch-automate/steps-c/step-04-validate-and-summarize.md`
- `src/workflows/testarch/bmad-testarch-automate/workflow-plan.md`
- `src/workflows/testarch/bmad-testarch-automate/workflow.yaml`
- `test/test-installation-components.js`

### Change Log

- 2026-06-30: Implemented structured TA evidence workflow contract and deterministic source tests.
