# Multi-stage Dockerfile for the frontend (Vite + React/TS)
# Stage 1: build
FROM node:20-alpine AS builder

WORKDIR /app

# Install build dependencies
RUN apk add --no-cache python3 make g++ git

# Enable corepack and prepare pnpm
RUN corepack enable && corepack prepare pnpm@latest --activate

# Copy package manifests first for better caching
COPY package.json pnpm-lock.yaml ./

# Copy the rest of the source
COPY . .

# Install and build
RUN pnpm install --frozen-lockfile
RUN pnpm build

# Stage 2: production image that serves the static build on port 3000
FROM node:20-alpine AS runner
WORKDIR /app

# Lightweight static server. npm is available in the Node image.
RUN npm install -g serve@14.1.2

# Copy built assets from the builder
COPY --from=builder /app/dist ./dist

ENV NODE_ENV=production
EXPOSE 3000

# Serve the static files on port 3000 so it matches traefik/service label expectations
CMD ["serve", "-s", "dist", "-l", "3000"]
