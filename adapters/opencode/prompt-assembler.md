# OpenCode Prompt Assembler

## Purpose

This document defines the deterministic adapter-side process that turns a planner-issued invocation request into the final prompt string passed to OpenCode.

It applies after pipeline selection and stage selection are already resolved.

## Inputs

The adapter assembles each prompt from five sources:

1. **Core agent definition** at `swarm/core/agents/{agent}.md`
2. **Mode overlays** at `swarm/core/modes/{agent}/{mode}.yaml`
3. **Pipeline stage contract** from the selected pipeline YAML
4. **Planner-provided task payload** (`goal`, `TaskProfile`, input refs, memory refs, behavior refs)
5. **Runtime routing data** from `adapters/opencode/runtime-config.yaml`

## Assembly Rules

### 1. Resolve the host agent

- Read `swarm/core/agents/{agent}.md`
- Extract these sections only:
  - `Identity`
  - `Permission Boundary`
  - `Reasoning Loop`
  - `Artifacts`
- Do not inline the agent's `Modes` list into the final prompt

### 2. Resolve the applied modes

- For each requested mode, read `swarm/core/modes/{agent}/{mode}.yaml`
- Preserve resolver order from the planner; do not re-sort in the adapter
- Extract from each mode:
  - `components.lens_preamble`
  - `components.checklist`
  - `components.output_schema`
  - `components.escalation_rules`

### 3. Resolve the stage contract

- Read the selected stage from the chosen pipeline YAML
- Extract:
  - stage id
  - agent
  - modes
  - `contract.must_provide`
  - `contract.must_not_provide`
- If the stage is nested under a parallel block, use the nested stage contract, not the parent block

### 4. Assemble the prompt sections

The final prompt is always rendered in this exact order:

1. `IDENTITY`
2. `LENS`
3. `CHECKLIST`
4. `TASK`
5. `CONTRACT`
6. `CONTEXT`

No extra top-level sections are allowed.

## Section Rendering Contract

### `IDENTITY`

Render the host agent's role and constraints from:

- `swarm/core/agents/{agent}.md#Identity`
- `swarm/core/agents/{agent}.md#Permission Boundary`
- `swarm/core/agents/{agent}.md#Reasoning Loop`
- `swarm/core/agents/{agent}.md#Artifacts`

Purpose: remind the runtime agent who it is, what it may do, and what kind of artifact it is expected to produce.

### `LENS`

Render all mode preambles in planner order.

- Single-mode invocation: emit one preamble
- Multi-mode invocation: concatenate preambles separated by a blank line
- After preambles, append escalation rules for those modes in the same order

Purpose: bias reasoning without changing the host agent identity.

### `CHECKLIST`

Render a single ordered checklist.

- Concatenate mode checklists in planner order
- Remove exact duplicates only
- Keep wording unchanged from the mode files

Purpose: provide fixed review or authoring questions that must be answered during the session.

### `TASK`

Render the planner's atomic objective.

This section must include:

- pipeline id
- stage id
- requested goal
- a one-line reminder of the expected action for the stage

Purpose: anchor the invocation to the exact stage being executed.

### `CONTRACT`

Render both the pipeline contract and the mode output obligations.

This section must contain:

- `must_provide`
- `must_not_provide`
- output schema requirements from all applied modes

Merge rules:

- `must_provide` and `must_not_provide` come from the pipeline stage contract
- output schema comes from the mode overlay
- if multiple modes are present, list each mode schema under its mode id; do not attempt schema fusion in the prompt

Purpose: make pipeline validation explicit before the agent responds.

### `CONTEXT`

Render only references and planner-produced task metadata.

This section must include:

- `TaskProfile` summary
- workspace artifact refs
- external or supporting refs
- memory refs when present

Do not inline file contents into this section. The adapter passes paths and identifiers, not copied artifact bodies.

## Prompt Template

The adapter renders the final prompt with this exact skeleton:

```text
IDENTITY
{agent identity block}

LENS
{mode lens preamble block}

CHECKLIST
{ordered checklist}

TASK
{stage-specific task block}

CONTRACT
{pipeline contract and output schema block}

CONTEXT
{task profile and refs block}
```

## Worked Example — `reviewer.security`

### Example source set

