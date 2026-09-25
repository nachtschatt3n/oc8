# syntax=docker/dockerfile:1
# Build context: frontend workspace (`./frontend` from the repository root).
FROM oven/bun:1 AS build
WORKDIR /app
COPY package.json bun.lock bunfig.toml ./
RUN bun install --frozen-lockfile
COPY . .
# Relative API base -> the same image runs on any server, no rebuild, no CORS.
ARG VITE_API_URL=/api/v1
# node-server preset: the repo default is cloudflare-module, which a container
# cannot run. Both env names are set because the effective one differs across
# TanStack Start / nitro versions; the build asserts the result below.
ENV VITE_API_URL=$VITE_API_URL \
    SERVER_PRESET=node-server \
    NITRO_PRESET=node-server
RUN bun run build && \
    test -f .output/server/index.mjs && \
    grep -q '"preset": "node-server"' .output/nitro.json

# Runtime needs only the node binary to run the nitro server -- not npm. The
# stock node image bundles a global npm whose own dependency tree lags upstream
# security fixes, so start from plain Debian and copy node alone. Same Debian
# release as the node image so glibc matches.
FROM debian:bookworm-slim AS runtime
RUN apt-get update && apt-get upgrade -y --no-install-recommends && \
    rm -rf /var/lib/apt/lists/*
COPY --from=node:22-bookworm-slim /usr/local/bin/node /usr/local/bin/node
RUN node --version
WORKDIR /app
COPY --from=build /app/.output ./.output
ENV PORT=3000
EXPOSE 3000
CMD ["node", ".output/server/index.mjs"]
