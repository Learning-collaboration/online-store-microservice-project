# --- Stage 1: Build & Compile (Pinned to Go 1.22 dev variant) ---
FROM cgr.dev/chainguard/go:latest-dev AS builder

WORKDIR /app

# Copy dependency manifests first for layer caching
COPY go.mod go.mod* go.sum* ./

# Download Go modules
RUN go mod download

# Copy the rest of the Go source code
COPY . /app

# Compile a fully static Go binary
# CGO_ENABLED=0 ensures no dynamic C library dependencies (like libc) are required at runtime
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o catalog-service .

# --- Stage 2: Production Runtime (Pinned to Chainguard Static/Base) ---
FROM cgr.dev/chainguard/static:latest

WORKDIR /app

# Copy the compiled static binary from the builder stage
COPY --from=builder /app/catalog-service /app/catalog-service

# Run under a secure non-root user (Chainguard static images run as nonroot by default)
USER nonroot:nonroot

# Expose port 8082 matching your Go application configuration
EXPOSE 8082

# Execute the binary directly
ENTRYPOINT ["/app/catalog-service"]