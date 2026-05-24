# Quick Start

Get your OSS-Swarm up and running in minutes.

## 1. Prerequisites

- Git
- An AI coding assistant runtime (currently supported: OpenCode with Claude, Antigravity with Gemini).
- MCP server integrations for your tools (GitHub, Jira, etc.) - *Optional but recommended.*

## 2. Install

```bash
# Clone this repository into your workspace
git clone <this-repo> oss-swarm

# The swarm is file-based — no build step required.
```

## 3. Configure your runtime adapter

OSS-Swarm supports multiple runtimes via adapters. Choose your preferred environment below.

### 🚀 Option A: Antigravity (Recommended)
Antigravity provides native asynchronous subagents and robust file system tools.

1. Copy the `adapters/antigravity/` contents to your Antigravity configuration.
2. The Antigravity Root Agent will automatically act as your Planner.
3. Model Tiering is pre-configured (`Gemini Pro` for deep reasoning, `Gemini Flash` for execution).

### Option B: OpenCode (Legacy)
1. Copy `adapters/opencode/` to your OpenCode configuration directory.
2. Setup the skills directories for each of the 6 core agents, pointing them to `swarm-core/behaviors/`.
3. Add swarm mode activation to your global instructions.

## 4. Initialize Memory

On first run, the system bootstraps by creating memory files in the `memory/` directory. 
The Planner will interview you during the first session to populate `memory/core/user-preferences.md`. 

After 5-10 interactions, memory effectiveness improves significantly!

## 5. Verify it Works

Ask your assistant: 
> *"Investigate how this repository is structured"*

This will trigger the `research-report` pipeline, spinning up the Researcher agent, and validating your setup!
