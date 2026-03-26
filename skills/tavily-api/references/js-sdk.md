# Tavily JavaScript/TypeScript SDK Reference

## Installation

```bash
npm i @tavily/core
```

## Client Setup

```javascript
const { tavily } = require("@tavily/core");
// or ES modules:
// import { tavily } from "@tavily/core";

const client = tavily({ apiKey: "tvly-YOUR_API_KEY" });
```

### Constructor Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `apiKey` | `string` | required | API key (prefix `tvly-`) |
| `projectId` | `string` | `undefined` | Organize/track usage by project |
| `proxies` | `object` | `undefined` | `{ http: "...", https: "..." }` proxy config |

Environment variables: `TAVILY_HTTP_PROXY`, `TAVILY_HTTPS_PROXY`, `TAVILY_PROJECT`

All methods return Promises (the client is async by default).

---

## client.search(query, options)

```javascript
const response = await client.search("your query here");
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `query` | `string` | **required** | Search query (keep under 400 chars) |
| `searchDepth` | `string` | `"basic"` | `"basic"`, `"advanced"`, `"fast"`, `"ultra-fast"` |
| `topic` | `string` | `"general"` | `"general"`, `"news"`, `"finance"` |
| `maxResults` | `number` | `5` | 0-20 results |
| `includeAnswer` | `boolean`/`string` | `false` | `true`/`"basic"` = quick; `"advanced"` = detailed |
| `includeRawContent` | `boolean`/`string` | `false` | `true`/`"markdown"` = markdown; `"text"` = plain |
| `includeImages` | `boolean` | `false` | Include image URLs |
| `includeImageDescriptions` | `boolean` | `false` | Add LLM-generated image descriptions |
| `includeDomains` | `string[]` | `[]` | Only these domains (max 300) |
| `excludeDomains` | `string[]` | `[]` | Skip these domains (max 150) |
| `timeRange` | `string` | `undefined` | `"day"`, `"week"`, `"month"`, `"year"` (or `"d"`,`"w"`,`"m"`,`"y"`) |
| `startDate` | `string` | `undefined` | `YYYY-MM-DD` format |
| `endDate` | `string` | `undefined` | `YYYY-MM-DD` format |
| `country` | `string` | `undefined` | Boost results from country (only when topic is `"general"`) |
| `chunksPerSource` | `number` | `3` | Chunks per source (advanced depth only, max 500 chars each) |
| `auto_parameters` | `boolean` | `false` | Auto-configure from query intent (may upgrade to advanced = 2 credits) |
| `exactMatch` | `boolean` | `false` | Only results containing exact quoted phrases |
| `includeFavicon` | `boolean` | `false` | Include favicon URL per result |
| `includeUsage` | `boolean` | `false` | Include credit usage info |
| `timeout` | `number` | `60` | Timeout in seconds |

### Response Object

```javascript
{
    query: "...",
    results: [
        {
            title: "...",
            url: "...",
            content: "...",        // AI-extracted snippet
            score: 0.85,           // relevance score
            rawContent: "...",     // if includeRawContent set
            publishedDate: "...",  // if topic="news"
            favicon: "...",        // if includeFavicon set
            images: [...]          // if includeImages set
        }
    ],
    answer: "...",          // if includeAnswer set
    images: [...],          // if includeImages set
    responseTime: 1.23,
    requestId: "..."
}
```

### Examples

```javascript
// Basic search
const response = await client.search("latest AI developments");

// News search with date filtering
const response = await client.search("tech earnings Q1 2025", {
    topic: "news",
    timeRange: "month",
    includeAnswer: true,
    maxResults: 10
});

// Advanced search with domain filtering
const response = await client.search("machine learning best practices", {
    searchDepth: "advanced",
    includeDomains: ["arxiv.org", "papers.nips.cc"],
    includeRawContent: true
});

// Exact match
const response = await client.search('"John Smith" CEO Acme Corp', {
    exactMatch: true
});
```

---

## client.extract(urls, options)

```javascript
const response = await client.extract(["https://example.com"]);
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `urls` | `string[]` | **required** | URLs to extract (max 20) |
| `extractDepth` | `string` | `"basic"` | `"basic"` or `"advanced"` (JS-rendered, tables) |
| `format` | `string` | `"markdown"` | `"markdown"` or `"text"` |
| `query` | `string` | `undefined` | Reranks chunks by relevance |
| `chunksPerSource` | `number` | `3` | 1-5 chunks (only when `query` provided) |
| `includeImages` | `boolean` | `false` | Include image URLs |
| `includeFavicon` | `boolean` | `false` | Include favicon URLs |
| `includeUsage` | `boolean` | `false` | Include credit usage |
| `timeout` | `number` | `undefined` | 1.0-60.0s (defaults: 10s basic, 30s advanced) |

### Response Object

```javascript
{
    results: [
        {
            url: "...",
            raw_content: "...",    // extracted content
            images: [...],         // if includeImages set
            favicon: "..."         // if includeFavicon set
        }
    ],
    failed_results: [
        { url: "...", error: "..." }
    ],
    response_time: 0.5,
    requestId: "..."
}
```

### Example

```javascript
const response = await client.extract(
    [
        "https://en.wikipedia.org/wiki/Artificial_intelligence",
        "https://en.wikipedia.org/wiki/Machine_learning"
    ],
    {
        query: "What are the key breakthroughs?",
        chunksPerSource: 3,
        includeImages: true
    }
);
```

