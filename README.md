# tts-docs

The documentation vault and published site for the **tts-service** product.

- `content/` is an **Obsidian vault** — open *this folder* in Obsidian to write notes.
- The site is built by **Quartz v5** and published to GitHub Pages via
  `.github/workflows/deploy.yml`.

## Authoring rules (portable Markdown)

So the same files render correctly in Obsidian, on GitHub, and on the Quartz site:

- Wikilinks **off** — use standard relative links: `[text](../adr/0001-record-architecture-decisions.md)`
- Callouts only: `NOTE`, `TIP`, `IMPORTANT`, `WARNING`, `CAUTION`
- Images: `![alt](./assets/x.png)` with relative paths
- Filenames lowercase-hyphenated; always include `.md` in links

## Publish

See [content/how-to/publish-the-docs-site.md](content/how-to/publish-the-docs-site.md).
In short: push to GitHub, set Pages **Source** to "GitHub Actions", and set `baseUrl` in
`quartz.config.yaml` to `<username>.github.io/tts-docs`.

## Local preview (optional)

```bash
npx quartz plugin install
npx quartz build --serve   # then open http://localhost:8080
```

> On **Windows**, `plugin install` creates symlinks, which need
> **Settings → For Developers → Developer Mode = On** (or an Administrator terminal).
> This is only needed for *local* preview — the GitHub Actions build runs on Linux and is
> unaffected.

Built with [Quartz](https://quartz.jzhao.xyz) (MIT) — see `LICENSE.txt`.
