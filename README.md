# Non-Caching Simple Reverse Proxy

## Overview
This repository provides a containerized Nginx reverse proxy designed to forward HTTP traffic to an upstream HTTPS endpoint. It is explicitly configured to operate without buffering or caching. The proxy relies on environment variables for runtime configuration and supports dynamic DNS resolution.

## Technical Specifications
- **Base Image:** `nginx:1.31.5-alpine`
- **Protocol:** HTTP (Listener) to HTTPS (Upstream)
- **Orchestration:** Docker Compose
- **Configuration Method:** `envsubst` via Nginx templates (`/etc/nginx/templates/`)

## Core Features
- **Disabled Buffering and Caching:** Enforces `proxy_buffering off;` and `proxy_cache off;` to ensure immediate stream processing and real-time data consistency.
- **Dynamic DNS Resolution:** Configures an explicit DNS resolver with a 10-second cache validity (`valid=10s`), enabling automatic adaptation to upstream IP address changes without container restarts.
- **Server Name Indication (SNI):** Enables `proxy_ssl_server_name` and sets `proxy_ssl_name` to ensure correct TLS routing on shared upstream hosts.
- **URI Rewriting:** Prepends a configurable base path to all incoming requests prior to upstream forwarding.
- **Extended Timeouts:** Sets `proxy_read_timeout` and `proxy_send_timeout` to 3600 seconds to accommodate long-lived connections and large payload transfers.

## Environment Variables
Configuration is managed via a `.env` file. An example configuration is provided in `.env.example`.

| Variable | Type | Description | Example |
| :--- | :--- | :--- | :--- |
| `TARGET_HOSTNAME` | String | The domain name of the upstream server. | `api.example.com` |
| `TARGET_PATH` | String | The prefix prepended to the request URI before forwarding. | `/v1` |
| `LISTEN_IP` | String | The local network interface to bind the listener to. | `127.0.0.1` |
| `LISTEN_PORT` | Integer | The local TCP port to bind the listener to. | `8080` |
| `DNS_RESOLVER` | String | The IP address of the DNS server used for upstream resolution. | `8.8.8.8` |

## Deployment

### Prerequisites
- Docker Engine
- Docker Compose

### Execution Steps
1. Clone the repository:
   ```bash
   git clone https://github.com/idashevskii/non-caching-simple-reverse-proxy.git
   cd non-caching-simple-reverse-proxy
   ```

2. Initialize the environment configuration:
   ```bash
   cp .env.example .env
   ```
   Modify the `.env` file to reflect the target infrastructure parameters.

3. Start the containerized proxy:
   ```bash
   docker compose up -d
   ```

## Routing Logic
The proxy intercepts incoming HTTP requests and applies the following transformations:
1. **URI Interception:** Listens on `http://${LISTEN_IP}:${LISTEN_PORT}`.
2. **Path Rewriting:** Modifies the request URI using the regular expression `^(.*)$`, replacing it with `${TARGET_PATH}$1`.
3. **Upstream Resolution:** Resolves `https://${TARGET_HOSTNAME}` using the specified `${DNS_RESOLVER}`.
4. **Forwarding:** Passes the request to the upstream server with modified headers (e.g., `Host` set to `${TARGET_HOSTNAME}`).
5. **Response Streaming:** Returns the upstream response to the client strictly without intermediate memory buffering or disk caching.

## License
This project is licensed under the terms specified in the [LICENSE](LICENSE) file.
