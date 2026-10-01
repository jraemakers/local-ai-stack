# Open WebUI + SearXNG Docker Compose Setup

## Requirements

- **Docker** with the Compose plugin (`docker compose version` should work).
- **Ollama** installed on the host (not in Docker), with at least one model:
  ```bash
  curl -fsSL https://ollama.com/install.sh | sh
  ollama pull qwen3:8b
  ```
- **Linux only:** Ollama must listen on all interfaces so the Open WebUI
  container can reach it:
  ```bash
  sudo systemctl edit ollama
  # add these two lines, save, then:
  #   [Service]
  #   Environment="OLLAMA_HOST=0.0.0.0"
  sudo systemctl restart ollama
  ```
  If you use a firewall (ufw), also see [Troubleshooting](#troubleshooting).
- **Optional:** an NVIDIA GPU with the
  [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).
  The models already run on your GPU through Ollama; this only speeds up
  Open WebUI's own features such as document search and speech-to-text.

## Quick Start

1. Clone the repo:
   ```bash
   git clone git@github.com:jraemakers/local-ai-stack.git
   cd local-ai-stack
   ```

2. Create your `.env` with fresh secrets:
   ```bash
   cp .env.example .env
   sed -i "s/^WEBUI_SECRET_KEY=.*/WEBUI_SECRET_KEY=$(openssl rand -hex 32)/" .env
   sed -i "s/^SEARXNG_SECRET=.*/SEARXNG_SECRET=$(openssl rand -hex 32)/" .env
   ```
   Optional settings in `.env`:
   - `BIND_ADDRESS=0.0.0.0` to reach the web UIs from other devices
     (LAN / Tailscale). The default `127.0.0.1` is this computer only.
   - Uncomment `COMPOSE_FILE=...` to use the NVIDIA GPU version.

3. Start everything:
   ```bash
   docker compose up -d
   ```

4. Verify SearXNG is working:
   ```bash
   curl "http://localhost:8080/search?q=hello&format=json"
   ```

5. Open http://localhost:3000 and create your account. The first account
   becomes the admin.

## Web Search

Web search works out of the box. Click the **Web Search** toggle in a chat
and your model can look things up on the internet.

To change web search settings, like the number of results, go to
**Admin Panel** → **Settings** → **Web Search**.

## Enable Web Search by Default for a Model

To avoid toggling web search on every chat:

1. Go to **Admin Panel** → **Settings** → **Models**
2. Click your model
3. Under **Default Features**, check **Web Search**
4. Save

## Important Notes

- Ollama runs on your PC directly (not in Docker). Open WebUI connects
  to it via `host.docker.internal:11434`.
- Your chats, settings, and users are stored in the `open-webui` volume.
- SearXNG config is in `searxng/settings.yml`.
- Your secrets are in `.env`, which is not committed to git. Keep it: if
  `WEBUI_SECRET_KEY` changes, everyone gets logged out.
- Ports published by Docker bypass ufw, so a ufw rule won't block 3000 or
  8080. Use `BIND_ADDRESS` in `.env` to control who can reach them.

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

Also check that Ollama listens on all interfaces (see [Requirements](#requirements)).

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

## License

[MIT](LICENSE)
