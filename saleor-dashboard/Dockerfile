# Stage 1: Build the Saleor Dashboard
FROM node:18-alpine AS builder

RUN apk add --no-cache bash

WORKDIR /app

# Copy dependency files first for Docker build caching
COPY package.json package-lock.json ./

ENV CI=1

RUN npm ci --legacy-peer-deps

# Copy the files required to build the dashboard
COPY nginx/ ./nginx/
COPY assets/ ./assets/
COPY locale/ ./locale/
COPY scripts/ ./scripts/
COPY src/ ./src/
COPY .featureFlags/ ./.featureFlags/

COPY vite.config.js tsconfig.json schema.graphql ./
COPY *.d.ts ./
COPY introspection*.json ./

# Dashboard build configuration
ARG API_URL
ARG APP_MOUNT_URI
ARG APPS_MARKETPLACE_API_URL
ARG APPS_TUNNEL_URL_KEYWORDS
ARG STATIC_URL
ARG SKIP_SOURCEMAPS
ARG LOCALE_CODE

ENV API_URL=${API_URL:-http://localhost:8000/graphql/} \
    APP_MOUNT_URI=${APP_MOUNT_URI:-/dashboard/} \
    APPS_MARKETPLACE_API_URL=${APPS_MARKETPLACE_API_URL:-https://apps.saleor.io/api/v2/saleor-apps} \
    APPS_TUNNEL_URL_KEYWORDS=${APPS_TUNNEL_URL_KEYWORDS} \
    STATIC_URL=${STATIC_URL:-/dashboard/} \
    SKIP_SOURCEMAPS=${SKIP_SOURCEMAPS:-true} \
    LOCALE_CODE=${LOCALE_CODE:-EN} \
    GENERATE_SOURCEMAP=false \
    NODE_OPTIONS="--max-old-space-size=4096"

RUN sed -i 's/8192/4096/g' package.json
RUN npm run build

# Stage 2: Serve the completed dashboard with Nginx
FROM nginx:stable-alpine

WORKDIR /app

COPY --from=builder /app/build/ /app/
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
COPY nginx/replace-env-vars.sh /docker-entrypoint.d/50-replace-env-vars.sh

RUN chmod +x /docker-entrypoint.d/50-replace-env-vars.sh

EXPOSE 80