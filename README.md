# Drop of Knowledge

Personal knowledge hub by Bhakti Pradhan.

**Architecture. Intelligence. Engineering.**

## Run locally

Requirements: Ruby, Bundler and Git.

```bash
bundle install
bundle exec jekyll serve
```

Open http://127.0.0.1:4000.

## Deploy

1. Create the public repository `bhakti.github.io`.
2. Add these files to the repository's `main` branch.
3. In Settings → Pages → Build and deployment, select **GitHub Actions**.
4. The workflow in `.github/workflows/pages.yml` builds and deploys the site after every push to `main`.

Site: https://bhakti.github.io/

## Add a post

Create a Markdown file in `_posts` using `YYYY-MM-DD-title.md`, and start it with YAML front matter:

```yaml
---
layout: post
title: "Your article title"
date: 2026-11-21 09:00:00 +0530
categories: [ai, architecture]
description: "A short summary."
---
```

## Publishing note

Review all content before publishing. Do not include confidential employer, customer, personal or proprietary information.
