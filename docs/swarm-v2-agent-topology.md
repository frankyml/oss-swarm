# Swarm v2 — Agent Topology & Relationships

## Core Agents & Their Modes

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        CORE AGENTS (6 total)                                  │
│              Each satisfies: Memory + Permissions + Reasoning Loop            │
└──────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────────────────┐    │
│  │   PLANNER    │     │  RESEARCHER  │     │         CODER            │    │
│  │              │     │              │     │                          │    │
│  │ Memory:      │     │ Memory:      │     │ Memory:                  │    │
│  │  routing     │     │  source rank │     │  repo conventions        │    │
│  │  heuristics  │     │  search heur │     │  build recipes           │    │
│  │  outcomes    │     │  domain prefs│     │  framework patterns      │    │
│  │              │     │              │     │                          │    │
│  │ Permissions: │     │ Permissions: │     │ Permissions:             │    │
│  │  orchestrate │     │  READ-ONLY   │     │  WRITE code/config/git   │    │
│  │  approve req │     │  web, docs   │     │  terminal, compiler      │    │
│  │  build plans │     │  databases   │     │  ONLY writer of src      │    │
│  │              │     │              │     │                          │    │
│  │ Modes: none  │     │ Modes:       │     │ Modes:                   │    │
│  │ (orchestrator│     │  ┌─────────┐ │     │  ┌───────────┐          │    │
│  │  role)       │     │  │ survey  │ │     │  │ implement │          │    │
│  │              │     │  │ compare │ │     │  │ patch     │          │    │
│  └──────────────┘     │  │ verify  │ │     │  │ refactor  │          │    │
│                       │  └─────────┘ │     │  │ migrate   │          │    │
│                       └──────────────┘     │  └───────────┘          │    │
│                                            └──────────────────────────┘    │
│                                                                              │
│  ┌──────────────────┐  ┌──────────────┐  ┌──────────────────────────┐      │
│  │   SPEC WRITER    │  │   REVIEWER   │  │    MEMORY CURATOR        │      │
│  │                  │  │              │  │                          │      │
│  │ Memory:          │  │ Memory:      │  │ Memory:                  │      │
│  │  decomp patterns │  │  defect patt │  │  compaction rules        │      │
│  │  templates       │  │  heuristics  │  │  promotion thresholds    │      │
│  │  AC patterns     │  │  historical  │  │  replay baselines        │      │
│  │                  │  │              │  │                          │      │
│  │ Permissions:     │  │ Permissions: │  │ Permissions:             │      │
│  │  docs, backlog   │  │  READ-ONLY   │  │  WRITE memory/ only     │      │
│  │  .sdd dirs       │  │  code, PRs   │  │  compaction, promotion   │      │
│  │  NO app code     │  │  specs       │  │  NO workspace artifacts  │      │
│  │                  │  │  NO writes   │  │                          │      │
│  │ Modes:           │  │              │  │ Modes:                   │      │
│  │  ┌────────────┐  │  │ Modes:       │  │  ┌─────────────────┐    │      │
│  │  │story-break │  │  │  ┌─────────┐ │  │  │ append          │    │      │
│  │  │adr         │  │  │  │security │ │  │  │ compact         │    │      │
│  │  │runbook     │  │  │  │arch     │ │  │  │ retrospective   │    │      │
│  │  └────────────┘  │  │  │qa       │ │  │  │ proposal-valid  │    │      │
│  │                  │  │  │contract │ │  │  └─────────────────┘    │      │
│  └──────────────────┘  │  │product  │ │  └──────────────────────────┘      │
│                        │  │failure  │ │                                     │
│                        │  │friction │ │                                     │
│                        │  └─────────┘ │                                     │
│                        └──────────────┘                                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

## v1 → v2 Agent Consolidation (Deprecation Map)

