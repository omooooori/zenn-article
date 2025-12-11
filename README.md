# zenn-article

Zenn articles and books managed with Zenn CLI.

## Requirements

- Node.js 14+

## Setup

```bash
# Install dependencies
npm install

# Initialize (already done for this repo)
npx zenn init
```

## Zenn CLI Commands

### Preview Articles Locally

```bash
npx zenn preview
```

Opens a local preview server at `http://localhost:8000`

Options:
- `--port 3000` - Use a custom port
- `--no-watch` - Disable auto-reload on file changes

### Create New Article

```bash
npx zenn new:article
```

Options:
- `--slug my-article` - Custom slug (a-z0-9, hyphens, underscores, 12-50 chars)
- `--title "Article Title"` - Set the title
- `--type tech` - Article type (`tech` or `idea`)
- `--emoji "🚀"` - Set the emoji

### Create New Book

```bash
npx zenn new:book
npx zenn new:book --slug my-book
```

### Update Zenn CLI

```bash
npm install zenn-cli@latest
```

## Article Front Matter

Articles in `articles/` directory use this format:

```yaml
---
title: "Article Title"
emoji: "😸"
type: "tech"  # "tech" or "idea"
topics: ["android", "kotlin"]  # up to 5 tags
published: true  # false for drafts
published_at: 2025-01-01 12:00  # optional, for scheduled publishing
---
```

## Book Structure

Books go in `books/` directory:

```
books/
└── my-book/
    ├── config.yaml
    ├── cover.png  # 500x700px recommended
    ├── chapter1.md
    └── chapter2.md
```

**config.yaml:**

```yaml
title: "Book Title"
summary: "Description"
topics: []  # up to 5 tags
published: true
price: 0  # 0 for free, 200-5000 for paid
chapters:
  - chapter1
  - chapter2
toc_depth: 2  # 0-3
```

## Publishing

Push to GitHub to auto-deploy. Add `[ci skip]` or `[skip ci]` to commit messages to skip deployment.

## References

- [Zenn CLI Guide](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [Install Zenn CLI](https://zenn.dev/zenn/articles/install-zenn-cli)
