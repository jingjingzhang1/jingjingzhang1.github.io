---
title: "Click2GO — Agentic Travel Intelligence Engine"
excerpt: "A multi-stage, agent-driven travel planning system that transforms unstructured social content from Xiaohongshu into intelligently curated itineraries, validated by Claude AI and optimized via K-Means clustering.<br/>"
collection: portfolio
---

<a href="https://github.com/jingjingzhang1/Vibe-Route-Planner" class="btn btn--info" target="_blank">
  <i class="fab fa-github"></i> View on GitHub
</a>
&nbsp;
<span class="btn btn--warning">🎬 Demo coming soon</span>

---

## Overview

Click2GO is a multi-stage, agent-driven travel planning system that transforms unstructured social content into intelligently curated itineraries. The platform scrapes authentic traveler narratives from **Xiaohongshu (Red Note)**, then routes each extracted location through a dedicated **Claude AI agent** for semantic validation, relevance scoring, and real-time status verification.

Validated points of interest are subsequently processed through **K-Means clustering** to algorithmically construct an optimized, day-by-day itinerary that balances geography, density, and travel efficiency. The final output is delivered as both a professionally formatted **PDF** and an **interactive HTML map** for dynamic exploration.

To ensure traceability, reproducibility, and robust debugging, every session, point of interest (POI), and AI verification result is persistently stored in **SQLite** using the SQLAlchemy ORM layer.

---

## Key Features

- **Agentic Location Validation** — Each POI is passed through a Claude Opus 4.6 agent that performs semantic relevance scoring and real-time status verification
- **Intelligent Clustering** — K-Means algorithm groups locations by geography and density to construct optimized day-by-day itineraries
- **Dual Output Formats** — Generates a professionally formatted PDF itinerary and an interactive Folium HTML map
- **Full Session Traceability** — All sessions, POIs, and AI verification results are persisted via SQLAlchemy + SQLite for reproducibility and debugging
- **REST API Backend** — FastAPI powers the full pipeline, containerized with Docker for easy deployment

---

## Technology Stack

| Layer | Technologies |
|---|---|
| Backend | FastAPI · Docker |
| AI Agent | Claude Opus 4.6 |
| Clustering | K-Means (scikit-learn) |
| Database | SQLAlchemy · SQLite |
| Output | ReportLab (PDF) · Folium (Map) |

---

## Architecture

```
Xiaohongshu Content
        ↓
  Content Scraper
        ↓
  Claude AI Agent  ←→  Semantic Validation + Relevance Scoring
        ↓
  K-Means Clustering  ←→  Geography + Density Optimization
        ↓
  ┌─────────────┐
  │  PDF Report │
  │ Interactive │
  │     Map     │
  └─────────────┘
        ↓
  SQLite (SQLAlchemy) — Full session & POI persistence
```
