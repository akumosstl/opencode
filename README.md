# OpenCode
Differents files (skills, commands, agents, etc) to build and run agentics flow using not only opencode but mainly.

Even though, the opencode was the target of this repo, everything or almost everything is suiteable for Claude Code and with few changes to Copilot and others.

Repository structure:

```bash
java/
|
springboot/
|
template/

```

- ***java***: prompts and .opencode (skills, commands, context and agents) folder based on java projects
- ***springboot***: prompts and .opencode (skills, commands, context and agents) folder based on spring-boot projects
- ***template**: templates files

# Execution Flow
The simple execution flow is:

```
Raw Spec
   ↓
spec-normalizer
   ↓
feature-planner
   ↓
task-breaker
   ↓
java-executor
```

Prompt:

```bash
Use spec-normalizer on:

"Users should be able to transfer money between wallets. It must not duplicate transactions. Add logging."


```

# Folder Structure

```bash
.opencode/
 ├── agents/
 │    ├── feature-planner.md
 │    ├── task-breaker.md
 │    └── java-executor.md
 ├── commands/
 │    └── plan-feature.md
 └── context/
      └── project-context.md


```

# Running

Having a feature normalize it:

```bash
.opencode/agents/spec-normalizer.md
```

***Prompt***

```bash
Use spec-normalizer on:

"Users should be able to transfer money between wallets. It must not duplicate transactions. Add logging."


```


Not having a pre written feature write one:

```bash
feature-spec-template.md
```

Valide it:

```bash
.opencode/agents/spec-validator.md

```

## 1 - Feature Planner

This agent reads a feature and creates a technical plan, not tasks.

```bash
.opencode/agents/feature-planner.md
```

This forces architectural thinking first.

## 2 - Task Breaker

Now convert the plan into actionable tasks.

```bash
.opencode/agents/task-breaker.md
```

No fluff. No vague items like “implement feature”.

## 3 - Code

Only now code.

```bash
.opencode/agents/java-executor.md
```
This prevents architectural drift.

# Orchestrator Command

Using a reusable pipeline command.

```bash
.opencode/commands/plan-feature.md
```

Now you can run:

```bash
/plan-feature "Implement wallet transfer with idempotency and audit logging"
```

And get structured tasks.

## Simple case

Step 1 — Write feature using this template

Do NOT skip sections. Missing sections create hallucinations.

Step 2 — Run Planner Agent, prompt:

```bash
Use feature-planner on this spec.
```

Planner now has structured input.

Step 3 — Run Task Breaker

```bash
Use task-breaker on planner output.
```

Now tasks are grounded in architecture and constraints.


# Why This Works

Context stabilizes decisions.

Planner thinks architecture.

Task breaker enforces engineering discipline.

Executor focuses on precision.

Each agent has a single responsibility.

If you let one agent do everything, you get average code.

If you split responsibility, you get senior-level output.


Firtlly have a 

"Users should be able to transfer money between wallets. It must not duplicate transactions. Add logging."



# How use agents in opencode?

In OpenCode, agents are specialized AI assistants that handle coding tasks. 

You can use primary agents for main chat, cycle between them using Tab, or configure custom behaviors using AGENTS.md files. 

Agents can be defined for specific tasks, and subagents can be triggered for specialized actions within a project, enabling collaborative, automated development workflows. 



***Key Ways to Use Agents in OpenCode:***



***Switching Agents:*** Use Tab to cycle between active agents in the terminal user interface (TUI).



***Custom Instructions (AGENTS.md):*** Create an AGENTS.md file in your project root to define specific rules for the AI, similar to Cursor's rules. Run /init in OpenCode to generate this automatically.



***Agent Configuration:*** Define specialized agents in ~/.config/opencode/agents/ or .opencode/agents/ to handle specific tasks.



***Subagent Invocation:*** Use Agent configuration in commands to specify which agent executes a task, allowing a main agent to delegate work to specialized subagents.



***Agent Skills:*** Use SKILL.md files in your repository to allow agents to discover and utilize reusable instructions.


***GitHub Integration:*** Set up the OpenCode GitHub app to run agents on pull requests and comments by including /oc or /opencode. 


***Common Agent Commands:***

/init: Initializes a new AGENTS.md file.

opencode --agent <name>: Starts a session directly with a specific agent. 

Ensure you have the required API keys (e.g., Anthropic) set up in your environment for the agents to function. 

