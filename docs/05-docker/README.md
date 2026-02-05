# Containerization (Docker)

[← Back to Index](../../README.md)

This section covers Docker fundamentals including images, containers, networking, volumes, Compose, and best practices.

---

<details>
<summary><strong>Q1: What is Docker and how does it differ from Virtual Machines?</strong></summary>

Docker is a platform for developing, shipping, and running applications in **containers** - lightweight, standalone packages that include everything needed to run an application.

| Aspect | Containers | Virtual Machines |
|--------|------------|------------------|
| **Architecture** | Share host OS kernel | Full OS per VM |
| **Size** | MBs (lightweight) | GBs (heavy) |
| **Startup** | Seconds | Minutes |
| **Isolation** | Process-level | Hardware-level |
| **Performance** | Near-native | Overhead from hypervisor |
| **Resource usage** | Efficient | Resource-intensive |

**Key Insight:** Containers virtualize the OS, VMs virtualize hardware.

</details>

<details>
<summary><strong>Q2: Explain the Docker architecture</strong></summary>

Docker uses a **client-server architecture**:

| Component | Role |
|-----------|------|
| **Docker Client** | CLI tool (`docker` command) that sends commands to daemon |
| **Docker Daemon** | Background service that manages containers, images, networks |
| **Docker Registry** | Stores Docker images (Docker Hub, ECR, GCR) |
| **containerd** | Container runtime that manages container lifecycle |
| **runc** | Low-level runtime that creates/runs containers |

**Flow:** Client → Daemon → containerd → runc → Container

</details>

<details>
<summary><strong>Q3: What is the difference between an Image and a Container?</strong></summary>

| Image | Container |
|-------|-----------|
| Read-only template | Running instance of an image |
| Blueprint/Class | Object/Instance |
| Stored on disk | Lives in memory |
| Immutable | Mutable (has writable layer) |
| Shareable | Isolated process |

```bash
# Image is the recipe, container is the cake
docker pull nginx          # Download image
docker run nginx           # Create container from image
docker ps                  # List running containers
docker images              # List images
```

</details>

<details>
<summary><strong>Q4: Explain Docker image layers and caching</strong></summary>

Docker images are built in **layers**. Each instruction in a Dockerfile creates a new layer. Layers are:

- **Cached** - unchanged layers are reused
- **Shared** - common base layers shared between images
- **Read-only** - only the top container layer is writable

```dockerfile
FROM node:18-alpine     # Layer 1: Base image
WORKDIR /app            # Layer 2
COPY package*.json ./   # Layer 3
RUN npm install         # Layer 4 (cached if package.json unchanged)
COPY . .                # Layer 5
CMD ["npm", "start"]    # Layer 6
```

**Best Practice:** Order instructions from least to most frequently changing to maximize cache hits.

</details>

<details>
<summary><strong>Q5: What are the most important Dockerfile instructions?</strong></summary>

| Instruction | Purpose |
|-------------|---------|
| `FROM` | Base image (required, must be first) |
| `RUN` | Execute commands during build |
| `COPY` | Copy files from host to image |
| `ADD` | Like COPY but can extract tars and fetch URLs |
| `WORKDIR` | Set working directory |
| `ENV` | Set environment variables |
| `EXPOSE` | Document which ports the container listens on |
| `CMD` | Default command when container starts (overridable) |
| `ENTRYPOINT` | Main executable (not easily overridden) |
| `ARG` | Build-time variables |
| `USER` | Set non-root user |
| `HEALTHCHECK` | Container health monitoring |

**CMD vs ENTRYPOINT:**
```dockerfile
ENTRYPOINT ["python", "app.py"]  # Fixed executable
CMD ["--port", "8080"]           # Default arguments (overridable)
```

</details>

<details>
<summary><strong>Q6: What is a multi-stage build and why use it?</strong></summary>

Multi-stage builds use multiple `FROM` statements to create smaller, more secure production images by separating build and runtime environments.

```dockerfile
# Stage 1: Build
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Stage 2: Production (minimal image)
FROM alpine:3.18
COPY --from=builder /app/myapp /myapp
CMD ["/myapp"]
```

**Benefits:**

| Without Multi-stage | With Multi-stage |
|--------------------|------------------|
| ~800MB (includes Go compiler) | ~15MB (just binary + alpine) |
| Build tools in production | Only runtime essentials |
| Larger attack surface | Minimal attack surface |

</details>

<details>
<summary><strong>Q7: Explain Docker networking modes</strong></summary>

| Network Mode | Description | Use Case |
|--------------|-------------|----------|
| **bridge** (default) | Private network, containers communicate via IP | Standard applications |
| **host** | Container shares host's network namespace | Maximum performance |
| **none** | No networking | Security-sensitive workloads |
| **overlay** | Multi-host networking for Swarm/K8s | Distributed applications |
| **macvlan** | Assigns MAC address, appears as physical device | Legacy apps requiring L2 |

```bash
# Create custom bridge network
docker network create mynet

# Run containers on same network (can communicate by name)
docker run -d --name db --network mynet postgres
docker run -d --name app --network mynet myapp
# app can reach db at hostname "db"
```

