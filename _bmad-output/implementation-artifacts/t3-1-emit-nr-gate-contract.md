# Story T3.1: Emit NR Gate Contract

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a test architect,
I want `NR` to emit an NFR evidence gate contract,
so that missing NFR evidence can route back to development through Archon's single loop.

## Acceptance Criteria

1. Given `NR` runs, when `tea-nr.gate.json` is emitted, then it includes `contract_version`, `workflow`, `story_ref`, `node`, `round`, `gate`, `nfr_categories`, `findings_count`, `blocking_findings_count`, and `report_file`.
2. Given missing security, reliability, performance, observability, data integrity, or integration evidence exists, when `NR` emits its contract, then route-blocking findings produce `gate: FAIL`.
3. Given NR execution fails, evidence is missing or invalid, worker summaries are untrusted, story identity cannot be trusted, the report cannot be written, or the contract output is invalid, when the contract is emitted or validated, then the contract produces `gate: ERROR`.
4. Given the human NFR report exists, when downstream orchestration needs a route decision, then it uses `tea-nr.gate.json` and the contract points to the report through `report_file`.
5. Given NFR categories are optional or intentionally skipped by Archon, when this story is implemented, then executed NR produces only `PASS`, `FAIL`, or `ERROR`; Story T3.2 owns `SKIPPED` compatibility.

## Tasks / Subtasks

- [ ] Define the NR gate contract output path and workflow metadata.
  - [ ] Add `nr_gate_output: "{test_artifacts}/tea-nr.gate.json"` to `src/workflows/testarch/bmad-testarch-nfr/workflow.yaml`.
  - [ ] Document the machine-readable NR contract fields near the output configuration, following the `trace` workflow style.
  - [ ] Update `src/workflows/testarch/bmad-testarch-nfr/instructions.md` if initialization should resolve `nr_gate_output`.
  - [ ] Update `src/workflows/testarch/bmad-testarch-nfr/workflow-plan.md` so the outputs list includes both `nfr-assessment.md` and `tea-nr.gate.json`.
- [ ] Emit `tea-nr.gate.json` from NFR aggregation.
  - [ ] Extend `src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04e-aggregate-nfr.md` after `executiveSummary` is built.
  - [ ] Map `executiveSummary.domain_assessments` into `nfr_categories`.
  - [ ] Map all NFR findings to `findings_count`.
  - [ ] Derive `blocking_findings_count` from route-blocking NFR findings.
  - [ ] Normalize worker `CONCERN` statuses and report `CONCERNS` terminology into one internal non-blocking warning category before gate mapping.
  - [ ] Treat worker findings with `status: FAIL`, compliance `FAIL`, overall `HIGH` risk, cross-domain `CRITICAL` or `HIGH` impact, and missing required evidence for route-critical categories as blocking.
  - [ ] Treat missing security, reliability, performance, observability, data integrity, or integration evidence as blocking when those categories are required by story, PRD, architecture, test-design, or route-loop input.
  - [ ] Emit `gate: FAIL` only for a trusted NR run with at least one blocking finding.
  - [ ] Emit `gate: PASS` only for a trusted NR run with zero blocking findings.
  - [ ] Emit `gate: ERROR` for execution, evidence, identity, write, validation, or contract-shape failures.
  - [ ] Keep `ERROR` separate from `FAIL`; do not convert infrastructure or trust failures into NFR evidence failures.
  - [ ] Do not emit `gate: SKIPPED` in this story; Story T3.2 owns skipped-contract compatibility.
- [ ] Preserve existing NFR workflow behavior.
  - [ ] Keep the four current domain workers as security, performance, reliability, and scalability.
  - [ ] Keep adaptive execution across `agent-team`, `subagent`, and `sequential` modes.
  - [ ] Keep worker temp outputs at `/tmp/tea-nfr-{domain}-${timestamp}.json` unless the full flow is updated consistently.
  - [ ] Keep `nfr-assessment.md` as the human-readable report.
  - [ ] Do not re-assess NFRs in Step 4E; aggregate worker output only.
  - [ ] Preserve the rule that unknown thresholds are not guessed.
