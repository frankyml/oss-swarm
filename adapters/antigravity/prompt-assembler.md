# Antigravity Prompt Assembler

This document dictates how the Antigravity Root Agent (Planner) must dynamically construct the `system_prompt` and configuration for a subagent before invoking it via the `define_subagent` tool.

## Assembly Workflow

When an execution plan dictates that a subagent must be launched, the Planner MUST follow these steps to assemble the configuration:

### Step 1: Read Base Definition
Read the target agent's base definition file located at `swarm-core/agents/{agent_name}.md`. 
Extract the core reasoning loop, memory ownership boundaries, and core invariant rules. This forms the base of the `system_prompt`.

### Step 2: Read Mode Overlays
Identify the `assuranceLenses` (modes) assigned to this task via the Semantic Router.
For each assigned mode, read the definition file from `swarm-core/modes/{agent_name}/{mode_name}.yaml`.

### Step 3: Concatenate System Prompt
Assemble the final `system_prompt` for the `define_subagent` tool by concatenating the components in the following order:

1. **Lens Preambles**: Combine the `lens_preamble` from all assigned modes. Insert this at the very top of the prompt to ensure it strongly biases the agent's behavior.
2. **Base Definition**: Insert the core agent instructions gathered in Step 1.
3. **Mode Checklists**: Create a "Required Validation Checklist" section combining the `checklist` arrays from all assigned modes.
4. **Output Schema**: If a mode dictates an `output_schema`, strictly instruct the subagent to structure its final artifact or response according to that schema.
5. **Escalation Rules**: Append the `escalation_rules` from the modes, instructing the subagent to halt and use `send_message` back to the Planner if the conditions are met.

### Step 4: Model Tier Assignment (Token Economy)
1. Read `memory/core/user-preferences.md` to determine the `active_profile` (e.g., `balanced`).
2. Read `adapters/antigravity/runtime-config.yaml` to find the semantic model alias assigned to the target agent under that specific economy profile (e.g., `reasoning_model`).
3. Resolve the semantic alias using the `model_aliases` mapping at the top of the config to find the exact provider model string (e.g., `gemini-pro`).

### Step 5: Define and Invoke
1. Call `define_subagent` with the assembled `system_prompt` and an appropriate `description`.
2. Call `invoke_subagent` to launch the execution, passing any necessary inputs via the `Prompt` argument.
