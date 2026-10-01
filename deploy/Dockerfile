# God's Eye View — self-hosted build for the openclaw (OCI arm64) box.
# Built by deploy/docker-compose.yml with the repo root as context.
FROM node:24-bookworm-slim

WORKDIR /app

# Puppeteer is only used by upstream's QA scripts; never download Chrome here.
# Cesium makes the production build memory-hungry on small ARM cores.
ENV PUPPETEER_SKIP_DOWNLOAD=1 \
    NODE_OPTIONS=--max-old-space-size=3072

# vite + the provider middleware deps are devDependencies, so install everything.
COPY package.json package-lock.json ./
RUN npm ci --no-audit --no-fund

COPY . .

# Client-exposed keys are baked into the bundle by Vite's `define` at build
# time (they are visible in the browser by design — restrict them provider-side).
# Server-side keys are NOT build args; they arrive at runtime via env_file.
ARG GOOGLE_MAPS_API_KEY=
ARG CESIUM_ION_TOKEN=
RUN GOOGLE_MAPS_API_KEY="$GOOGLE_MAPS_API_KEY" CESIUM_ION_TOKEN="$CESIUM_ION_TOKEN" npm run build \
 && mkdir -p .gev-cache .gev-logs \
 && chown node:node .gev-cache .gev-logs

USER node
ENV HOST=0.0.0.0 \
    PORT=4173
EXPOSE 4173

# `vite preview` serves dist/ plus every /api provider (configurePreviewServer);
# the dev-only key-setup panel is not registered under preview.
CMD ["npx", "vite", "preview", "--host", "0.0.0.0", "--port", "4173", "--strictPort"]