- [ ] Link and validate the contract from the human report path.
  - [ ] Update `src/workflows/testarch/bmad-testarch-nfr/steps-c/step-05-generate-report.md` so it verifies `tea-nr.gate.json` exists and is readable before completion.
  - [ ] Update `src/workflows/testarch/bmad-testarch-nfr/nfr-report-template.md` to point readers to `tea-nr.gate.json` for route decisions.
  - [ ] Update `src/workflows/testarch/bmad-testarch-nfr/checklist.md` so validation covers the full envelope, PASS, FAIL, ERROR, `nfr_categories`, count fields, report pointer, and no Markdown parsing.
  - [ ] Inspect `src/workflows/testarch/bmad-testarch-nfr/steps-v/step-01-validate.md` and update it only if validate mode should load or schema-check `tea-nr.gate.json`.
- [ ] Update public docs and command metadata if the artifact is documented as a user-facing or integrator-facing output.
  - [ ] Update `docs/how-to/workflows/run-nfr-assess.md` to prefer reading `tea-nr.gate.json` over parsing Markdown for route decisions.
  - [ ] Update `docs/reference/commands.md` and `docs/reference/configuration.md` so `tea-nr.gate.json` appears in the NR output list.
  - [ ] Update `docs/explanation/tea-overview.md` and `src/module-help.csv` only if the public command catalog should advertise the contract.
  - [ ] Do not manually edit `CHANGELOG.md`; follow the maintainer release process if a changelog entry is required.
