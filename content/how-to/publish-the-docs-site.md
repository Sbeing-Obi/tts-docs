---
title: How to publish the docs site
tags: [how-to, docs, quartz]
---

# How to publish the docs site

Goal: get this vault online as a website at `https://<username>.github.io/tts-docs`, and
keep it updated automatically on every push.

## One-time setup

1. **Set your site URL.** In `quartz.config.yaml`, change `baseUrl` from the placeholder to
   your real Pages URL (no `https://`, no trailing slash):

   ```yaml
   baseUrl: <username>.github.io/tts-docs
   ```

2. **Create the GitHub repo and push.**

   ```bash
   git remote add origin https://github.com/<username>/tts-docs.git
   git push -u origin main
   ```

3. **Enable Pages.** On GitHub → repo **Settings → Pages → Source** → choose
   **"GitHub Actions"**. (Do *not* pick "Deploy from a branch".)

That's it. The workflow in `.github/workflows/deploy.yml` runs on every push to `main`,
builds Quartz on Linux, and deploys to Pages.

## Everyday flow

Write notes in Obsidian (open the `content/` folder as the vault), then:

```bash
git add -A
git commit -m "docs: <what changed>"
git push
```

Within a minute or two the live site updates. You can also use `npx quartz sync` which
commits, pulls, and pushes in one step.

## Local preview (optional)

```bash
npx quartz plugin install
npx quartz build --serve   # http://localhost:8080
```

> [!WARNING]
> On **Windows**, `npx quartz plugin install` creates symlinks and will fail with
> `EPERM: operation not permitted, symlink` unless you enable
> **Settings → For Developers → Developer Mode**, or run the terminal as Administrator.
> This affects **local preview only** — the GitHub Actions deploy runs on Linux, where
> symlinks work, so publishing is unaffected either way.

## Related

- [Decision: separate Quartz docs vault](../adr/0005-separate-quartz-docs-vault.md)
- [Docs home](../index.md)
