# Tavily Python SDK Reference

## Installation

```bash
pip install tavily-python
```

## Client Setup

```python
from tavily import TavilyClient

client = TavilyClient(api_key="tvly-YOUR_API_KEY")
```

### Async Client

```python
from tavily import AsyncTavilyClient

client = AsyncTavilyClient(api_key="tvly-YOUR_API_KEY")
```

### Constructor Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `api_key` | `str` | required | API key (prefix `tvly-`) |
| `project_id` | `str` | `None` | Organize/track usage by project |
| `proxies` | `dict` | `None` | `{"http": "...", "https": "..."}` proxy config |

Environment variables: `TAVILY_PROJECT`, `TAVILY_HTTP_PROXY`, `TAVILY_HTTPS_PROXY`

---

## client.search()

```python
response = client.search("your query here")
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `query` | `str` | **required** | Search query (keep under 400 chars) |
| `search_depth` | `str` | `"basic"` | `"basic"`, `"advanced"`, `"fast"`, `"ultra-fast"` |
| `topic` | `str` | `"general"` | `"general"`, `"news"`, `"finance"` |
| `max_results` | `int` | `5` | 0-20 results |
| `include_answer` | `bool`/`str` | `False` | `True`/`"basic"` = quick answer; `"advanced"` = detailed |
| `include_raw_content` | `bool`/`str` | `False` | `True`/`"markdown"` = markdown; `"text"` = plain text |
| `include_images` | `bool` | `False` | Include image URLs in results |
| `include_image_descriptions` | `bool` | `False` | Add LLM-generated image descriptions |
| `include_domains` | `list[str]` | `[]` | Only these domains (max 300) |
| `exclude_domains` | `list[str]` | `[]` | Skip these domains (max 150) |
| `time_range` | `str` | `None` | `"day"`, `"week"`, `"month"`, `"year"` (or `"d"`,`"w"`,`"m"`,`"y"`) |
| `start_date` | `str` | `None` | `YYYY-MM-DD` format |
| `end_date` | `str` | `None` | `YYYY-MM-DD` format |
| `country` | `str` | `None` | Boost results from country (only when topic is `"general"`) |
| `chunks_per_source` | `int` | `3` | Relevant chunks per source (advanced depth only, max 500 chars each) |
| `auto_parameters` | `bool` | `False` | Auto-configure params from query intent (may upgrade to advanced = 2 credits) |
| `exact_match` | `bool` | `False` | Only results containing exact quoted phrases |
| `include_favicon` | `bool` | `False` | Include favicon URL per result |
| `include_usage` | `bool` | `False` | Include credit usage info |
| `timeout` | `float` | `60` | Request timeout in seconds |

### Response (dict)

```python
{
    "query": "...",
    "results": [
        {
            "title": "...",
            "url": "...",
            "content": "...",        # AI-extracted snippet
            "score": 0.85,           # relevance score
            "raw_content": "...",    # if include_raw_content set
            "published_date": "...", # if topic="news"
            "favicon": "...",        # if include_favicon set
            "images": [...]          # if include_images set
        }
    ],
    "answer": "...",          # if include_answer set
    "images": [...],          # if include_images set
    "response_time": 1.23,
    "request_id": "..."
}
```

### Examples

```python
# Basic search
response = client.search("latest AI developments")

# News search with date filtering
response = client.search(
    "tech earnings Q1 2025",
    topic="news",
    time_range="month",
    include_answer=True,
    max_results=10
)

# Advanced search with domain filtering
response = client.search(
    "machine learning best practices",
    search_depth="advanced",
    include_domains=["arxiv.org", "papers.nips.cc"],
    include_raw_content=True
)

