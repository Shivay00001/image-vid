# Shivay00001/image-vid

A lightweight, high-performance Go HTTP service, containerized for frictionless deployment on any machine — from a developer laptop to a production server.

## 🚀 Overview

**image-vid** is a minimal yet production-ready Go microservice that exposes an HTTP endpoint reporting live system status with a server-side timestamp. It is built on Go's standard library only — **zero external dependencies** — resulting in a tiny attack surface, fast builds, and an extremely small container footprint via Alpine Linux.

## ✨ Features

- **Zero-dependency architecture** — built entirely on Go's standard library (`net/http`, `log`, `time`, `fmt`).
- **High-performance HTTP server** — leverages Go's goroutine-per-connection model for efficient concurrency out of the box.
- **Docker-native deployment** — ships with a ready-to-use `Dockerfile` based on `golang:1.20-alpine`.
- **Fail-fast logging** — startup and fatal errors are logged via the standard `log` package for easy observability in container logs.
- **Single static binary** — compiled to one executable (`app`), ideal for immutable infrastructure.

## 🏗️ Architecture / How It Works

The entire service lives in `main.go` and follows a clean, single-responsibility design:

1. **Route Registration**
   ```go
   http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
       fmt.Fprintf(w, "System Operational: %s", time.Now())
   })
   ```
   A handler is registered on the root path `/` using Go's `DefaultServeMux`. On every incoming request — regardless of HTTP method — it writes a plain-text response containing the string `System Operational:` followed by the **current server timestamp** (`time.Now()`). This makes the endpoint useful as a **health check / liveness probe**: a successful `200 OK` with a fresh timestamp confirms the service is alive and processing requests.

2. **Server Bootstrap**
   ```go
   log.Println("Starting high-performance service on :8080")
   log.Fatal(http.ListenAndServe(":8080", nil))
   ```
   The service listens on **port `8080`** on all interfaces (`0.0.0.0`). `http.ListenAndServe` blocks and serves requests concurrently (one goroutine per connection). If the server ever fails to start or crashes, `log.Fatal` prints the error to stderr and exits with a non-zero status — which, under Docker's default restart policies, makes failures immediately visible.

3. **Build & Runtime Pipeline (Docker)**
   - Base image: `golang:1.20-alpine` — the Go toolchain on a minimal Alpine Linux base.
   - Working directory: `/app`.
   - The full source tree is copied in, and `go build -o app` compiles the module (`github.com/Shivay00001/image-vid`, per `go.mod`) into a single binary.
   - The container's default command runs `./app`, starting the HTTP server on port `8080`.

```
Client ──HTTP──▶ :8080 ──▶ DefaultServeMux ──▶ / handler ──▶ "System Operational: <timestamp>"
```

## 🐳 Docker Deployment (Recommended)

This is the fastest way to run the service on any laptop or server with Docker installed.

### 1. Build the image
```bash
docker build -t shivay00001/image-vid .
```

### 2. Run the container
```bash
docker run -d -p 8080:8080 --name image-vid shivay00001/image-vid
```

### 3. Verify it's operational
```bash
curl http://localhost:8080/
# Expected output: System Operational: 2024-01-01 12:00:00.000000000 +0000 UTC ...
```

### 4. Manage the container
```bash
docker logs -f image-vid      # stream logs
docker stop image-vid         # stop the service
docker rm image-vid           # remove the container
```

> **Note:** This repository does not include a `docker-compose.yml`. The plain `docker build` / `docker run` flow above is the canonical deployment method. If you prefer Compose, create a minimal `docker-compose.yml`:
> ```yaml
> services:
>   image-vid:
>     build: .
>     ports:
>       - "8080:8080"
> ```
> then run `docker-compose up -d --build`.

## 🛠️ Local Execution (Without Docker)

Requires **Go 1.20+** installed:

```bash
# Clone and enter the repository
git clone https://github.com/Shivay00001/image-vid.git
cd image-vid

# Run directly
go run main.go

# Or compile a binary
go build -o app
./app
```

The service will be available at `http://localhost:8080/`.

## 📁 Project Structure

```
image-vid/
├── main.go        # HTTP service: handler + server bootstrap
├── go.mod         # Go module definition (Go 1.20, no dependencies)
├── Dockerfile     # Container build (golang:1.20-alpine)
├── .gitignore     # Excludes IDE files, build artifacts, env files
├── LICENSE        # VisionQuantech Custom Commercial License
└── README.md      # This document
```

## ⚙️ Configuration

| Setting | Value | Notes |
|---|---|---|
| Port | `8080` | Hardcoded in `main.go`; remap via `-p HOST:8080` in Docker |
| Endpoint | `GET /` (all methods) | Returns `System Operational: <timestamp>` |
| Dependencies | None | Standard library only |

## 📄 License

This project is distributed under the **VisionQuantech Custom Commercial License**:

- **Non-financial / personal / educational use** — free.
- **Personal revenue-generating use** — requires a 15–30% revenue share.
- **Business / enterprise use** — requires a separate commercial license.

For commercial licensing inquiries, contact: **visionquantech@proton.me**

See [LICENSE](LICENSE) for full terms.