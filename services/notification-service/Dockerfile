# ==========================================
# Stage 1: Build & Dependencies
# ==========================================
FROM ruby:3.3-slim AS builder

# Install system build tools and PostgreSQL development headers (Debian equivalent of apk packages)
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY Gemfile Gemfile.lock* ./

# Explicitly install the exact Bundler version matching your lockfile
RUN gem install bundler -v 4.0.20

# Install gems locally into vendor/bundle
RUN bundle config set --local path 'vendor/bundle' && \
    bundle config set --local without 'development test' && \
    bundle install

# Copy application source code
COPY . /app

# ==========================================
# Stage 2: Production Runtime
# ==========================================
FROM ruby:3.3-slim

# Install only the lightweight runtime PostgreSQL library (libpq5) required by the pg gem
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# 1. Copy the compiled local gems from the builder stage
COPY --from=builder /app/vendor/bundle ./vendor/bundle
COPY --from=builder /app/.bundle ./.bundle

# 2. Copy application files (`app.rb`, Gemfile, Gemfile.lock)
COPY --from=builder /app/app.rb ./app.rb
COPY --from=builder /app/Gemfile ./Gemfile
COPY --from=builder /app/Gemfile.lock ./Gemfile.lock

# 3. Tell Bundler explicitly where to find vendor/bundle at runtime
ENV BUNDLE_PATH="/app/vendor/bundle"

# Create a secure non-root user and set permissions
RUN useradd -u 1000 ruby && chown -R ruby:ruby /app
USER ruby

# Expose the port and run Puma
EXPOSE 8086

CMD ["bundle", "exec", "ruby", "app.rb"]