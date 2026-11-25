# --- Stage 1: Build & Bundle ---
FROM cgr.dev/chainguard/node:latest-dev AS builder

WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci || npm install
COPY . /app
RUN npm run build

# --- Stage 2: Temporary Debug Runtime ---
FROM nginx:alpine

# Copy compiled static assets from builder stage
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy your custom Nginx routing config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 8080
EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]