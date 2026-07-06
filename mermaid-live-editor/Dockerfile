# Ad-free build of the Mermaid Live Editor.
#
# The workflow checks the upstream source out into ./mermaid-src (pinned tag), so the
# build context is this directory. All MERMAID_* knobs are Vite-inlined at build time,
# so they are set here as ENV (loadEnv picks up MERMAID_*-prefixed process env):
#   MERMAID_RENDERER_URL=""        removes the external mermaid.ink PNG/SVG links.
#   MERMAID_KROKI_RENDERER_URL="/kroki"  same-origin path -> nginx proxies it to the
#                                  in-cluster kroki service; no host baked in, so no
#                                  rebuild when the route/namespace changes.
#   MERMAID_IS_ENABLED_MERMAID_CHART_LINKS unset  hides all promo/upsell (flag is falsy).
#   MERMAID_ANALYTICS_URL / MERMAID_DOMAIN unset  no analytics beacon.

FROM node:24-alpine AS builder
RUN apk --no-cache add build-base git python3 && rm -rf /var/cache/apk/*
RUN corepack enable pnpm
WORKDIR /app
COPY mermaid-src/ ./
ENV MERMAID_RENDERER_URL="" \
    MERMAID_KROKI_RENDERER_URL="/kroki" \
    MERMAID_BASE_PATH=""
RUN pnpm install --frozen-lockfile && pnpm build

# nginx-unprivileged: listens on :8080 as a non-root UID with writable pid/cache, so it
# runs under restricted container SCCs (arbitrary UID + GID 0).
FROM nginxinc/nginx-unprivileged:1.28-alpine
COPY default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/docs /usr/share/nginx/html
EXPOSE 8080
