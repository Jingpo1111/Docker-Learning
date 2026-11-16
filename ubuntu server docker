# Deploy from Docker Hub to Ubuntu Server

Build the image on the laptop, push it to Docker Hub, then pull and run it on the server.
The server doesn't need the source code, only the image.

- Docker Hub user: `jingpophai`
- Server: `jingpo@192.168.18.39`
- Image: `jingpophai/hello-docker:v1`

> Run commands **one line at a time**. Copy only the command, not the comments.

---

## 1. On the laptop: build and push

```bash
cd ~/hello-docker
```

```bash
docker login -u jingpophai
```

```bash
docker build -t jingpophai/hello-docker:v1 .
```

```bash
docker push jingpophai/hello-docker:v1
```

Check that it's there at **hub.docker.com → Repositories → hello-docker → Tags**.

---

## 2. On the server: pull and run

```bash
ssh jingpo@192.168.18.39
```

```bash
mkdir -p ~/services/hello-docker
```

```bash
cd ~/services/hello-docker
```

```bash
nano compose.yaml
```

Paste this:

```yaml
services:
  web:
    image: jingpophai/hello-docker:v1
    restart: unless-stopped
    ports:
      - "127.0.0.1:3000:3000"
    logging:
      options:
        max-size: "10m"
        max-file: "3"
```

Save with **Ctrl+O**, then **Enter**, then exit with **Ctrl+X**.

| Line | Meaning |
|---|---|
| `image:` | Download this image from Docker Hub (there's no `build:` line, so nothing is built on the server) |
| `restart: unless-stopped` | Restart after crashes and reboots, like pm2 |
| `127.0.0.1:3000:3000` | Only the server itself (and the Cloudflare Tunnel) can reach the app |
| `logging` | Keep at most 3 log files of 10 MB each |

### Log in (private repository only)

If the repository is **private**, log in on the server first. For a public one, skip this.

```bash
docker login -u jingpophai
```

At `Password:`, paste your **access token** with **Ctrl+Shift+V**.

### Start it

```bash
docker compose pull
```

```bash
docker compose up -d
```

### Check it

```bash
docker compose ps
```

STATUS should say **Up**.

```bash
curl http://localhost:3000
```

Your Cloudflare Tunnel then serves it at your domain as before.

---

## 3. Updating to a new version

### On the laptop

```bash
docker build -t jingpophai/hello-docker:v2 .
```

```bash
docker push jingpophai/hello-docker:v2
```

### On the server

Change `v1` to `v2` in `compose.yaml`:

```bash
cd ~/services/hello-docker
```

```bash
nano compose.yaml
```

Then:

```bash
docker compose pull
```

```bash
docker compose up -d
```

Compose replaces the old container with the new version.

### Going back

If `v2` has a problem, change the tag back to `v1` in `compose.yaml`, then:

```bash
docker compose up -d
```

### Clean up old images

```bash
docker image prune -a
```

---

## 4. Useful commands (on the server)

| Command | What it does |
|---|---|
| `docker compose ps` | Show status |
| `docker compose logs -f web` | Follow the logs (Ctrl+C to exit) |
| `docker compose restart` | Restart the app (same version) |
| `docker compose stop` | Stop it |
| `docker compose start` | Start it again |
| `docker compose down` | Stop and remove the container |
| `docker images` | List downloaded images |

Always run `docker compose` commands **inside the folder** that has `compose.yaml`.

---

## 5. Troubleshooting

### `not found` when pulling

```
failed to resolve reference "docker.io/jingpophai/...": not found
```

The image name in `compose.yaml` doesn't match Docker Hub. Check:

- **Name:** it must match exactly, e.g. `jingpophai/hello-docker`.
- **Tag:** with no tag, Docker uses `:latest`. If you pushed `:v1`, write `:v1`.
- **Private repo:** run `docker login -u jingpophai` on the server first.

On the laptop, see what you pushed:

```bash
docker images | grep jingpophai
```

### `port is already allocated`

Something else is already using port 3000.

```bash
docker ps
```

Stop the other container (`docker rm -f <name>`), or stop the old project with `docker compose down` in its folder.

### `incorrect username or password` on login

- Use an **access token** from **app.docker.com/settings → Personal access tokens**, not your password.
- Paste with **Ctrl+Shift+V**, not Ctrl+V.
