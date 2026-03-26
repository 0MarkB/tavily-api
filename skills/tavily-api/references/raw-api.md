# Tavily Raw HTTP API Reference

Use this only when neither the Python nor JavaScript SDK can be installed. The SDKs are strongly preferred — they handle authentication headers, error parsing, retries, and response typing automatically. This reference covers doing it manually via direct HTTP requests.

**Base URL:** `https://api.tavily.com`
**Auth:** Bearer token header: `Authorization: Bearer tvly-YOUR_API_KEY`

All endpoints accept JSON request bodies and return JSON responses.

---

## POST /search

```bash
curl -X POST https://api.tavily.com/search \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer tvly-YOUR_API_KEY" \
  -d '{
    "query": "your search query",
    "search_depth": "basic",
    "max_results": 5
  }'
```

### Request Body

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `query` | `string` | **required** | Search query (keep under 400 chars) |
| `search_depth` | `string` | `"basic"` | `"basic"`, `"advanced"`, `"fast"`, `"ultra-fast"` |
| `topic` | `string` | `"general"` | `"general"`, `"news"`, `"finance"` |
| `max_results` | `int` | `5` | 0-20 |
| `include_answer` | `bool`/`string` | `false` | `true`/`"basic"` or `"advanced"` |
| `include_raw_content` | `bool`/`string` | `false` | `true`/`"markdown"` or `"text"` |
| `include_images` | `bool` | `false` | Include image URLs |
| `include_image_descriptions` | `bool` | `false` | LLM-generated image descriptions |
| `include_domains` | `string[]` | `[]` | Only these domains (max 300) |
| `exclude_domains` | `string[]` | `[]` | Skip these domains (max 150) |
| `time_range` | `string` | `null` | `"day"`, `"week"`, `"month"`, `"year"` |
| `start_date` | `string` | `null` | `YYYY-MM-DD` |
| `end_date` | `string` | `null` | `YYYY-MM-DD` |
| `country` | `string` | `null` | Boost results from country (only when topic is `"general"`) |
| `chunks_per_source` | `int` | `3` | Chunks per source (advanced depth only) |
| `auto_parameters` | `bool` | `false` | Auto-configure from query intent |
| `exact_match` | `bool` | `false` | Only exact quoted phrase matches |
| `include_favicon` | `bool` | `false` | Favicon per result |
| `include_usage` | `bool` | `false` | Credit usage info |

### Response (200)

```json
{
  "query": "...",
  "results": [
    {
      "title": "...",
      "url": "...",
      "content": "...",
      "score": 0.85,
      "raw_content": "...",
      "published_date": "...",
      "favicon": "...",
      "images": [{"url": "...", "description": "..."}]
    }
  ],
  "answer": "...",
  "images": [{"url": "...", "description": "..."}],
  "response_time": 1.23,
  "request_id": "..."
}
```

---

## POST /extract

```bash
curl -X POST https://api.tavily.com/extract \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer tvly-YOUR_API_KEY" \
  -d '{
    "urls": ["https://example.com"],
    "extract_depth": "basic"
  }'
```

### Request Body

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `urls` | `string` or `string[]` | **required** | URL(s) to extract (max 20) |
| `extract_depth` | `string` | `"basic"` | `"basic"` or `"advanced"` |
| `format` | `string` | `"markdown"` | `"markdown"` or `"text"` |
| `query` | `string` | `null` | Reranks chunks by relevance |
| `chunks_per_source` | `int` | `3` | 1-5 (only when `query` provided) |
| `include_images` | `bool` | `false` | Include images |
| `include_favicon` | `bool` | `false` | Include favicons |
| `include_usage` | `bool` | `false` | Credit usage |
| `timeout` | `float` | `null` | 1.0-60.0s |

### Response (200)

```json
{
  "results": [
    {"url": "...", "raw_content": "...", "images": [...], "favicon": "..."}
  ],
  "failed_results": [
    {"url": "...", "error": "..."}
  ],
  "response_time": 0.5,
  "request_id": "..."
}
```

---

## POST /crawl

```bash
curl -X POST https://api.tavily.com/crawl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer tvly-YOUR_API_KEY" \
  -d '{
    "url": "https://docs.example.com",
    "instructions": "Find the API docs",
    "limit": 20
  }'
```

