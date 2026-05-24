# Swarm v2 — Execution Flow Diagrams

## 1. Main Orchestration Flow

```
┌────────────────────────────────────────────────────────────────────────────┐
│                        SWARM EXECUTION FLOW                                 │
└────────────────────────────────────────────────────────────────────────────┘

  ┌──────────┐
  │   User   │
  │  Request │
  └────┬─────┘
       │
       ▼
┌──────────────┐
│   PLANNER    │
│              │
│ 1. Classify  │──── Is it ambiguous? ───▶ Ask user (1 question)
│ 2. Profile   │                                    │
│ 3. Route     │◀───────────────────────────────────┘
└──────┬───────┘
       │ generates
       ▼
┌──────────────────┐
│   TaskProfile    │
│                  │
│ • artifactType   │
│ • riskScore      │
│ • reviewDepth    │
│ • authOrSecrets  │
│ • publicSurface  │
│ • userFacing     │
│ • externalInteg  │
└────────┬─────────┘
         │
         ▼
┌──────────────────────────────────────┐
│         MODE RESOLVER                 │
│    (Deterministic decision table)     │
│                                       │
│  TaskProfile ──▶ assuranceLenses[]    │
│                                       │
│  Rules evaluated in order:            │
│  • reviewDepth=2 + code → arch        │
│  • riskScore≥2 + auth → security      │
│  • delivery_gate + AC → qa            │
│  • externalInteg + public → contract  │
│  • userFacing + spec → product        │
│  • workflow + ops risk → failure       │
│  • userFacing + UX → friction         │
└────────┬─────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────┐
│      EXECUTION PLAN                   │
│                                       │
│  pipeline: "implement"                │
│  stages: [research, code, review...]  │
│  lenses: [security, architecture]     │
│  approvalGates: [destructive_actions] │
└────────┬─────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────┐
│      PIPELINE EXECUTOR                │
│                                       │
│  For each stage:                      │
│    1. Build InvocationRequest         │
│    2. Call adapter.invoke()           │
│    3. Collect InvocationResult        │
│    4. Check escalation_rules          │
│    5. Pass artifacts to next stage    │
│                                       │
│  On halt_pipeline → stop + notify     │
│  On widen_review → add mode + rerun   │
└────────┬─────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────┐
│      AUDIT & MEMORY                   │
│                                       │
│  • Record events to events.jsonl      │
│  • Snapshot memory state              │
│  • Store change proposals             │
│  • Memory Curator appends learnings   │
└──────────────────────────────────────┘
```

## 2. Pipeline Stage Execution (implement.yaml example)

```
┌──────────────────────────────────────────────────────────────────────┐
│                  IMPLEMENT PIPELINE                                    │
└──────────────────────────────────────────────────────────────────────┘

Stage 1 (sequential, optional)          Stage 2 (sequential, mandatory)
┌─────────────────────────┐             ┌─────────────────────────────┐
│ research_dependencies   │             │ write_code                  │
│                         │             │                             │
│ Agent: Researcher       │────────────▶│ Agent: Coder                │
│ Mode:  verify           │             │ Mode:  implement            │
│                         │             │                             │
│ Output:                 │             │ Output:                     │
│  dependency_validation_ │             │  source_code                │
│  report.md              │             │  unit_tests                 │
│                         │             │  build_success_log          │
│ Cannot: mutate code     │             │ Cannot: untested stubs      │
└─────────────────────────┘             └──────────────┬──────────────┘
                                                       │
                                                       ▼
                          Stage 3 (PARALLEL, mandatory)
         ┌─────────────────────────────────────────────────────────────┐
         │                   validation_layer                           │
         │                                                             │
         │  ┌─────────────────┐ ┌─────────────────┐ ┌──────────────┐ │
         │  │ security_audit  │ │ qa_verification │ │ architecture │ │
         │  │                 │ │                 │ │ _review      │ │
         │  │ Agent: Reviewer │ │ Agent: Reviewer │ │ Agent:Review │ │
         │  │ Mode: security  │ │ Mode: qa        │ │ Mode: arch   │ │
         │  │                 │ │                 │ │              │ │
         │  │ Out: verdict.json│ │ Out: verdict    │ │ Out: verdict │ │
         │  │ Cannot: fix code│ │ +coverage report│ │              │ │
         │  └─────────────────┘ └─────────────────┘ └──────────────┘ │
         │          │                    │                   │         │
         └──────────┼────────────────────┼───────────────────┼─────────┘
                    │                    │                   │
                    ▼                    ▼                   ▼
              ┌──────────────────────────────────────────────────┐
              │  Escalation Check                                 │
              │                                                   │
              │  critical_vulnerabilities > 0 → HALT PIPELINE     │
              │  auth change + unverified arch → WIDEN REVIEW     │
              │  all pass → continue                              │
              └──────────────────────┬───────────────────────────┘
                                     │
                                     ▼
                     Stage 4 (sequential, mandatory)
                     ┌────────────────────────────┐
                     │ memory_consolidation       │
                     │                            │
                     │ Agent: Memory Curator      │
                     │ Mode:  append              │
                     │                            │
                     │ Output: memory_patch       │
                     │ Cannot: workspace artifacts│
                     └────────────────────────────┘
```

