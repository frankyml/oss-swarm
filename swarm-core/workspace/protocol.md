# Workspace Protocol

## Purpose

This protocol defines the standard shared workspace used by the Portable Core and any runtime adapter for artifact-mediated handoffs.

The standard root is:

```text
.sdd/{task-id}/
```

This root is created once per execution and retained as the replayable evidence bundle for the task.

## Required Directory Layout

Every task workspace uses the same top-level structure.

```text
.sdd/{task-id}/
├── planner/
├── researcher/
├── coder/
├── spec-writer/
├── reviewer/
├── memory-curator/
└── shared/
```

### Directory ownership

| Directory | Primary writer | Purpose |
|---|---|---|
| `planner/` | Planner | Task profile, execution plan, run manifest, approval requests |
| `researcher/` | Researcher | Research notes, evidence summaries, comparison tables |
| `coder/` | Coder | Receipts describing code mutations, test logs, build logs |
| `spec-writer/` | Spec Writer | Stories, ADRs, runbooks, structured docs |
| `reviewer/` | Reviewer | Architecture, security, QA, contract, product, failure, and friction verdicts |
| `memory-curator/` | Memory Curator | Memory patch proposals, compaction summaries, retrospective notes |
| `shared/` | Planner or adapter | Cross-stage index files, artifact manifests, shared reference lists |

Agents may read any file under `.sdd/{task-id}/` but may write only inside their owned directory.

## Standard Artifacts

The following files are standard and should exist whenever applicable.

### Planner

- `planner/task-profile.yaml`
- `planner/execution-plan.yaml`
- `planner/run-manifest.yaml`

### Researcher

- `researcher/{stage-id}-report.md`
- `researcher/{stage-id}-sources.yaml`

### Coder

- `coder/{stage-id}.artifacts.yaml`
- `coder/{stage-id}.build.log`
- `coder/{stage-id}.test.log`

The coder does not copy source files into `.sdd`. Source code remains in the repository. The workspace stores receipts that point to those files.

### Spec Writer

- `spec-writer/{stage-id}.md`
- `spec-writer/{stage-id}.checklist.yaml`

### Reviewer

- `reviewer/{stage-id}.{mode}.json`

Examples:

- `reviewer/security_audit.security.json`
- `reviewer/architecture_review.architecture.json`
- `reviewer/delivery_gate.qa.json`

### Memory Curator

- `memory-curator/{stage-id}.memory-patch.yaml`
- `memory-curator/{stage-id}.summary.md`

### Shared

- `shared/context-refs.yaml`
- `shared/artifact-index.yaml`
- `shared/handoff-log.md`

## Artifact Naming Rules

1. File names are deterministic and derived from stage id plus mode when needed.
2. Use kebab-case for task ids and stage ids.
3. Use the owning directory to express author identity; do not encode author twice in the file name.
4. Use these suffixes only:
   - `.md` for narrative artifacts
   - `.yaml` for manifests and receipts
   - `.json` for structured verdicts
   - `.log` for build, test, or execution evidence

## Handoff Rules

### Rule 1 — Pass paths, not copied content

Downstream stages receive artifact paths, never pasted artifact bodies, unless a runtime limitation makes path-based reading impossible.

Correct:

- `Read .sdd/task-42/reviewer/security_audit.security.json before producing the delivery summary.`

Incorrect:

- Copying the full JSON verdict into the next agent prompt.

### Rule 2 — Handoffs are manifest-driven

Each stage should emit a receipt or manifest when its primary output is not itself a single file.

Examples:

- Coder writes `coder/write_code.artifacts.yaml` listing changed repository files, build evidence, and test evidence.
- Reviewer writes `reviewer/security_audit.security.json` and the planner passes only that path forward.

### Rule 3 — Shared index stays authoritative

`shared/artifact-index.yaml` is the planner-maintained map of stage ids to artifact paths. When in doubt, downstream stages consult the index instead of guessing file names.

## Contract Validation

Contract validation happens after each stage completes and before dependents start.

### Validation inputs

- the completed stage's `must_provide` list
- the completed stage's `must_not_provide` list
- the artifact receipts written by that stage
- the actual files under `.sdd/{task-id}/`

### Validation procedure

1. **Locate owner directory** for the stage's agent.
2. **Collect emitted artifacts** from that directory and any referenced repository paths listed in receipts.
3. **Validate `must_provide`**:
   - every required artifact class must be represented by an actual file or a receipt entry
   - examples:
     - `security_verdict.json` means the exact structured verdict file must exist
     - `source_code` means the coder receipt must list changed repository files
     - `build_success_log` means a build log must exist and record success
4. **Validate `must_not_provide`**:
   - forbidden artifact classes must be absent from both workspace files and receipt declarations
   - examples:
     - `code_fixes` is violated if a reviewer artifact declares changed code or if reviewer-owned files contain patch instructions presented as applied fixes
     - `workspace_artifacts` is violated if a memory-curator stage writes outside `memory-curator/`
5. **Validate ownership boundary**:
   - the stage must not create files in another agent's directory
6. **Record result** in `shared/handoff-log.md`

### Validation outcome

- **pass**: all required artifacts present, no forbidden artifacts present, ownership respected
- **fail**: any missing required artifact, any forbidden artifact, or any cross-owner write

Dependent stages may start only after a pass.

## Repository vs. Workspace Artifacts

The workspace is not a mirror of the repository.

- Repository files remain the source of truth for code and config
- Workspace artifacts are receipts, verdicts, manifests, and handoff documents
- Any pipeline contract that refers to code or tests must be represented in the workspace by evidence files or receipts

## Replay Guarantee

An execution is replayable when the workspace contains, at minimum:

- `planner/task-profile.yaml`
- `planner/execution-plan.yaml`
- at least one stage artifact per executed stage
- enough receipts to reconstruct which repository files were read or changed

Under that condition, a planner can reconstruct the path from request to task invocation without needing copied prompt bodies from prior stages.
