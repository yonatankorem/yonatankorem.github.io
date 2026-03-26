---
name: yonatankorem.github.io
status: active
repo: https://github.com/yonatankorem/yonatankorem.github.io
tags: [claude-code, agentic-workflows, github-pages]
---

This site is a personal experiment in agentic development — built and evolved entirely through Claude Code workflows.

## What it is

A static GitHub Pages site with no build step, no framework, and no bundler. All routing, rendering, and content loading happens in a single `index.html` file, driven by hash-based navigation and `fetch()` calls to markdown files.

## How it's built

Every feature — from the initial design to this very page — was planned and implemented by AI agents working from structured PRDs and phased implementation plans. The workflow looks like this:

1. Write a PRD describing what to build
2. Generate a phased implementation plan
3. Implement each phase on its own branch
4. Review, approve, and merge

## Why

To demonstrate what it looks like when agentic workflows own not just code generation, but the full software lifecycle: architecture, content, design iteration, and ongoing maintenance.