## 3. Self-Improvement & Policy Mutation Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│               SELF-IMPROVEMENT PROTOCOL                               │
└──────────────────────────────────────────────────────────────────────┘

           SAFE (automatic)              GUARDED (5-step)
         ┌─────────────────┐          ┌──────────────────────────────┐
         │ Memory Evolution │          │ Policy Mutation               │
         │                  │          │                              │
         │ Memory Curator   │          │ 1. Agent proposes patch      │
         │ can freely:      │          │ 2. Curator validates format  │
         │  • Append        │          │ 3. Replay harness benchmarks │
         │  • Compact       │          │ 4. Human reviews diff        │
         │  • Promote       │          │ 5. Versioned merge           │
         │                  │          │                              │
         │ Targets:         │          │ Targets:                     │
         │  active-context  │          │  routing rules               │
         │  agents/*.md     │          │  mode definitions            │
         │  projects/*      │          │  pipeline templates          │
         │                  │          │  invariants                  │
         │ Constraints:     │          │  schemas                     │
         │  schema-valid    │          │  adapter interfaces          │
         │  no secrets      │          │                              │
         │  no role changes │          └──────────────────────────────┘
         │  own namespace   │
         └─────────────────┘
                                       INVARIANTS (immutable)
                                      ┌──────────────────────────────┐
                                      │ Cannot be changed ever:       │
                                      │  • Goal integrity             │
                                      │  • Human approval gates       │
                                      │  • Memory ownership bounds    │
                                      │  • Max feedback iterations    │
                                      │  • No secrets in memory       │
                                      │  • Core/Adapter separation    │
                                      └──────────────────────────────┘
```

## 4. Adapter Invocation Sequence

```
Planner                Adapter                  Backend (OpenCode)
  │                      │                          │
  │ invoke(request)      │                          │
  │─────────────────────▶│                          │
  │                      │  assemble prompt         │
  │                      │  (agent def + modes +    │
  │                      │   behaviors + memory)    │
  │                      │                          │
  │                      │  task(category, skills)  │
  │                      │─────────────────────────▶│
  │                      │                          │
  │                      │                          │ agent executes
  │                      │                          │ (tools, reads, writes)
  │                      │                          │
  │                      │  InvocationResult        │
  │                      │◀─────────────────────────│
  │                      │                          │
  │                      │  recordEvent(RunEvent)   │
  │                      │──────▶ events.jsonl      │
  │                      │                          │
  │ InvocationResult     │                          │
  │◀─────────────────────│                          │
  │                      │                          │
  │ (check escalation)   │                          │
  │                      │                          │
```
