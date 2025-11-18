# Carnival Network GraphQL - Warp AI Context

## Project Overview

**carnival-network-graphql** is a companion Obsidian plugin that provides a GraphQL interface for the [carnival-network](../carnival-network) REST API.

### Architecture Philosophy
- **Zero coupling**: Never imports carnival-network internals
- **Pure REST client**: All data fetched via HTTP from `localhost:27123/api/*`
- **Thin wrapper**: GraphQL queries translate directly to REST calls
- **Optional installation**: Users install only if they need GraphQL

## Key Files

```
carnival-network-graphql/
├── src/
│   ├── main.ts                  # Plugin entry + GraphQL endpoint registration
│   ├── graphql/
│   │   ├── schema.ts           # GraphQL schema definition
│   │   ├── resolvers.ts        # Query/Mutation resolvers (call REST)
│   │   └── types.ts            # TypeScript type definitions
│   ├── rest/
│   │   ├── client.ts           # REST API client wrapper
│   │   └── types.ts            # REST response types
│   └── utils/
│       ├── logger.ts           # Logging utilities
│       └── errors.ts           # Error handling
├── .warp/
│   ├── IMPLEMENTATION-PLAN.md  # Full implementation roadmap
│   └── conversations/          # AI session archives
├── manifest.json               # Plugin metadata + dependencies
└── package.json                # Dependencies: graphql, graphql-http
```

## Development Commands

```bash
# Install dependencies
npm install

# Build (production)
npm run build

# Watch mode (development)
npm run dev

# Lint
npm run lint

# Clean build artifacts
npm run clean
```

## Testing

```bash
# Run GraphQL test queries
.warp/scripts/test-queries.sh

# Manual testing with curl
curl -X POST http://localhost:27123/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ networkStatus { health } }"}'
```

## Dependencies

### Required Plugins
1. **carnival-network** - Provides the REST API this plugin wraps
2. **obsidian-local-rest-api** - HTTP server for endpoint registration

### npm Dependencies
- `graphql` - GraphQL query parsing and execution
- `graphql-http` - HTTP handler for GraphQL endpoint
- `obsidian` - Obsidian API (dev dependency)
- `typescript` - TypeScript compiler

## Implementation Notes

### How GraphQL → REST Translation Works

