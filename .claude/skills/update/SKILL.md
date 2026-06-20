---
name: update
description: Use when the user runs /update on the kademai-website repo - replaces index.html with the newest akademai landing-page export from ~/Downloads and pushes it straight to main as the akademai account.
---

# update

## Overview

Publishes a freshly exported landing page. Takes the **newest** `*landing-page*.html`
file from `~/Downloads`, overwrites the repo's `index.html` with it, then commits and
pushes directly to `main` — as the **akademai** account (`nir@akademai.ai`), without
touching the user's global `nirkarpati` git setup. No PR. This repo is the GitHub Pages
site, so a push to `main` updates the live site.

## Auth model (already configured — do not "fix" it)

- Repo remote `origin` is `git@github-akademai:nir-aka/kademai-website.git`. The
  `github-akademai` host alias in `~/.ssh/config` uses `~/.ssh/id_ed25519_akademai`,
  which authenticates as the **nir-aka** account → `git push` works.
- Repo-local identity is `nir-aka <nir@akademai.ai>` (so commits display under the
  akademai account). Global identity stays `nirkarpati` — leave it alone.
- Only `git push` is used. No `gh` / GitHub API calls are needed.

## Steps

### 1. Find the newest landing-page export
```bash
SRC=$(find ~/Downloads -maxdepth 1 -type f -iname '*landing-page*.html' \
        -printf '%T@ %p\n' | sort -rn | head -1 | cut -d' ' -f2-)
echo "Selected: $SRC"
```
If `$SRC` is empty, stop and tell the user no `*landing-page*.html` file was found in
`~/Downloads`.

### 2. Confirm the pick
Show the user the selected filename and its modified time. Detection is by mtime, so a
stale file can win — let them confirm before overwriting and pushing to the live site.

### 3. Replace index.html
```bash
cp "$SRC" /home/compute/dev/kademai-website/index.html
git -C /home/compute/dev/kademai-website status --short
```

### 4. Commit and push to main (as akademai)
```bash
cd /home/compute/dev/kademai-website
git checkout main
git add index.html
git commit -m "Update landing page from $(basename "$SRC")"
git push origin main
```
The push uses the SSH alias → authenticates as nir-aka (repo owner). If it fails with
`Permission denied (publickey)`, the public key has not been added to the nir-aka
account yet — see "First-time setup" below. Report the pushed commit hash when done.

## First-time setup (only if push fails with publickey error)
The akademai public key must be registered on the nir-aka GitHub account once:
```bash
cat ~/.ssh/id_ed25519_akademai.pub
```
Have the user paste it at https://github.com/settings/keys **while logged in as the
akademai (nir-aka) account**. Verify with:
```bash
ssh -T git@github-akademai   # expect: "Hi nir-aka! You've successfully authenticated"
```

## Common mistakes
- **Don't** change the global git identity or `gh` account to make this work; the
  per-repo SSH alias + repo-local email is the whole point.
- **Don't** assume the most recently *downloaded* file is right if the user re-exported
  an older page — always confirm the selected filename in step 2.
- Pushing to `main` updates the live GitHub Pages site immediately — confirm in step 2.