---

## client.crawl(url, options)

```javascript
const response = await client.crawl("https://docs.example.com", {
    instructions: "Find the API reference"
});
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` | `string` | **required** | Root URL to crawl |
| `maxDepth` | `number` | `1` | Exploration depth (1-5) |
| `maxBreadth` | `number` | `20` | Links per level (1-500) |
| `limit` | `number` | `50` | Total pages before stopping |
| `instructions` | `string` | `undefined` | Natural language guidance (doubles mapping cost) |
| `chunksPerSource` | `number` | `3` | 1-5 chunks (only when `instructions` set) |
| `selectPaths` | `string[]` | `[]` | Regex patterns for allowed paths |
| `selectDomains` | `string[]` | `[]` | Regex for allowed domains |
| `excludePaths` | `string[]` | `[]` | Regex for excluded paths |
| `excludeDomains` | `string[]` | `[]` | Regex for excluded domains |
| `allowExternal` | `boolean` | `true` | Follow external links |
| `includeImages` | `boolean` | `false` | Extract images |
| `extractDepth` | `string` | `"basic"` | `"basic"` or `"advanced"` |
| `format` | `string` | `"markdown"` | `"markdown"` or `"text"` |
| `includeFavicon` | `boolean` | `false` | Include favicons |
| `timeout` | `number` | `150` | 10-150 seconds |
| `includeUsage` | `boolean` | `false` | Credit usage info |

### Response Object

```javascript
{
    baseUrl: "...",
    results: [
        {
            url: "...",
            rawContent: "...",
            images: [...],
            favicon: "..."
        }
    ],
    responseTime: 5.2,
    requestId: "..."
}
```

### Example

```javascript
const response = await client.crawl("https://docs.example.com", {
    instructions: "Find all pages about authentication",
    maxDepth: 2,
    limit: 30,
    selectPaths: ["/docs/auth.*"]
});
```

---

## client.map(url, options)

```javascript
const response = await client.map("https://docs.example.com");
```

### Parameters

Same as `crawl()` except no extraction-related params (`extractDepth`, `format`, `includeImages`, `chunksPerSource`, `includeFavicon`).

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` | `string` | **required** | Root URL to map |
| `maxDepth` | `number` | `1` | Exploration depth |
| `maxBreadth` | `number` | `20` | Links per level |
| `limit` | `number` | `50` | Total links before stopping |
| `instructions` | `string` | `undefined` | Guidance (doubles cost) |
| `selectPaths` | `string[]` | `[]` | Regex include patterns |
| `selectDomains` | `string[]` | `[]` | Regex domain patterns |
| `excludePaths` | `string[]` | `[]` | Regex exclude patterns |
| `excludeDomains` | `string[]` | `[]` | Regex domain exclusions |
| `allowExternal` | `boolean` | `true` | Follow external links |
| `timeout` | `number` | `150` | 10-150 seconds |
| `includeUsage` | `boolean` | `false` | Credit usage info |

### Response Object

```javascript
{
    baseUrl: "...",
    results: ["https://...", "https://...", ...],  // list of discovered URLs
    responseTime: 2.1,
    requestId: "..."
}
```

### Example

```javascript
// Map first to discover structure
const urls = await client.map("https://docs.example.com", {
    instructions: "Find the API reference pages",
    limit: 100
});

// Then crawl only what you need
const response = await client.crawl("https://docs.example.com", {
    selectPaths: ["/api/.*"],
    limit: 20
});
```

---

## client.research(input, options)

Deep multi-step research that automatically searches, reads, and synthesizes a report.

```javascript
const response = await client.research("What are the latest developments in quantum computing?");
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `input` | `string` | **required** | Research question or task |
| `model` | `string` | `"auto"` | `"mini"` (focused, 4-110 credits), `"pro"` (deep, 15-250 credits), `"auto"` |
| `stream` | `boolean` | `false` | Stream progress via SSE |
| `outputSchema` | `object` | `undefined` | JSON Schema for structured output |
| `citationFormat` | `string` | `"numbered"` | `"numbered"`, `"mla"`, `"apa"`, `"chicago"` |

### Non-streaming workflow

```javascript
// Start research (returns pending status)
const response = await client.research("your question");
const requestId = response.requestId;

// Poll for completion
const poll = async () => {
    while (true) {
        const result = await client.getResearch(requestId);
        if (result.status === "completed") {
            console.log(result.content);   // the report
            console.log(result.sources);   // [{title, url, favicon}]
            return result;
        }
        if (result.status === "failed") {
            throw new Error("Research failed");
        }
        await new Promise(r => setTimeout(r, 5000));
    }
};
await poll();
```

### Streaming

```javascript
const stream = await client.research("your question", {
    model: "pro",
    stream: true
});
for await (const chunk of stream) {
    console.log(chunk.toString('utf-8'));
}
```

### Structured output

```javascript
const response = await client.research("Compare React vs Vue in 2025", {
    outputSchema: {
        properties: {
            comparison: {
                type: "array",
                description: "Comparison points",
                items: {
                    type: "object",
                    properties: {
                        category: { type: "string", description: "Category" },
                        react: { type: "string", description: "React assessment" },
                        vue: { type: "string", description: "Vue assessment" }
                    }
                }
            },
            recommendation: { type: "string", description: "Overall recommendation" }
        },
        required: ["comparison", "recommendation"]
    }
});
```
