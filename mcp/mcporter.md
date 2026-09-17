### MCPorter: A Runtime for MCP Tool Composition

[MCPorter](https://github.com/steipete/mcporter) is a TypeScript runtime and CLI toolkit that makes calling MCP servers practical without boilerplate — and helps reduce context bloat through selective tool exposure and typed wrappers.

**What it solves:** Instead of loading all tool definitions from all MCP servers upfront, MCPorter lets you discover, inspect, and call specific tools on demand — keeping your context lean.

**Key features:**

| Feature | Description |
|---------|-------------|
| **Zero-config discovery** | Auto-discovers MCP servers from Cursor, Claude, Codex, or local configs |
| **Typed tool clients** | `mcporter emit-ts` generates `.d.ts` interfaces and ready-to-run wrappers |
| **Composable API** | `createServerProxy()` exposes tools as camelCase methods with `.text()`, `.json()`, `.markdown()` helpers |
| **CLI generation** | `mcporter generate-cli` converts any MCP server into a standalone CLI with `--include-tools` / `--exclude-tools` filtering |
| **Parameter hiding** | Optional parameters stay hidden by default, reducing schema verbosity |

**Installation:**

```bash
npx mcporter list          # No install required — discover servers instantly
pnpm add mcporter          # Add to a project
brew install steipete/tap/mcporter  # macOS via Homebrew
```

**Example — composing tools in TypeScript:**

```typescript
import { createRuntime, createServerProxy } from "mcporter";

const runtime = await createRuntime();
const gdrive = createServerProxy(runtime, "google-drive");
const salesforce = createServerProxy(runtime, "salesforce");

// Data flows between tools without passing through the model context
const doc = await gdrive.getDocument({ documentId: "abc123" });
await salesforce.updateRecord({
  objectType: "SalesMeeting",
  recordId: "00Q5f000001abcXYZ",
  data: { Notes: doc.text() }
});
```

**Example — CLI tool call:**

```bash
# Call a specific tool directly
npx mcporter call linear.create_comment issueId:ENG-123 body:'Looks good!'

# List available servers and tools
npx mcporter list
```

MCPorter complements the code-execution approach described above by providing the runtime infrastructure for calling MCP tools as typed APIs — making it straightforward to keep intermediate data out of the model context.
