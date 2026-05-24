# OSS-Swarm (PersonalAgent Swarm v2)

A portable, strictly-decoupled multi-agent swarm system for software engineering. Features 6 core agents, behavioral mode overlays, sandboxed self-improvement, and external package support.

## Elevator Pitch

Unlike rigid, monolithic AI coding assistants, **OSS-Swarm** decouples cognitive orchestration from the execution backend. 

We avoid infinite agent sprawl by relying on exactly 6 foundational core agents, dynamically injecting "Modes" (like a Security Lens or Architecture Checklist) to specialize their behavior on the fly. With an architecture built for self-improvement guarded by strict human invariants, OSS-Swarm is designed to safely learn your repository conventions over time.

You can run OSS-Swarm on various runtimes (like Antigravity or OpenCode) simply by configuring the proper `adapter`.

## Quick Start

### 1. Prerequisites
- An AI coding assistant runtime (Currently natively supporting **Antigravity** and **OpenCode**).

### 2. Install
```bash
# Clone this repository into your workspace
git clone <this-repo> oss-swarm

# The swarm is file-based — no build step required.
```

### 3. Setup your Adapter
Choose your runtime and configure it using the files in the `adapters/` directory:
- **For Antigravity**: Copy `adapters/antigravity/` to your config. Antigravity will handle parallel execution and model tiering out of the box.
- **For OpenCode**: Copy `adapters/opencode/` to your skills directory and configure your prompts.

### 4. Verify
Trigger your first swarm pipeline by asking your assistant: 
> *"Investigate how this repository is structured"*

## Documentation

For deep-dive documentation on how the Semantic Router works, the details of all assurance modes, and the two-layer architecture design, **[visit the OSS-Swarm Wiki](wiki/Home.md)**!