1. **Client sends GraphQL query** to `POST /graphql`
2. **graphql-http parses** the query into AST
3. **Resolvers execute** for each field:
   ```typescript
   records: async (_, args, context) => {
     // Call REST API
     const response = await fetch(
       `http://localhost:27123/api/records?limit=${args.limit}`
     );
     return response.json();
   }
   ```
4. **GraphQL formats** response according to schema

### REST Endpoints Used
- `GET /api/records` - Query records with pagination
- `POST /api/records` - Create new record
- `GET /api/records/:id` - Get specific record
- `POST /api/search` - Search records
- `GET /api/network/status` - Network health
- `GET /api/territories` - Territory list
- `GET /api/analytics` - Analytics data

## Common Tasks

### Adding a New GraphQL Query

1. **Add to schema** (`src/graphql/schema.ts`):
   ```graphql
   type Query {
     myNewQuery(arg: String!): MyResult!
   }
   ```

2. **Add resolver** (`src/graphql/resolvers.ts`):
   ```typescript
   Query: {
     myNewQuery: async (_, { arg }, context) => {
       const response = await context.restClient.fetch(
         `/api/new-endpoint?arg=${arg}`
       );
       return response.data;
     }
   }
   ```

3. **Add REST client method** (`src/rest/client.ts`):
   ```typescript
   async callNewEndpoint(arg: string): Promise<MyResult> {
     return this.fetch(`/api/new-endpoint?arg=${arg}`);
   }
   ```

### Debugging GraphQL Queries

1. Check console logs in Obsidian Developer Tools
2. Use test script: `.warp/test-queries.sh`
3. Test REST API directly first to isolate issues
4. Use GraphQL client tools (Postman, Altair, etc.)

## Session Documentation

At the end of each development session:

1. **Create conversation archive**:
   ```bash
   cp .warp/CONVERSATION-template.md \
      .warp/conversations/YYYY-MM-DD-session-name.md
   ```

2. **Document**:
   - What was implemented
   - Decisions made
   - Next steps
   - Any blockers

3. **Commit changes**:
   ```bash
   git add .
   git commit -m "Implement: [feature description]"
   ```

## Project Status

Refer to `.warp/IMPLEMENTATION-PLAN.md` for:
- Phase-by-phase implementation roadmap
- Deliverables checklist
- Estimated timeline (~14-21 hours total)

Current phase progress tracked in conversation archives.

## Related Projects

- **carnival-network** - The REST API this wraps (sibling directory)
- **carnival-records** - Record management UI plugin
- **obsidian-local-rest-api** - HTTP server dependency

## Constraints & Best Practices

### Never Do
- ❌ Import carnival-network source files
- ❌ Directly access carnival-network's services
- ❌ Duplicate business logic from carnival-network
- ❌ Cache data (let REST API handle it)

### Always Do
- ✅ Call REST API for all data
- ✅ Handle network errors gracefully
- ✅ Validate GraphQL inputs
- ✅ Document schema changes
- ✅ Test with real carnival-network running

### Plugin Size
Target: Keep bundle < 500KB gzipped
- GraphQL dependencies add ~2MB
- This is acceptable since plugin is optional
- carnival-network stays lean (~200KB)

## Troubleshooting

### "Carnival Network plugin required" error
- Ensure carnival-network is installed and enabled
- Check Settings → Community Plugins

### "Local REST API plugin required" error
- Install obsidian-local-rest-api from Community Plugins
- Verify it's running on port 27123

### GraphQL endpoint not responding
1. Check if carnival-network REST API is running:
   ```bash
   curl http://localhost:27123/api/network/status
   ```
2. Check Obsidian console for errors
3. Verify plugin is enabled

### REST API connection refused
- Ensure Obsidian is running
- Verify local-rest-api is enabled
- Check firewall settings

## AI Agent Guidelines

### Project-Specific Directives

When working on this project:

1. **Reference REST API first**: Check carnival-network's REST API before implementing
2. **No tight coupling**: Never suggest importing carnival-network internals
3. **Test REST separately**: Verify REST endpoints work before debugging GraphQL
4. **Follow schema**: Always match GraphQL types to REST response types
5. **Document sessions**: Use `.warp/CONVERSATION-template.md` at session end

### Obsidian Plugin Development Guidelines

#### Environment & Tooling
- **Package manager**: npm (required - `package.json` defines scripts and dependencies)
- **Bundler**: esbuild (required - see `esbuild.config.mjs`)
- TypeScript with `"strict": true` preferred
- Node.js 18+ LTS recommended

#### File Organization & Architecture
- **Keep `main.ts` minimal**: Focus only on plugin lifecycle (onload, onunload, addCommand calls)
- **Split large files**: If any file exceeds ~200-300 lines, break into smaller modules
- **Use clear module boundaries**: Each file should have a single, well-defined responsibility
- Source lives in `src/` - organize by feature/domain:
  ```
  src/
    main.ts           # Plugin entry point, lifecycle only
    graphql/          # GraphQL schema, resolvers, types
    rest/             # REST API client
    utils/            # Utility functions, helpers
  ```

#### Build & Release
- Bundle everything into `main.js` (no unbundled runtime deps)
- **Never commit build artifacts**: Don't commit `node_modules/`, `main.js`, or generated files
- Release artifacts must be at plugin root: `main.js`, `manifest.json`, `styles.css`
- Manual test install: copy artifacts to `<Vault>/.obsidian/plugins/<plugin-id>/`

#### Manifest Rules (`manifest.json`)
- Required fields: `id`, `name`, `version` (SemVer), `minAppVersion`, `description`, `isDesktopOnly`
- **Never change `id` after release** - treat as stable API
- For this plugin: include `dependencies: ["carnival-network", "obsidian-local-rest-api"]`
- Keep `minAppVersion` accurate when using newer APIs

#### Commands & Settings
- Add commands via `this.addCommand(...)` with stable IDs
- **Never rename command IDs once released**
- Provide settings tab with sensible defaults if configurable
- Persist settings using `this.loadData()` / `this.saveData()`

#### Security & Privacy
- Default to local/offline operation
- Only make network requests when essential (this plugin: localhost only)
- No hidden telemetry - require explicit opt-in if collecting analytics
- Never execute remote code or auto-update outside normal releases
- Minimize scope: read/write only what's necessary
- Register and clean up all listeners using `this.register*` helpers

#### Performance
- Keep startup light - defer heavy work until needed
- Avoid long-running tasks during `onload` - use lazy initialization
- Batch disk access and avoid excessive vault scans
- Debounce/throttle expensive operations

#### Mobile Compatibility
- Test on iOS and Android where feasible
- Don't assume desktop-only behavior unless `isDesktopOnly: true`
- Avoid Node/Electron APIs for mobile compatibility
- Be mindful of memory and storage constraints

#### Coding Conventions
- Prefer `async/await` over promise chains
- Handle errors gracefully
- Use `this.register*` helpers for everything needing cleanup
- Write idempotent code - reload/unload shouldn't leak listeners

#### Agent Do/Don't

**Do:**
- Add commands with stable IDs (don't rename once released)
- Provide defaults and validation in settings
- Use `this.register*` helpers for everything that needs cleanup
- Split functionality across separate modules
- Keep `main.ts` focused on lifecycle only

**Don't:**
- Introduce network calls without obvious user-facing reason
- Ship features requiring cloud services without disclosure and opt-in
- Store or transmit vault contents unless essential and consented
- Put all code in `main.ts` - organize into focused modules
- Commit build artifacts (`main.js`, `node_modules/`)

#### Common Patterns

**Register listeners safely:**
```typescript
this.registerEvent(this.app.workspace.on("file-open", f => { /* ... */ }));
this.registerDomEvent(window, "resize", () => { /* ... */ });
this.registerInterval(window.setInterval(() => { /* ... */ }, 1000));
```

**Persist settings:**
```typescript
interface MySettings { enabled: boolean }
const DEFAULT_SETTINGS: MySettings = { enabled: true };

async onload() {
  this.settings = Object.assign({}, DEFAULT_SETTINGS, await this.loadData());
  await this.saveData(this.settings);
}
```

**Organize modular code:**
```typescript
// main.ts (minimal, lifecycle only)
import { Plugin } from "obsidian";
import { registerGraphQLEndpoint } from "./graphql";

export default class MyPlugin extends Plugin {
  async onload() {
    await registerGraphQLEndpoint(this);
  }
}
```

#### References
- Obsidian API: https://docs.obsidian.md
- Developer policies: https://docs.obsidian.md/Developer+policies
- Plugin guidelines: https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines
- Sample plugin: https://github.com/obsidianmd/obsidian-sample-plugin
