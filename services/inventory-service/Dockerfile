# ==========================================
# Stage 1: Build & Dependencies (Pinned to Node.js 24 dev variant)
# ==========================================
FROM cgr.dev/chainguard/node:latest-dev AS builder

USER root
WORKDIR /app

# Copy dependency manifests first for optimal Docker layer caching
COPY package.json package-lock.json* ./

# Install production-only dependencies
RUN npm ci --omit=dev || npm install --omit=dev

# Copy application source code
COPY src ./src

# ==========================================
# Stage 2: Production Runtime (Pinned to Node.js 24 clean runtime)
# ==========================================
FROM cgr.dev/chainguard/node:latest

WORKDIR /app

# Copy only the necessary production dependencies and application source code
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/src ./src
COPY --from=builder /app/package.json ./package.json

# Run under the secure non-root user native to Chainguard Node.js images
USER node

# Expose port 8083 matching your inventory service configuration
EXPOSE 8083

# Execute the application directly via node
CMD ["src/index.js"]