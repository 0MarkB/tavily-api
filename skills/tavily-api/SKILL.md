---
name: tavily-api
description: Use the Tavily API (search, extract, crawl, map, research) via Python or JavaScript SDK. Trigger this skill whenever the user wants to search the web via Tavily, extract content from URLs, crawl websites, map site structure, or run deep research tasks using the Tavily API. Also trigger when you see imports of `tavily` or `@tavily/core`, or the user mentions Tavily by name, or asks you to write code that integrates Tavily into their project.
---

# Tavily API Skill

This skill enables you to write code that uses the Tavily API through its official SDKs. Tavily provides web search, content extraction, site crawling, site mapping, and deep research capabilities.

## Step 1: Detect Available Runtimes

Before loading any SDK reference, check what's actually installed:

1. Run `python --version` (or `python3 --version`) to check for Python
2. Run `node --version` to check for Node.js/JavaScript

## Step 2: Choose Which SDK Reference to Load

Apply this logic in order:

1. **User explicitly asked for a language** (e.g., "use the JS SDK", "write this in Python") — load only that reference file. If that runtime isn't available, tell the user.

2. **User didn't specify a preference:**
   - If Python is available, read `references/python-sdk.md` and use that.
   - If Python is NOT available but Node.js is, read `references/js-sdk.md` and use that.
   - If neither runtime is available, **ask the user** before proceeding:
     - Option A: Install Python or Node.js first **(Recommended)** — the SDKs handle auth, retries, and response parsing for you
     - Option B: Proceed with raw HTTP calls — read `references/raw-api.md`

     Wait for the user's choice. Do not default to raw API without asking.

3. **Fallback on failure:** If you wrote Python code and it fails due to environment issues (missing pip, broken install, etc.), and Node.js IS available, read `references/js-sdk.md` and retry with JavaScript. If JS also fails or isn't available, ask the user the same choice as above (install a runtime vs. raw API).

Only read ONE reference file. Do not read both unless the user explicitly asks for multiple approaches.

## Step 3: API Key

The Tavily API key is required. It has the prefix `tvly-`. Check for it in this order:

1. Environment variable `TAVILY_API_KEY` — check if it's already set
2. If not set, ask the user for their API key
3. Keys are available at app.tavily.com (free tier: 1,000 credits/month, no credit card)

## Step 4: Write the Code

Follow the instructions in whichever reference file you loaded. Key things to keep in mind regardless of language:

### Available Operations
| Operation | What it does | Credit cost |
|-----------|-------------|-------------|
| **Search** | Web search with AI-ranked results | 1 (basic) or 2 (advanced) per request |
| **Extract** | Pull content from specific URLs | 1 (basic) or 2 (advanced) per 5 URLs |
| **Crawl** | Traverse a website from a starting URL | mapping + extraction combined |
| **Map** | Discover URLs across a site (no content extraction) | 1 per 10 pages (2 with instructions) |
| **Research** | Deep multi-step research on a topic | 4-250 credits depending on model |

### Best Practices

**Search:**
- Keep queries under 400 characters — concise, focused
- Split complex questions into multiple separate searches
- Start with `search_depth="basic"` (1 credit); only use `"advanced"` (2 credits) when high relevance matters
- Use `topic="news"` for current events (adds `published_date` to results)
- Use `time_range` or `start_date`/`end_date` to filter by recency
- `include_answer=True` gets an LLM-generated summary; `"advanced"` for a detailed one
- Filter results by `score` (higher = more relevant, e.g., keep `> 0.7`)
- `auto_parameters=True` is convenient but may silently upgrade to advanced (2 credits)

**Extract:**
- Use the `query` parameter to rerank chunks by relevance (avoids dumping entire pages)
- Control volume with `chunks_per_source` (1-5, needs `query` set)
- Basic depth is fine for simple text; use `"advanced"` for JS-rendered pages or tables
- Batch up to 20 URLs per call
- Failed extractions are free

**Crawl:**
- Always set a `limit` to cap pages and costs
- Start conservative: `max_depth=1`, `max_breadth=20`, `limit=20`
- Use `instructions` for semantic filtering (but doubles mapping cost)
- **Map first, then crawl** — discover structure cheaply, then do targeted extraction
- Use `select_paths`/`exclude_paths` regex patterns to stay focused

**Map:**
- Use before crawling to understand site structure
- Returns just URLs (cheap: 1 credit per 10 pages)
- Use `instructions` only when you need semantic filtering (doubles cost)

**Research:**
- `model="mini"` for focused narrow questions (4-110 credits)
- `model="pro"` for complex multi-subtopic research (15-250 credits)
- Use `stream=True` for real-time progress
- Use `output_schema` to get structured JSON output instead of markdown

### Rate Limits
| Endpoint | Dev RPM | Prod RPM |
|----------|---------|----------|
| Search / Extract | 100 | 1,000 |
| Crawl | 100 | 100 |
| Research | 20 | 20 |

On 429 errors, respect the `retry-after` header.
