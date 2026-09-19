---
title: "Building This Blog: Hugo, PaperMod, and a Few GitHub Detours"
date: 2026-09-19T09:00:00+02:00
draft: false
tags: ["meta", "hugo", "github-pages"]
---

This site is built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, hosted for free on GitHub Pages, and deployed automatically on every push. Here's how it came together — including the handful of snags that made it interesting.

## The stack

The goal was the easiest possible path to setup *and* ongoing updates. Hugo won out for build speed, PaperMod for a clean, fast, no-nonsense theme, and GitHub Pages because hosting is free and deploys can be fully automated with GitHub Actions.

On the tooling side, [Homebrew](https://brew.sh/) handled installing Hugo and Git — one command, no manual PATH wrangling beyond the standard post-install step.

## The repo name has to match exactly

First real gotcha: for GitHub Pages to serve a site at the root of `username.github.io`, the **repository itself** has to be named exactly `username.github.io`. I'd started with a repo just called `andrejspisiak`, which pushed fine but never went live — no error, it just silently wasn't a "user site" as far as GitHub Pages was concerned. Renaming the repo fixed the whole chain in one move.

## Themes as submodules

PaperMod gets pulled in as a git submodule rather than copied in directly:

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

Skip the `submodule add` step and just clone the theme into the folder, and Git will warn about an "embedded repository" — the theme's own `.git` history gets nested inside the main repo instead of being tracked as a proper reference, which breaks for anyone else who clones the site. The fix is a `.gitmodules` file pointing at the theme's URL, which is what `submodule add` creates for you.

## Automating the deploy

Rather than building locally and pushing the compiled `public/` folder, a GitHub Actions workflow rebuilds and redeploys the site on every push to `main` — using GitHub's official `actions/deploy-pages` action. That's the whole "easy to update" part: write a post, `git push`, done.

Two things tripped up the first deploy:

- **Version mismatch** — the workflow pinned an older Hugo version than PaperMod's current minimum requirement, so the build failed with `hugo v0.146.0 or greater is required`. Bumping the pinned version fixed it.
- **Environment protection rules** — GitHub auto-creates a `github-pages` deployment environment on first use, and it defaulted to only allowing a branch name that no longer matched after renaming `master` to `main`. Widening the allowed branches in the environment settings unblocked it.

## Two GitHub accounts, one Mac

I split personal and work GitHub accounts a while back, which meant SSH needed to know which key to use for which repo. The fix is an alias per account in `~/.ssh/config`:

```
Host github.com-andrejspisiak
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_andrejspisiak
  IdentitiesOnly yes
```

Repos under the personal account point their remote at `github.com-andrejspisiak` instead of plain `github.com`, and SSH picks the right key automatically — no more logging in and out of anything to switch accounts.

## Cleanup

Along the way, a stray `master` branch survived the rename to `main` — left over locally and on GitHub from before the switch. Once confirmed it had no unique commits of its own, it was safe to delete on both ends and set `main` as the actual default branch.

## Where it stands

Toolchain installed, theme wired up, deploys automated, accounts sorted, branches cleaned up — and this post is the first thing published through the whole pipeline, end to end.
