# God's Eye View — public self-hosted image for the openclaw (OCI arm64) box.
#
# Why not upstream's root Dockerfile: it runs the dev server and trusts the
# container gateway for Provider Settings, which is only safe while published on
# loopback. This image serves the production build with `vite preview`, where the
# key-entry panel is not registered at all.
#
# Built by deploy/docker-compose.yml with the repo root as context, so upstream's
# root .dockerignore applies and keeps .env, .git and local state out of the image.
FROM node:24.22.2-slim

# Puppeteer is only used by upstream's QA scripts; never download Chrome here.
ENV PUPPETEER_SKIP_DOWNLOAD=1 \
    NODE_OPTIONS=--max-old-space-size=3072

WORKDIR /app
RUN chown node:node /app
USER node

# vite and the provider middleware deps are devDependencies, so install everything.
COPY --chown=node:node package.json package-lock.json ./
RUN npm ci --no-audit --no-fund

COPY --chown=node:node . .

# Client-exposed tokens are baked into the bundle by Vite's `define` at build
# time (visible in the browser by design; restrict them at the provider).
# Server-side keys are NOT build args; they arrive at runtime via env_file.
ARG GOOGLE_MAPS_API_KEY=
ARG CESIUM_ION_TOKEN=
ARG MAPILLARY_CLIENT_TOKEN=
RUN GOOGLE_MAPS_API_KEY="$GOOGLE_MAPS_API_KEY" \
    CESIUM_ION_TOKEN="$CESIUM_ION_TOKEN" \
    MAPILLARY_CLIENT_TOKEN="$MAPILLARY_CLIENT_TOKEN" \
    npm run build \
 && mkdir -p .gev-cache .gev-logs

ENV NODE_OPTIONS= \
    HOST=0.0.0.0 \
    PORT=4173
EXPOSE 4173

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD ["node", "-e", "fetch('http://127.0.0.1:4173/').then((r)=>process.exit(r.ok?0:1),()=>process.exit(1))"]

CMD ["npx", "vite", "preview", "--host", "0.0.0.0", "--port", "4173", "--strictPort"]
