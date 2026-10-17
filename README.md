# breezy-search

A lightweight, browserless, open-source scraper and search engine built for LLM pipelines, with a Firecrawl-compatible API.

> **Status: early alpha.** Core features work on server-rendered pages. See [Limitations](#limitations).

## Why

Most scrapers launch a headless browser for every page: heavy, slow, and hard to self-host. breezy-search fetches with `httpx`, extracts with `trafilatura`, and stores everything in a single SQLite file. No Docker, no Redis, no Postgres, no paid APIs.

## Features

- Clean markdown output (main-content extraction, hidden-element stripping)
- Politely crawls: robots.txt, per-host delay, sitemap discovery, depth/limit/path filters
- Query-aware link scoring (optional `query` guides which links the crawler follows first)
- Heading-path chunks (`Asyncio > Tasks > gather`) ready for RAG
- Local cache with conditional GET (ETag / If-Modified-Since) and change tracking
- `highlights` format: query-focused text extraction
- PDF support (optional), structured error codes
- Optional Playwright renderer via plugin, so the core stays browserless
- Pluggable discovery backends (Wikipedia, Hacker News, arXiv built in; add your own)

## Install

    pip install -e ".[server]"        # core + HTTP server
    pip install -e ".[pdf]"           # optional PDF parsing

## Use

CLI:

    breezy-search scrape https://example.com
    breezy-search crawl https://docs.python.org/3/ --limit 20 --query asyncio
    breezy-search query "how does asyncio gather work" --budget 2000

Python:

    from breezy_search import Breezy

    with Breezy() as b:
        doc = b.scrape("https://example.com", formats=["markdown", "links"])
        print(doc.markdown)

HTTP server (Firecrawl-style):

    breezy-search serve
    curl -X POST localhost:3002/v2/scrape \
      -H "content-type: application/json" \
      -d '{"url":"https://example.com","formats":["markdown"]}'

## API compatibility

| Endpoint | Status |
|---|---|
| `POST /v2/scrape` (`/v1` alias) | Implemented, checked against Firecrawl docs |
| `POST /v2/crawl` + `GET /v2/crawl/{id}` | Implemented; status shape not yet verified |
| `POST /v2/map` | Implemented, checked against Firecrawl docs |
| `POST /v2/search` | Experimental |
| `GET /r/<url>` (Jina-style) | Experimental |
| `POST /breezy/query` | Native: token-budgeted, cited chunks |

## Plugins

Discovery backends and the browser renderer are plugins. Copy `plugins/_template.py` to `plugins/my_backend.py` and implement `search()`. Files in `plugins/` are gitignored except the templates. You are responsible for the terms of service of any backend you add.

## Limitations

- No JavaScript rendering by default. A fallback recovers text from JSON-LD and `__NEXT_DATA__`, which is a hack that misses pure SPAs. Use the Playwright plugin for those.
- No anti-bot evasion or proxy rotation.
- Unsupported formats (`json`, `screenshot`, `summary`, ...) are ignored with a warning.
- Not designed for large-scale distributed crawling.

## License

MIT
