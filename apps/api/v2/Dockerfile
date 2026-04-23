FROM node:20-alpine AS build

ARG DATABASE_DIRECT_URL
ARG DATABASE_URL

WORKDIR /calcom

RUN set -eux;
RUN ln -s /usr/lib/libssl.so.3 /lib/libssl.so.3
RUN apk add --no-cache python3 make g++

ENV NODE_ENV="production"
ENV NODE_OPTIONS="--max-old-space-size=8192"
ENV PYTHON=/usr/bin/python3
ENV DATABASE_DIRECT_URL=${DATABASE_DIRECT_URL}
ENV DATABASE_URL=${DATABASE_URL}
ENV USE_POOL="true"
# Prevent postinstall from running husky (requires .git which is excluded in .dockerignore)
ENV HUSKY=0

COPY . .

RUN npm install -g bun
RUN bun install --ignore-scripts
RUN bunx prisma@6.16.1 generate --schema=packages/prisma/schema.prisma
RUN bun --filter @calcom/platform-constants run build && \
    bun --filter @calcom/platform-enums run build && \
    bun --filter @calcom/platform-utils run build && \
    bun --filter @calcom/platform-types run build && \
    bun --filter @calcom/platform-libraries run build && \
    bun --filter @calcom/trpc run build:server
RUN bun --filter @calcom/api-v2 run build

EXPOSE 80

CMD [ "bun", "--filter", "@calcom/api-v2", "run", "start:prod"]
