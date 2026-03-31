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