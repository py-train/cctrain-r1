# Claude Plan Mode vs Explore Agent: A comparison


## Overview

Plan Mode and the Explore subagent in Claude Code both let Claude read and analyze your codebase without making changes, but they serve different parts of the workflow.   


## Plan Mode 

- What it is: A global permission mode for your entire Claude Code session. 
- How it works: Disables write and edit tools globally so Claude cannot modify files or run side-effecting commands. 
- Purpose: Designed for interactive architecture brainstorming, asking clarifying questions, and getting your explicit approval on a step-by-step roadmap before any coding begins. 
- How to access: Press  to cycle through permission modes until Plan Mode is active, or start via the command line.   

## Explorer Agent (Plan Subagent) 

- What it is: A specialized, read-only internal subagent that Claude delegates tasks to. 
- How it works: When Claude needs deep codebase research during planning, it spins up this subagent to keep heavy exploration outputs isolated in a separate context window. 
- Purpose: Acts as a background researcher to gather dependencies, search files, and analyze patterns without cluttering your main conversation window. 
- How to access: Triggered automatically by Claude during Plan Mode, or run independently when you just want a quick summary of an unfamiliar project.   

## Key Differences 

| Feature | Plan Mode | Explorer / Plan Subagent  |
| --- | --- | --- |
| Scope | Session-level permission state | Task-level delegated worker  |
| Context Window | Main conversation | Isolated separate context  |
| Primary Goal | Formulate an approved implementation roadmap | Gather raw technical context and research  |

