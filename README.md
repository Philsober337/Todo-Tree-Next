# Todo Tree_Next

**Todo Tree_Next** is a modern rewrite of the classic VS Code Todo Tree extension. It keeps the familiar tree, highlighting, filtering, and navigation workflow, while adding a Rust scanner, TypeScript modules, Git-aware task views, and an AI Agent interface.

The extension is published on the VS Code Marketplace as **Todo Tree_Next**.

## Highlights

| Area | What changed |
| --- | --- |
| Scanner | Rust native scanner with ripgrep fallback |
| Refresh | Fast file-level incremental scan for save/open events |
| Metadata | `P0`-`P3`, `TODO!`, `TODO?`, `@assignee`, `due:YYYY-MM-DD`, `#labels` |
| Markdown | Native `- [ ]`, `- [x]`, and numbered task support |
| Filtering | Structured queries such as `tag:TODO path:src priority:P0` |
| Git | Scan changed/staged files and export TODO debt reports |
| Dashboard | Webview dashboard with counts, charts, scanner controls, and Git actions |
| AI Agent | Structured TODO context and editor annotations for AI coding tools |

## Installation

Install from the VS Code Marketplace by searching for:

```text
Todo Tree_Next
```

You can also install a packaged `.vsix` manually:

1. Open VS Code Extensions.
2. Choose `...` > `Install from VSIX...`.
3. Select the generated `todo-tree-*.vsix` package.

## Core Workflow

Open the Todo Tree activity bar view to see TODOs grouped by file, tag, or flat list. Click an item to jump to the source line. The extension updates on workspace scans, open file scans, current file scans, and file save events.

Common commands:

```text
Todo Tree: Refresh
Todo Tree: Open Dashboard
Todo Tree: Scan Changed Files
Todo Tree: Scan Staged Files
Todo Tree: Export TODO Debt Report
Todo Tree: Get Agent TODO Context
Todo Tree: Clear Agent Annotations
```

## Smart Filtering

The tree filter supports plain text and field queries:

```text
auth
tag:TODO
path:src
file:README.md
text:refactor
priority:P0
status:open
tag:FIXME path:src priority:P1
```

Supported fields:

| Field | Matches |
| --- | --- |
| `tag` | `TODO`, `FIXME`, `BUG`, `[ ]`, `[x]`, custom tags |
| `path` | Full file path |
| `file` | File name only |
| `text` | TODO text |
| `priority` | `P0`, `P1`, `P2`, `P3`, `none` |
| `status` | Markdown task status: `open` or `done` |

## Priority And Metadata

```javascript
// TODO:P0 fix auth bug @alice due:2026-06-01 #security
// FIXME:P1 memory leak @bob #backend
// TODO! urgent task      -> P0
// TODO? needs discussion -> P2
```

The Rust scanner extracts metadata into structured fields so the tree, dashboard, export, and AI Agent interface can reason about task urgency and ownership.

## Dashboard

Open with:

```text
Todo Tree: Open Dashboard
```

The dashboard provides:

- Total TODO count and tag distribution
- Priority distribution
- Tag charts and trend chart
- Scanner engine switch: `auto`, `rust`, `ripgrep`
- Scan mode controls
- Max file size setting
- Smart filter input
- Git changed/staged scan actions

## Git Integration

```text
Todo Tree: Scan Changed Files
Todo Tree: Scan Staged Files
Todo Tree: Export TODO Debt Report
```

Git-aware scans help focus on TODOs introduced or touched by the current branch. The debt report compares the current branch against a base branch and summarizes added/removed TODOs.

## AI Agent Interface

Todo Tree_Next exposes TODO debt as structured data for AI coding tools. Agents can read ranked TODO context and write temporary editor annotations back into VS Code.

VS Code command API:

```javascript
const context = await vscode.commands.executeCommand('todo-tree.getAgentContext');

await vscode.commands.executeCommand('todo-tree.annotateAgentFinding', {
  file: 'src/auth.ts',
  line: 42,
  column: 5,
  severity: 'warning',
  message: 'P0 TODO touches authentication code; review before merge.'
});

await vscode.commands.executeCommand('todo-tree.clearAgentAnnotations');
```

Rust CLI:

```bash
todo-scanner agent-context --root . --config todo-scanner-config.json
```

The Agent context includes file path, line/column, tag, priority, assignee, due date, labels, Git status, approximate age, code context snippet, recommended action, and recommended processing order.

See [docs/AGENT_INTERFACE.md](docs/AGENT_INTERFACE.md) for the full schema.

## Architecture

```text
VS Code extension
  src/extension.js          entry point and legacy glue
  src/scannerClient.ts      Rust CLI JSON protocol
  src/agentInterface.ts     AI Agent context and annotations
  src/dashboard.ts          Webview dashboard
  src/tree.ts               Tree data provider
  src/filterQuery.ts        Structured filter parser
  src/gitScanner.ts         Git changed/staged scan
  src/debtReport.ts         Git TODO debt report
  src/fileWatcher.ts        File event handling
  src/statusBar.ts          Status bar and activity badge

Rust scanner
  scanner/src/main.rs       scan-workspace, scan-file, agent-context, benchmark
  scanner/src/walker.rs     .gitignore-aware traversal
  scanner/src/matcher.rs    TODO matching and metadata extraction
  scanner/src/output.rs     JSON output schema
```

## Development

Requirements:

- Node.js
- Rust toolchain
- VS Code

```bash
npm install
npm run scanner:build
npm run webpack
npm test
cargo test --manifest-path scanner/Cargo.toml
```

Package a VSIX:

```bash
npm run vscode:prepublish
npx --yes @vscode/vsce package
```

Current test coverage:

| Type | Count |
| --- | ---: |
| QUnit tests | 97 |
| Rust tests | 38 |
| Total | 135 |

## Configuration

```json
{
  "todo-tree.scanner.engine": "auto",
  "todo-tree.scanner.path": "",
  "todo-tree.scanner.maxFileSize": 1048576
}
```

| Value | Behavior |
| --- | --- |
| `auto` | Use Rust scanner when available, fallback to ripgrep |
| `rust` | Force Rust scanner |
| `ripgrep` | Use the original ripgrep scanner |

## Documentation

- [Rewrite notes](docs/REWRITE.md)
- [Feature compatibility](docs/COMPATIBILITY.md)
- [AI Agent interface](docs/AGENT_INTERFACE.md)
- [Benchmark report](docs/BENCHMARK.md)

## License

MIT. Based on the original [Todo Tree](https://github.com/Gruntfuggly/todo-tree) extension by Gruntfuggly.
