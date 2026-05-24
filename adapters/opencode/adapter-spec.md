# OpenCode Runtime Adapter

This adapter implements the `RuntimeAdapter` interface for the OpenCode execution environment.

## Capabilities
- Parallel agent execution via `task(run_in_background=true)`
- Human approval via interactive `question()` tool
- File-based artifact persistence
- Session continuity via `task_id`

## Tool Mapping

| Abstract Method | OpenCode Implementation |
|---|---|
| `invoke()` | `task(category=..., load_skills=[...], run_in_background=false)` |
| `invokeParallel()` | Multiple `task(run_in_background=true)` + `background_output()` |
| `readRef()` | `read(filePath=ref.uri)` |
| `writeArtifact()` | `write(filePath=ref.uri, content=...)` |
| `requestApproval()` | `question(questions=[...])` |
| `recordEvent()` | Append to `swarm/runs/{runId}/events.jsonl` |
| `getCapabilities()` | Static config from `capabilities.yaml` |

## Agent → Category Mapping

| Core Agent | Default Route |
|---|---|
| planner | Orchestrator (Sisyphus) — not delegated |
| researcher | `subagent_type="librarian"` |
| coder | `category={tier}` based on complexity |
| spec-writer | `category={tier}` based on complexity |
| reviewer | `subagent_type="oracle"` |
| memory-curator | `category="quick"` |

## Mode → Skill Mapping

Modes are injected as skills via `load_skills`. The adapter reads the mode YAML and assembles the prompt with lens_preamble + checklist prepended.
