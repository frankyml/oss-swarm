# System Architecture

OSS-Swarm v2 is built on the principle of **Strict Decoupling**.

## Two-Layer Design

### 1. Portable Core
The cognitive source of truth. It owns task profiling, routing rules, pipeline templates, workspace protocol, agent definitions, and mode definitions. It contains NO backend-specific tool definitions.
- Lives in `swarm-core/`

### 2. Runtime Adapters
Connects the Portable Core to an LLM backend (e.g., Antigravity, OpenCode). It handles prompt assembly, parallel execution, tool exposing, and human approvals.
- Lives in `adapters/`

## Key Mechanisms

### The Semantic Router
Instead of brittle rule-based routing, the Planner uses an LLM-powered **Semantic Router**. It evaluates an incoming `TaskProfile` against the descriptions of all available mode overlays to dynamically assign `assuranceLenses`.

### Hybrid Communication
- **In-Memory Event Bus**: Used for high-frequency feedback loops (e.g., Reviewer sending diff feedback to the Coder) to prevent I/O bottlenecks.
- **Artifact-Mediated Workspace**: Used for major checkpoints (`.sdd/{task}/`) to preserve the audit trail and guarantee state persistence.

### Self-Improvement & Invariants
The swarm can update its own `memory/` files (Safe) and propose policy mutations (Guarded). However, strict invariants (`policies/invariants.yaml`) ensure the swarm can never drift from its goals or execute destructive actions without explicit human approval.
