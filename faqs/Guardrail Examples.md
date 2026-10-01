# GuardRail Examples


Claude Code evaluates your settings.json permissions in a strict waterfall order: deny rules take absolute precedence, followed by ask rules, and finally allow rules. If a tool call doesn't match a rule, it falls back to the environment’s defaultMode.  

## 1. Bare Tool Disabling (Context Stripping)

If you place a bare tool name (like WebFetch) in a deny list, Claude Code completely strips that tool from the LLM's context window. The model won't even realize the tool exists, removing any opportunity for it to plan an exploit.  


* Check: deny: `["WebFetch"]` $\rightarrow$ Blocks all outbound network requests initiated by the native fetch tool.  


## 2. Parameter-Scoped Tool Restrictions
You can map specific parameter matches using wildcards (*) to create fine-grained command or file boundaries.  


* Check: deny: `["Bash(curl *)", "Bash(wget *)"]` $\rightarrow$ Allows normal terminal use but immediately blocks commands attempting outbound web downloads.
* Check: ask: `["Bash(git push *)"]` $\rightarrow$ Allows local version control to proceed silently, but freezes execution to ask for human confirmation before code leaves the machine.
* Check: deny: `["Read(**/secrets/**)", "Write(./.env)"]` $\rightarrow$ Prevents the model's native filesystem utilities from reading configuration secrets or overwriting environmental profiles.  


------------------------------

# Examples of settings.json Configurations

Claude Code aggregates settings hierarchically across multiple scopes: Managed Organization Policy (highest priority), Project Local (.claude/settings.local.json), Project Shared (.claude/settings.json), and User Global (~/.claude/settings.json). A deny rule explicitly set anywhere in these layers remains enforced.  

Here are three templates tailored for different security postures:

## 1. Strict Enterprise Repository Guardrails (.claude/settings.json)

Commit this to the root of a shared repository to enforce a safe, zero-trust collaborative baseline for your entire team.  

```json
{
  "permissions": {
    "defaultMode": "default",
    "disableBypassPermissionsMode": "disable",
    "disableAutoMode": "disable",
    "deny": [
      "WebFetch",
      "Bash(curl*)",
      "Bash(wget*)",
      "Bash(rm -rf /)",
      "Read(**/.env*)",
      "Write(**/.env*)",
      "Read(**/infra/terraform.tfstate)",
      "Write(**/infra/terraform.tfstate)",
      "mcp__*"
    ],
    "ask": [
      "Bash(git push*)",
      "Bash(npm publish*)",
      "Bash(docker run*)",
      "Write(**/*.config*)"
    ],
    "allow": [
      "Read(src/**)",
      "Glob",
      "Grep"
    ]
  }
}
```


* Why it works: It forces interactive mode (default), completely disables unsafe flags (disableBypassPermissionsMode), removes all third-party Model Context Protocol (MCP) server capabilities (mcp__*), and isolates access exclusively to the src/ directory.  


## 2. Balanced Local Developer Setting (.claude/settings.local.json)

Place this gitignored file in your local workspace when you want high-velocity development without constant confirmation popups for safe adjustments.  

```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "deny": [
      "Read(~/.*)", 
      "Read(**/*.pem)",
      "Read(**/*.key)"
    ],
    "ask": [
      "Bash(npm run deploy*)",
      "Bash(db:migrate*)"
    ],
    "allow": [
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(npm test)",
      "Bash(npm run build)"
    ]
  },
  "sandbox": {
    "enabled": true
  }
}
```


* Why it works: Setting defaultMode to "acceptEdits" allows Claude to seamlessly modify code architecture without stopping to prompt for minor file updates. However, it safely keeps native OS sandboxing enabled, protects system-level directories (~/.*), and forces confirmation before schema migrations or deployments.  


## 3. Total Non-Mutation Policy (~/.claude/settings.json)

Apply this globally in your user profile to restrict Claude Code to read-only analysis when auditing unfamiliar codebases or open-source software.  

```json
{
  "permissions": {
    "defaultMode": "plan",
    "deny": [
      "Write",
      "Edit",
      "Bash"
    ],
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Task"
    ]
  }
}
```


* Why it works: By leveraging "plan" mode and hard-denying mutation tools like Write, Edit, and Bash, the model is strictly limited to scanning structural logic and reasoning without any capacity to alter state.  

