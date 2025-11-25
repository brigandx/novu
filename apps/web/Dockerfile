# start build stage
# start build stage
FROM node:20-slim AS builder
RUN apt-get update && apt-get install -y g++ make python3 python3-pip git
ENV NX_DAEMON=false

WORKDIR /usr/src/app


RUN apt-get install -y bash
RUN npm install -g pnpm@10.16.1 --loglevel notice

COPY .npmrc .
COPY package.json .

COPY apps/web ./apps/web
COPY libs ./libs
COPY packages ./packages
COPY enterprise ./enterprise
COPY tsconfig.json .

COPY nx.json .
COPY pnpm-workspace.yaml .
COPY pnpm-lock.yaml .

# Increase memory limit for the build
ENV NODE_OPTIONS="--max-old-space-size=8192"

RUN --mount=type=cache,id=pnpm-store-web,target=/root/.pnpm-store\
  pnpm install --no-frozen-lockfile

RUN pnpm add @babel/core -w

RUN CI='' pnpm build:web --skip-nx-cache
# end build stage

# start production stage
FROM node:20-slim

RUN apt-get update && apt-get install -y bash
RUN npm install -g pnpm@10.16.1 http-server --loglevel notice

USER 1000
WORKDIR /app

COPY --chown=1000:1000 --from=builder /usr/src/app/apps/web/env.sh /app/env.sh

COPY --chown=1000:1000 --from=builder /usr/src/app/apps/web/build /app/build
COPY --chown=1000:1000 --from=builder /usr/src/app/apps/web/.env.sample /app/.env.sample
COPY --chown=1000:1000 --from=builder /usr/src/app/apps/web/package.json /app/package.json

RUN pnpm run envsetup:docker
CMD [ "pnpm", "start:static:build" ]
# end production stage
