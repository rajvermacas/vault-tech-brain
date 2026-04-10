---
title: "Index-Based Navigation"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---LLM-Wiki-Idea-File]]"
tags:
  - architecture
  - methodology
---

# Index-Based Navigation

A technique used in the [[LLM-Wiki-Pattern]] where the LLM reads a curated `index.md` file to locate relevant wiki pages before answering a query, rather than using embedding-based vector search.

## How It Works

1. `index.md` catalogs every wiki page with a link and one-line summary.
2. When answering a query, the LLM reads the index to identify relevant pages.
3. The LLM then reads those specific pages for full context.

## Scale

Works well at moderate scale (~100 sources, hundreds of wiki pages). At larger scale, dedicated search tools like [[qmd]] supplement or replace index-based lookup.

## Advantages

- No embedding infrastructure required.
- The index is human-readable — useful for browsing in [[Obsidian]].
- The LLM can reason about page relevance from summaries, not just keyword/vector similarity.
