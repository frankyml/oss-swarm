# PersonalAgent Swarm v2 — Architecture RFC

## Status & Metadata
- **Status**: Proposed
- **Version**: 2.0
- **Date**: 2025-05-21
- **Author**: Sisyphus (Orchestrator) + Oracle (Architect)
- **Scope**: Core Swarm Architecture, Agent Topology, and Runtime Abstraction

## Supersedes
- PersonalAgent Swarm v1 (OpenCode-coupled architecture)
- All `.aicontext/` spec files (swarm-system-spec.md, orchestration-protocol.md, swarm-pipelines.md, resource-governance.md, memory-architecture.md)
- `memory/core/agent-registry.md` (current agent roster)

---

## 1. Principles

- **Strict Decoupling**: Separation of the cognitive orchestration (Portable Core) from the execution backend (Runtime Adapter).
- **Agent Consolidation**: Distinguish true identity (memory/permission boundary) from functional lens (Mode).
- **Deterministic Routing**: Modes and lenses are applied algorithmically based on an explicit `TaskProfile`.
- **Immutable Audit**: All states, events, and mutations are captured in versioned, replayable bundles.

---

## 2. Two-Layer Architecture

PersonalAgent is split into a **Portable Core** and a **Runtime Adapter**.

### 2.1 Portable Core

The Portable Core is the cognitive source of truth. It owns task profiling, routing rules, pipeline templates, workspace protocol, agent definitions, mode definitions, memory schemas, invariants, and audit/replay rules. It contains NO backend-specific tool definitions (e.g., `task()`, `load_skills`, `subagent_type`, `~/.claude/skills/`).

**Portable Core File Tree:**
```
swarm/
  core/
    schemas/
      task-profile.schema.json
      execution-plan.schema.json
      invocation.schema.json
      memory-entry.schema.json
      mode-overlay.schema.json
    policies/
      routing-policy.yaml
      risk-policy.yaml
      review-policy.yaml
      mutation-policy.yaml
      invariants.yaml
    pipelines/
      research-report.yaml
      spec-to-issues.yaml
      implement.yaml
      audit.yaml
      fix.yaml
      review-pr.yaml
    agents/
      planner.md
      researcher.md
      coder.md
      spec-writer.md
      reviewer.md
      memory-curator.md
    modes/
      reviewer/
        security.yaml
        architecture.yaml
        qa.yaml
        contract.yaml
        product.yaml
        failure.yaml
        friction.yaml
      researcher/
        survey.yaml
        compare.yaml
        verify.yaml
      coder/
        implement.yaml
        patch.yaml
        refactor.yaml
        migrate.yaml
      spec-writer/
        story-breakdown.yaml
        adr.yaml
        runbook.yaml
      memory-curator/
        append.yaml
        compact.yaml
        retrospective.yaml
        proposal-validation.yaml
    workspace/
      protocol.md
      artifact-map.yaml
    behaviors/
      shared/
      domains/
```

### 2.2 Runtime Adapter

The Runtime Adapter connects the Portable Core to a backend (e.g., OpenCode, Claude Code). It knows how to assemble prompts, launch agent sessions, execute parallelization, expose capabilities, request approvals, and persist artifacts.

**Runtime Adapter File Tree (OpenCode implementation):**
```
adapters/
  opencode/
    adapter-spec.md
    capabilities.yaml
    toolmap.yaml
    prompt-assembler.md
    runtime-config.yaml
```

### 2.3 Abstract Adapter Interface

The Portable Core defines its requirements via an abstract interface. The Runtime Adapter implements this interface.

