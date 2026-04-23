FROM oven/bun:1-alpine AS build

ARG DATABASE_DIRECT_URL
ARG DATABASE_URL

WORKDIR /calcom

# Build tools needed for any remaining native modules (e.g. sharp, @prisma/engines)
RUN apk add --no-cache python3 make g++ openssl openssl-dev

ENV NODE_ENV="production"
ENV NODE_OPTIONS="--max-old-space-size=8192"
ENV PYTHON=/usr/bin/python3
ENV DATABASE_DIRECT_URL=${DATABASE_DIRECT_URL}
ENV DATABASE_URL=${DATABASE_URL}
ENV USE_POOL="true"
# Prevent postinstall from running husky (requires .git which is excluded in .dockerignore)
ENV HUSKY=0

COPY . .

# bun install reads yarn.lock natively; --ignore-scripts skips postinstall (husky, etc.)
# bun also installs a `yarn` shim so internal scripts that call `yarn workspace` still work
RUN bun install --ignore-scripts

RUN bun run --filter '@calcom/api-v2' generate-schemas
RUN bun run --filter '@calcom/api-v2' build:docker
RUN bun run --filter '@calcom/api-v2' build

EXPOSE 80

# Start with node directly — the prod build is a plain JS bundle
CMD ["bun", "run", "--filter", "@calcom/api-v2", "start:prod"]
