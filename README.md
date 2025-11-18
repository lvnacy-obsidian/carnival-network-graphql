# Carnival Network GraphQL

> GraphQL interface for the [Carnival Network](../carnival-network) REST API

## Overview

**carnival-network-graphql** is a lightweight Obsidian plugin that provides a GraphQL endpoint for querying and managing Carnival Network data. It's a thin wrapper around the carnival-network REST API, offering GraphQL's flexibility without adding bloat to the core plugin.

## Features

- 🎯 **Clean GraphQL interface** for all Carnival Network operations
- 🔌 **Zero coupling** to carnival-network internals
- 📦 **Optional installation** - install only if you need GraphQL
- 🚀 **Full API coverage** - records, search, network status, analytics
- 🧪 **Easy testing** with included test scripts
- 📚 **Auto-documented** via GraphQL introspection

## Installation

### Prerequisites

1. **carnival-network plugin** - Must be installed and running
2. **obsidian-local-rest-api plugin** - Required for HTTP endpoints

### Install Steps

1. Copy this folder to `.obsidian/plugins/carnival-network-graphql/`
2. Reload Obsidian
3. Enable in Settings → Community Plugins → carnival-network-graphql

### Verify Installation

```bash
curl http://localhost:27123/graphql \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"query": "{ networkStatus { health } }"}'
```

## Usage

### GraphQL Endpoint

```
POST http://localhost:27123/graphql
```

### Example Queries

#### Query Records with Pagination

```graphql
{
  records(territory: "backstage", limit: 10, offset: 0) {
    records {
      id
      title
      territory
      type
      content
      createdAt
      status
    }
    pagination {
      total
      hasNext
      hasPrevious
    }
    timestamp
  }
}
```

#### Get Single Record

```graphql
{
  record(id: "abc123") {
    id
    title
    territory
    content
    metadata
    createdAt
    updatedAt
  }
}
```

#### Search Records

```graphql
{
  search(query: "performance", territories: ["backstage", "necropolis"], limit: 5) {
    query
    resultCount
    hasMore
    results {
      record {
        id
        title
        territory
      }
      matchedFields
      relevance
    }
  }
}
```

#### Network Status

```graphql
{
  networkStatus {
    health
    uptime {
      milliseconds
      formatted
    }
    network {
      totalPerformers
      connectedPerformers
      territories
      activeRegistries
    }
    capabilities
  }
}
```

#### List Territories

```graphql
{
  territories {
    name
    performerCount
    status
  }
}
```

#### Get Analytics

```graphql
{
  analytics(metrics: [RECORDS, ACTIVITY, CAPABILITIES], timeframe: "7d") {
    timeframe
    records {
      # Record metrics data
    }
    activity {
      # Activity metrics data
    }
    capabilities
    timestamp
  }
}
```

### Mutations

#### Create Record

```graphql
mutation CreateRecord($input: CreateRecordInput!) {
  createRecord(input: $input) {
    record {
      id
      title
      territory
      createdAt
    }
    message
    timestamp
  }
}
```

**Variables:**
```json
{
  "input": {
    "title": "New Feature Deployed",
    "territory": "backstage",
    "type": "changelog",
    "content": "Deployed GraphQL interface for Carnival Network",
    "broadcast": true
  }
}
```

### Using Variables

```bash
curl -X POST http://localhost:27123/graphql \
  -H "Content-Type: application/json" \
  -d '{
    "query": "query SearchRecords($q: String!, $limit: Int) { search(query: $q, limit: $limit) { resultCount results { record { title } } } }",
    "variables": {
      "q": "test",
      "limit": 5
    }
  }'
```

## GraphQL Clients

Use any GraphQL client to explore the API:

### Recommended Tools

