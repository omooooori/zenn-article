# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Zenn article repository managed with Zenn CLI. Zenn is a Japanese tech blog platform. Articles are written in Markdown and deployed to zenn.dev via GitHub integration.

## Commands

```bash
# Preview articles locally (opens http://localhost:8000)
npx zenn preview

# Create new article
npx zenn new:article
npx zenn new:article --slug my-slug --title "Title" --type tech --emoji "🚀"

# Create new book
npx zenn new:book

# Update Zenn CLI
npm install zenn-cli@latest
```

## Structure

- `articles/` - Markdown files for individual articles
- `books/` - Directories for longer-form book content

## Article Format

Articles require YAML front matter:

```yaml
---
title: "Article Title"
emoji: "😸"
type: "tech"  # or "idea"
topics: ["tag1", "tag2"]  # up to 5
published: true  # false for drafts
---
```

Slug constraints: a-z0-9, hyphens, underscores, 12-50 characters.

## Publishing

Push to GitHub to auto-deploy. Use `[ci skip]` in commit messages to skip deployment.

## Commit Guidelines

- Write commit messages in English, one line, concise
- Do NOT include AI stamps or co-author tags
