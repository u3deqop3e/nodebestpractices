# Node Best Practices - Multi-stage Dockerfile Example

FROM node:20-alpine AS base

WORKDIR /app

COPY package*.json ./

FROM base AS dependencies
# Use --omit=dev instead of deprecated --only=production
RUN npm ci --omit=dev

FROM base AS build
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS release
WORKDIR /app
ENV NODE_ENV=production

USER node

COPY --chown=node:node --from=dependencies /app/node_modules ./node_modules
COPY --chown=node:node --from=build /app/dist ./dist
COPY --chown=node:node package*.json ./

EXPOSE 3000

CMD ["node", "dist/index.js"]