# Antigravity Runtime Adapter Specification

This document details how the Antigravity framework fulfills the abstract adapter interface defined in `docs/RFC-001-Swarm-v2.md`. 
The Antigravity Root Agent acts as the Planner/Orchestrator, bridging the Portable Core cognitive logic with Antigravity's native execution capabilities.

## Interface Mapping

### RuntimeCapabilities
- `supportsParallelExecution`: `true` (via `invoke_subagent` which accepts an array of subagents to launch concurrently).
- `maxContextTokens`: Automatically handled by Antigravity's native context management window (using Gemini Pro/Flash).
- `availableTools`: Dynamically derived from the Antigravity native tools (mapped in `runtime-config.yaml`).
- `approvalMechanisms`: Uses native Antigravity prompt interactions and native user approvals during `run_command` execution.

### Subagent Execution
- **`invoke`**: Maps to a single `invoke_subagent` call. The subagent is passed the unified `system_prompt` containing its core definition, assigned modes, lenses, checklist, and output schemas.
- **`invokeParallel`**: Maps to passing multiple agent specifications into the `Subagents` array of a single `invoke_subagent` call.
- **Inter-Agent Communication**: Antigravity's `send_message` tool handles all message routing between the Planner (Root) and the Subagents (or peer-to-peer if configured).

### Ref Interactions
- **Artifact Refs**: Antigravity's `write_to_file` and code replacement tools handle creating and updating artifacts (`type: artifact`).
- **Memory Refs**: Antigravity reads from and writes to the `memory/` namespace on the local filesystem.

### Approval Flow
- **`requestApproval`**: Handled natively by Antigravity's conversational interface or `ask_question` tool, asking the user explicitly for confirmation before executing.

### Run Events & Logging
- **`recordEvent`**: Major events and checkpoints are written to `.sdd/` or logged in the Antigravity artifact store. High-frequency feedback loops use in-memory `send_message` instead of disk I/O to avoid bottlenecks.
