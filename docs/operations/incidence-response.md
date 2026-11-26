## Incident: 500 Internal Server Error on User Registration via Nginx Reverse Proxy

   1. Symptom:
      After deploying the polyglot stack with Docker Compose, POST requests to the registration endpoint (/api/auth/auth/register) consistently failed at the gateway with an HTTP 500 Internal Server Error before ever reaching auth-service.

   2. Root Cause:
      Nginx acts as the API gateway routing incoming browser requests to internal Docker Compose microservices using regular expression location blocks and dynamic proxy_pass targets (incorporating captured URI variables like $1). In Nginx, whenever proxy_pass contains a variable, upstream DNS resolution is deferred from configuration load time to request runtime—bypassing the container's /etc/resolv.conf. Because no explicit DNS resolver was defined, Nginx could not resolve the auth-service container hostname to its internal bridge IP address (no resolver defined to resolve auth-service), triggering an immediate internal 500 failure.

   3. Resolution:
      Configured Docker's embedded DNS server (127.0.0.11) globally in the http context of nginx.conf with a 30-second cache TTL:
      ```Nginx

      http {
        # Use Docker's embedded DNS server for runtime variable resolution in proxy_pass
        resolver 127.0.0.11 valid=30s;

        server {
           listen 80;

           # 1. Auth Service (Java - port 8081) - handles optional duplicate /auth/auth/
           location ~ ^/api/auth(?:/auth)?/(.*)$ {
             proxy_pass http://auth-service:8081/auth/$1$is_args$args;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
           }
        }
      }
      ```
     4. Verification & Impact:
        Reloaded Nginx and re-tested the registration flow. Nginx successfully resolved auth-service dynamically at runtime and forwarded the request upstream, eliminating the 500 gateway errors and preventing stale IP caching when backend containers restart.