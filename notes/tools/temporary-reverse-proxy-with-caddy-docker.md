# Temporary Local Reverse Proxy with Caddy Docker

## What it is

A zero-install, disposable one-liner that uses Caddy inside Docker to forward any local port (e.g., `localhost:8888`) to an external website or domain (e.g., `https://example.com`), while automatically rewriting the HTTP `Host` header.

## Why it matters

During debugging, local development, or quick experiments, you frequently need requests sent to `localhost:<port>` to transparently hit a remote domain or API. 

Common alternatives fall short:
- `/etc/hosts` cannot map ports (it only resolves hostnames to IPs).
- Setting up Nginx requires installing packages and writing config files.
- `socat` forwards raw TCP but keeps the `Host: localhost:<port>` header, which causes modern web servers and CDNs (Cloudflare, Vercel, AWS) to return `400 Bad Request` or `404 Not Found`.

Running Caddy inside a temporary container handles TLS upstream and rewrites the `Host` header on the fly with no persistent config.

## Key idea

Run Caddy in Docker with `--change-host-header` and `--rm` so the proxy cleans up completely the moment you stop it:

```bash
docker run --rm -it -p 8888:80 caddy caddy reverse-proxy --from :80 --to https://example.com --change-host-header
```

## Example

### 1. Quick Interactive Run (Fore-ground)

Forward `localhost:8888` to `https://example.com`:

```bash
docker run --rm -it -p 8888:80 caddy caddy reverse-proxy --from :80 --to https://example.com --change-host-header
```

Test it in another terminal:

```bash
curl -I http://localhost:8888
```

Press `Ctrl+C` to stop and automatically delete the container.

---

### 2. Detached Run (Background for the day)

Start the proxy as a named container:

```bash
docker run -d --name proxy-example -p 8888:80 caddy caddy reverse-proxy --from :80 --to https://example.com --change-host-header
```

Verify it:

```bash
curl -I http://localhost:8888
```

Inspect traffic logs:

```bash
docker logs -f proxy-example
```

Tear down and free the port when finished:

```bash
docker rm -f proxy-example
```

---

### 3. Optional Shell Helper

Add this function to your `~/.bashrc` (or `~/.zshrc`) for instant throwaway proxies:

```bash
proxy-to() {
  local port="${1:-8888}"
  local target="${2:-https://example.com}"
  echo "Forwarding localhost:${port} -> ${target} (Ctrl+C to exit)..."
  docker run --rm -it -p "${port}:80" caddy caddy reverse-proxy --from :80 --to "${target}" --change-host-header
}
```

Usage:

```bash
proxy-to 8888 https://example.com
proxy-to 3000 https://puku.sh
```

## Pitfalls

- **Missing `--change-host-header`**: Without this flag, Caddy passes `Host: localhost:8888` to the target server. Most modern servers (especially behind Cloudflare or Nginx vhosts) will reject the request.
- **Upstream Protocol**: Use `https://` if the target domain requires SSL, or `http://` if it does not.
- **Port Conflicts**: If the port is already bound locally (e.g. `jupyter-notebook` on `8888`), Docker will fail to bind `0.0.0.0:8888`. Check running processes with `ss -tulpn | grep <port>` or use another port like `8889`.

## Related

- [`open-alias-for-xdg-open.md`](./open-alias-for-xdg-open.md) — Shell alias for opening files and URLs
- [`dns-blocking-and-dns-over-tls.md`](../networking/dns-blocking-and-dns-over-tls.md) — DNS resolution and networking notes
