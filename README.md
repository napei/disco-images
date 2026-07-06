# Container images for disconnected environments

This repo builds container images of open-source **apps and tools** for use in
internet-restricted (air-gapped) environments. Each image is a self-contained,
minimal build — a static bundle served by nginx — so it's easy to transport into
those environments as a single artifact.

Sibling in spirit to a docs-focused build repo, but for runnable tooling rather
than documentation sites.

> NOTE: This repo is entirely unaffiliated with any of the upstream projects built
> here. Images are produced from each project's public source, under that project's
> own license (e.g. Mermaid Live Editor is MIT). No endorsement is implied.

## How it works

One directory per image, each with a `Dockerfile` and an nginx `default.conf`.
The upstream source is **not vendored** — the GitHub Actions workflow checks it out
at a pinned ref, builds it, and pushes the result to GHCR. A daily schedule keeps
images fresh; `workflow_dispatch` lets you build a single image on demand.

## Images

### `mermaid-live-editor`

An ad-free build of the Mermaid Live Editor. The upstream project bakes all config
into its JS bundle at build time and ships its public image with promotional
banners and upsell links enabled; this build turns them off:

- Mermaid Chart promo banners + upsell links: **disabled**
- Analytics beacon: **disabled**
- External `mermaid.ink` PNG/SVG links: **removed** (unreachable when disconnected)
- Kroki export: pointed at the **relative** path `/kroki`, which nginx reverse-proxies
  to an in-cluster `kroki` service (see below). Because it's same-origin, the image is
  portable across hosts/namespaces with no rebuild, and diagram share links (which are
  just the browser URL) work on whatever hostname serves it.

**Run it:** serves on `:8080` (non-root nginx, so it runs under restricted container
platforms). Diagrams render in-browser with no dependencies. To make the **Kroki**
export button work, run a `kroki` service (image `yuzutech/kroki`, with the
`yuzutech/kroki-mermaid` companion) reachable at `http://kroki:8000` from the editor
pod's network; the editor proxies `/kroki/*` to it. Without a `kroki` service the
editor still works — only the Kroki export link is inert.

**Runtime dependencies (optional, for Kroki export only):** these are *not* built
here — pull them from upstream (mirror into your registry for disconnected use):
- `yuzutech/kroki` — the Kroki gateway (`:8000`)
- `yuzutech/kroki-mermaid` — the Mermaid companion the gateway delegates to (`:8002`,
  wired via `KROKI_MERMAID_HOST`)

There are no build-time dependencies to source separately; the upstream editor source
is fetched by CI at build.

Built from upstream `master` (rebuilt daily). Each image is tagged with the exact
upstream package version and short commit SHA, plus `latest`.
