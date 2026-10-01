# Learnings from the `Claude Code Training`

## Areas covered
We covered following areas:

### Memory (Claude.md)

- What locations can they exist in
- What is their loading mechanism
- What is the context cost
- Does specialization reduce the context tax? (Ans: No!)

### Skills

- The need for Skills
- The skills.md file format
- Frontmatter `yaml` fields
- Progressive Disclosure
- How it saves tax on the context by loading on-need
- Recommended headers for markdown body

### MCP

- Why MCP?
- How MCP is the `USB-C` of the Agentic world
- The cost of MCP tools on context
- MCP Tool Search
- Brings Productivity Tools into the Agentic Ecosystem
- The deterministic part of a probabilistic system

### SubAgents

- Bundled subagents: claude-code-guide, explore, general-purpose
- Custom subgents
- Agent .md front-matter fields
- How subagents have their own context space
- Using multiple agents gives more combined context space
- Flow: Main agent (spawns) → subagent (solves, returns distilled findings) → Main agentf

### Plugins

- Bundles for reuse
- Include: skills, subagents, hooks, mcp servers
- Available via marketplaces
- Need to be installed
- Some already available: claude-official-plugins
- Useful plugins tried: playwright, skill-creator, playground
- Useful marketplaces: claude-community, composio-community
- Directory structure and manifet json format for creating own plugins

### Guardrails

- Combination: determinstic, probabilistic
- Determinstic:
    - allowedTools filters: Bash(cd src/*), Read(src/*)
    - disallowedTools filters: Bash(rm -rf *)
    - hooks
- Probabilistic:
    - Do's/Don't sections in skill/subagent files
- Safety layers
    - Organizational policy
    - Project level
    - User Level
    - Project level local

### References

- Claude content repository: https://github.com/py-train/cctrain-r1
- Example project for hands-on(python): Steel Drawing Parser: https://github.com/py-train/steel-drawing-parser
- Example project for hands-on(node/nextjs): https://github.com/py-train/hookhub