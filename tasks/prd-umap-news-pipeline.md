# PRD: UMAP News Clustering Pipeline

## Introduction

A pipeline that aggregates news articles from free sources, clusters them using UMAP + DBSCAN, and generates LLM summaries of each cluster. The goal is to help users stay informed about the news cycle by presenting a digestible overview of the day's major topics without manually reading dozens of articles.

## Goals

- Aggregate news from free sources (RSS feeds, web scraping, free APIs)
- Embed article text using local sentence-transformers models
- Reduce dimensionality with UMAP for clustering
- Identify topic clusters with DBSCAN
- Generate human-readable summaries of each cluster using Ollama
- Present results as a static HTML report for human review
- Support both on-demand and scheduled execution

## User Stories

### US-001: Project setup and dependencies
**Description:** As a developer, I need the project scaffolded with dependencies so I can build the pipeline.

**Acceptance Criteria:**
- [ ] Python project with pyproject.toml or requirements.txt
- [ ] Dependencies: requests, beautifulsoup4, feedparser, sentence-transformers, umap-learn, hdbscan/scikit-learn, ollama, jinja2
- [ ] Basic project structure: src/, tests/, templates/, output/
- [ ] README with setup instructions
- [ ] Typecheck/lint passes (ruff or flake8 configured)

### US-002: News source configuration
**Description:** As a user, I want to configure which news sources to pull from so I can customize my news intake.

**Acceptance Criteria:**
- [ ] Config file (YAML or JSON) for defining sources
- [ ] Support RSS feed URLs
- [ ] Support web scrape targets with CSS selectors
- [ ] Default config with 3-5 free sources (AP, Reuters RSS, BBC, NPR)
- [ ] Typecheck/lint passes

### US-003: RSS feed fetcher
**Description:** As a developer, I need to fetch articles from RSS feeds so I can aggregate news content.

**Acceptance Criteria:**
- [ ] Function to fetch and parse RSS feeds using feedparser
- [ ] Extract: title, link, published date, summary/description
- [ ] Handle fetch errors gracefully (timeout, invalid feed)
- [ ] Return list of article dicts with consistent schema
- [ ] Unit tests for RSS parsing
- [ ] Typecheck/lint passes

### US-004: Web scraper module
**Description:** As a developer, I need to scrape articles from web pages for sources without RSS.

**Acceptance Criteria:**
- [ ] Function to scrape article content given URL and CSS selectors
- [ ] Extract: title, body text, published date (if available)
- [ ] Respect robots.txt and add delays between requests
- [ ] Handle errors gracefully (404, timeout, missing elements)
- [ ] Unit tests with mocked responses
- [ ] Typecheck/lint passes

### US-005: Article aggregator
**Description:** As a developer, I need to combine articles from all sources into a unified dataset.

**Acceptance Criteria:**
- [ ] Load sources from config file
- [ ] Fetch from all RSS and scrape sources
- [ ] Deduplicate by URL
- [ ] Output unified list with fields: id, title, text, url, source, date
- [ ] Typecheck/lint passes

### US-006: Text embedding with sentence-transformers
**Description:** As a developer, I need to embed article text so I can cluster by semantic similarity.

**Acceptance Criteria:**
- [ ] Use sentence-transformers (all-MiniLM-L6-v2 or similar)
- [ ] Embed article title + text combined
- [ ] Return numpy array of embeddings
- [ ] Handle empty or very short texts gracefully
- [ ] Typecheck/lint passes

### US-007: UMAP dimensionality reduction
**Description:** As a developer, I need to reduce embedding dimensions so DBSCAN can cluster effectively.

**Acceptance Criteria:**
- [ ] Apply UMAP to reduce embeddings to 2D or configurable dimensions
- [ ] Configurable UMAP parameters (n_neighbors, min_dist)
- [ ] Return reduced coordinates
- [ ] Typecheck/lint passes

### US-008: DBSCAN clustering
**Description:** As a developer, I need to cluster the UMAP-reduced embeddings to identify topic groups.

