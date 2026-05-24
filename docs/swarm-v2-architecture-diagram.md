# Swarm v2 — Architecture Diagram

## Two-Layer Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           PORTABLE CORE                                      │
│                    (Backend-agnostic cognitive layer)                         │
│                                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Schemas    │  │  Policies   │  │  Pipelines   │  │  Agent Defs      │  │
│  │             │  │             │  │              │  │                  │  │
│  │ task-profile│  │ routing     │  │ implement    │  │ planner.md       │  │
│  │ exec-plan   │  │ risk        │  │ audit        │  │ researcher.md    │  │
│  │ invocation  │  │ review      │  │ fix          │  │ coder.md         │  │
│  │ memory-entry│  │ mutation    │  │ research     │  │ spec-writer.md   │  │
│  │ mode-overlay│  │ invariants  │  │ spec-to-issue│  │ reviewer.md      │  │
│  │             │  │             │  │ review-pr    │  │ memory-curator.md│  │
│  └─────────────┘  └─────────────┘  └──────────────┘  └──────────────────┘  │
│                                                                              │
│  ┌──────────────────────────────────┐  ┌─────────────────────────────────┐  │
│  │  Modes (Overlays)                │  │  Workspace Protocol             │  │
│  │                                  │  │                                 │  │
│  │  reviewer/ security, arch, qa,   │  │  protocol.md                    │  │
│  │           contract, product,     │  │  artifact-map.yaml              │  │
│  │           failure, friction      │  │                                 │  │
│  │  researcher/ survey, compare,    │  │  Behaviors/                     │  │
│  │             verify               │  │    shared/                      │  │
│  │  coder/ implement, patch,        │  │    domains/                     │  │
│  │         refactor, migrate        │  │                                 │  │
│  │  spec-writer/ story, adr, runbook│  │                                 │  │
│  │  memory-curator/ append, compact,│  │                                 │  │
│  │                  retro, proposal │  │                                 │  │
│  └──────────────────────────────────┘  └─────────────────────────────────┘  │
│                                                                              │
└──────────────────────────────────┬───────────────────────────────────────────┘
                                   │
                     ┌─────────────▼─────────────┐
                     │  Abstract Adapter Interface │
                     │                            │
                     │  getCapabilities()         │
                     │  startRun(RunContext)       │
                     │  invoke(InvocationRequest)  │
                     │  invokeParallel(requests[]) │
                     │  readRef(Ref)              │
                     │  writeArtifact(Ref, ...)   │
                     │  requestApproval(...)      │
                     │  recordEvent(RunEvent)     │
                     └─────────────┬─────────────┘
                                   │
┌──────────────────────────────────▼───────────────────────────────────────────┐
│                          RUNTIME ADAPTER                                      │
│                    (Backend-specific execution layer)                         │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  adapters/opencode/                                                  │    │
│  │                                                                      │    │
│  │  adapter-spec.md ─── How the adapter maps core concepts to OpenCode  │    │
│  │  capabilities.yaml ─ Runtime feature flags & limits                  │    │
│  │  toolmap.yaml ────── Core actions → OpenCode tool calls              │    │
│  │  prompt-assembler.md  System prompt construction rules               │    │
│  │  runtime-config.yaml  Model selection, token limits, parallelism     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Could also be:  adapters/claude-code/  adapters/cursor/  adapters/custom/   │
└──────────────────────────────────────────────────────────────────────────────┘
```

## Data Flow

```
User Request
     │
     ▼
┌──────────┐    TaskProfile     ┌───────────────┐
│  Planner │ ──────────────────▶│ Mode Resolver │
│          │                    │ (deterministic)│
└──────────┘                    └───────┬───────┘
     │                                  │
     │  Execution Plan                  │ assuranceLenses[]
     ▼                                  ▼
┌──────────────────────────────────────────────┐
│            Pipeline Executor                  │
│                                              │
│   InvocationRequest[] ──▶ RuntimeAdapter     │
│   (agent + modes + refs)     .invoke()       │
└──────────────────────────────────────────────┘
     │
     ▼
┌──────────────────────┐
│  Audit Trail         │
│  swarm/runs/{run-id}/│
│    manifest.json     │
│    events.jsonl      │
│    memory-snapshot   │
└──────────────────────┘
```

## External Packages Integration

```
┌─────────────────────────────────────────────────┐
│              PORTABLE CORE                        │
│  (swarm/core/)                                   │
└────────────────────────┬────────────────────────┘
                         │ loads via manifest
                         ▼
┌─────────────────────────────────────────────────┐
│            EXTERNAL PACKAGES                     │
│  (swarm/packages/{name}/)                        │
│                                                  │
│  Trust Tiers:                                    │
│  ┌────────────┐ ┌────────────┐ ┌─────────────┐ │
│  │ Behaviors  │ │   Modes    │ │   Agents    │ │
│  │ (low risk) │ │ (med risk) │ │ (high risk) │ │
│  │ auto-load  │ │ routed by  │ │ explicit    │ │
│  │ read-only  │ │ planner    │ │ registration│ │
│  └────────────┘ └────────────┘ └─────────────┘ │
│                                                  │
│  Isolation:                                      │
│  • Cannot override core agents                   │
│  • Memory scoped to memory/packages/{name}/      │
│  • No floating versions                          │
│  • Undeclared capabilities rejected              │
└─────────────────────────────────────────────────┘
```
