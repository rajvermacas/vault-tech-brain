---
title: "LLM Wiki Pattern"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---LLM-Wiki-Idea-File]]"
tags:
  - methodology
  - knowledge-management
---

# LLM Wiki Pattern

A method for building personal knowledge bases where an LLM incrementally builds and maintains a persistent wiki of interlinked markdown files, rather than relying on [[RAG]]-style retrieval at query time.

## How It Differs From RAG

| Aspect | RAG | LLM Wiki |
|--------|-----|----------|
| Knowledge state | Re-derived per query | Compiled once, kept current |
| Cross-references | Discovered at query time | Pre-built and maintained |
| Contradictions | May go unnoticed | Explicitly flagged |
| Accumulation | None — each query starts fresh | Compounding — wiki grows richer |

## Core Principle

The human curates sources and directs analysis. The LLM handles all bookkeeping — summarizing, cross-referencing, filing, maintenance. The wiki stays maintained because the cost of maintenance is near zero for an LLM.

## Three-Layer Architecture

1. **Raw sources** — immutable input documents
2. **Wiki** — LLM-generated structured pages
3. **Schema** — configuration governing structure and workflows

## Three Operations

- **Ingest** — process new sources into the wiki
- **Query** — search wiki, synthesize answers, optionally file them back
- **Lint** — health-check for contradictions, orphans, staleness

## Use Cases

Personal growth, research, book companioning, business intelligence, competitive analysis, course notes, hobby deep-dives — any domain where knowledge accumulates over time.

## Inspiration

Related to [[Vannevar-Bush]]'s [[Memex]] (1945). The LLM solves the maintenance problem that made Bush's vision impractical.
