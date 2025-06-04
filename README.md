# go2web

A simple command-line web client written in Python that can:
- Make HTTP/HTTPS requests
- Cache GET responses to speed up repeated requests
- Perform DuckDuckGo searches and display results
- Follow redirects up to a configurable limit
- Handle basic JSON and HTML formatting in the terminal

This README explains how to install, configure, and use **go2web**.

---

## Table of Contents

1. [Project Overview](#project-overview)  
2. [Features](#features)  
3. [Prerequisites](#prerequisites)  
4. [Installation](#installation)  
5. [Usage](#usage)  
   - [Making HTTP Requests](#making-http-requests)  
   - [Searching with DuckDuckGo](#searching-with-duckduckgo)  
   - [Accessing a Link From Search Results](#accessing-a-link-from-search-results)  
6. [Caching Behavior](#caching-behavior)  
7. [File Structure](#file-structure)  
8. [Extending or Contributing](#extending-or-contributing)  
9. [License](#license)  

---

## Project Overview

`go2web` is a minimalistic HTTP client and search tool you can run from the terminal. It is designed for quick lookups of webpages and basic JSON APIs. Key goals:

- **Lightweight**: No external HTTP libraries—uses Python’s built-in `socket` (and `ssl` for HTTPS).  
- **Caching**: GET requests are cached in `~/.go2web_cache/` so you don’t re-download the same resource within a configurable time window.  
- **Search Integration**: Query DuckDuckGo, parse the results with BeautifulSoup, and show titles and URLs.  
- **Redirect Handling**: Follows HTTP redirects (301, 302, 303, 307, 308) up to 5 times by default.  
- **Basic Formatting**: Prints out parsed HTML text (stripped of tags) or prettified JSON for API endpoints.  

This tool is especially handy if you need to quickly inspect an endpoint or search term directly from a terminal without a full browser.

---

## Features

- **`make_http_request`**  
  - Performs low-level HTTP/HTTPS requests over a socket.  
  - Handles request headers, decompresses `gzip`/`deflate`, and decodes character sets.  
  - Automatically follows up to 5 redirects.  
  - Caches 200-OK GET responses (JSON or HTML) in `~/.go2web_cache`.  

- **`search(term, search_engine="duckduckgo")`**  
  - Sends a DuckDuckGo query (using the HTML-only endpoint).  
  - Parses result titles and real URLs (unshortening DuckDuckGo’s intermediate links).  
  - Returns up to 10 results as `(title, url)` tuples.  

- **`format_html_content(content)`**  
  - Strips raw HTML tags and prints page text in a readable format.  
  - Lists all `<a>` links at the bottom with numeric indexes.  

- **`format_json_content(content)`**  
  - Pretty-prints JSON blobs with 2-space indentation if valid JSON.  

- **Command-Line Interface**  
  ```bash
  go2web -u <URL> [--html | --json]
  go2web -s "<search query>"
  go2web --link <number>

  # 1) Simple HTTP request (auto-detect HTML/JSON):
go2web -u https://jsonplaceholder.typicode.com/posts/1

# 2) Force JSON pretty-print:
go2web -u https://jsonplaceholder.typicode.com/posts/1 --json

# 3) Force HTML formatting (strip tags and list links):
go2web -u https://example.com --html

# 4) DuckDuckGo search:
go2web -s "python socket programming"

# 5) Fetch the 3rd result from your last search:
go2web --link 3

## Caching Behavior
- Cached responses are stored in `~/.go2web_cache/`.
- Filenames are derived from the URL (with unsafe characters replaced by underscores).
- By default, a cached GET response expires after 3600 seconds (1 hour).
- You can change the TTL by editing the `max_age` parameter in `Cache.get(...)`.

