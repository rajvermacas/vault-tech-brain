---
title: "RAG"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---LLM-Wiki-Idea-File]]"
tags:
  - architecture
  - information-retrieval
---

# RAG (Retrieval-Augmented Generation)

A common pattern for combining LLMs with document collections. The LLM retrieves relevant chunks from a corpus at query time and generates an answer based on them.

## How It Works

1. Documents are split into chunks and embedded into a vector store.
2. At query time, the user's question is embedded and similar chunks are retrieved.
3. Retrieved chunks are passed to the LLM as context for answer generation.

## Limitations (per the LLM Wiki analysis)

- **No accumulation** — knowledge is re-derived from scratch on every query.
- **No pre-built synthesis** — cross-document connections must be discovered each time.
- **Contradictions go unnoticed** — no mechanism to flag when sources disagree.
- **No compounding** — asking good questions doesn't make future queries better.

Products using this pattern include NotebookLM, ChatGPT file uploads, and most enterprise document Q&A systems.

## Contrast

The [[LLM-Wiki-Pattern]] inverts RAG by compiling knowledge into a persistent wiki that grows richer over time, rather than retrieving from raw documents on every query.
