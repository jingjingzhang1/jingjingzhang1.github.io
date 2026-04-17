---
title: "Click2GO — Autonomous Multi-Agent Travel Planner"
excerpt: "A LangGraph-orchestrated multi-agent system that scrapes Xiaohongshu, AI-verifies every location, and clusters POIs into geographically optimized daily routes — split across three specialized agents (Knowledge, Route, Design) with conditional supervisor routing.<br/><br/><code>FastAPI</code> &nbsp;<code>LangGraph</code> &nbsp;<code>Multi-Agent</code> &nbsp;<code>K-Means</code> &nbsp;<code>MCP</code> &nbsp;<code>SQLAlchemy</code> &nbsp;<code>Folium</code><br/><br/><a href='/portfolio/click2go/' style='display:inline-block; margin-top:6px; padding:8px 18px; background:#0066cc; color:#fff; border-radius:4px; font-weight:600; text-decoration:none;'>View Project Details →</a>"
collection: portfolio
---

<a href="https://github.com/jingjingzhang1/Vibe-Route-Planner" class="btn btn--info" target="_blank">
  <i class="fab fa-github"></i> View on GitHub
</a>
&nbsp;
<span class="btn btn--warning">🎬 Demo coming soon</span>

---

## Overview

> *Plan perfectly. Arrive curious.*

Click2GO is a multi-agent AI travel planner that scrapes real social media posts from **Xiaohongshu (Red Note)**, runs every location through an AI verification pipeline, clusters them into geographically optimized daily routes, and generates an interactive map with a styled PDF itinerary — all from a single request.

The headline upgrade in this version is a full re-architecture from a single monolithic agent into a **three-agent pipeline** orchestrated by a **LangGraph Supervisor**. Each agent owns a distinct domain (data, logistics, presentation), with conditional routing between them.

---

## Multi-Agent Architecture

```
START
  │
  ▼
┌────────────────────┐
│  Agent 1           │   Knowledge Manager
│  Knowledge Manager │   • Persona-aware Xiaohongshu scraping
└─────────┬──────────┘   • AI verification (status / season / persona fit)
          │              • POI cache with freshness + coverage checks
          ▼
   ┌─────────────┐
   │ Sufficiency │ ──retry──▶ Agent 1 (broader queries)
   │   Check     │ ──force──▶ Agent 2 (proceed with what we have)
   └──────┬──────┘
          │ ok
          ▼
┌────────────────────┐
│  Agent 2           │   Route Optimizer
│  Route Optimizer   │   • K-Means day clustering
└─────────┬──────────┘   • Greedy nearest-neighbour sorting
          │              • Transit gap filling via read-only Postgres MCP
          ▼
┌────────────────────┐
│  Agent 3           │   Design & UI Agent
│  Design & UI Agent │   • Folium interactive map (category-coded markers)
└─────────┬──────────┘   • ReportLab branded PDF itinerary
          │
          ▼
         END
```

---

## Technical Challenges

### 1. Conditional supervisor routing with retry loops

Splitting the pipeline into independent agents only works if the supervisor can react to *outcomes*, not just sequence steps. The Knowledge Manager doesn't always return enough verified POIs on the first pass — niche personas, weak destinations, or scraper rate limits can produce a thin result set.

The supervisor evaluates a **sufficiency check** between agents: it requires at least `max(days * 2, 4)` included POIs. If the threshold isn't met, it loops back to the Knowledge Manager with broader queries. If two attempts still fall short, it forces the pipeline forward rather than failing — better to deliver a sparse plan than no plan. This is implemented as a `StateGraph` with explicit conditional edges in LangGraph.

### 2. The persona-category cache contamination bug

The hardest bug uncovered during the multi-agent split was a cache invariant violation. POIs were being tagged with the full set of *user-selected personas* (e.g., `["chilling", "foodie"]`) rather than the POI's *actual category*. A coffee shop scraped under a "chilling + foodie" request got tagged with both — so when a later "foodie only" request hit the cache, that coffee shop matched and showed up as a foodie recommendation.

The fix required two changes: introducing a true `category` field set at scrape time to the specific persona that sourced each POI, and adding a **persona coverage check** to the cache layer — the cache is bypassed unless every requested persona category is represented, even when the total POI count is sufficient. This forces a fresh scrape when a user adds a new travel style instead of silently reusing mistagged data.

### 3. Hybrid MCP / ORM read-write boundary

The Route Optimizer needs to *read* from `poi_cache` to find filler POIs for transit gaps > 3km, but it must never *write* — all itinerary writes go through a strict SQLAlchemy ORM layer with validated schemas. To enforce this, reads go through a **read-only Postgres MCP server** that statically blocks `INSERT`, `UPDATE`, `DELETE`, and `DROP` at the tool boundary, while writes go through hand-rolled ORM functions. This split means the agent can use natural-language SQL for flexible lookups without ever risking a destructive write.

### 4. Geographic day clustering + intra-day routing

Each day's plan is two algorithms stacked: **K-Means** partitions all POIs into `num_days` geographic clusters (one per day), then a **greedy nearest-neighbour heuristic** sorts within each cluster — starting from the northernmost POI as a natural "morning start" — to minimize backtracking. Transit gaps between consecutive stops are detected via Haversine distance and filled with cached nearby POIs to keep walking distances reasonable.

### 5. Seed database merge without clobbering user state

The repo ships a pre-scraped `seed_database.sqlite` of POIs for popular cities, but the user's local `click2go.db` contains their own itineraries and chat history. On startup, if the seed file is newer, only `poi_cache` rows are merged forward — `planning_sessions`, `pois`, and `itinerary_days` are left untouched. This lets me ship better cache data via Git without ever overwriting a user's plan.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Web framework | FastAPI (async, BackgroundTasks) |
| Agent orchestration | LangGraph StateGraph (Supervisor pattern) |
| AI verification | OpenAI gpt-4o-mini (structured JSON output) |
| Routing algorithms | scikit-learn (KMeans) + NumPy |
| Map generation | Folium (Leaflet.js, custom DivIcon markers) |
| PDF generation | ReportLab |
| Database | SQLAlchemy 2.0 + SQLite (seed-database strategy) |
| Read-only data access | Postgres MCP server (statically blocks writes) |
| Social scraping | Xiaohongshu MCP server (Docker) |
| Frontend | Vanilla HTML/CSS/JS — no build step |
| Testing | pytest + FastAPI TestClient (58 tests) |

---

## Graceful Degradation

Every external dependency has a fallback path so the full pipeline runs end-to-end with **zero API keys configured**:

| Dependency | Fallback |
|---|---|
| Xiaohongshu MCP scraper | Curated mock POIs (8 templates per persona) |
| OpenAI API | Neutral verification (`is_open: true`, score `7.0`) |
| Google Maps geocoding | 60+ city lookup table with fuzzy substring matching |
| ReportLab | Plain text `.txt` export |
| Folium | GeoJSON export |
