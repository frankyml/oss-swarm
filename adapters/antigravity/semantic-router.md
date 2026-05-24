# Semantic Router Prompt Template

The Semantic Router is a decision-making prompt used by the Antigravity Planner to dynamically resolve which `assuranceLenses` (modes) should be applied to a core agent based on the current `TaskProfile`.

## Routing Instructions

Whenever a new `TaskProfile` is generated, evaluate it against the following logic to determine the active modes.

### Input
You will be provided with:
1. A **TaskProfile** containing the execution context, risk score, artifact types, and target constraints.
2. The catalog of available mode overlays located in `swarm-core/modes/`.

### Processing Rules
Cross-reference the `TaskProfile` parameters against the mode catalogs. Use the following heuristic mapping as a baseline:

**Reviewer Lenses:**
- If `reviewDepth == 2` AND `artifactType == "code"` -> assign `reviewer.architecture`
- If `riskScore >= 2` AND (`authOrSecrets == true` OR `publicSurface == true`) -> assign `reviewer.security`
- If pipeline stage is `delivery_gate` AND acceptance criteria exist -> assign `reviewer.qa`
- If `externalIntegration == true` OR `publicSurface == true` -> assign `reviewer.contract`
- If `userFacing == true` AND `artifactType == "spec"` -> assign `reviewer.product`
- If operational risk is high -> assign `reviewer.failure`
- If changes involve CLI, UX, or agent interaction flows -> assign `reviewer.friction`

**Researcher Lenses:**
- If domain is unknown and exploration is needed -> assign `researcher.survey`
- If evaluating trade-offs -> assign `researcher.compare`
- If external claims require factual confirmation -> assign `researcher.verify`

**Coder Lenses:**
- If building a new capability -> assign `coder.implement`
- If fixing a bounded bug -> assign `coder.patch`
- If refactoring existing structures -> assign `coder.refactor`
- If bumping frameworks or configs -> assign `coder.migrate`

### Output Requirement
You MUST output your decision as a strict JSON array of strings, where each string represents an applied mode.

Example Output:
```json
{
  "assuranceLenses": [
    "coder.implement",
    "reviewer.security",
    "reviewer.architecture"
  ]
}
```