**Acceptance Criteria:**
- [ ] Apply DBSCAN to UMAP coordinates
- [ ] Configurable eps and min_samples parameters
- [ ] Assign cluster labels to each article (-1 for noise/outliers)
- [ ] Return articles grouped by cluster
- [ ] Typecheck/lint passes

### US-009: Cluster summarization with Ollama
**Description:** As a user, I want each cluster summarized so I can quickly understand the topic.

**Acceptance Criteria:**
- [ ] Connect to local Ollama instance
- [ ] For each cluster, select representative articles (e.g., 3-5 closest to centroid)
- [ ] Prompt LLM with article titles/excerpts to generate cluster summary
- [ ] Summary includes: topic label, 2-3 sentence description, key themes
- [ ] Handle Ollama connection errors gracefully
- [ ] Typecheck/lint passes

### US-010: Static HTML report generator
**Description:** As a user, I want results as an HTML report so I can review the news clusters easily.

**Acceptance Criteria:**
- [ ] Jinja2 template for HTML report
- [ ] Display each cluster with: summary, article count, list of article titles with links
- [ ] Show UMAP 2D scatter plot visualization (matplotlib or plotly static image)
- [ ] Include generation timestamp
- [ ] Output to output/report-YYYY-MM-DD.html
- [ ] Report is readable and styled (basic CSS)
- [ ] Typecheck/lint passes

### US-011: CLI entry point
**Description:** As a user, I want to run the pipeline from the command line.

**Acceptance Criteria:**
- [ ] CLI command: `python -m umap_pipeline run`
- [ ] Optional flags: --config, --output-dir, --date
- [ ] Runs full pipeline: fetch → embed → UMAP → DBSCAN → summarize → report
- [ ] Prints progress to stdout
- [ ] Typecheck/lint passes

### US-012: Scheduled execution support
**Description:** As a user, I want to schedule the pipeline to run automatically.

**Acceptance Criteria:**
- [ ] Documentation for cron/launchd setup
- [ ] Example cron entry in README
- [ ] Script handles being run non-interactively (no prompts)
- [ ] Logs to file when run scheduled
- [ ] Typecheck/lint passes

## Functional Requirements

- FR-1: Load news sources from a YAML/JSON configuration file
- FR-2: Fetch articles from RSS feeds using feedparser
- FR-3: Scrape articles from web pages using BeautifulSoup
- FR-4: Deduplicate articles by URL before processing
- FR-5: Embed article text using sentence-transformers (local model)
- FR-6: Reduce embeddings to 2D using UMAP
- FR-7: Cluster reduced embeddings using DBSCAN
- FR-8: Generate cluster summaries using Ollama (local LLM)
- FR-9: Render results as static HTML with Jinja2
- FR-10: Provide CLI interface for on-demand execution
- FR-11: Support scheduled execution via cron

## Non-Goals

- No paid API subscriptions (NewsAPI premium, etc.)
- No real-time streaming or live updates
- No user authentication or multi-user support
- No database storage (stateless, processes fresh each run)
- No web server or interactive dashboard (static HTML only)
- No mobile app or push notifications

## Technical Considerations

- Python 3.10+ required
- Ollama must be installed and running locally with a model (llama3, mistral, etc.)
- sentence-transformers downloads models on first run (~100MB)
- UMAP and DBSCAN parameters may need tuning based on article volume
- Web scraping should respect rate limits and robots.txt

## Success Metrics

- Pipeline completes in under 5 minutes for 100 articles
- Clusters are semantically coherent (similar topics grouped together)
- Summaries accurately represent cluster content
- HTML report is readable on desktop browsers
- User can identify top 5-10 news topics of the day in under 2 minutes

## Open Questions

- Which Ollama model works best for summarization? (Start with llama3 or mistral)
- Should we cache embeddings to speed up re-runs?
- How to handle paywalled articles that appear in RSS but require login?
