---
name: gh-username-verification
description: Use when writing a GitHub username into a published install command, plugin marketplace path, manifest homepage, README clone URL, or any other public URL where the wrong account handle yields a 404.
---

# Verify GitHub Handles Before Publishing URLs

Run `gh auth status` before baking a GitHub username into any install command, marketplace source path, manifest homepage, or public README URL.

## The Trap

A person's display name, domain, and GitHub handle are often different. Only the GitHub handle belongs in:

- `/plugin marketplace add <username>/<repo>`
- `"url": "https://github.com/<username>"` in a manifest
- `"homepage": "https://github.com/<username>/<repo>"` in plugin metadata
- clone URLs
- links to related repositories

## The Check

Run:

```bash
gh auth status
```

The output includes the authenticated GitHub account handle. Use that handle for repos owned by the authenticated user.

If the repo will be owned by an organization or a different person, ask explicitly. Do not infer from an email address, website domain, display name, or local folder name.

## Why It Matters

Broken install commands are easy to miss before publishing and annoying for users to debug afterward. A one-command check prevents 404s in READMEs, plugin manifests, and setup guides.

## Real-World Use

During the first `daves-digi-doodads` release, the maintainer caught a mismatch between an assumed short handle and the authenticated GitHub handle before publishing the README install command. Without the check, the public plugin instructions would have pointed at a nonexistent account.