```typescript
interface RuntimeCapabilities {
  supportsParallelExecution: boolean;
  maxContextTokens: number;
  availableTools: string[];
  approvalMechanisms: string[];
}

interface ToolPolicy {
  allowedTools: string[];
  forbiddenTools: string[];
  requireApprovalFor: string[];
}

interface Ref {
  type: "artifact" | "memory" | "behavior" | "schema";
  uri: string;
}

interface ArtifactRef extends Ref { type: "artifact" }
interface MemoryRef extends Ref { type: "memory" }

interface InvocationRequest {
  coreAgent: "planner" | "researcher" | "coder" | "spec-writer" | "reviewer" | "memory-curator";
  modes: string[];
  goal: string;
  inputs: Ref[];
  memoryRefs: Ref[];
  behaviorRefs: Ref[];
  toolPolicy: ToolPolicy;
  outputSchema: Ref;
}

interface InvocationResult {
  status: "success" | "failure" | "yield";
  artifactsProduced: Ref[];
  events: RunEvent[];
  rawOutput: string;
}

interface ApprovalRequest {
  action: string;
  impact: string;
  reversible: boolean;
  contextRef: Ref;
}

interface ApprovalDecision {
  granted: boolean;
  modifications?: string;
  reason?: string;
}

interface RunEvent {
  timestamp: string;
  type: "tool_call" | "artifact_write" | "checkpoint" | "error";
  payload: any;
}

interface RunContext {
  runId: string;
  pipeline: string;
  taskProfile: any;
}

interface RunHandle {
  runId: string;
  status: string;
  abort(): Promise<void>;
}

interface RuntimeAdapter {
  getCapabilities(): RuntimeCapabilities;
  startRun(run: RunContext): Promise<RunHandle>;
  invoke(request: InvocationRequest): Promise<InvocationResult>;
  invokeParallel(requests: InvocationRequest[]): Promise<InvocationResult[]>;
  readRef(ref: ArtifactRef | MemoryRef): Promise<string>;
  writeArtifact(ref: ArtifactRef, content: string, owner: string): Promise<void>;
  requestApproval(req: ApprovalRequest): Promise<ApprovalDecision>;
  recordEvent(event: RunEvent): Promise<void>;
}
```

---

## 3. Core Agents

A Core Agent is defined strictly by three criteria: (a) distinct memory ownership, (b) distinct write/permission boundary, and (c) distinct artifact or reasoning loop. If a persona does not meet all three, it is a Mode.

### 3.1 Planner
- **(a) Memory Ownership**: Owns routing heuristics, pipeline outcomes, historical escalation patterns, and personal-context.
- **(b) Write/Permission Boundary**: Allowed to trigger pipeline orchestration, request human execution approvals, and construct `TaskProfile` documents. Cannot mutate code or execute tools directly outside orchestration.
- **(c) Artifact/Reasoning Loop**: Reasons about process routing. Writes execution plans, run manifests, approval requests, and orchestration logs.

### 3.2 Researcher
- **(a) Memory Ownership**: Owns source rankings, search heuristics, verification notes, and domain source preferences.
- **(b) Write/Permission Boundary**: Allowed read-only web access, internal document parsing, and database reads. FORBIDDEN from writing to the workspace or issuing deployment commands.
- **(c) Artifact/Reasoning Loop**: Iterative evidence-gathering loop. Cross-references external claims. Writes research artifacts and data summaries only.

### 3.3 Coder
- **(a) Memory Ownership**: Owns repository conventions, build/test recipes, framework repair patterns, and tool recovery notes.
- **(b) Write/Permission Boundary**: The ONLY execution agent permitted to mutate source code, configurations, and workspace states. Has access to compiler, terminal, and git tools.
- **(c) Artifact/Reasoning Loop**: Implementation and repair loop (Code → Test → Diagnose). Writes source code, scripts, configuration files, and implementation artifacts.

### 3.4 Spec Writer
- **(a) Memory Ownership**: Owns decomposition patterns, structured authoring templates, acceptance-criteria patterns, and doc style conventions.
- **(b) Write/Permission Boundary**: Restricted to documentation, backlog systems (e.g., Jira/GitHub issues via API), and `.sdd` directories. Cannot mutate application code.
- **(c) Artifact/Reasoning Loop**: Structured-authoring loop (Decompose → Map Contracts → Fill Templates). Writes specs, ADRs, user stories, epics, and runbooks.

