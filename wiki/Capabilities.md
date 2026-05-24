# Swarm Capabilities

OSS-Swarm's power lies in its modular structure. We define a tight set of **Core Agents**, dynamic **Modes**, and structured **Pipelines**.

## Core Agents

A Core Agent is defined strictly by three criteria: (1) distinct memory ownership, (2) distinct permission boundary, and (3) distinct artifact loop.

1. **Planner**: The Orchestrator. Evaluates tasks, selects pipelines, delegates work, and handles verification.
2. **Coder**: The execution agent. The only agent permitted to mutate source code, run tests, and execute builds.
3. **Researcher**: The librarian. Gathers information, cross-references claims, and reads external sources.
4. **Spec Writer**: The product owner. Breaks down epics into issues and writes structured documentation.
5. **Reviewer**: The oracle. Read-only validation loop that verifies implementations against specifications.
6. **Memory Curator**: The archivist. Compiles learnings, manages cross-pollination, and prunes stale memory.

## Assurance Modes (Lenses)

Instead of creating hundreds of specialized agents, OSS-Swarm uses **Modes**. A Mode is a behavioral overlay injected into a core agent.

For example, a `Reviewer` can be assigned the `Security` mode. It receives a specific preamble, checklist, output schema, and escalation rules dedicated to finding vulnerabilities.

**Available Modes include:**
- **Reviewer**: `architecture`, `security`, `qa`, `contract`, `product`, `failure`, `friction`.
- **Researcher**: `survey`, `compare`, `verify`.
- **Coder**: `implement`, `patch`, `refactor`, `migrate`, `skill-author`.

## Pipeline Templates

Pipelines dictate the execution graph. Standard pipelines include:
- `research-report`: Investigate topics and output structured findings.
- `spec-to-issues`: Break down a feature into GitHub/Jira tickets.
- `implement`: Code -> Test -> Review (Security/QA/Architecture) -> Memory Consolidate.
- `audit`: Deep codebase reviews looking for prioritized gaps.
- `skill-evolution`: Autonomously discover, author, benchmark, evaluate, and propose new agent skills.

## External Packages & Custom Skills (Trust Tiers)

You can extend the swarm using custom skills or the `swarm-packages/` directory.

- **Tier 1 (Behaviors)**: Low risk, read-only context injections.
- **Tier 2 (Modes/Agents)**: Medium risk. Jailed to their own memory namespace. Strictly sandboxed.
- **Tier 3 (Quarantined/Policies)**: High risk. Includes newly authored, quarantined self-improving skills and policy modifications. Under Antigravity, Tier 3 limits file access strictly to synthetic TaskProfile scopes and blocks terminal execution and web requests with immediate execution halts (no bypass allowed).
