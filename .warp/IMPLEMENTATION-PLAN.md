# Carnival Network GraphQL Plugin - Implementation Plan

## Project Overview

**Goal**: Create a lightweight Obsidian plugin that provides a GraphQL interface for the Carnival Network REST API.

**Key Principles**:
- Zero coupling to carnival-network internals
- All data fetched via REST API calls
- Thin translation layer (GraphQL → REST)
- Optional installation (doesn't bloat core plugin)

---

## Architecture

```
┌─────────────────────────────────────────────┐
│         External GraphQL Clients            │
│    (curl, Postman, custom apps, etc.)       │
└──────────────────┬──────────────────────────┘
                   │ POST /graphql
                   │
┌──────────────────▼──────────────────────────┐
│      carnival-network-graphql Plugin        │
│  ┌────────────────────────────────────────┐ │
│  │  GraphQL Schema & Resolvers            │ │
│  │  - Parse GraphQL queries               │ │
│  │  - Translate to REST API calls         │ │
│  │  - Format responses                    │ │
│  └────────────────┬───────────────────────┘ │
└───────────────────┼─────────────────────────┘
                    │ fetch()
┌───────────────────▼─────────────────────────┐
│       carnival-network REST API             │
│    GET /api/records                         │
│    POST /api/records                        │
│    POST /api/search                         │
│    GET /api/network/status                  │
│    GET /api/territories                     │
│    GET /api/analytics                       │
└─────────────────────────────────────────────┘
```

---

## Phase 1: Project Setup (1-2 hours)

### 1.1 Project Structure
```
carnival-network-graphql/
├── .warp/                          # ✅ Already created
│   ├── IMPLEMENTATION-PLAN.md
│   ├── CONVERSATION-template.md
│   └── scripts/
├── src/
│   ├── main.ts                     # Plugin entry point
│   ├── graphql/
│   │   ├── schema.ts              # GraphQL schema definition
│   │   ├── resolvers.ts           # Query/Mutation resolvers
│   │   └── types.ts               # TypeScript types
│   ├── rest/
│   │   └── client.ts              # REST API client
│   └── utils/
│       ├── logger.ts              # Logging utilities
│       └── errors.ts              # Error handling
├── manifest.json
├── package.json
├── tsconfig.json
├── esbuild.config.mjs
├── .eslintrc.json
├── .gitignore
├── README.md
└── WARP.md                        # Warp AI context
```

### 1.2 Dependencies
```json
{
  "dependencies": {
    "graphql": "^16.8.1",
    "graphql-http": "^1.22.0"
  },
  "devDependencies": {
    "@types/node": "^16.11.6",
    "esbuild": "^0.25.11",
    "obsidian": "latest",
    "typescript": "^5.9.3"
  }
}
```

### 1.3 Deliverables
- [ ] Project initialized with package.json
- [ ] TypeScript configuration
- [ ] Build system (esbuild)
- [ ] Manifest.json with carnival-network dependency

---

## Phase 2: REST API Client (2-3 hours)

### 2.1 REST Client Implementation

**File**: `src/rest/client.ts`

```typescript
/**
 * REST API Client for Carnival Network
 * Wraps all REST API calls with error handling
 */
export class CarnivalRestClient {
  constructor(
    private baseUrl: string = 'http://localhost:27123'
  ) {}

  // Records
  async queryRecords(params: RecordQueryParams): Promise<RecordQueryResponse>
  async createRecord(data: CreateRecordInput): Promise<CreateRecordResponse>
  async getRecord(id: string): Promise<Record>
  
  // Search
  async search(query: SearchInput): Promise<SearchResults>
  
  // Network
  async getNetworkStatus(): Promise<NetworkStatus>
  async getTerritories(): Promise<Territory[]>
  async getAnalytics(metrics?: string[]): Promise<Analytics>
  
  private async fetch<T>(endpoint: string, options?: RequestInit): Promise<T>
}
```

### 2.2 Type Definitions

**File**: `src/rest/types.ts`

Mirror the REST API response types from carnival-network's `local-rest-api-types.ts`

### 2.3 Deliverables
- [ ] REST client with all endpoint methods
- [ ] Type definitions matching REST API
- [ ] Error handling and retry logic
- [ ] Connection validation

---

## Phase 3: GraphQL Schema (3-4 hours)

### 3.1 Schema Definition

**File**: `src/graphql/schema.ts`

```graphql
type Query {
  # Records
  records(
    territory: String
    type: String
    limit: Int = 10
    offset: Int = 0
  ): RecordQueryResult!
  
  record(id: ID!): Record
  
  # Search
  search(
    query: String!
    territories: [String!]
    limit: Int = 20
  ): SearchResult!
  
  # Network
  networkStatus: NetworkStatus!
  territories: [Territory!]!
  analytics(
    metrics: [MetricType!]
    timeframe: String
  ): Analytics!
}

type Mutation {
  createRecord(input: CreateRecordInput!): CreateRecordResult!
}

type Record {
  id: ID!
  title: String!
  territory: String!
  type: String!
  content: String
  metadata: JSON
  createdAt: String!
  updatedAt: String
  status: String!
}

type RecordQueryResult {
  records: [Record!]!
  pagination: Pagination!
  timestamp: String!
}

type Pagination {
  limit: Int!
  offset: Int!
  total: Int!
  hasNext: Boolean!
  hasPrevious: Boolean!
}

type SearchResult {
  query: String!
  results: [SearchMatch!]!
  resultCount: Int!
  hasMore: Boolean!
  timestamp: String!
}

type SearchMatch {
  record: Record!
  matchedFields: [String!]!
  relevance: Float
}

type NetworkStatus {
  health: String!
  uptime: Uptime!
  network: NetworkInfo!
  capabilities: [String!]!
  timestamp: String!
}

type NetworkInfo {
  totalPerformers: Int!
  connectedPerformers: Int!
  territories: Int!
  activeRegistries: Int!
}

type Territory {
  name: String!
  performerCount: Int!
  status: TerritoryStatus!
}

type Analytics {
  timeframe: String!
  records: RecordMetrics
  activity: ActivityMetrics
  capabilities: [String!]
  performance: PerformanceMetrics
  timestamp: String!
}

input CreateRecordInput {
  title: String!
  territory: String!
  type: String!
  content: String
  metadata: JSON
  requireAck: Boolean
  broadcastToAll: Boolean
  targetTerritories: [String!]
  broadcast: Boolean
}

enum TerritoryStatus {
  ACTIVE
  INACTIVE
}

enum MetricType {
  RECORDS
  ACTIVITY
  CAPABILITIES
  PERFORMANCE
}

scalar JSON
```

### 3.2 Deliverables
- [ ] Complete GraphQL schema
- [ ] Type definitions
- [ ] Custom scalars (JSON)
- [ ] Input validation rules

---

## Phase 4: GraphQL Resolvers (4-6 hours)

### 4.1 Query Resolvers

**File**: `src/graphql/resolvers.ts`

```typescript
export const resolvers = {
  Query: {
    records: async (_, args, context) => {
      const { territory, type, limit, offset } = args;
      const response = await context.restClient.queryRecords({
        territory,
        type,
        limit,
        offset
      });
      return {
        records: response.data,
        pagination: response.pagination,
        timestamp: response.timestamp
      };
    },
    
    record: async (_, { id }, context) => {
      const response = await context.restClient.getRecord(id);
      return response.data;
    },
    
    search: async (_, args, context) => {
      const { query, territories, limit } = args;
      const response = await context.restClient.search({
        query,
        territories,
        limit
      });
      return {
        query: response.data.query,
        results: response.data.results.map(r => ({
          record: {
            id: r.id,
            title: r.title,
            type: r.type,
            territory: r.territory,
            content: r.content,
            createdAt: r.createdAt,
            status: 'active'
          },
          matchedFields: r.matchedFields,
          relevance: r.relevance
        })),
        resultCount: response.data.resultCount,
        hasMore: response.data.hasMore,
        timestamp: response.timestamp
      };
    },
    
    networkStatus: async (_, __, context) => {
      const response = await context.restClient.getNetworkStatus();
      return response.data;
    },
    
    territories: async (_, __, context) => {
      const response = await context.restClient.getTerritories();
      return response.data.territories;
    },
    
    analytics: async (_, args, context) => {
      const { metrics, timeframe } = args;
      const response = await context.restClient.getAnalytics(
        metrics || ['RECORDS', 'ACTIVITY']
      );
      return response.data;
    }
  },
  
  Mutation: {
    createRecord: async (_, { input }, context) => {
      const response = await context.restClient.createRecord(input);
      return {
        record: response.data,
        message: response.message,
        timestamp: response.timestamp
      };
    }
  }
};
```

### 4.2 Context Setup

```typescript
export interface GraphQLContext {
  restClient: CarnivalRestClient;
}

export function createContext(): GraphQLContext {
  return {
    restClient: new CarnivalRestClient()
  };
}
```

### 4.3 Deliverables
- [ ] All query resolvers implemented
- [ ] Mutation resolvers implemented
- [ ] Context factory
- [ ] Error mapping (REST → GraphQL)

---

## Phase 5: Plugin Integration (2-3 hours)

### 5.1 Main Plugin Class

**File**: `src/main.ts`

```typescript
import { Plugin, Notice } from 'obsidian';
import { createHandler } from 'graphql-http/lib/use/http';
import { buildSchema } from 'graphql';
import { schema } from './graphql/schema';
import { resolvers } from './graphql/resolvers';
import { createContext } from './graphql/context';

export default class CarnivalNetworkGraphQLPlugin extends Plugin {
  private graphqlHandler: any;
  
  async onload() {
    console.log('Loading Carnival Network GraphQL plugin');
    
    // Verify carnival-network is installed
    if (!this.verifyCarnivalNetwork()) {
      return;
    }
    
    // Create GraphQL handler
    this.graphqlHandler = createHandler({
      schema: buildSchema(schema),
      rootValue: resolvers,
      context: createContext()
    });
    
    // Register with local-rest-api
    this.app.workspace.onLayoutReady(() => {
      this.registerGraphQLEndpoint();
    });
    
    new Notice('Carnival Network GraphQL enabled');
  }
  
  private verifyCarnivalNetwork(): boolean {
    const carnivalPlugin = (this.app as any).plugins.getPlugin('carnival-network');
    
    if (!carnivalPlugin) {
      new Notice(
        'Carnival Network plugin required. Please install carnival-network first.',
        10000
      );
      return false;
    }
    
    return true;
  }
  
  private registerGraphQLEndpoint() {
    const localRestAPI = (this.app as any).plugins.getPlugin('obsidian-local-rest-api');
    
    if (!localRestAPI) {
      new Notice(
        'Local REST API plugin required. Please install obsidian-local-rest-api.',
        10000
      );
      return;
    }
    
    try {
      const api = localRestAPI.getPublicApi?.(this.manifest);
      
      if (!api) {
        throw new Error('Could not get Local REST API public API');
      }
      
      // Register GraphQL endpoint
      api.addRoute('/graphql').all(async (req: any, res: any) => {
        try {
          const result = await this.graphqlHandler(req);
          res.status(result.status).json(result.body);
        } catch (error) {
          console.error('GraphQL error:', error);
          res.status(500).json({
            errors: [{ message: 'Internal GraphQL error' }]
          });
        }
      });
      
      console.log('GraphQL endpoint registered: POST /graphql');
      new Notice('GraphQL endpoint available at /graphql');
      
    } catch (error) {
      console.error('Failed to register GraphQL endpoint:', error);
      new Notice('Failed to register GraphQL endpoint');
    }
  }
  
  async onunload() {
    console.log('Unloading Carnival Network GraphQL plugin');
  }
}
```

### 5.2 Manifest

**File**: `manifest.json`

```json
{
  "id": "carnival-network-graphql",
  "name": "Carnival Network GraphQL",
  "version": "1.0.0",
  "minAppVersion": "0.15.0",
  "description": "GraphQL interface for Carnival Network REST API",
  "author": "L V N A C Y",
  "authorUrl": "https://github.com/yourusername",
  "isDesktopOnly": false,
  "dependencies": ["carnival-network", "obsidian-local-rest-api"]
}
```

### 5.3 Deliverables
- [ ] Plugin class with initialization
- [ ] Dependency verification
- [ ] GraphQL endpoint registration
- [ ] Error handling and notices
- [ ] Manifest with dependencies

---

## Phase 6: Testing & Documentation (2-3 hours)

### 6.1 Testing Script

**File**: `.warp/test-queries.sh`

```bash
#!/bin/bash

# GraphQL testing script for Carnival Network
BASE_URL="http://localhost:27123/graphql"

echo "🧪 Testing Carnival Network GraphQL API"
echo "========================================"

# Query records
echo -e "\n📋 Query Records:"
curl -X POST $BASE_URL \
  -H "Content-Type: application/json" \
  -d '{
    "query": "{ records(limit: 5) { records { id title territory } pagination { total } } }"
  }' | jq

# Get network status
echo -e "\n🌐 Network Status:"
curl -X POST $BASE_URL \
  -H "Content-Type: application/json" \
  -d '{
    "query": "{ networkStatus { health uptime { formatted } network { totalPerformers } } }"
  }' | jq

# Search
echo -e "\n🔍 Search:"
curl -X POST $BASE_URL \
  -H "Content-Type: application/json" \
  -d '{
    "query": "query Search($q: String!) { search(query: $q, limit: 3) { resultCount results { record { title } } } }",
    "variables": { "q": "test" }
  }' | jq

# Create record
echo -e "\n✏️  Create Record:"
curl -X POST $BASE_URL \
  -H "Content-Type: application/json" \
  -d '{
    "query": "mutation CreateRecord($input: CreateRecordInput!) { createRecord(input: $input) { record { id title } message } }",
    "variables": {
      "input": {
        "title": "GraphQL Test Record",
        "territory": "backstage",
        "type": "changelog",
        "content": "Created via GraphQL"
      }
    }
  }' | jq

echo -e "\n✅ Testing complete"
```

### 6.2 README

**File**: `README.md`

```markdown
# Carnival Network GraphQL

GraphQL interface for the [Carnival Network](../carnival-network) REST API.

## Features

- 🎯 **Clean GraphQL interface** for Carnival Network
- 🔌 **Zero coupling** to internal implementation
- 📦 **Optional installation** - doesn't bloat core plugin
- 🚀 **Full API coverage** - all REST endpoints available

## Installation

### Prerequisites
1. Install [carnival-network](../carnival-network) plugin
2. Install [obsidian-local-rest-api](https://github.com/coddingtonbear/obsidian-local-rest-api) plugin

### Install Plugin
1. Copy this folder to `.obsidian/plugins/carnival-network-graphql/`
2. Enable in Obsidian Settings → Community Plugins

## Usage

### GraphQL Endpoint
```
POST http://localhost:27123/graphql
```

### Example Queries

**Query Records:**
```graphql
{
  records(territory: "backstage", limit: 10) {
    records {
      id
      title
      territory
      createdAt
    }
    pagination {
      total
      hasNext
    }
  }
}
```

**Search:**
```graphql
{
  search(query: "test", territories: ["backstage"]) {
    resultCount
    results {
      record {
        title
        content
      }
      matchedFields
      relevance
    }
  }
}
```

**Create Record:**
```graphql
mutation {
  createRecord(input: {
    title: "New Record"
    territory: "backstage"
    type: "changelog"
    content: "Record content"
  }) {
    record {
      id
      title
    }
    message
  }
}
```

**Network Status:**
```graphql
{
  networkStatus {
    health
    uptime {
      formatted
    }
    network {
      totalPerformers
      connectedPerformers
      territories
    }
  }
}
```

## GraphQL Playground

Use tools like:
- [GraphQL Playground](https://github.com/graphql/graphql-playground)
- [Altair GraphQL Client](https://altairgraphql.dev/)
- [Postman](https://www.postman.com/)
- curl (see `.warp/test-queries.sh`)

## Architecture

This plugin is a thin wrapper that:
1. Accepts GraphQL queries
2. Translates them to REST API calls
3. Returns formatted GraphQL responses

All data flows through the carnival-network REST API.

## Development

```bash
# Install dependencies
npm install

# Build
npm run build

# Dev mode (watch)
npm run dev
```

## License

MIT
```

### 6.3 Deliverables
- [ ] Test script with example queries
- [ ] Comprehensive README
- [ ] Example queries documented
- [ ] Manual testing completed

---

## Success Criteria

- [ ] Plugin loads without errors
- [ ] GraphQL endpoint responds at `/graphql`
- [ ] All query types work correctly
- [ ] Mutations create records successfully
- [ ] Error handling works properly
- [ ] Documentation is clear and complete
- [ ] No increase in carnival-network bundle size

---

## Estimated Timeline

- **Phase 1**: 1-2 hours (Setup)
- **Phase 2**: 2-3 hours (REST Client)
- **Phase 3**: 3-4 hours (Schema)
- **Phase 4**: 4-6 hours (Resolvers)
- **Phase 5**: 2-3 hours (Integration)
- **Phase 6**: 2-3 hours (Testing/Docs)

**Total**: ~14-21 hours

---

## Next Steps

1. Review this implementation plan
2. Set up project structure (Phase 1)
3. Implement REST client (Phase 2)
4. Build GraphQL layer (Phases 3-4)
5. Integrate with Obsidian (Phase 5)
6. Test and document (Phase 6)