### 3.5 Reviewer
- **(a) Memory Ownership**: Owns recurring defect patterns, review heuristics, and mode-specific historical findings.
- **(b) Write/Permission Boundary**: Strictly read-only access to workspace code, pull requests, and architecture specs. Cannot write code or fix bugs directly.
- **(c) Artifact/Reasoning Loop**: Validation loop. Reads implementation vs. specifications. Writes review artifacts, verdicts (pass/fail), and findings reports.

### 3.6 Memory Curator
- **(a) Memory Ownership**: Owns compaction rules, learning promotion thresholds, replay baselines, and memory hygiene rules.
- **(b) Write/Permission Boundary**: The ONLY agent allowed to mutate the `memory/` namespace and live learnings. Access restricted to memory stores.
- **(c) Artifact/Reasoning Loop**: Compaction and consolidation loop. Writes memory files, memory change proposals, and compaction summaries.

---

## 4. Modes System

### 4.1 Definition (4 Components)

A mode is an overlay applied to a host core agent. It does not get its own routing identity or memory file. A mode consists of EXACTLY four components:

1. **Lens Preamble**: The context injection that biases the agent's focus (e.g., "review only for security concerns").
2. **Checklist**: Fixed, explicit questions the agent must answer.
3. **Output Schema**: Required sections and severity rubrics for the agent's deliverables.
4. **Escalation Rules**: Concrete triggers dictating when to halt execution or expand the review scope.

### 4.2 Mode Resolver Flow

The Mode Resolver is a deterministic function executed by the Planner. The Planner evaluates the environment to produce a `TaskProfile`, which is then passed through the Mode Resolver decision rules to select `assuranceLenses`.

**Decision Rules (TaskProfile → assuranceLenses):**

| Condition | Action | Applied Mode |
|---|---|---|
| `reviewDepth == 2` AND `artifactType == "code"` | `push` | `reviewer.architecture` |
| `riskScore >= 2` AND (`authOrSecrets == true` OR `publicSurface == true`) | `push` | `reviewer.security` |
| Pipeline stage == `delivery_gate` AND acceptance criteria present | `push` | `reviewer.qa` |
| `externalIntegration == true` OR `publicSurface == true` | `push` | `reviewer.contract` |
| `userFacing == true` AND `artifactType == "spec"` | `push` | `reviewer.product` |
| `artifactType == "workflow"` OR risk score operational | `push` | `reviewer.failure` |
| `userFacing == true` OR involves CLI/Agent UX | `push` | `reviewer.friction` |

### 4.3 Complete Example: `reviewer.security`

```yaml
# swarm/core/modes/reviewer/security.yaml
id: "reviewer.security"
host_agent: "reviewer"

components:
  lens_preamble: >
    You are operating in SECURITY mode. Your sole focus is identifying vulnerabilities,
    authentication bypasses, secret leaks, and boundary violations. Ignore code style,
    performance (unless a DoS vector), and functional completeness. Assume all external
    inputs are malicious.

  checklist:
    - "Are any secrets, tokens, or credentials hardcoded or logged?"
    - "Is all user input validated, sanitized, and escaped before processing?"
    - "Are authorization boundaries enforced on all new or modified public interfaces?"
    - "Does the change introduce injection vectors (SQL, Command, XSS, SSRF)?"
    - "Are insecure cryptographic primitives used?"

  output_schema:
    type: "object"
    required: ["verdict", "critical_vulnerabilities", "warnings", "remediation_requirements"]
    properties:
      verdict:
        type: "string"
        enum: ["pass", "fail"]
      critical_vulnerabilities:
        type: "array"
        items:
          type: "object"
          properties:
            line_range: { type: "string" }
            cwe: { type: "string" }
            description: { type: "string" }
      warnings:
        type: "array"
        items:
          type: "string"
      remediation_requirements:
        type: "array"
        items:
          type: "string"

  escalation_rules:
    - condition: "critical_vulnerabilities.length > 0"
      action: "halt_pipeline"
      reason: "Immediate security remediation required before further validation."
    - condition: "authOrSecrets == true AND architecture is unverified"
      action: "widen_review"
      target_mode: "reviewer.architecture"
      reason: "Security boundary change requires architecture validation."
```

