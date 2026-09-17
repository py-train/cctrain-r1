## Memory Architecture

Memory in Claude Code follows a hierarchical system where different scopes serve different purposes. Unlike Claude Web/Desktop's 24-hour synthesis cycle Claude Code has two memory systems that both load at the start of every session and update continuously, not on a timer:

```mermaid
graph TB
    A["Session Start"]
    B["CLAUDE.md Files<br/>(you write)"]
    C["Auto Memory<br/>(Claude writes)"]
    D["Claude Session"]
    E["Your Correction /<br/>Preference"]

    B -->|loaded in full| A
    C -->|MEMORY.md loaded| A
    A --> D
    D -->|"Remember that..."| E
    E -->|writes during session| C
    D -->|"add this to CLAUDE.md"| B
```
