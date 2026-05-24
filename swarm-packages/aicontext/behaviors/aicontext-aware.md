# Behavior: AI Context Awareness

> When loaded, agents understand how to discover and use `.aicontext/` project context files.

## When This Applies

This behavior activates when working in a repository that contains an `.aicontext/` directory.

## Discovery Protocol

1. Check for `.aicontext/project.md` — the entry point for project identity and architecture
2. Check for `.aicontext/structure.md` — commands, services, file layout
3. Scan for any additional `.aicontext/*.md` files relevant to the current task

## Usage Rules

- READ `.aicontext/project.md` before making assumptions about project purpose or conventions
- READ `.aicontext/structure.md` before searching for files or running commands
- NEVER modify `.aicontext/` files unless explicitly asked — they are project-level documentation
- If `.aicontext/` is absent, skip this behavior entirely

## Integration with Swarm

- Planner: reads project.md to understand scope before decomposing goals
- Researcher: uses structure.md to orient codebase exploration
- Coder: reads conventions from project.md before implementing
- Spec Writer: references project identity when writing specs

## File Priority

If both `.aicontext/` and `swarm/core/` contain conflicting information:
- `swarm/core/` is authoritative for swarm behavior (agents, modes, pipelines, policies)
- `.aicontext/` is authoritative for project-specific context (architecture, conventions, structure)