### 4.4 Full Mode Catalog

**Reviewer Modes**

| Mode | Trigger | Focus |
|---|---|---|
| `architecture` | `reviewDepth=2`, cross-module change, new pattern | Structural integrity, pattern adherence |
| `security` | `riskScore>=2`, auth/secrets/input validation/public endpoint | Vulnerabilities, boundary violations |
| `qa` | delivery gate, acceptance criteria exist, integration risk | Functional correctness vs. AC |
| `contract` | public API/CLI/schema/tool/interface changed | Backward compatibility, API design |
| `product` | reviewing specs/plans with user-facing scope | Value delivery, user intent alignment |
| `failure` | automation/workflow/runtime change with operational risk | Degradation modes, retry limits |
| `friction` | user flow, CLI flow, agent UX, approval UX change | Usability, cognitive load, UX |

**Researcher Modes**

| Mode | Trigger | Focus |
|---|---|---|
| `survey` | unknown domain, exploration needed | Breadth of landscape, landscape mapping |
| `compare` | user asks for options/trade-offs | Pros/cons, matrix evaluation |
| `verify` | external claim must be confirmed before decision | Fact-checking, source validation |

**Coder Modes**

| Mode | Trigger | Focus |
|---|---|---|
| `implement` | new capability | Scaffold, implement, wire |
| `patch` | bounded bug fix | Isolate, reproduce, repair |
| `refactor` | behavior-preserving structural change | AST manipulation, state preservation |
| `migrate` | framework/API/config migration | Version bumps, deprecation handling |

**Spec Writer Modes**

| Mode | Trigger | Focus |
|---|---|---|
| `story-breakdown` | epics/stories/tasks | Granularity, AC definition |
| `adr` | architecture decision | Context, alternatives, consequences |
| `runbook` | operational/process documentation | Step-by-step clarity, recovery steps |

**Memory Curator Modes**

| Mode | Trigger | Focus |
|---|---|---|
| `append` | safe memory evolution | Add facts without losing context |
| `compact` | token/size threshold hit | Summarize, prune dead paths |
| `retrospective` | repeated friction or scheduled review | Root cause analysis, systemic fixes |
| `proposal-validation` | policy change proposed | Schema validation, invariant checks |

---

## 5. TaskProfile Schema

The `TaskProfile` is generated by the Planner and strictly defines the parameters of the operation, dictating the downstream `assuranceLenses`.

```json
  {
    "content": "Read RFC §3 for agent source material",
    "status": "in_progress",
    "priority": "high"
  }
  }
}
```

---

## 6. Pipeline System

### 6.1 Pipeline YAML Format

Pipelines dictate execution graphs. A pipeline file defines:
- `stages`: Sequential or parallel node definitions.
- `agent`: The core agent targeted.
- `modes`: The modes applied to the agent.
- `parallel`: Boolean indicating if this block executes concurrently with siblings.
- `mandatory`: Boolean indicating if the stage can be skipped by the Planner.
- `contract`: Input/output guarantees (`must_provide`, `must_not_provide`).

### 6.2 Complete Example: `implement.yaml`