# Exact match for specific entity lookup
response = client.search(
    '"John Smith" CEO Acme Corp',
    exact_match=True
)
```

---

## client.extract()

```python
response = client.extract(urls="https://example.com")
# or multiple URLs:
response = client.extract(urls=["https://example.com", "https://other.com"])
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `urls` | `str` or `list[str]` | **required** | URL(s) to extract (max 20) |
| `extract_depth` | `str` | `"basic"` | `"basic"` or `"advanced"` (JS-rendered, tables) |
| `format` | `str` | `"markdown"` | `"markdown"` or `"text"` |
| `query` | `str` | `None` | Reranks chunks by relevance to this query |
| `chunks_per_source` | `int` | `3` | 1-5 chunks (only when `query` provided) |
| `include_images` | `bool` | `False` | Include image URLs |
| `include_favicon` | `bool` | `False` | Include favicon URLs |
| `include_usage` | `bool` | `False` | Include credit usage |
| `timeout` | `float` | `None` | 1.0-60.0s (defaults: 10s basic, 30s advanced) |

### Response (dict)

```python
{
    "results": [
        {
            "url": "...",
            "raw_content": "...",    # extracted content (chunks joined by [...] when query set)
            "images": [...],         # if include_images set
            "favicon": "..."         # if include_favicon set
        }
    ],
    "failed_results": [
        {"url": "...", "error": "..."}
    ],
    "response_time": 0.5,
    "request_id": "..."
}
```

### Example

```python
response = client.extract(
    urls=[
        "https://en.wikipedia.org/wiki/Artificial_intelligence",
        "https://en.wikipedia.org/wiki/Machine_learning"
    ],
    query="What are the key breakthroughs?",
    chunks_per_source=3,
    include_images=True
)
```

---

## client.crawl()

```python
response = client.crawl("https://docs.example.com", instructions="Find the API reference")
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` | `str` | **required** | Root URL to begin crawling |
| `max_depth` | `int` | `1` | How deep to explore (1-5) |
| `max_breadth` | `int` | `20` | Links per level (1-500) |
| `limit` | `int` | `50` | Total pages before stopping |
| `instructions` | `str` | `None` | Natural language guidance (doubles mapping cost) |
| `chunks_per_source` | `int` | `3` | 1-5 chunks (only when `instructions` set) |
| `select_paths` | `list[str]` | `None` | Regex patterns for allowed paths |
| `select_domains` | `list[str]` | `None` | Regex for allowed domains |
| `exclude_paths` | `list[str]` | `None` | Regex for excluded paths |
| `exclude_domains` | `list[str]` | `None` | Regex for excluded domains |
| `allow_external` | `bool` | `True` | Follow external domain links |
| `include_images` | `bool` | `False` | Extract images |
| `extract_depth` | `str` | `"basic"` | `"basic"` or `"advanced"` |
| `format` | `str` | `"markdown"` | `"markdown"` or `"text"` |
| `include_favicon` | `bool` | `False` | Include favicons |
| `timeout` | `float` | `150` | 10-150 seconds |
| `include_usage` | `bool` | `False` | Credit usage info |

### Response (dict)

```python
{
    "base_url": "...",
    "results": [
        {
            "url": "...",
            "raw_content": "...",
            "images": [...],
            "favicon": "..."
        }
    ],
    "response_time": 5.2,
    "request_id": "..."
}
```

### Example

```python
response = client.crawl(
    "https://docs.example.com",
    instructions="Find all pages about authentication",
    max_depth=2,
    limit=30,
    select_paths=[r"/docs/auth.*"]
)
```

---

## client.mapping()

```python
response = client.mapping("https://docs.example.com")
```

### Parameters

Same as `crawl()` except no extraction-related params (`extract_depth`, `format`, `include_images`, `chunks_per_source`, `include_favicon`).

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` | `str` | **required** | Root URL to map |
| `max_depth` | `int` | `1` | Exploration depth |
| `max_breadth` | `int` | `20` | Links per level |
| `limit` | `int` | `50` | Total links before stopping |
| `instructions` | `str` | `None` | Natural language guidance (doubles cost) |
| `select_paths` | `list[str]` | `None` | Regex include patterns |
| `select_domains` | `list[str]` | `None` | Regex domain patterns |
| `exclude_paths` | `list[str]` | `None` | Regex exclude patterns |
| `exclude_domains` | `list[str]` | `None` | Regex domain exclusions |
| `allow_external` | `bool` | `True` | Follow external links |
| `timeout` | `float` | `150` | 10-150 seconds |
| `include_usage` | `bool` | `False` | Credit usage info |

### Response (dict)

```python
{
    "base_url": "...",
    "results": ["https://...", "https://...", ...],  # list of discovered URLs
    "response_time": 2.1,
    "request_id": "..."
}
```

### Example

```python
# Map first to discover structure
urls = client.mapping(
    "https://docs.example.com",
    instructions="Find the API reference pages",
    limit=100
)

