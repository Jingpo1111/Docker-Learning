# Docker Command Guide

A practical reference for learning Docker with Node.js on Arch Linux.
Every command is explained, in the order you'll usually need it.

---

## Table of Contents

1. [Key Ideas](#1-key-ideas)
2. [Install and Setup](#2-install-and-setup)
3. [Docker Service (Daemon)](#3-docker-service-daemon)
4. [Images](#4-images)
5. [Containers](#5-containers)
6. [docker run Options](#6-docker-run-options)
7. [Building Images (Dockerfile)](#7-building-images-dockerfile)
8. [Inspecting and Debugging](#8-inspecting-and-debugging)
9. [Volumes (Saving Data)](#9-volumes-saving-data)
10. [Networks](#10-networks)
11. [Docker Compose](#11-docker-compose)
12. [Registry (Docker Hub)](#12-registry-docker-hub)
13. [Cleanup](#13-cleanup)
14. [System Info](#14-system-info)
15. [Example Project: Node.js](#15-example-project-nodejs)
16. [Everyday Workflow](#16-everyday-workflow)
17. [Troubleshooting](#17-troubleshooting)

---

## 1. Key Ideas

| Term | Meaning |
|---|---|
| **Image** | A read-only template: an operating system plus your app and its dependencies. Like a recipe. |
| **Container** | A running copy of an image. Like a dish made from the recipe. You can run many from one image. |
| **Dockerfile** | A text file of steps that tells Docker how to build an image. |
| **Volume** | Storage that survives when a container is deleted. |
| **Bind mount** | A folder on your computer linked into a container, so edits show up live. |
| **Port mapping** | Connects a port on your computer to a port in a container (`host:container`). |
| **Network** | Lets containers talk to each other by name. |
| **Compose** | Runs several containers together from one `compose.yaml` file. |
| **Registry** | An online store of images, such as Docker Hub. |
| **Daemon** | The Docker background service. Every `docker` command talks to it. |

---

## 2. Install and Setup

```bash
sudo pacman -Syu
sudo pacman -S nodejs npm docker docker-compose docker-buildx
```
| Package | Purpose |
|---|---|
| `nodejs` / `npm` | Runs JavaScript on your machine / installs Node packages |
| `docker` | The Docker engine and CLI |
| `docker-compose` | Adds the `docker compose` command |
| `docker-buildx` | The image builder; recent Compose versions need it |

On Ubuntu, use instead:
```bash
sudo apt install docker.io docker-compose-v2 docker-buildx
```

Allow your user to run Docker without `sudo`:
```bash
sudo usermod -aG docker $USER
```
- `-aG docker` means append (`-a`) your user to the group (`-G`) named `docker`.
- **Log out and back in** afterwards for this to take effect.

Test everything:
```bash
node -v                  # Node version
npm -v                   # npm version
docker --version         # Docker version
docker compose version   # Compose version
docker run hello-world   # Downloads a tiny test image and runs it
```

---

## 3. Docker Service (Daemon)

```bash
sudo systemctl start docker     # Start Docker now
sudo systemctl stop docker      # Stop Docker
sudo systemctl restart docker   # Restart Docker (e.g. after changing config)
sudo systemctl enable docker    # Start automatically at boot
sudo systemctl disable docker   # Don't start at boot
systemctl status docker         # Check status; look for "active (running)"
```

Shortcut to start now **and** enable at boot:
```bash
sudo systemctl enable --now docker
```

---
## Build the image from your Dockerfile

```bash
docker build -t hello-docker .
````
## 4. Images

```bash
docker pull nginx               # Download an image from Docker Hub
docker pull node:20-slim        # Download a specific version (tag)
docker images                   # List images on your machine
docker image ls                 # Same as above
docker rmi nginx                # Delete an image
docker rmi -f nginx             # Force delete, even if a stopped container uses it
docker image prune              # Delete dangling (untagged) images
docker image prune -a           # Delete all images not used by any container
docker tag myapp myapp:v1       # Give an image an extra name/tag
docker history myapp            # Show the layers an image was built from
docker image inspect myapp      # Show full details as JSON
```

**Tags:** `node:20-slim` means image `node`, tag `20-slim`. With no tag, Docker uses `latest`.

---

## 5. Containers

```bash
docker run nginx                # Create and start a container
docker ps                       # List running containers
docker ps -a                    # List all containers, including stopped ones
docker stop web                 # Stop a container gracefully
docker kill web                 # Stop immediately
docker start web                # Start a stopped container again
docker restart web              # Stop and start
docker pause web                # Freeze the processes
docker unpause web              # Unfreeze
docker rm web                   # Delete a stopped container
docker rm -f web                # Force delete, even if running
docker rename old new           # Rename a container
docker create nginx             # Create a container without starting it
```

You can refer to a container by its **name** (e.g. `web`) or its **ID** (the first few characters are enough, e.g. `a1b2`).

---

## 6. docker run Options

```bash
docker run -d -p 8080:80 --name web --rm nginx
```

| Option | Meaning |
|---|---|
| `-d` | Detached: run in the background |
| `-p 8080:80` | Map port `host:container`; open `localhost:8080` to reach port 80 inside |
| `--name web` | Give the container a name instead of a random one |
| `--rm` | Delete the container automatically when it stops |
| `-it` | Interactive terminal: lets you type into the container |
| `-e KEY=value` | Set an environment variable |
| `--env-file .env` | Load environment variables from a file |
| `-v mydata:/data` | Attach a named volume |
| `-v $(pwd):/app` | Bind mount the current folder into `/app` |
| `-w /app` | Set the working directory inside the container |
| `--network mynet` | Connect to a specific network |
| `--restart unless-stopped` | Restart automatically after crashes or reboots |
| `--memory 512m` | Limit memory |
| `--cpus 1.5` | Limit CPU |

Examples:
```bash
docker run -it --rm node:20-slim bash        # Open a shell in a Node container
docker run -it --rm node:20-slim node        # Open a Node REPL
docker run -d -p 3000:3000 -e NODE_ENV=production myapp
```

---

## 7. Building Images (Dockerfile)

### Example Dockerfile
```dockerfile
FROM node:20-slim          # Base image to start from
WORKDIR /app               # Folder inside the image where the next steps run
COPY package*.json .       # Copy package files first (for caching)
RUN npm install            # Run a command while building
COPY . .                   # Copy the rest of the code
EXPOSE 3000                # Documents which port the app uses
CMD ["node", "app.js"]     # Command run when a container starts
```

| Instruction | Meaning |
|---|---|
| `FROM` | Base image |
| `WORKDIR` | Set the working folder |
| `COPY` | Copy files from your computer into the image |
| `ADD` | Like COPY, but can also extract archives and download URLs |
| `RUN` | Run a command at **build** time |
| `CMD` | Default command at **run** time (can be overridden) |
| `ENTRYPOINT` | Fixed command at run time; CMD becomes its arguments |
| `ENV` | Set an environment variable |
| `ARG` | A variable used only during the build |
| `EXPOSE` | Documents a port (doesn't publish it; `-p` does that) |
| `USER` | Switch to a non-root user |

### .dockerignore
Files listed here are not copied into the image:
```
node_modules
.git
.env
```

### Build commands
```bash
docker build -t myapp .                  # Build from the Dockerfile in this folder
docker build -t myapp:v1 .               # Build with a tag
docker build -f Dockerfile.dev -t myapp .  # Use a different Dockerfile
docker build --no-cache -t myapp .       # Rebuild every step, ignoring the cache
```
- `-t` is the name/tag.
- `.` is the **build context**, the folder Docker can copy files from.

**Layer caching:** Each instruction is a layer. If nothing changed, Docker reuses it.
Copying `package.json` and running `npm install` **before** copying your code means
code-only changes rebuild in seconds.

---

## 8. Inspecting and Debugging

```bash
docker logs web                 # Show a container's output
docker logs -f web              # Follow the logs live (Ctrl+C to stop)
docker logs --tail 50 web       # Show only the last 50 lines
docker exec -it web sh          # Open a shell inside a running container
docker exec -it web bash        # Same, if the image has bash
docker exec web ls /app         # Run one command inside
docker inspect web              # Full details: IP, mounts, environment, etc.
docker top web                  # Processes running inside
docker stats                    # Live CPU / memory usage (Ctrl+C to exit)
docker port web                 # Show the port mappings
docker diff web                 # Files changed inside the container
docker cp web:/app/log.txt .    # Copy a file from the container to your computer
docker cp file.txt web:/app/    # Copy a file into the container
```

Type `exit` to leave a shell opened with `exec`.

---

## 9. Volumes (Saving Data)

Containers are throwaway: deleting one deletes its files. Volumes keep data.

```bash
docker volume create mydata     # Create a named volume
docker volume ls                # List volumes
docker volume inspect mydata    # Show details, including where it's stored
docker volume rm mydata         # Delete a volume
docker volume prune             # Delete all unused volumes
```

Using them:
```bash
docker run -v mydata:/data redis           # Named volume (managed by Docker)
docker run -v $(pwd):/app node:20-slim     # Bind mount (your folder)
docker run -v $(pwd):/app:ro node:20-slim  # Read-only bind mount
```

| Type | Use for |
|---|---|
| **Named volume** | Databases and data you want to keep |
| **Bind mount** | Development: live code editing |

---

## 10. Networks

```bash
docker network ls                       # List networks
docker network create mynet             # Create a network
docker network inspect mynet            # Show which containers are connected
docker network connect mynet web        # Add a container to a network
docker network disconnect mynet web     # Remove it
docker network rm mynet                 # Delete a network
docker network prune                    # Delete unused networks
```

Containers on the same network reach each other **by name**:
```bash
docker network create mynet
docker run -d --name redis --network mynet redis
docker run -d --name web --network mynet -p 3000:3000 myapp
# Inside "web", the database address is: redis:6379
```
Compose creates a network for you automatically.

---

## 11. Docker Compose

Compose runs a whole app (several containers) from one file: `compose.yaml`.

### Example compose.yaml
```yaml
services:
  web:
    build: .                  # Build from the Dockerfile here
    restart: unless-stopped   # Restart after crashes or reboots
    ports:
      - "3000:3000"           # host:container
    environment:
      - NODE_ENV=development
    volumes:
      - ./:/app               # Bind mount: live code editing
      - /app/node_modules     # Keep the container's own packages
    command: node --watch app.js   # Auto-restart when files change
    depends_on:
      - redis                 # Start redis first

  redis:
    image: redis:7            # Use a ready-made image
    volumes:
      - redisdata:/data       # Save the database data

volumes:
  redisdata:
```

### Compose commands
```bash
docker compose up               # Build if needed, start everything, show logs
docker compose up -d            # Same, in the background
docker compose up -d --build    # Rebuild images, then start in the background
docker compose down             # Stop and remove containers and the network
docker compose down -v          # Also delete volumes (data is lost!)
docker compose ps               # List this project's containers
docker compose logs             # Logs from all services
docker compose logs -f web      # Follow logs for one service
docker compose exec web sh      # Shell into a running service
docker compose run --rm web npm test   # Run a one-off command in a new container
docker compose build            # Build images without starting
docker compose pull             # Download the latest images
docker compose stop             # Stop without removing
docker compose start            # Start stopped services again
docker compose restart web      # Restart one service
docker compose up -d --scale web=3     # Run 3 copies of a service
docker compose config           # Check and print the final config
docker compose watch            # Auto-sync/rebuild on file changes (needs a "develop" section)
```

Compose looks for `compose.yaml` (or `docker-compose.yml`) in the current folder.
Use `-f other.yaml` to pick a different file.

---

## 12. Registry (Docker Hub)

Share your images by pushing them to a registry.

```bash
docker login                            # Sign in to Docker Hub
docker tag myapp username/myapp:v1      # Name it as username/image:tag
docker push username/myapp:v1           # Upload
docker pull username/myapp:v1           # Download (on any machine)
docker search nginx                     # Search Docker Hub
docker logout                           # Sign out
```

Moving an image without a registry:
```bash
docker save -o myapp.tar myapp          # Export an image to a file
docker load -i myapp.tar                # Import it on another machine
```

---

## 13. Cleanup

```bash
docker container prune          # Delete all stopped containers
docker image prune              # Delete dangling images
docker image prune -a           # Delete all unused images
docker volume prune             # Delete unused volumes (careful: data!)
docker network prune            # Delete unused networks
docker builder prune            # Delete the build cache
docker system prune             # Delete stopped containers, unused networks, dangling images, cache
docker system prune -a          # Also delete all unused images
docker system prune -a --volumes  # Everything unused, including volumes
```
Add `-f` to skip the "Are you sure?" question.

Stop and delete **all** containers:
```bash
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```
- `-q` prints only IDs; `-a` includes stopped containers.

---

## 14. System Info

```bash
docker version          # Client and server versions
docker info             # Full system info: containers, images, storage, etc.
docker system df        # Disk space used by images, containers, volumes, cache
docker events           # Live stream of Docker events
docker help             # List commands
docker run --help       # Help for one command
```

---

## 15. Example Project: Node.js

### Create the project
```bash
mkdir hello-docker && cd hello-docker
npm init -y                 # Create package.json
npm install express         # Install Express
```

**app.js**
```javascript
const express = require("express");
const app = express();
app.get("/", (req, res) => res.send("Hello! Running on Node " + process.version));
app.listen(3000, "0.0.0.0", () => console.log("Running on port 3000"));
```
`0.0.0.0` makes the app listen on all addresses, which is required inside a container.

**Dockerfile**
```dockerfile
FROM node:20-slim
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
CMD ["node", "app.js"]
```

**.dockerignore**
```
node_modules
```

**compose.yaml**
```yaml
services:
  web:
    build: .
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - ./:/app
      - /app/node_modules
    command: node --watch app.js
```

### Run it three ways
```bash
# 1. Without Docker: uses your laptop's Node
node app.js

# 2. With Docker: uses the container's Node 20
docker build -t hello-docker .
docker run --rm -p 3000:3000 hello-docker

# 3. With Compose: live reload while you edit
docker compose up -d
```
Open `http://localhost:3000` or run `curl http://localhost:3000`.

---

## 16. Everyday Workflow

```bash
docker compose up -d            # Start
# edit app.js → save → it reloads automatically
docker compose logs -f web      # Watch the logs
docker compose up -d --build    # Rebuild ONLY after npm install or Dockerfile changes
docker compose exec web sh      # Look inside
docker compose down             # Stop
```

| You changed... | Do this |
|---|---|
| Code (`app.js`) | Nothing; `node --watch` reloads it |
| Packages (`npm install x`) | `docker compose up -d --build` |
| `Dockerfile` | `docker compose up -d --build` |
| `compose.yaml` | `docker compose up -d` |

---

## 17. Troubleshooting

### "port is already allocated"
Something else is already using the port.
```bash
docker ps                       # Is a container already using it?
docker compose down             # Stop the Compose app
docker stop <name>              # Or stop the container that has it
sudo ss -ltnp | grep 3000       # Is a non-Docker program using it?
kill <PID>                      # Stop that program
```
Or use another host port: `-p 3001:3000`, then open `localhost:3001`.

`ss` options: `-l` listening, `-t` TCP, `-n` numbers, `-p` show program.

### "permission denied ... docker.sock"
```bash
sudo usermod -aG docker $USER   # Then log out and back in
groups                          # Check that "docker" is listed
```

### "Cannot connect to the Docker daemon"
```bash
sudo systemctl start docker
systemctl status docker
```

### Container keeps exiting
```bash
docker compose ps               # Shows "Exited"
docker compose logs web         # Read the error
docker ps -a                    # See the exit code
```

### "compose build requires buildx"
```bash
sudo pacman -S docker-buildx
```

### Changes not showing up
- Code change: check the bind mount (`./:/app`) and `node --watch` in `compose.yaml`.
- Package change: run `docker compose up -d --build`.
- Still stuck: `docker compose build --no-cache`, then `docker compose up -d`.

### Disk is full
```bash
docker system df                # See what's using space
docker system prune -a          # Clean up unused items
```
