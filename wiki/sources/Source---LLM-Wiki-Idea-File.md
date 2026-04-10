---
title: "Source - LLM Wiki Idea File"
type: source
created: 2026-04-10
updated: 2026-04-10
sources:
  - "raw/llm-wiki-idea-file.md"
tags:
  - methodology
  - knowledge-management
  - founding-document
---

# Source - LLM Wiki Idea File

## Summary

A design document describing the **LLM Wiki** pattern — a method for building personal knowledge bases where the LLM incrementally builds and maintains a persistent, interlinked wiki of markdown files rather than relying on RAG-style retrieval at query time.

The core insight: most LLM-document workflows (RAG, NotebookLM, ChatGPT uploads) rediscover knowledge from scratch on every query. The LLM Wiki inverts this — knowledge is **compiled once and kept current** through a structured wiki that compounds over time.

## Architecture

Three layers:
1. **Raw sources** — immutable source documents (articles, papers, data). The source of truth.
2. **The wiki** — LLM-generated markdown pages (summaries, entities, concepts, analyses). The LLM owns this layer entirely.
3. **The schema** — a configuration file (e.g. CLAUDE.md) defining structure, conventions, and workflows. Co-evolved by human and LLM.

## Operations

Three core workflows:
- **Ingest** — process a new source, create/update wiki pages, update index and log.
- **Query** — search the wiki, synthesize an answer, optionally file it back as a new page.
- **Lint** — health-check for contradictions, orphans, stale claims, missing pages.

## Key Takeaways

- The wiki is a **persistent, compounding artifact** — not ephemeral retrieval.
- The human curates sources and asks questions; the LLM does all bookkeeping.
- The [[Index-Based-Navigation]] approach (reading index.md to find relevant pages) works well up to ~100 sources / hundreds of pages, avoiding the need for embedding infrastructure.
- Good query answers should be **filed back into the wiki** so explorations compound.
- The pattern applies broadly: personal growth, research, book reading, business intelligence, competitive analysis.
- Inspired by [[Vannevar-Bush]]'s [[Memex]] (1945) — a personal knowledge store with associative trails. The LLM solves the maintenance problem Bush couldn't.

## Connections

- Contrasts with [[RAG]] as an approach to LLM + document workflows.
- Recommends [[Obsidian]] as the browsing/viewing layer.
- Suggests [[qmd]] as an optional search tool at scale.
- References tools: [[Obsidian-Web-Clipper]], [[Marp]], [[Dataview]].