# Then crawl only what you need
response = client.crawl(
    "https://docs.example.com",
    select_paths=[r"/api/.*"],
    limit=20
)
```

---

## client.research()

Deep multi-step research that automatically searches, reads, and synthesizes a report.

```python
response = client.research("What are the latest developments in quantum computing?")
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `input` | `str` | **required** | Research question or task |
| `model` | `str` | `"auto"` | `"mini"` (focused, 4-110 credits), `"pro"` (deep, 15-250 credits), `"auto"` |
| `stream` | `bool` | `False` | Stream progress via SSE |
| `output_schema` | `dict` | `None` | JSON Schema for structured output |
| `citation_format` | `str` | `"numbered"` | `"numbered"`, `"mla"`, `"apa"`, `"chicago"` |

### Non-streaming workflow

Research is async on the server. The SDK call returns immediately with a pending status, then you poll:

```python
# Start research
response = client.research("your question")
request_id = response["request_id"]

# Poll for completion
import time
while True:
    result = client.get_research(request_id)
    if result["status"] == "completed":
        print(result["content"])   # the report (str or dict if output_schema used)
        print(result["sources"])   # list of {"title", "url", "favicon"}
        break
    elif result["status"] == "failed":
        print("Research failed")
        break
    time.sleep(5)
```

### Streaming

```python
stream = client.research("your question", model="pro", stream=True)
for chunk in stream:
    print(chunk.decode('utf-8'))
```

Stream events arrive as SSE: Planning -> WebSearch (with queries/sources) -> ResearchSubtopic (pro only) -> Generating -> Content chunks -> Sources -> Done.

### Structured output

```python
response = client.research(
    "Compare React vs Vue in 2025",
    output_schema={
        "properties": {
            "comparison": {
                "type": "array",
                "description": "Comparison points",
                "items": {
                    "type": "object",
                    "properties": {
                        "category": {"type": "string", "description": "Comparison category"},
                        "react": {"type": "string", "description": "React assessment"},
                        "vue": {"type": "string", "description": "Vue assessment"}
                    }
                }
            },
            "recommendation": {"type": "string", "description": "Overall recommendation"}
        },
        "required": ["comparison", "recommendation"]
    }
)
```

---

## Async Usage

All methods are available on `AsyncTavilyClient` with the same signatures:

```python
import asyncio
from tavily import AsyncTavilyClient

async def main():
    client = AsyncTavilyClient(api_key="tvly-YOUR_API_KEY")

    # Parallel searches
    results = await asyncio.gather(
        client.search("query 1"),
        client.search("query 2"),
        client.search("query 3"),
        return_exceptions=True
    )

    for r in results:
        if isinstance(r, Exception):
            print(f"Failed: {r}")
        else:
            print(r["results"])

asyncio.run(main())
```

---

## Hybrid RAG Client (Advanced)

Combines Tavily web search with a local MongoDB vector database. Requires `cohere` package.

```python
from pymongo import MongoClient
from tavily import TavilyHybridClient

db = MongoClient("mongodb+srv://YOUR_URI")["YOUR_DB"]

hybrid = TavilyHybridClient(
    api_key="tvly-YOUR_API_KEY",
    db_provider="mongodb",
    collection=db.get_collection("YOUR_COLLECTION"),
    index="YOUR_VECTOR_INDEX",
    embeddings_field="embeddings",
    content_field="content"
)

# Search across web + local DB
results = hybrid.search("your query", max_results=10)
# Returns: [{"content": "...", "score": 0.9, "origin": "local"|"foreign"}, ...]

# Save web results to local DB
results = hybrid.search("your query", save_foreign=True)

# Custom embedding/ranking functions supported via constructor params
```