- [ ] Add focused automated coverage.
  - [ ] Extend `test/test-installation-components.js` or add a focused test file for the NR contract workflow metadata.
  - [ ] Assert `workflow.yaml` declares `nr_gate_output` as `{test_artifacts}/tea-nr.gate.json`.
  - [ ] Assert Step 4E writes `{nr_gate_output}` and uses `JSON.stringify(..., null, 2)`.
  - [ ] Assert Step 4E maps `domain_assessments` to `nfr_categories`.
  - [ ] Assert Step 4E maps all worker findings to `findings_count`.
  - [ ] Assert Step 4E maps route-blocking findings to `blocking_findings_count`.
  - [ ] Assert checklist text covers the full envelope and PASS, FAIL, ERROR behavior.
  - [ ] Add fixture-level validation for PASS, FAIL, and ERROR if a reusable TEA gate validator is introduced.
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
  "workflow": "bmad-testarch-nfr",
  "story_ref": "resolved-story-ref-or-null",
  "node": "NR",
  "round": 1,
  "gate": "PASS",
  "generated_at": "2026-06-30T00:00:00.000Z",
  "nfr_categories": [
    {
      "name": "security",
      "status": "PASS",
      "risk_level": "LOW",
      "findings_count": 0,
      "blocking_findings_count": 0,
      "evidence_count": 2
    },
    {
      "name": "performance",
      "status": "PASS",
      "risk_level": "LOW",
      "findings_count": 0,
      "blocking_findings_count": 0,
      "evidence_count": 1
    },
    {
      "name": "reliability",
      "status": "PASS",
      "risk_level": "LOW",
      "findings_count": 0,
      "blocking_findings_count": 0,
      "evidence_count": 1
    },
    {
      "name": "scalability",
      "status": "PASS",
      "risk_level": "LOW",
      "findings_count": 0,
      "blocking_findings_count": 0,
      "evidence_count": 1
    }
  ],
  "findings_count": 0,
  "blocking_findings_count": 0,
  "report_file": "nfr-assessment.md",
  "artifact_pointers": {
    "nfr_assessment_report": "nfr-assessment.md"
  },
  "risk_breakdown": {
    "security": "LOW",
    "performance": "LOW",
    "reliability": "LOW",
    "scalability": "LOW"
  },
  "compliance_summary": {},
  "blocking_findings": []
}
```

`story_ref` must be the resolved story target when NR is running for a story.
If a standalone legacy invocation has no story target, keep the key present and use `null` rather than fabricating an identity.
For Archon-routed invocations, a missing or mismatched story identity is an `ERROR`.

`round` must be the Archon loop round when it is supplied.
Use `1` only for standalone runs that have no loop metadata.

`nfr_categories` must summarize every assessed NFR domain.
At minimum, map the current worker domains: security, performance, reliability, and scalability.
When evidence maps to architecture categories such as observability, data integrity, integration boundaries, permissions, retries, timeouts, queues, background jobs, caching, logging, monitoring, audit, public APIs, provider adapters, CI, or critical state machines, include those categories as additional entries or category aliases.

`findings_count` should count all worker findings that require attention.
A practical default is every finding whose `status` is not `PASS` or `N/A`.

`blocking_findings_count` should count findings that block route progress.
A practical default is every finding with `status: FAIL`, every compliance `FAIL`, every cross-domain `CRITICAL` or `HIGH` risk, and every missing required NFR evidence item that the route loop expected.

`report_file` must point to the human `nfr-assessment.md` report.
Downstream orchestration must not parse Markdown for the route decision.

### Implementation Guidance

The best emission point is `src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04e-aggregate-nfr.md`.
That step already reads the four worker JSON files, calculates overall risk, aggregates compliance, identifies cross-domain risks, builds `executiveSummary`, and writes `/tmp/tea-nfr-summary-${timestamp}.json`.

Use `src/workflows/testarch/bmad-testarch-nfr/steps-c/step-05-generate-report.md` to validate that the JSON exists and to link it from the human report.
Do not derive the route contract from Markdown.

The existing `trace` workflow is the closest local precedent for machine-readable route artifacts.
It declares machine-readable outputs in `workflow.yaml` and writes route-facing JSON from the decision step.

The prior RV story is the closest local handoff precedent for common envelope shape and `ERROR` separation.
If T2.1 or T2.2 has already introduced a shared validator by the time T3.1 is implemented, reuse it rather than adding a parallel NR-only validator.

Do not edit `src/agents/bmad-tea/customize.toml`.
That file is generated and the current NR menu already dispatches to `bmad-testarch-nfr`.

Treat Archon stories A3.1 and A3.2 as external integration assumptions.
This story should be implementable and testable locally with fixtures and without pulling external Archon project files.

Story T1.1 is only `ready-for-dev` at story creation time.
Do not assume `tea-ta.evidence.json` has already been implemented.
If NR needs TA evidence during development, use the local T1.1 contract shape as an optional fixture or compatibility assumption.

### Current NR Workflow State

`src/workflows/testarch/bmad-testarch-nfr/workflow.yaml` currently declares only `{test_artifacts}/nfr-assessment.md` as the default output.
It does not declare `nr_gate_output`.

`src/workflows/testarch/bmad-testarch-nfr/steps-c/step-02-define-thresholds.md` selects ADR quality readiness categories and never guesses thresholds.
Preserve that behavior.

`src/workflows/testarch/bmad-testarch-nfr/steps-c/step-03-gather-evidence.md` gathers evidence and marks missing evidence as `CONCERNS`.
For the route contract, do not let missing route-required NFR evidence silently become a pass.

`src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04-evaluate-and-score.md` dispatches four workers and validates all four temp outputs exist.
Missing worker output must become `ERROR`, not `FAIL`.

`src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04a-subagent-security.md` writes security output with `domain`, `risk_level`, `findings`, `compliance`, `priority_actions`, and `summary`.
Preserve that worker output shape unless an additive field is needed for blocking classification.

`src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04b-subagent-performance.md` writes performance output with the same broad shape.
Preserve that worker output shape unless an additive field is needed for blocking classification.

`src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04c-subagent-reliability.md` writes reliability output with the same broad shape.
Preserve that worker output shape unless an additive field is needed for blocking classification.

`src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04d-subagent-scalability.md` writes scalability output with the same broad shape.
Preserve that worker output shape unless an additive field is needed for blocking classification.

`src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04e-aggregate-nfr.md` currently builds `executiveSummary` with `overall_risk`, `domain_assessments`, `compliance_summary`, `cross_domain_risks`, `priority_actions`, `risk_breakdown`, `subagent_execution`, and `performance_gain`.
Preserve those fields and add contract emission around them.

The current worker examples use singular `CONCERN`, while report and checklist language often uses plural `CONCERNS`.
Normalize both spellings before computing counts so the route contract does not split warning semantics across two labels.

`src/workflows/testarch/bmad-testarch-nfr/steps-c/step-05-generate-report.md` currently writes the human report and validates it with the checklist.
Keep that responsibility and add JSON existence or shape validation rather than moving aggregation into Step 5.

### Gate Semantics

`PASS` means the NR run was trusted, the contract is valid, and no route-blocking NFR findings are open.

`FAIL` means the NR run was trusted and development work is needed to resolve NFR evidence gaps or failed NFR evidence.

`ERROR` means TEA cannot trust execution, evidence, contract shape, story identity, or output.
`ERROR` must not be converted into `FAIL`.

Do not emit `CONCERNS` as the route contract gate value in this v2 story.
The existing NFR report may keep category-level `CONCERNS`, but the route-facing contract must emit `PASS`, `FAIL`, or `ERROR` for executed NR.

Do not emit `SKIPPED` in this story.
Story T3.2 owns `tea-nr-skipped.gate.json` compatibility.

### Testing Notes

Prefer plain Node tests that inspect workflow files and fixture output.
The repository already uses Node-based validation scripts and does not need a new dependency for this story.

Use fixture-driven contract tests if a reusable TEA gate validator is introduced.
Recommended fixture path is `test/fixtures/tea-gate-contracts/`.

If a JSON schema file is added, use an explicitly declared validator that supports the selected schema draft.
Do not rely on transitive `ajv` availability.

The current package engine is `node >=20.0.0`.
Implementation can use modern Node file system APIs that are available in Node 20 and later.

As of 2026-06-30, Node 20 is past its planned maintenance end.
Changing the package engine is outside this story.

### Project Structure Notes

Canonical workflow source lives under `src/workflows/testarch/bmad-testarch-nfr/`.
Generated or installed skill copies under `.agents/` and `.claude/` must not be treated as the implementation source.

Public documentation lives under `docs/`.
Only update docs when the implementation changes the documented user-facing output surface.

Release metadata must stay synchronized, but this story should not require package version changes.
Run `npm run test:release-metadata` after implementation to prove no accidental drift was introduced.

### References

- [Source: `_bmad-output/planning-artifacts/epics.md` Story T3.1]
- [Source: `_bmad-output/planning-artifacts/prd.md` T-FR-4]
- [Source: `_bmad-output/planning-artifacts/architecture.md` T-AD-3, T-AD-6, contract envelope, and `tea-nr.gate.json`]
- [Source: `_bmad-output/planning-artifacts/implementation-readiness-report-2026-06-30.md` story quality findings]
- [Source: `_bmad-output/implementation-artifacts/t1-1-emit-structured-ta-evidence.md` dependency context]
- [Source: `_bmad-output/implementation-artifacts/t2-1-emit-rv-gate-contract.md` common gate contract precedent]
- [Source: `_bmad-output/implementation-artifacts/t2-2-validate-rv-skipped-contract-compatibility.md` skipped-contract precedent]
- [Source: `src/workflows/testarch/bmad-testarch-nfr/workflow.yaml`]
- [Source: `src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04-evaluate-and-score.md`]
- [Source: `src/workflows/testarch/bmad-testarch-nfr/steps-c/step-04e-aggregate-nfr.md`]
- [Source: `src/workflows/testarch/bmad-testarch-nfr/steps-c/step-05-generate-report.md`]
- [Source: `src/workflows/testarch/bmad-testarch-nfr/resources/knowledge/nfr-criteria.md`]
- [Source: `src/workflows/testarch/bmad-testarch-nfr/resources/knowledge/adr-quality-readiness-checklist.md`]
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
