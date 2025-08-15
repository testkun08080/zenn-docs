# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Zenn documentation repository that contains technical articles written in Japanese. The repository uses Zenn CLI for managing and previewing articles, and also includes Qiita CLI for publishing articles to Qiita platform.

## Repository Structure

- `articles/` - Contains Zenn articles in Markdown format with frontmatter
- `books/` - Directory for Zenn books (currently empty)
- `images/` - Image assets organized by article slug
- `qiita/public/` - Articles formatted for Qiita publication
- `draft/` - Draft articles not yet published

## Article Format

Articles use YAML frontmatter with the following structure:
```yaml
---
title: "Article Title"
emoji: "🐍"
type: "tech" | "idea"
topics: ["python", "fastapi"]
published: true | false
published_at: 2025-06-08 12:00  # Optional
---
```

## Common Commands

### Zenn CLI Commands
```bash
# Create new article
npx zenn new:article
npx zenn new:article --slug article-slug --title "Title" --type idea --emoji ✨

# Create new book
npx zenn new:book

# Preview articles locally
npx zenn preview
```

<!-- ### Qiita CLI Commands
The repository includes @qiita/qiita-cli as a dev dependency for publishing to Qiita. -->

## Development Workflow

1. Articles are written in `articles/` directory using Zenn format
2. Images are stored in `images/{article-slug}/` directories
3. Use `npx zenn preview` to preview articles locally before publishing
4. Qiita versions are maintained separately in `qiita/public/`

## Image Management

- Images are organized by article slug in the `images/` directory
- Reference images in articles using `/images/{article-slug}/filename.png`
- Common images can be stored in `images/common/`