```yaml
# swarm/core/pipelines/implement.yaml
id: "pipeline.implement"
description: "Standard code implementation pipeline with deep validation"

stages:
  - id: "research_dependencies"
    agent: "researcher"
    modes: ["verify"]
    parallel: false
    mandatory: false
    contract:
      must_provide: ["dependency_validation_report.md"]
      must_not_provide: ["code_mutations"]

  - id: "write_code"
    agent: "coder"
    modes: ["implement"]
    parallel: false
    mandatory: true
    contract:
      must_provide: ["source_code", "unit_tests", "build_success_log"]
      must_not_provide: ["untested_stubs"]

  - id: "validation_layer"
    parallel: true
    mandatory: true
    sub_stages:
      - id: "security_audit"
        agent: "reviewer"
        modes: ["security"]
        contract:
          must_provide: ["security_verdict.json"]
          must_not_provide: ["code_fixes"]

      - id: "qa_verification"
        agent: "reviewer"
        modes: ["qa"]
        contract:
          must_provide: ["qa_verdict.json", "coverage_report"]
          must_not_provide: ["code_fixes"]

      - id: "architecture_review"
        agent: "reviewer"
        modes: ["architecture"]
        contract:
          must_provide: ["architecture_verdict.json"]
          must_not_provide: []

  - id: "memory_consolidation"
    agent: "memory-curator"
    modes: ["append"]
    parallel: false
    mandatory: true
    contract:
      must_provide: ["memory_patch"]
      must_not_provide: ["workspace_artifacts"]
```

---

## 7. External Packages

External domains are injected without modifying the Portable Core files via the external packages system.

### 7.1 Trust Tiers

| Tier | What | Trust Level | Activation |
|---|---|---|---|
| **Behaviors** | Prompt fragments, checklists, domain knowledge | Low risk — read-only context injection | Auto-loaded by domain match |
| **Modes** | New review lenses, coder specializations | Medium risk — changes evaluation/output | Planner routes via manifest |
| **Agents** | Entirely new core agents | High risk — new memory/write boundaries | Requires explicit registration + policy update |

### 7.2 Package Manifest

Packages must declare their contents strictly.

```yaml
# swarm/packages/aicontext/package.yaml
name: "aicontext-behaviors"
version: "1.2.0"
description: "AI Context evaluation modes and standards"
provides:
  agents: []
  modes:
    - host: "reviewer"
      id: "aicontext-compliance"
      overlay: "modes/reviewer/aicontext-compliance.yaml"
  behaviors:
    - id: "aicontext-conventions"
      path: "behaviors/aicontext-conventions.md"
  pipelines: []
```

### 7.3 Installation & Isolation

**Installation Methods:**

| Method | When |
|---|---|
| **Git submodule** | Versioned, shared across machines |
| **Symlink** | Local development, fast iteration |
| **Vendored copy** | Hermetic, CI-friendly |
| **Registry fetch** | Future: `swarm install pkg@version` |

**Isolation Rules (from `invariants.yaml`):**
```yaml
external_packages:
  override_core_agents: never
  policy_mutation: proposal_only
  undeclared_capabilities: rejected
  memory_namespace: "memory/packages/{package-name}/"
  floating_versions: forbidden
```

Memory namespace strict isolation ensures a package cannot corrupt global swarm routing memories.

---

## 8. Self-Improvement Protocol

The swarm's ability to update itself is restricted into three strict categories.

### 8.1 Safe: Memory Evolution

Can change automatically through the `memory-curator`.

- **Targets**: `memory/core/active-context.md`, `memory/agents/*.md`, `memory/projects/*`, source rankings, convention lists, failure patterns.
- **Constraints**:
  - Must be schema-validated.
  - Append, compact, or promote ONLY.
  - NO secrets allowed.
  - NO role identity changes.
  - NO permission/capability changes.
  - Agents cannot edit outside their owned namespace.

### 8.2 Guarded: Policy Mutation

Must NEVER change live during normal execution.

- **Targets**: Routing rules, mode activation rules, pipeline templates, base agent prompts, mode definitions, tool permissions, approval gates, invariants, schemas, adapter interfaces.
- **Workflow (5-Step)**:
  1. Agent proposes change artifact (`change-proposal.patch`).
  2. Memory Curator validates format and scope.
  3. Replay harness runs fixed benchmark tasks against the proposed patch.
  4. Diff is reviewed by a Human.
  5. Approved change is versioned and merged.

### 8.3 Invariants

