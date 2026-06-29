# Trolley-X project blog

A minimal Jekyll site (minima theme) for the PDE4445 dissertation blog, hosted on GitHub Pages.

## Go live (one time)

1. Create a new GitHub repository (e.g. `trolley-x-blog`) and push the contents of this folder to it.
2. In the repo: **Settings → Pages → Build and deployment → Source: Deploy from a branch**, pick `main` and `/ (root)`, save.
3. Wait ~1 minute. Your blog is live at `https://<your-username>.github.io/trolley-x-blog/`.
4. Submit that link to the module as your blog link.

## Add a new post

Create a file in `_posts/` named `YYYY-MM-DD-title.md` with this front-matter:

```
---
layout: post
title: "Your title"
date: 2026-07-08
author: Your Name
tags: [software, testing]
---
```

Then write the body in Markdown. Posts appear on the home page automatically, newest first.

## Images / video

- Put images in `assets/images/` and reference them as `![caption](/assets/images/your-file.jpg)`.
- For video, embed a YouTube link or upload to the repo and link it.

## Run locally (optional)

```
bundle install
bundle exec jekyll serve
```
Then open `http://localhost:4000`.

## Cadence & integrity

- Two posts per week (Stream A progress log + Stream B deep dive) — see the blog plan.
- The blog is summative (40%). Write posts in your own voice; if your module permits AI assistance, declare its use and the prompts.

## Notes

- The four starter posts in `_posts/` are **draft scaffolds** — headings and prompts are in place; replace the italic prompts and image placeholders with your own words and media.
