# Open WebUI + SearXNG Docker Compose Setup

## Quick Start

1. Clone the repo:
   ```bash
   git clone git@github.com:jraemakers/local-ai-stack.git
   cd local-ai-stack
   ```

2. If you already have Open WebUI and SearXNG running, stop and remove them:
   ```bash
   docker stop open-webui && docker rm open-webui
   docker stop searxng && docker rm searxng
   ```

3. Start everything:
   ```bash
   docker compose up -d
   ```

4. Verify SearXNG is working:
   ```bash
   curl "http://localhost:8080/search?q=hello&format=json"
   ```

5. Open http://localhost:3000 and create your account.

## Web Search

Web search is configured automatically, so there's nothing to set up in the UI:

- `searxng/settings.yml` turns on SearXNG's JSON output, which Open WebUI needs.
- The `environment` section in `docker-compose.yml` turns on web search in
  Open WebUI and points it at SearXNG (5 results, 10 concurrent requests).

These values are only defaults. If you change web search settings in
**Admin Panel** → **Settings** → **Web Search** and hit Save, the saved
values take priority over `docker-compose.yml` from then on.

## Enable Web Search by Default for a Model

To avoid toggling web search on every chat:

1. Go to **Admin Panel** → **Settings** → **Models**
2. Click your model (e.g. Qwen 2.5 14B)
3. Under **Default Features**, check **Web Search**
4. Save

## Important Notes

- Ollama runs on your PC directly (not in Docker). Open WebUI connects
  to it via `host.docker.internal:11434`.
- Your chats, settings, and users are stored in the `open-webui` volume.
- SearXNG config is in `searxng/settings.yml`.

## Troubleshooting

### Open WebUI shows no models (can't reach Ollama)

Symptoms: the model list is empty, and `docker logs open-webui` shows
`open_webui.routers.ollama ... Connection error` on every request.

Cause: the firewall blocks port 11434, so the Open WebUI container can't
reach Ollama on the host.

Fix: open port 11434 for Docker's networks. Use `insert 1` so the rule
goes above any existing DENY rule for that port (ufw uses the first rule
that matches):

```bash
sudo ufw insert 1 allow from 172.16.0.0/12 to any port 11434 proto tcp
```

Check it from inside the container:
```bash
docker exec open-webui python3 -c 'import urllib.request;print(urllib.request.urlopen("http://host.docker.internal:11434/api/tags",timeout=5).read()[:200])'
```

Ollama itself must also listen on all interfaces (`OLLAMA_HOST=0.0.0.0`
in the ollama systemd service), not just on 127.0.0.1.

## Updating

```bash
docker compose pull
docker compose up -d
```

Your data and settings are safe in the volumes.

## Stopping

```bash
docker compose down
```

To stop AND delete all data (careful!):
```bash
docker compose down -v
```