### Request Body

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` | `string` | **required** | Root URL to crawl |
| `max_depth` | `int` | `1` | 1-5 |
| `max_breadth` | `int` | `20` | 1-500 |
| `limit` | `int` | `50` | Total pages cap |
| `instructions` | `string` | `null` | Natural language guidance (doubles mapping cost) |
| `chunks_per_source` | `int` | `3` | 1-5 (only when `instructions` set) |
| `select_paths` | `string[]` | `null` | Regex include patterns |
| `select_domains` | `string[]` | `null` | Regex domain patterns |
| `exclude_paths` | `string[]` | `null` | Regex exclude patterns |
| `exclude_domains` | `string[]` | `null` | Regex domain exclusions |
| `allow_external` | `bool` | `true` | Follow external links |
| `include_images` | `bool` | `false` | Extract images |
| `extract_depth` | `string` | `"basic"` | `"basic"` or `"advanced"` |
| `format` | `string` | `"markdown"` | `"markdown"` or `"text"` |
| `include_favicon` | `bool` | `false` | Favicons |
| `timeout` | `float` | `150` | 10-150s |
| `include_usage` | `bool` | `false` | Credit usage |

### Response (200)

```json
{
  "base_url": "...",
  "results": [
    {"url": "...", "raw_content": "...", "images": [...], "favicon": "..."}
  ],
  "response_time": 5.2,
  "request_id": "..."
}
```

---

## POST /map

```bash
curl -X POST https://api.tavily.com/map \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer tvly-YOUR_API_KEY" \
  -d '{
    "url": "https://docs.example.com",
    "limit": 50
  }'
```

### Request Body

Same as crawl but without extraction params (`extract_depth`, `format`, `include_images`, `chunks_per_source`, `include_favicon`).

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` | `string` | **required** | Root URL to map |
| `max_depth` | `int` | `1` | 1-5 |
| `max_breadth` | `int` | `20` | 1-500 |
| `limit` | `int` | `50` | Total links cap |
| `instructions` | `string` | `null` | Guidance (doubles cost) |
| `select_paths` | `string[]` | `null` | Regex include |
| `select_domains` | `string[]` | `null` | Regex domains |
| `exclude_paths` | `string[]` | `null` | Regex exclude |
| `exclude_domains` | `string[]` | `null` | Regex domain exclusions |
| `allow_external` | `bool` | `true` | External links |
| `timeout` | `float` | `150` | 10-150s |
| `include_usage` | `bool` | `false` | Credit usage |

### Response (200)

```json
{
  "base_url": "...",
  "results": ["https://...", "https://..."],
  "response_time": 2.1,
  "request_id": "..."
}
```

---

## POST /research

```bash
curl -X POST https://api.tavily.com/research \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer tvly-YOUR_API_KEY" \
  -d '{
    "input": "What are the latest developments in quantum computing?",
    "model": "mini"
  }'
```

### Request Body

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `input` | `string` | **required** | Research question |
| `model` | `string` | `"auto"` | `"mini"`, `"pro"`, `"auto"` |
| `stream` | `bool` | `false` | SSE streaming |
| `output_schema` | `object` | `null` | JSON Schema for structured output |
| `citation_format` | `string` | `"numbered"` | `"numbered"`, `"mla"`, `"apa"`, `"chicago"` |

### Response (201) — task queued

```json
{
  "request_id": "123e4567-...",
  "created_at": "2025-01-15T10:30:00Z",
  "status": "pending",
  "input": "...",
  "model": "mini"
}
```

### GET /research/{request_id} — poll for results

```bash
curl https://api.tavily.com/research/REQUEST_ID \
  -H "Authorization: Bearer tvly-YOUR_API_KEY"
```

**202** (still processing): `{"request_id": "...", "status": "pending"|"in_progress"}`

**200** (completed): `{"request_id": "...", "status": "completed", "content": "...", "sources": [{"title": "...", "url": "...", "favicon": "..."}]}`

**200** (failed): `{"request_id": "...", "status": "failed"}`

---

## Error Responses

All errors return: `{"detail": {"error": "message"}}`

| Code | Meaning |
|------|---------|
| 400 | Bad request / invalid params |
| 401 | Missing or invalid API key |
| 403 | URL not supported |
| 429 | Rate limit — check `retry-after` header |
| 432 | Plan/key limit exceeded |
| 433 | Pay-as-you-go limit exceeded |
| 500 | Server error |

---

## Quick Examples in Common Languages

### Python (no SDK, just requests)

```python
import requests

headers = {
    "Content-Type": "application/json",
    "Authorization": "Bearer tvly-YOUR_API_KEY"
}

response = requests.post("https://api.tavily.com/search", headers=headers, json={
    "query": "latest AI news",
    "max_results": 5
})
data = response.json()
```

### JavaScript (no SDK, just fetch)

```javascript
const response = await fetch("https://api.tavily.com/search", {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer tvly-YOUR_API_KEY"
    },
    body: JSON.stringify({
        query: "latest AI news",
        maxResults: 5
    })
});
const data = await response.json();
```

### Bash (curl)

```bash
curl -s -X POST https://api.tavily.com/search \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer tvly-YOUR_API_KEY" \
  -d '{"query": "latest AI news", "max_results": 5}' | jq .
```
