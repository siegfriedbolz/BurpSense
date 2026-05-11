<h1 align="center">BurpSense for Cursor</h1>
<div align="center">Burp Suite integration for VS Code and Cursor</div>

<div align="center">

[![Build and Publish Release](https://github.com/siegfriedbolz/BurpSense/actions/workflows/release.yml/badge.svg)](https://github.com/siegfriedbolz/BurpSense/actions/workflows/release.yml)

[![Visual Studio Marketplace Version](https://img.shields.io/visual-studio-marketplace/v/siegfriedbolz.burpsense-cursor?include_prereleases&style=flat-square&label=Visual%20Studio%20Marketplace&color=blue)](https://marketplace.visualstudio.com/items?itemName=siegfriedbolz.burpsense-cursor)
[![Open VSX Version](https://img.shields.io/open-vsx/v/siegfriedbolz/burpsense-cursor?style=flat-square&label=Open%20VSX%20Marketplace)](https://open-vsx.org/extension/siegfriedbolz/burpsense-cursor)

</div>

BurpSense bridges security testing in **Burp Suite** and your editor: map findings to **lines of source code**, with inline diagnostics, full advisories, and persistent mappings.

This repository is a **monorepo**: the **Java Burp bridge** lives under [`burp-bridge/`](burp-bridge/), the **VS Code / Cursor extension** under [`vscode-extension/`](vscode-extension/). For command palette, settings keys, troubleshooting, and marketplace notes specific to the editor extension, see **[vscode-extension/README.md](vscode-extension/README.md)**.

## Fork, upstream, and compatibility

**BurpSense for Cursor** is a **fork** of the original **BurpSense** by **Arqsz** ([TheArqsz/BurpSense](https://github.com/TheArqsz/BurpSense); Marketplace **`arqsz.burpsense`**). **Siegfried-Thor Bolz** maintains this fork ([siegfriedbolz/BurpSense](https://github.com/siegfriedbolz/BurpSense)) so the **bridge** and **extension** stay aligned with **Cursor 3** (reference: **3.3.30**, embedded VS Code engine **1.105.1**) and **Burp Suite v2026** (reference: **v2026.4.2**), including **Montoya** `montoya-api` **2026.4** for the Java extension. The published editor package declares **`engines.vscode` `^1.105.0`** so it installs on current **Cursor** and **VS Code 1.105+**. Use recent Cursor, VS Code, and Burp updates for the best-supported experience.

![BurpSense main view](assets/main_view.png)

## What this solves?

If you've done web application security testing, you've probably experienced the friction of correlating Burp Suite findings with actual source code. You find an SQL injection, note the URL and parameter, then hunt through your codebase to figure out where that endpoint lives. Then you need to remember which file and line to fix, or worse, communicate this to someone else on your team who wasn't sitting next to you during testing.

Burp Suite does have a built-in API integration feature, but if you've tried using it, you know it's... let's say "minimalist".

![Burp API](assets/burp_off_api.png)

The extension eliminates most of this friction. The Burp bridge exposes scan results through a clean local API and the VS Code extension consumes them in real-time. You get a live view of all findings and you can annotate specific lines of code with issues. These annotations persist in your workspace, so you can pick up where you left off without losing context.

## How it works?

The project has two components:

The **Burp bridge** is a Java extension that runs inside Burp Suite. It starts an HTTP server on `localhost` (default port `1337`) and exposes your scan results through a REST API. It also maintains WebSocket connections to push updates when new issues appear or existing ones get removed. The bridge applies your filters (severity, confidence, scope) server-side, so only relevant issues get sent to clients.

The **VS Code / Cursor extension** connects to this bridge and displays issues in a dedicated tree view. You can browse, search, and filter findings, then map them to specific lines in your source files. Mapped issues appear as diagnostics in the Problems panel, and clicking on one opens a detailed advisory with full HTTP request/response data. When you refactor code and lines move around, the extension attempts to track those changes and adjust mappings automatically.

Communication happens over HTTP for queries and WebSocket for real-time updates. The protocol is differential - after the initial sync, the bridge only sends what's changed (new issues, removed issues) rather than the entire dataset each time.

## Requirements

**Editor extension (VS Code or Cursor)**

1. **Cursor 3** (reference **3.3.30**, engine **1.105.1**) or **VS Code 1.105+** — the published extension uses `engines.vscode` **^1.105.0**.
2. **Burp Suite v2026.4.2** (or another build on the **2026.4** line; Professional or Community) with the Montoya API, plus the **BurpSense Bridge** JAR from [Releases](https://github.com/siegfriedbolz/BurpSense/releases) or built from `burp-bridge/` (below).
3. Node.js **18+** if you build the extension from source.

**Burp bridge (Java)**

- Same Burp version line as above; the bridge is compiled against `montoya-api` **2026.4** (Maven Central).
- **Java 21+** and **Maven 3.6+** to build from source.

### Manual smoke check (bridge)

After loading the bridge JAR in Burp: open **BurpSense Bridge Settings**, start the server, then `GET http://<host>:<port>/health`. With issues in the site map, call `GET /issues` (with your API key if configured). Integration coverage: `mvn verify` in `burp-bridge/`.

### Marketplace icon (published extension)

Registries expect a **128×128 PNG** at [`vscode-extension/images/icon.png`](vscode-extension/images/icon.png). List/search thumbnails in **Cursor** / **Open VSX** / **Visual Studio Marketplace** come from store metadata after you **publish a new version** — not from a locally installed VSIX alone. Reload the editor after an update if the icon is cached.

## Installation and setup

### Bridge extension (Burp Suite side)

![the Bridge](assets/bridge.png)

Start by building the bridge:

```bash
cd burp-bridge
mvn clean package
```

This produces `target/burpsense-bridge-*.jar`. Load it in Burp Suite through the Extensions tab (click "Add" and select the JAR). After loading, you'll see a new "BurpSense Bridge Settings" tab appear.

Generate an API token using the "Generate New Key" button and copy it somewhere safe. Then click "Start Server" to begin listening. By default, the server binds to `127.0.0.1:1337`, but you can change this in the settings panel if needed.

> The bridge requires Burp Suite with the Montoya extension API (current **2026.4** Burp line).

### VS Code / Cursor extension

Install from [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=siegfriedbolz.burpsense-cursor), [Open VSX](https://open-vsx.org/extension/siegfriedbolz/burpsense-cursor), a **VSIX** from [GitHub Releases](https://github.com/siegfriedbolz/BurpSense/releases), or build from source.

**Package a VSIX locally (same as CI `vsix` script):**

```bash
cd vscode-extension
npm ci
npm run vsix
```

This produces `burpsense-cursor-*.vsix`. In **VS Code** or **Cursor**, Extensions (`Ctrl+Shift+X` / `Cmd+Shift+X`) → **⋯** → **Install from VSIX**.

**Equivalent without the npm script:**

```bash
cd vscode-extension
npm install
npm run compile
npx --yes @vscode/vsce package
```

**From source (for development):**

```bash
cd vscode-extension
npm install
npm run compile
```

Then press `F5` in VS Code to launch the Extension Development Host with the extension loaded and debugger attached.

### Connecting the two

After installing both components, run **BurpSense: Set API Token** from the command palette in **VS Code** or **Cursor** and paste the token from Burp. You should see `BurpSense: Connected [X issues]` in the status bar; if not, click the status bar for diagnostics.

> **First-time users:** A welcome panel and walkthrough are available; see [vscode-extension/README.md](vscode-extension/README.md) for commands, settings, and troubleshooting.

## Using the extension

### Browsing issues

The BurpSense view appears in the activity bar (the shield icon on the left side). Opening it shows a tree of issues organized by severity. Each issue displays its name, confidence level, and affected URL. The tree uses lazy loading, so it stays responsive even when you have hundreds of findings.

You can filter issues in several ways:

- Text search (searches names, URLs and issue IDs)
- Minimum severity (high, medium, low, information)
- Minimum confidence (certain, firm, tentative)
- Scope filter (show only in-scope issues)

The toolbar icons let you adjust these filters on the fly. There's also a "quick filter" menu that applies common presets like "high severity only" or "in-scope high and medium".

### Mapping issues to code

To map an issue to a line of code:

1. Open the file and position your cursor on the relevant line
2. Right-click and select `BurpSense: Map Burp Issue to this Line`

![context menu](assets/context_menu.png)

3. Choose the issue from the dropdown

![select issues](assets/issue_select.png)

The mapping gets saved to `.burpsense/mappings.json` in your workspace. This file can be committed to version control so team members see the same annotations.

Mapped issues appear in two places: the Problems panel (with appropriate severity icons) and directly in the editor as squiggly underlines. Click any diagnostic to open a detailed advisory showing the full issue description, remediation advice, and HTTP traffic if available.

When mapping, the extension analyzes your code context and suggests relevant issues. For example, if the line contains SQL keywords like `SELECT` or `WHERE`, it prioritizes SQL injection issues. This works for common patterns like file operations, authentication code, XSS-prone templates, and API endpoints.

![Problems tab](assets/problems_tab.png)

![advisory](assets/advisory.png)

### Drift detection

![Drift](assets/drift.png)

When you refactor code, lines move around. BurpSense tries to keep mappings accurate by checking whether the text you originally mapped still exists nearby. When you edit a file with mappings, the extension:

1. Checks if the exact text still appears on the mapped line
2. If not, searches within +/- 20 lines for similar content (>80% match)
3. Calculates similarity using character-level comparison with position weighting
4. Updates the line number if it finds a good match

This works well for typical refactoring like adding functions or moving code blocks. It won't track cases where you completely rewrite the vulnerable code, but that's intentional - if the code changed that much, the original issue may no longer apply and you should verify it manually.

The extension can notify you when it adjusts mappings, though you can disable these notifications in settings if they're distracting.

### Removing mappings

Remove a single mapping by right-clicking the line and selecting `BurpSense: Remove Mapping from this Line`. To remove all mappings for a particular issue, hover over it in the tree view and click the trashbin icon. For bulk removal, use `BurpSense: Remove multiple Mappings` from the command palette.

### Sharing mappings with your team

The mappings file (`.burpsense/mappings.json`) is workspace-specific and can be committed to your repository. This means when a teammate clones the project and connects their own VS Code to a shared Burp Project, they'll see the same annotations you created. They'll need their own bridge API token, but the mappings themselves are just file paths and line numbers with issue IDs.

You can also export mappings to a standalone JSON file and import them in a different workspace if needed with dedicated commands.

## Configuration

Settings live under **BurpSense for Cursor** in the editor preferences (namespace `burpsense-cursor.*`):

```json
{
  "burpsense-cursor.bridgeIp": "127.0.0.1",
  "burpsense-cursor.bridgePort": 1337,
  "burpsense-cursor.inScopeOnly": true,
  "burpsense-cursor.minSeverity": "INFORMATION",
  "burpsense-cursor.minConfidence": "TENTATIVE",
  "burpsense-cursor.showDriftNotifications": true,
  "burpsense-cursor.confirmMappingDeletion": true,
  "burpsense-cursor.autoCleanOrphanedMappings": false,
  "burpsense-cursor.logLevel": "info"
}
```

The IP and port should match where your bridge is listening. If you're running Burp on a different machine, adjust accordingly. The extension will reconnect automatically when these settings change.

Filter settings (severity, confidence, scope) apply server-side, which means the bridge only sends issues matching your criteria. Changing filters triggers a refetch, so there might be a brief delay while the new filtered set loads.

Drift notifications can be disabled if you find them noisy. The extension will still adjust mappings, just without notifying you.

Auto-clean orphaned mappings, when enabled, removes mappings for files that get deleted. This is disabled by default because you might temporarily remove files during refactoring and want the mappings to persist.

The logLevel setting controls logging verbosity. Set to error for minimal logs, warn for warnings only, info for normal operation, or debug for detailed troubleshooting output.

## Bridge API Reference

The Burp bridge exposes the following REST endpoints:

- `GET /health`

Health check endpoint. Returns `{"status":"ok"}` when the server is running.

Response: 

```json
{"status":"ok"}
```

- `GET /issues`

Fetches all issues matching current filter criteria (severity, confidence, scope).

Headers:

- `Authorization: Bearer <token>`

Response: 

```json
[
  {
    "id": "unique-issue-id",
    "name": "SQL injection",
    "severity": "HIGH",
    "confidence": "CERTAIN",
    "host": "example.com",
    "path": "/api/users",
    ...
  }
]
```

- `POST /issues`

Differential sync endpoint. Send known issue IDs to receive only changes since last fetch.

Headers:

- `Authorization: Bearer <token>`
- `Content-Type: application/json`

Body:

```json
{
  "knownIds": ["id1", "id2", "id3"]
}
```

Response: 

```json
{
  "newIssues": [...],
  "removedIds": ["id2"]
}
```

- `GET /issues/{id}`

Fetches a single issue by ID with full details including HTTP messages.

Headers:

- `Authorization: Bearer <token>`

Response: `200 OK` or `404 Not Found`

- `GET /ws`

WebSocket endpoint for real-time issue updates. Server broadcasts "refresh" message when issue count changes.

Headers:

- `Upgrade: websocket`
- `Authorization: Bearer <token>`

### Security features

The bridge includes several security hardening measures:

- **Rate Limiting**: Fixed-window rate limiter protects against brute force attacks on the API token. Default: 100 requests per 60-second window. Rate limit headers (`X-RateLimit-Limit`, `X-RateLimit-Remaining`) are included in responses.
- **CORS**: Cross-Origin Resource Sharing middleware with `localhost`/`127.0.0.1` defaults. This prevents unauthorized web pages from accessing the bridge API.
- **Constant-time token comparison**: Authentication uses constant-time comparison to prevent timing attacks on API tokens.
- **Localhost binding**: By default, the server only listens on `127.0.0.1`, preventing external network access. Change this setting with caution.

## Technical details

### Synchronization protocol

The extension uses a hybrid push/pull model. On initial load, it fetches all issues via `GET /issues`. Subsequent requests to `POST /issues` include the IDs of issues the client already knows about, and the bridge responds with:

```json
{
  "newIssues": [...],
  "removedIds": [...]
}
```

This differential sync significantly reduces network traffic during active scanning sessions where issues frequently appear and disappear.

WebSocket connections handle real-time updates. The bridge monitors its issue list every 2 seconds and broadcasts a refresh message when the count changes. Clients respond by fetching an incremental update.

### Caching strategy

Issues are cached locally with a 30 second TTL. After expiration, the tree view fetches fresh data the next time you expand a node. This avoids hammering the bridge with requests while keeping the display reasonably current.

The cache has two levels:

- Issue-level cache (raw scan data)
- Grouped data cache (tree nodes organized by severity/name)

When an incremental update arrives, both caches are invalidated, but the tree doesn't refetch unless something actually changed. This keeps the UI responsive even during heavy scanning activity.

### Performance characteristics

The initial load with several hundred issues can take a few seconds while the tree builds its internal structure. After that, navigation is fast because the tree uses lazy loading - child nodes aren't created until you expand their parent.

The extension processes incremental updates synchronously, which means if you have a large removal event (like changing filters to exclude 500 issues), there might be a noticeable pause. This is a trade-off for keeping the sync logic simple.

## Known quirks and limitations

### Manual disconnect behavior

If you manually disconnect using the status bar, the extension won't fetch data for any operation, including opening issue details. This is intentional - when you're disconnected, you're working with stale cache only. Reconnecting re-enables all fetches.

### Cache invalidation timing

When the cache expires (after 30 seconds), the extension doesn't immediately refetch. It waits until you next interact with the tree view. This is by design to avoid background fetches when you're not actively looking at the extension.

### Large issue sets

With 1000+ issues, the tree can feel sluggish. This is partly because VS Code's tree view API isn't optimized for that many items, and partly because the grouping logic has to sort and organize everything. Consider using stricter filters to reduce the working set.

### WebSocket reconnection gap

If the Burp bridge restarts, the WebSocket takes a few seconds to reconnect. During this window, you won't receive real-time updates, though the extension continues working with cached data. Once the connection re-establishes, you'll get an incremental sync of anything that changed.

### Mapping file format

The mappings file is plain JSON and gets rewritten entirely on every change. This means if two people edit mappings simultaneously, the last write wins. Git can handle this in most cases, but be aware that mappings don't merge intelligently across branches.

## Troubleshooting

### Status bar shows "Not Connected"

Work through these in order:

1. Is Burp Suite running?
2. Is the bridge extension loaded? (Check Extensions tab in Burp)
3. Is the server started? (Check Bridge Settings tab)
4. Does your API token match between Burp and VS Code?
5. Is something blocking set port? (Firewall, another service)

The status bar is clickable and opens a quick actions menu with diagnostics. Check the VS Code output panel (`BurpSense: Show Logs` command) for connection logs.

### Issues aren't showing up

First, verify the issues actually exist in Burp's sitemap. Then check your filter settings - you might have the minimum severity set too high, or scope filtering enabled when the issues aren't in scope. Try clicking the refresh button in the tree view toolbar. If that doesn't work, check the output panel for errors.

### Mappings not appearing as diagnostics

Make sure the file path in the mappings matches your workspace structure exactly. The extension uses paths relative to the workspace root, so if your workspace root isn't what you expect, mappings might not resolve. Check `.burpsense/mappings.json` to see what paths are stored, and verify they match your actual file locations.

### Drift detection moving mappings incorrectly

The similarity algorithm is conservative (70% match threshold), but false positives can still happen if you have very similar code repeated in nearby lines. If drift detection is causing problems, you can turn off notifications in settings. The extension will still adjust mappings, but you can check the Problems panel to verify they're still pointing to the right code.

## Development and contributing

To build from source, you need Maven for the bridge and Node.js for the extension.

### Bridge

```bash
cd burp-bridge
mvn clean package
```

Output is in `target/burpsense-bridge-*.jar`. Load this in Burp's Extensions tab.

To run tests:

```
# Unit tests only
mvn test

# Unit + integration tests
mvn verify

# Only integration tests
mvn verify -DskipUnitTests

# With coverage report
mvn clean verify
```

The project maintains 70% line coverage. Tests use JUnit 5, Mockito and REST Assured.

### Extension

```bash
cd vscode-extension
npm install
npm run compile
```

For active development, use `npm run watch` to recompile automatically when you change files. Press F5 in VS Code to launch the Extension Development Host with debugger attached.

Code quality:

```
# Type check
npm run check-types

# Lint
npm run lint

# Watch mode (type check + compile)
npm run watch
```

Issues and pull requests are welcome. For major changes, please open an issue first to discuss the approach. When reporting bugs, include VS Code and Burp Suite versions, extension version, and logs from both components (VS Code output panel and Burp's extension output tab).

## Project structure

```
burpsense/
├── burp-bridge/
│   └── src/main/java/com/arqsz/burpsense/
│       ├── api/              # HTTP endpoints and handlers
│       │   ├── handler/      # Health, issues, single issue endpoints
│       │   └── middleware/   # Auth, CORS, rate limiting
│       ├── service/          # Issue filtering and management
│       ├── config/           # Settings persistence and key storage
│       ├── constants/        # Server, security, and UI constants
│       ├── ui/               # Bridge settings panel and components
│       └── util/             # JSON mapping and validation
└── vscode-extension/
    └── src/
        ├── commands/         # Command palette handlers
        ├── events/           # Connection state change events
        ├── providers/        # Tree view, diagnostics, code actions
        ├── services/         # Connection, caching, mapping, smart suggestions
        ├── ui/               # Webview panels for advisories and welcome
        └── extension.ts      # Entry point and activation
```

## Security and privacy

Traffic between the editor and the bridge should stay on **localhost** (`127.0.0.1` by default). API tokens are stored in the editor’s secret storage. No third-party telemetry is sent by this project; mappings are local path/line/issue data. See also the **Security & Privacy** section in [vscode-extension/README.md](vscode-extension/README.md).

## License

[MIT](LICENSE.md)

## Acknowledgments

- **Arqsz** — original **BurpSense** ([TheArqsz/BurpSense](https://github.com/TheArqsz/BurpSense); Marketplace **`arqsz.burpsense`**).
- **Siegfried-Thor Bolz** — fork maintainer: **Cursor 3** / **Burp Suite v2026** compatibility, packaging (`npm run vsix`), and documentation in [siegfriedbolz/BurpSense](https://github.com/siegfriedbolz/BurpSense).

Built with Burp Suite’s Montoya API, Undertow (bridge HTTP/WebSocket), `ws` / `fetch` in the extension, and **sanitize-html** for advisory webviews.

## Extension README

Full extension-only reference (commands, settings list, marketplace icon notes): **[vscode-extension/README.md](vscode-extension/README.md)**.
