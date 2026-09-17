# Context Consumption Comparison

To customize and extend Claude Code, developers primarily leverage three distinct mechanisms: 

- CLAUDE.md (Memory)
- Claude Skills, and 
- Model Context Protocol (MCP) Servers

While they all shape how Claude behaves, they interact with the LLM's finite context window differently and serve unique roles.  


## Context Consumption Characteristics

The primary structural difference lies in how and when these layers feed data into Claude's context window.

 
- **CLAUDE.md / Memory (Static & Global)**: This file is an always-on, session-start context injection. Claude reads the full contents of CLAUDE.md at the start of every session. Because it is sent with every single request in the conversation history, large CLAUDE.md files act as a permanent tax on your token window.  
- **Claude Skills (Progressive & On-Demand)**: Skills use a highly optimized, three-tier progressive disclosure mechanism to preserve context. At session start, Claude only loads the basic names and descriptions of available skills. If a user request matches a skill, Claude dynamically pulls the full .md file, and only executes or loads associated scripts or templates if and when they are strictly needed.  
- **MCP Servers (Verbose & Upfront Bloat)**: MCP servers expose schemas, tool definitions, arguments, and metadata directly to the agent. Because an MCP server presents its entire functional surface area (multiple tools, parameters, descriptions, and examples) to the client, it often introduces extreme upfront context bloat. Connecting a handful of complex MCP servers can consume tens of thousands of tokens before you even type your first prompt.   


## Direct Parameter Comparison

| Feature / Parameter | CLAUDE.md / Memory | Claude Skills | MCP Servers |
|---|---|---|---|
| Primary Purpose | Stable project-level alignment, coding style rules, and setup commands. | Reusable procedural workflows, expert instructions, and text templates. | Secure gateway to external APIs, databases, filesystems, and custom apps. |
| Context Loading Strategy | Static, permanent payload injected on every message. | On-demand, adaptive progressive disclosure. | High upfront footprint due to full schema definitions and tool descriptions. |
| Data Nature | Static text / Markdown instructions. | Structured workflows, reasoning prompts, and local scripts. | Real-time, stateful, dynamic or binary data (e.g., live databases, API endpoints). |
| Execution Domain | Internal reasoning and rule adherence. | Local execution layer, text manipulation, and sequential logic. | External server-side processes (can handle binary payloads, streams, heavy compute). |
| Portability & Reuse | Scoped strictly to the specific project directory. | Shared via local file structures or text templates. | Highly portable; language-agnostic protocol reusable across Cursor, Windsurf, Claude Desktop, etc. |
| Setup Complexity | Zero. Just drop a markdown file in the root directory. | Low. Simple file tracking under .claude/commands/ or settings. | High. Requires running background processes, managing network connections, or setting up auth. |


## Architectural Synergy: How They Work Together

Rather than competing, these three features work best when layered hierarchically:  

   1. CLAUDE.md provides the overarching identity and baseline constraints for your repository (e.g., "We use TypeScript; run tests via npm test").  
   2. MCP Servers provide the physical "hands" to reach outside of Claude Code—connecting to corporate Slack channels, querying production Postgres databases, or calling specific SaaS APIs.  
   3. Claude Skills provide the operational "intelligence" or playbook telling Claude exactly how to coordinate those MCP tools to achieve complex, multi-step actions cleanly.  

