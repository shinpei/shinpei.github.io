# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal Jekyll blog hosted on GitHub Pages at `https://shinpei.github.io`. Pushing to `master` triggers automatic build and deploy.

## Tech Stack

- **Jekyll 3.10** (GitHub Pages supports Jekyll 3.x only — do not upgrade to 4.x)
- **Ruby 3.2.0** via rbenv
- **Bundler 4.x** via rbenv
- **GA4** for analytics, configured via `google_analytics_id` in `_config.yml`

## Commands

Always prefix with `rbenv exec` to use the correct Ruby/Bundler version.

```bash
rbenv exec bundle install
rbenv exec bundle exec jekyll serve --baseurl ""
rbenv exec bundle exec rake post title="Post Title"
```

`--baseurl ""` is required locally — `_config.yml` sets `baseurl: "/"` which causes double slashes without it.

## Creating Posts

Filename format: `_posts/YYYY-MM-DD-title.md`

```yaml
---
layout: post
title: "Post Title"
date: 2026-04-21 00:00:00 +0900
comments: false
tags: []
---
```

## Architecture

- `_layouts/default.html` — base layout, injects analytics only when `jekyll.environment == 'production'`
- `_includes/analytics.html` — GA4 gtag.js snippet
- `_config.yml` — site settings, permalink structure (`/blog/:year/:month/:day/:title`)
- `css/main.scss` — imports from `_sass/` partials
- `_pages/` — static pages, included via `include: [_pages]` in `_config.yml`