Cannot be self-modified under any circumstance. Declared in `swarm/core/policies/invariants.yaml`.

- Goal integrity (no intent drift).
- Human approval required for all destructive actions.
- Memory ownership boundaries are absolute.
- Max feedback iterations limit.
- Max retry limits.
- No secret persistence in memory.
- Adapter interface contract strictness.
- Complete distinction between portable core and runtime adapter.

---

## 9. Audit Trail & Reversibility

Every swarm execution captures its context and mutations for replay and auditability.

**Run Bundle Structure:**
```
swarm/runs/{run-id}/
  manifest.json
  events.jsonl
  inputs/
  outputs/
  memory-snapshot.json
  change-proposals/
```

- **Manifest Contents**: Policy version, agent definition version, mode version set, adapter version, memory snapshot hashes, runtime capability set.
- **Patch Storage Path**: Every memory or policy change generates a `.patch` file at `swarm/changes/{timestamp}-{change-id}.patch`.
- **Reversibility Mechanism**: Any operational state can be reverted by (a) restoring a prior `memory-snapshot.json`, or (b) applying the inverse of the respective `.patch` file via the adapter.

---

## 10. Migration Plan

Transitioning from v1 (OpenCode-coupled) to v2 requires 7 phases.

| Phase | Description | Deliverable | Effort |
|---|---|---|---|
| **1. Extract Core** | Extract portable core out of OpenCode logic. | `swarm/core/` directory created with executable specs. | 1 week |
| **2. Adapter Boundary** | Build the boundary. Planner calls abstract adapter, not `task()` directly. | `RuntimeAdapter` implemented in `adapters/opencode/`. | 2 weeks |
| **3. Mode Collapse** | Collapse Evaluator/Sentinel/Guardian into Reviewer Modes. | Deletion of old agents, creation of `reviewer/*.yaml`. | 1 week |
| **4. Source of Truth Move** | Move behavior definitions from `~/.claude/skills/` to repo. | `swarm/core/behaviors/` populated. | 3 days |
| **4.5. Packaging** | Define package manifest schema. First external package. | `swarm/packages/aicontext` as first external pack. | 1 week |
| **5. Reproducibility** | Build audit logging and replay harnesses. | Run manifests, snapshots, `.patch` system live. | 2 weeks |
| **6. Lockdown** | Disable live prompt/policy self-editing. | `invariants.yaml` enforced by Adapter. | 2 days |

---

## 11. Deprecation Map

| Old Entity | New Location |
|---|---|
| Security Sentinel | `Reviewer.security` mode |
| QA Guardian | `Reviewer.qa` mode |
| Contract Critic | `Reviewer.contract` mode |
| Product Owner | `Reviewer.product` mode |
| Friction Auditor | `Reviewer.friction` mode |
| Failure Dramatist | `Reviewer.failure` mode |
| Retrospective Agent | `MemoryCurator.retrospective` mode |
| Personal Assistant | `Planner.personal-context` (utility mode) |

---

## Appendix: Requirements Traceability

| # | Requirement | Section |
|---|---|---|
| 1 | Core Agent Rule (3 criteria per agent) | §3 |
| 2 | Mode Definition (4 components + full YAML) | §4.1, §4.3 |
| 3 | TaskProfile JSON Schema (9 fields) | §5 |
| 4 | External Packages (3 tiers, manifest, isolation) | §7 |
| 5 | Self-Improvement (Safe/Guarded/Invariants) | §8 |
| 6 | Audit Trail (bundle, manifest, patches, revert) | §9 |
| 7 | Mode Resolver (decision table) | §4.2 |
| 8 | Pipeline Format (complete YAML) | §6 |
| 9 | Reviewer Modes Catalog (7 modes + triggers) | §4.4 |
| 10 | File Trees (core + adapter) | §2.1, §2.2 |
| 11 | Adapter Interface (full TS pseudocode) | §2.3 |
| 12 | Migration Plan (7 phases + effort) | §10 |