```
         SWARM v1 (12 agents)                    SWARM v2 (6 agents)
    ┌─────────────────────────┐            ┌────────────────────────────┐
    │                         │            │                            │
    │  Planner ───────────────┼───────────▶│  Planner                   │
    │  Researcher ────────────┼───────────▶│  Researcher                │
    │  Coder ─────────────────┼───────────▶│  Coder                     │
    │  Spec Writer ───────────┼───────────▶│  Spec Writer               │
    │                         │            │                            │
    │  ┌─ Security Sentinel ──┼──┐         │  Reviewer                  │
    │  │  QA Guardian ────────┼──┤         │    ├── .security           │
    │  │  Contract Critic ────┼──┤────────▶│    ├── .qa                 │
    │  │  Product Owner ──────┼──┤         │    ├── .contract           │
    │  │  Friction Auditor ───┼──┤         │    ├── .product            │
    │  │  Failure Dramatist ──┼──┘         │    ├── .friction           │
    │  └──────────────────────┼──          │    ├── .failure            │
    │                         │            │    └── .architecture       │
    │  Retrospective Agent ───┼──┐         │                            │
    │  Memory Curator ────────┼──┤────────▶│  Memory Curator            │
    │                         │  │         │    ├── .append             │
    │  Personal Assistant ────┼──┘         │    ├── .compact            │
    │                         │            │    ├── .retrospective      │
    └─────────────────────────┘            │    └── .proposal-validation│
                                           │                            │
                                           │  (Personal Assistant →      │
                                           │   Planner.personal-context) │
                                           └────────────────────────────┘

    Result: 12 agents → 6 agents + 22 modes
    Benefit: Clearer boundaries, less routing ambiguity, shared memory per agent
```

## Permission Boundaries

```
┌──────────────────────────────────────────────────────────────────┐
│                    WRITE PERMISSIONS MAP                           │
└──────────────────────────────────────────────────────────────────┘

                    Source     .sdd/     memory/    Backlog   Terminal
                    Code      Specs     Files      (Jira/GH) (Build)
                    ─────     ─────     ───────    ────────  ────────
  Planner            ✗         ✗         ✗          ✗         ✗
  Researcher         ✗         ✗         ✗          ✗         ✗
  Coder              ✓         ✗         ✗          ✗         ✓
  Spec Writer        ✗         ✓         ✗          ✓         ✗
  Reviewer           ✗         ✗         ✗          ✗         ✗
  Memory Curator     ✗         ✗         ✓          ✗         ✗

  Legend: ✓ = allowed   ✗ = forbidden
```

## Mode Application Mechanics

```
┌─────────────────────────────────────────────────────────────────┐
│                  HOW MODES OVERLAY AN AGENT                       │
└─────────────────────────────────────────────────────────────────┘

                 ┌─────────────────────────────┐
                 │      Core Agent Definition   │
                 │                              │
                 │  • Base identity             │
                 │  • Memory ownership          │
                 │  • Permission boundary       │
                 │  • Reasoning loop type       │
                 └──────────────┬───────────────┘
                                │
                    Mode applied │ (overlay, not replacement)
                                ▼
    ┌───────────────────────────────────────────────────────┐
    │              MODE OVERLAY (4 components)               │
    │                                                       │
    │  ┌─────────────────┐    ┌──────────────────────────┐ │
    │  │ 1. Lens Preamble│    │ 2. Checklist             │ │
    │  │                 │    │                          │ │
    │  │ Biases agent's  │    │ Mandatory questions      │ │
    │  │ focus area      │    │ agent must answer        │ │
    │  │ (injected into  │    │                          │ │
    │  │  system prompt) │    │                          │ │
    │  └─────────────────┘    └──────────────────────────┘ │
    │                                                       │
    │  ┌─────────────────┐    ┌──────────────────────────┐ │
    │  │ 3. Output Schema│    │ 4. Escalation Rules      │ │
    │  │                 │    │                          │ │
    │  │ Required format │    │ When to:                 │ │
    │  │ for deliverable │    │  • halt_pipeline         │ │
    │  │ (verdict, items │    │  • widen_review          │ │
    │  │  severity, etc.)│    │  • expand_scope          │ │
    │  └─────────────────┘    └──────────────────────────┘ │
    └───────────────────────────────────────────────────────┘
                                │
                                ▼
                 ┌──────────────────────────────┐
                 │   RESULTING INVOCATION        │
                 │                               │
                 │   Agent: reviewer             │
                 │   Mode:  security             │
                 │   = Reviewer with security    │
                 │     lens, checklist, schema,  │
                 │     and escalation rules      │
                 │                               │
                 │   Still READ-ONLY (inherits   │
                 │   base agent permissions)     │
                 └──────────────────────────────┘
```