- **[Altair GraphQL Client](https://altairgraphql.dev/)** - Desktop app with introspection
- **[GraphQL Playground](https://github.com/graphql/graphql-playground)** - Web-based IDE
- **[Postman](https://www.postman.com/)** - REST & GraphQL testing
- **curl** - Command-line testing (see test script)

### Introspection Query

GraphQL supports introspection to explore the schema:

```graphql
{
  __schema {
    queryType {
      fields {
        name
        description
        args {
          name
          type {
            name
          }
        }
      }
    }
  }
}
```

## Testing

### Automated Test Script

```bash
# Run all example queries
.warp/test-queries.sh
```

The test script includes:
- Query records
- Network status
- Search
- Create record mutation
- Territory listing

### Manual Testing

```bash
# Test network status
curl -X POST http://localhost:27123/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ networkStatus { health } }"}'

# Test record query
curl -X POST http://localhost:27123/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ records(limit: 3) { records { title } } }"}'
```

## Architecture

```
┌─────────────────────────────────┐
│    GraphQL Client               │
│  (curl, Postman, Altair, etc.)  │
└────────────┬────────────────────┘
             │ POST /graphql
             │
┌────────────▼────────────────────┐
│  carnival-network-graphql       │
│  ┌─────────────────────────┐   │
│  │  GraphQL Handler        │   │
│  │  - Parse query          │   │
│  │  - Execute resolvers    │   │
│  └────────┬────────────────┘   │
└───────────┼─────────────────────┘
            │ HTTP fetch()
┌───────────▼─────────────────────┐
│  carnival-network REST API      │
│  - GET /api/records             │
│  - POST /api/records            │
│  - POST /api/search             │
│  - GET /api/network/status      │
│  - GET /api/territories         │
│  - GET /api/analytics           │
└─────────────────────────────────┘
```

### How It Works

1. **Client sends GraphQL query** to `/graphql` endpoint
2. **Plugin parses query** using `graphql` library
3. **Resolvers translate** GraphQL fields to REST API calls
4. **REST API returns data** from carnival-network
5. **GraphQL formatter** structures response per schema
6. **Client receives** properly typed GraphQL response

### Why This Design?

- ✅ **Separation of concerns** - GraphQL layer is independent
- ✅ **No code duplication** - Reuses existing REST API
- ✅ **Optional feature** - carnival-network stays lean
- ✅ **Easy to maintain** - Changes to REST API don't break GraphQL
- ✅ **Type safety** - GraphQL schema enforces contracts

## Development

### Setup Development Environment

```bash
# Navigate to plugin directory
cd .obsidian/plugins/carnival-network-graphql

# Install dependencies
npm install

# Start development build (watch mode)
npm run dev
```

### Build Commands

```bash
# Development build with watch mode
npm run dev

# Production build
npm run build

# Lint code
npm run lint

# Clean build artifacts
npm run clean
```

### Project Structure

```
carnival-network-graphql/
├── src/
│   ├── main.ts              # Plugin entry point
│   ├── graphql/
│   │   ├── schema.ts        # GraphQL schema definition
│   │   ├── resolvers.ts     # Query/Mutation resolvers
│   │   └── types.ts         # TypeScript type definitions
│   ├── rest/
│   │   ├── client.ts        # REST API client
│   │   └── types.ts         # REST response types
│   └── utils/
│       ├── logger.ts        # Logging utilities
│       └── errors.ts        # Error handling
├── .warp/
│   ├── IMPLEMENTATION-PLAN.md   # Full implementation guide
│   ├── conversations/           # Development session logs
│   └── scripts/
│       └── test-queries.sh      # Testing script
├── manifest.json            # Plugin metadata
├── package.json             # Dependencies
├── tsconfig.json            # TypeScript config
├── esbuild.config.mjs       # Build configuration
└── README.md                # This file
```

## Troubleshooting

### Plugin doesn't load

**Symptom:** No notice when enabling plugin

**Solutions:**
1. Verify carnival-network is installed and enabled
2. Verify obsidian-local-rest-api is installed and enabled
3. Check Obsidian Developer Console for errors (Ctrl+Shift+I)

### GraphQL endpoint not responding

**Symptom:** Connection refused or timeout errors

**Solutions:**
```bash
# 1. Verify REST API is running
curl http://localhost:27123/api/network/status

# 2. Check if local-rest-api is on correct port
# Settings → Local REST API → Check port number

# 3. Verify plugin is enabled
# Settings → Community Plugins → carnival-network-graphql
```

### GraphQL errors in responses

**Symptom:** `{"errors": [...]}`in response

**Solutions:**
1. Check query syntax is valid GraphQL
2. Verify field names match schema
3. Test underlying REST API endpoint directly
4. Check Obsidian console for detailed error logs

### "Carnival Network plugin required" error

**Solution:** Install and enable the carnival-network plugin first

### Large bundle size

**Note:** This plugin adds ~2MB to Obsidian due to GraphQL dependencies. This is expected and acceptable since it's optional. The core carnival-network plugin remains lean.

## FAQ

**Q: Why not include GraphQL in carnival-network directly?**

A: Separation keeps carnival-network lean (~200KB) for users who don't need GraphQL. Optional installation means only GraphQL users pay the 2MB cost.

**Q: Can I use this without carnival-network?**

A: No, this plugin requires carnival-network's REST API to function.

**Q: Does this support subscriptions?**

A: Not currently. The plugin only supports queries and mutations over HTTP. Subscriptions would require WebSocket support.

**Q: Can I add custom GraphQL fields?**

A: Yes! Edit `src/graphql/schema.ts` and add resolvers in `src/graphql/resolvers.ts`. See the WARP.md file for examples.

**Q: How do I update the schema?**

A: Edit the schema string in `src/graphql/schema.ts`, add corresponding resolvers, rebuild with `npm run build`, and reload Obsidian.

## Performance

- **Query overhead:** +5-15ms per request vs direct REST
- **Memory usage:** ~10-30MB baseline (GraphQL schema + runtime)
- **Bundle size:** ~2MB (graphql + graphql-http libraries)
- **Concurrent requests:** Limited by local-rest-api (typically 10-20)

For high-performance needs, consider using the REST API directly.

## Contributing

This is a personal project, but suggestions are welcome:

1. Test the plugin thoroughly
2. Document issues with clear reproduction steps
3. Suggest improvements via GitHub issues (if public repo)

## License

MIT

## Related Projects

- **[carnival-network](../carnival-network)** - The REST API this wraps
- **[carnival-records](../carnival-records)** - Record management UI
- **[obsidian-local-rest-api](https://github.com/coddingtonbear/obsidian-local-rest-api)** - HTTP server dependency

## Version History

See [CHANGELOG.md](CHANGELOG.md) for version history and updates.

---

**Built with ❤️ for the Carnival of Calamity**
