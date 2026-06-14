## Docker

### Containers

```bash
docker run -d --name <name> <image>        # start detached
docker run -it --rm <image> sh             # interactive, auto-remove on exit
docker ps                                  # running containers
docker ps -a                               # all containers
docker stop <name>
docker start <name>
docker rm <name>                           # remove stopped container
docker rm -f <name>                        # force remove running container
docker logs <name>
docker logs -f <name>                      # follow logs
docker exec -it <name> sh                  # shell into running container
docker exec -it postgres psql -U postgres  # psql into postgres
docker inspect <name>                      # full container metadata
```

### Images

```bash
docker images                              # list local images
docker pull <image>:<tag>
docker build -t <name>:<tag> .
docker rmi <image>                         # remove image
docker image prune                         # remove dangling images
docker image prune -a                      # remove all unused images
```

### Volumes

```bash
docker volume ls
docker volume create <name>
docker volume rm <name>
docker volume prune                        # remove unused volumes
docker run -v <volume>:/path <image>       # named volume
docker run -v $(pwd):/app <image>          # bind mount
```

### Networks

```bash
docker network ls
docker network create <name>
docker network connect <network> <container>
docker network inspect <name>
```

### Cleanup

```bash
docker system prune                        # dangling images + stopped containers + unused networks
docker system prune -a --volumes           # nuclear: everything unused
docker system df                           # disk usage breakdown
```

### Common Services

```bash
# Postgres
docker run -d --name postgres \
  -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 \
  postgres:17

# Redis
docker run -d --name redis \
  -p 6379:6379 \
  redis:7-alpine

# Mongo
docker run -d --name mongo \
  -p 27017:27017 \
  mongo:8
```

### Dockerfile (multi-stage, Rust example)

```dockerfile
FROM rust:1.87 AS builder
WORKDIR /app
COPY . .
RUN cargo build --release

FROM debian:bookworm-slim
COPY --from=builder /app/target/release/<binary> /usr/local/bin/
CMD ["<binary>"]
```

### Dockerfile (Python with uv)

```dockerfile
FROM python:3.14-slim
WORKDIR /app
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev
COPY src/ src/
CMD ["uv", "run", "python", "-m", "<package>"]
```

### Compose

```yaml
# compose.yaml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
    env_file: .env

  db:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  fedora:
    image: fedora:latest
    stdin_open: true
    tty: true
    deploy:
      resources:
        limits:
          cpus: "4.0"
          memory: 2g
        reservations:
          cpus: "1.0"
          memory: "256m"

volumes:
  pgdata:
```

```bash
docker compose up -d                       # start all
docker compose down                        # stop + remove containers
docker compose down -v                     # also remove volumes
docker compose logs -f <service>
docker compose exec <service> sh
docker compose build                       # rebuild images
```

### Exec into Services

```bash
# Fedora shell
docker compose exec fedora bash

# Postgres shell (psql)
docker compose exec db psql -U postgres

# Redis CLI
docker compose exec redis redis-cli

# Mongo shell
docker compose exec mongo mongosh

# Run a one-off command
docker compose exec fedora dnf install -y curl
docker compose exec db psql -U postgres -c "SELECT version();"
```

### Copy Files

```bash
docker cp <container>:/path/to/file ./local
docker cp ./local <container>:/path/to/file
```

### Environment Variables

```bash
docker run -e KEY=value <image>
docker run --env-file .env <image>
```

### Restart Policies

```bash
docker run -d --restart unless-stopped <image>
# options: no, on-failure, always, unless-stopped
```

### Health Checks (compose)

```yaml
services:
  db:
    image: postgres:17
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  app:
    depends_on:
      db:
        condition: service_healthy
```

### Registry

```bash
docker tag <image> registry.example.com/<image>:<tag>
docker push registry.example.com/<image>:<tag>
docker login registry.example.com
```

### .dockerignore

```
.git
.venv
target/
node_modules/
dist/
*.env
```
