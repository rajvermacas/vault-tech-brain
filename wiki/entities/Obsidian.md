---
title: "Obsidian"
type: entity
created: 2026-04-10
updated: 2026-04-10
sources:
  - "[[Source---LLM-Wiki-Idea-File]]"
tags:
  - tool
  - knowledge-management
---

# Obsidian

A markdown-based note-taking application with a strong focus on interlinking (wikilinks) and a graph view for visualizing connections between notes.

## Role in the LLM Wiki

Serves as the **browsing and viewing layer** — the human reads the wiki in Obsidian while the LLM maintains it. Key features used:

- **Graph view** — visualize wiki structure, spot orphans and hubs.
- **Wikilinks** — `[[Page Name]]` syntax for cross-references.
- **[[Obsidian-Web-Clipper]]** — browser extension for capturing articles as markdown sources.
- **[[Dataview]]** plugin — dynamic queries over page frontmatter.
- **[[Marp]]** plugin — slide deck generation from markdown.
