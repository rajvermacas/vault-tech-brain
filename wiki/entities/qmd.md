---
title: "qmd"
type: entity
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---LLM-Wiki-Idea-File]]"
tags:
  - tool
  - search
---

# qmd

A local search engine for markdown files with hybrid BM25/vector search and LLM re-ranking. Runs entirely on-device. Available as both a CLI tool and an MCP server.

## Role in the LLM Wiki

Supplements [[Index-Based-Navigation]] at scale. When the wiki grows beyond what the index file can efficiently serve (~hundreds of pages), qmd provides proper full-text and semantic search that the LLM can invoke directly.