- Agent: `swarm/core/agents/reviewer.md`
- Mode: `swarm/core/modes/reviewer/security.yaml`
- Pipeline stage: `swarm/core/pipelines/implement.yaml` → `validation_layer.security_audit`
- Runtime route: `adapters/opencode/runtime-config.yaml` → `reviewer.route=subagent`, `subagent_type=oracle`

### Example planner payload

- pipeline id: `pipeline.implement`
- stage id: `security_audit`
- goal: `Review the produced code changes for security regressions before delivery.`
- TaskProfile:
  - `mutating: true`
  - `riskScore: 3`
  - `reviewDepth: 2`
  - `artifactType: code`
  - `publicSurface: true`
  - `userFacing: false`
  - `authOrSecrets: true`
  - `externalIntegration: true`
  - `assuranceLenses: [reviewer.architecture, reviewer.security, reviewer.contract]`
- input refs:
  - `.sdd/task-auth-hardening/coder/write_code.artifacts.yaml`
  - `.sdd/task-auth-hardening/planner/task-profile.yaml`
  - `src/auth/session.ts`
  - `src/api/webhook.ts`

### Exact assembled prompt text

```text
IDENTITY
You are the Reviewer.
Identity: The validation specialist responsible for verifying implementation correctness, security, and quality against specifications.
Permission boundary:
- ALLOWED: Read-only access to workspace code, pull requests, and architecture specs.
- FORBIDDEN: Writing code, fixing bugs directly, or mutating the application state.
Reasoning loop: Read Context → Analyze vs. Specs → Identify Gaps → Issue Verdict.
Artifacts: Review reports, pass/fail verdicts, and detailed finding reports.

LENS
You are operating in SECURITY mode. Your sole focus is identifying vulnerabilities,
authentication bypasses, secret leaks, and boundary violations. Ignore code style,
performance (unless a DoS vector), and functional completeness. Assume all external
inputs are malicious.

Escalation rules:
- Halt the pipeline if `critical_vulnerabilities.length > 0`.
- Widen review to `reviewer.architecture` if `authOrSecrets == true AND architecture is unverified`.

CHECKLIST
1. Are any secrets, tokens, or credentials hardcoded or logged?
2. Is all user input validated, sanitized, and escaped before processing?
3. Are authorization boundaries enforced on all new or modified public interfaces?
4. Does the change introduce injection vectors (SQL, Command, XSS, SSRF)?
5. Are insecure cryptographic primitives used?

TASK
Pipeline: pipeline.implement
Stage: security_audit
Goal: Review the produced code changes for security regressions before delivery.
Action: Inspect the implementation output from `write_code` and issue a security verdict without modifying code.

CONTRACT
Stage contract:
- must_provide: security_verdict.json
- must_not_provide: code_fixes

Required output schema for reviewer.security:
- verdict: pass | fail
- critical_vulnerabilities: array of objects with `line_range`, `cwe`, and `description`
- warnings: array of strings
- remediation_requirements: array of strings

Return a result that satisfies the schema and the stage contract. If the contract cannot be satisfied, fail explicitly.

CONTEXT
TaskProfile:
- mutating: true
- riskScore: 3
- reviewDepth: 2
- artifactType: code
- publicSurface: true
- userFacing: false
- authOrSecrets: true
- externalIntegration: true
- assuranceLenses: reviewer.architecture, reviewer.security, reviewer.contract

Read these refs before answering:
- .sdd/task-auth-hardening/coder/write_code.artifacts.yaml
- .sdd/task-auth-hardening/planner/task-profile.yaml
- src/auth/session.ts
- src/api/webhook.ts

Support references:
- swarm/core/agents/reviewer.md
- swarm/core/modes/reviewer/security.yaml
- swarm/core/pipelines/implement.yaml
```

### Resulting OpenCode invocation descriptor

The assembled prompt above is then routed with these adapter-side decisions:

- route type: `subagent`
- target runtime identity: `oracle`
- background execution: depends on the pipeline stage, not on prompt assembly
- prompt body: the exact assembled text above

## Determinism Guarantees

The adapter is deterministic if all of the following hold:

1. The planner passes a fixed agent, mode list, pipeline id, and stage id
2. The adapter preserves mode order from the planner
3. The prompt uses the exact section order defined here
4. The adapter passes refs by path instead of copying file bodies
5. Contract rendering always includes both stage obligations and mode schema obligations

Under those constraints, the path from `TaskProfile` and selected stage to OpenCode prompt text is replayable.