</details>

<details>
<summary><strong>Q8: What are Docker volumes and why use them?</strong></summary>

Volumes provide **persistent storage** for containers. Container data is ephemeral by default - volumes survive container deletion.

| Storage Type | Description | Use Case |
|--------------|-------------|----------|
| **Volumes** | Managed by Docker, stored in Docker area | Production, databases |
| **Bind mounts** | Map host directory to container | Development, config files |
| **tmpfs** | Stored in host memory only | Sensitive data, caches |

```bash
# Named volume (recommended)
docker volume create mydata
docker run -v mydata:/app/data nginx

# Bind mount
docker run -v /host/path:/container/path nginx

# Anonymous volume
docker run -v /app/data nginx
```

**Best Practice:** Use named volumes for databases and persistent data.

</details>

<details>
<summary><strong>Q9: What is Docker Compose?</strong></summary>

Docker Compose defines and runs **multi-container applications** using a YAML file.

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://db:5432/app
    depends_on:
      - db
    
  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=secret

volumes:
  pgdata:
```

**Common Commands:**
```bash
docker compose up -d      # Start all services
docker compose down       # Stop and remove
docker compose logs -f    # Follow logs
docker compose ps         # List services
docker compose exec web sh # Shell into service
```

</details>

<details>
<summary><strong>Q10: How do you reduce Docker image size?</strong></summary>

| Technique | Impact |
|-----------|--------|
| Use **alpine** or **distroless** base images | 5MB vs 100MB+ |
| **Multi-stage builds** | Remove build tools |
| Combine RUN commands | Fewer layers |
| Use `.dockerignore` | Exclude unnecessary files |
| Clean up in same layer | `apt-get clean && rm -rf /var/lib/apt/lists/*` |
| Use specific tags | Avoid `latest` bloat |

**Example: Optimized Dockerfile**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
USER node
CMD ["node", "server.js"]
```

</details>

<details>
<summary><strong>Q11: What are Docker security best practices?</strong></summary>

| Practice | Why |
|----------|-----|
| **Run as non-root user** | Limit container privileges |
| **Use read-only filesystem** | `--read-only` flag |
| **Scan images for vulnerabilities** | `docker scout`, Trivy, Snyk |
| **Use minimal base images** | Smaller attack surface |
| **Don't store secrets in images** | Use Docker secrets or env vars |
| **Set resource limits** | Prevent DoS |
| **Use specific image tags** | Avoid supply chain attacks |
| **Enable content trust** | `DOCKER_CONTENT_TRUST=1` |

```bash
# Run container securely
docker run \
  --user 1000:1000 \
  --read-only \
  --cap-drop ALL \
  --security-opt no-new-privileges \
  --memory 512m \
  --cpus 0.5 \
  myapp
```

</details>

<details>
<summary><strong>Q12: How do you inspect and debug containers?</strong></summary>

```bash
# View container logs
docker logs <container> -f --tail 100

# Execute command in running container
docker exec -it <container> sh

# Inspect container details (network, mounts, env)
docker inspect <container>

# View resource usage
docker stats

# View running processes
docker top <container>

# Copy files from container
docker cp <container>:/path/file ./local

# View container filesystem changes
docker diff <container>

# Check why container exited
docker inspect <container> --format='{{.State.ExitCode}}'
docker logs <container> 2>&1 | tail -20
```

</details>

<details>
<summary><strong>Q13: What is the difference between COPY and ADD?</strong></summary>

| Feature | COPY | ADD |
|---------|------|-----|
| Copy local files | ✅ | ✅ |
| Extract tar archives | ❌ | ✅ (auto-extracts) |
| Fetch remote URLs | ❌ | ✅ |
| Transparency | More explicit | Magic behavior |

**Best Practice:** Use `COPY` unless you specifically need tar extraction.

```dockerfile
# Prefer COPY
COPY ./src /app/src

# Use ADD only for tar extraction
ADD archive.tar.gz /app/

# DON'T use ADD for URLs (use curl instead)
# BAD: ADD https://example.com/file /app/
# GOOD:
RUN curl -o /app/file https://example.com/file
```

</details>

<details>
<summary><strong>Q14: What is a Docker Registry?</strong></summary>

A registry stores and distributes Docker images.

| Registry | Type | Use Case |
|----------|------|----------|
| **Docker Hub** | Public/Private | Default, open-source images |
| **Amazon ECR** | Private | AWS workloads |
| **Google GCR/Artifact Registry** | Private | GCP workloads |
| **Azure ACR** | Private | Azure workloads |
| **GitHub Container Registry** | Public/Private | GitHub-integrated projects |
| **Harbor** | Self-hosted | Enterprise, air-gapped environments |

```bash
# Push to registry
docker tag myapp:v1 myregistry.com/myapp:v1
docker push myregistry.com/myapp:v1

# Pull from registry
docker pull myregistry.com/myapp:v1

# Login to private registry
docker login myregistry.com
```

</details>

