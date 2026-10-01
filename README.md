# Open WebUI + SearXNG Docker Compose Setup

## Quick Start

1. Copy `docker-compose.yml` to a folder on your machine.

2. If you already have Open WebUI and SearXNG running, stop and remove them:
   ```bash
   docker stop open-webui && docker rm open-webui
   docker stop searxng && docker rm searxng
   ```

3. Start everything:
   ```bash
   docker compose up -d
   ```

4. Enable JSON format in SearXNG (only needed once):
   ```bash
   docker exec searxng sed -i '/- html/a\    - json' /etc/searxng/settings.yml
   docker restart searxng
   ```

5. Verify SearXNG is working:
   ```bash
   curl "http://localhost:8080/search?q=hello&format=json"
   ```

6. Open http://localhost:3000 and create your account.

## Configure Web Search in Open WebUI

After logging in, you need to set up web search in the UI:

1. Go to **Admin Panel** → **Settings** → **Web Search**
2. Toggle **Web Search** ON
3. Set **Web Search Engine** to **searxng**
4. Set **Searxng Query URL** to: `http://searxng:8080/search?q=<query>`
5. Set **Search Result Count** to **5**
6. Set **Concurrent Requests** to **10**
7. Hit **Save**

This only needs to be done once. Settings persist across updates.

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
- SearXNG config is stored in the `searxng_data` volume.

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