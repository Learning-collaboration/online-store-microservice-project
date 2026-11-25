# ==========================================
# Stage 1: Builder (Has apk package manager to install extensions & libraries)
# ==========================================
FROM cgr.dev/chainguard/php:latest-dev AS builder

USER root

# Install PHP extensions along with their required system dependencies
RUN apk update && apk add --no-cache \
    php-pdo_pgsql \
    php-pgsql \
    php-curl \
    php-openssl

WORKDIR /app
COPY router.php /app/router.php

# ==========================================
# Stage 2: Production Runtime (Zero apk, locked down and hardened)
# ==========================================
FROM cgr.dev/chainguard/php:latest

WORKDIR /app

# 1. Copy PHP extension binaries and INI configuration files
COPY --from=builder /usr/lib/php /usr/lib/php
COPY --from=builder /etc/php /etc/php

# 2. Copy underlying system libraries required for PostgreSQL (libpq)
COPY --from=builder /usr/lib/libpq.so* /usr/lib/
COPY --from=builder /usr/lib/libpgcommon.so* /usr/lib/
COPY --from=builder /usr/lib/libpgport.so* /usr/lib/

# 3. Copy underlying system libraries required for cURL (libcurl)
COPY --from=builder /usr/lib/libcurl.so* /usr/lib/

# Copy application source code from builder stage and assign ownership to php user
COPY --chown=php:php --from=builder /app /app

# Enforce secure non-root execution (native 'php' user in Chainguard images)
USER php

EXPOSE 8087

# Pass only server arguments; Chainguard's ENTRYPOINT handles the 'php' binary
CMD ["-S", "0.0.0.0:8087", "router.php"]