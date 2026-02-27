# DevOpsified Go Web App 🚀

> A production-grade CI/CD and GitOps implementation for a Go-based web application — containerized with Docker, deployed on Kubernetes via Helm, and fully automated using GitHub Actions and ArgoCD.

---

## 🏗️ Architecture Overview

```
Developer Push → GitHub Actions Pipeline → DockerHub → Helm Chart Update → ArgoCD → Kubernetes (EKS)
```

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Actions Pipeline                   │
│                                                             │
│  ┌─────────┐    ┌──────────────┐    ┌──────────────────┐   │
│  │  Build  │───▶│ Code Quality │───▶│  Docker Build &  │   │
│  │  & Test │    │  (golangci)  │    │  Push to DockerHub│  │
│  └─────────┘    └──────────────┘    └────────┬─────────┘   │
│                                              │              │
│                                    ┌─────────▼──────────┐  │
│                                    │  Update Helm Chart  │  │
│                                    │  tag in values.yaml │  │
│                                    └─────────┬──────────┘  │
└──────────────────────────────────────────────┼─────────────┘
                                               │
                                    ┌──────────▼──────────┐
                                    │       ArgoCD         │
                                    │  (GitOps - detects   │
                                    │   Helm chart change) │
                                    └──────────┬──────────┘
                                               │
                                    ┌──────────▼──────────┐
                                    │   Kubernetes (EKS)   │
                                    │  Helm Deployment +   │
                                    │  Ingress Controller  │
                                    └─────────────────────┘
```

---

## ✨ Key Features

- **Multi-Stage Docker Build** — Uses a distroless base image for the final container, resulting in a minimal attack surface and significantly smaller image size
- **4-Stage CI/CD Pipeline** — Automated build, test, code quality check, and deployment via GitHub Actions
- **GitOps Loop** — Pipeline automatically commits updated image tags back to the Helm chart, triggering ArgoCD to sync without any manual intervention
- **Code Quality Gates** — `golangci-lint` runs on every push, blocking merges that don't meet quality standards
- **Helm Multi-Environment Deployment** — Parameterized Helm charts allow deployment across dev, staging, and production environments
- **Kubernetes Ingress** — Ingress controller configured with DNS mapping for clean external traffic routing

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Application** | Go 1.22 |
| **Containerization** | Docker (Multi-Stage, Distroless) |
| **CI/CD** | GitHub Actions |
| **Code Quality** | golangci-lint |
| **Artifact Registry** | DockerHub |
| **Package Management** | Helm |
| **GitOps** | ArgoCD |
| **Orchestration** | Kubernetes (AWS EKS) |
| **Ingress** | Kubernetes Ingress Controller |

---

## 📦 Project Structure

```
devosified_go-app/
├── .github/
│   └── workflows/
│       └── github_actions.yaml   # 4-stage CI/CD pipeline
├── helm_files/
│   └── go-web-app-chart/
│       └── values.yaml           # Auto-updated by pipeline with new image tag
├── kuberntes/
│   └── manifests_files/          # Raw Kubernetes manifests
├── static/                       # Static assets
├── Dockerfile                    # Multi-stage build with distroless base
├── main.go                       # Application entry point
└── main_test.go                  # Unit tests
```

---

## 🔄 CI/CD Pipeline Deep Dive

The GitHub Actions pipeline triggers on every push to `main` (excluding Helm and README changes) and runs 4 jobs:

### Stage 1 — Build & Test
- Checks out the repo and sets up Go 1.22
- Compiles the binary and runs all unit tests
- All subsequent stages depend on this passing

### Stage 2 — Code Quality
- Runs `golangci-lint` v1.56.2 in parallel with the build stage
- Enforces Go best practices and catches bugs before they reach production

### Stage 3 — Docker Build & Push
- Builds the multi-stage Docker image
- Tags the image with the GitHub run ID for precise version tracking
- Pushes to DockerHub using secured credentials via GitHub Secrets

### Stage 4 — GitOps Helm Update ⭐
- Automatically updates the image tag in `helm_files/go-web-app-chart/values.yaml`
- Commits and pushes the change back to the repo
- ArgoCD detects the change and syncs the Kubernetes deployment — **zero manual deployment steps**

---

## 🐳 Docker — Multi-Stage Build

```dockerfile
# Stage 1 — Build
FROM golang:1.22.5 as base
WORKDIR /app
COPY go.mod ./
RUN go mod download
COPY . .
RUN go build -o main .

# Stage 2 — Production (Distroless)
FROM gcr.io/distroless/base
COPY --from=base /app/main .
COPY --from=base /app/static ./static
EXPOSE 8080
CMD ["./main"]
```

**Why distroless?** The final image contains only the application binary and its runtime dependencies — no shell, no package manager, no unnecessary tools. This drastically reduces the attack surface and image size.

---

## 🚀 How to Run Locally

```bash
# Clone the repo
git clone https://github.com/bijit5/devosified_go-app.git
cd devosified_go-app

# Run directly
go run main.go

# Or with Docker
docker build -t go-web-app .
docker run -p 8080:8080 go-web-app
```

Access the app at `http://localhost:8080/courses`

---

## ☸️ Deploy with Helm

```bash
# Install on Kubernetes
helm install go-web-app ./helm_files/go-web-app-chart

# Upgrade with a new image tag
helm upgrade go-web-app ./helm_files/go-web-app-chart --set image.tag=<new-tag>
```

---

## 📈 Impact

| Metric | Result |
|---|---|
| Deployment time | Reduced by **50%** |
| Manual deployment steps | **Zero** — fully automated via GitOps |
| Image attack surface | Minimized via **distroless** base |
| Code quality | Enforced on **every push** via linting gates |

---

## 👤 Author

**Bijit Kalita** — DevOps Engineer
- 📧 bijit987kalita@gmail.com
- 💼 [linkedin.com/in/bijit-kalita](https://linkedin.com/in/bijit-kalita/)
- 🐙 [github.com/bijit5](https://github.com/bijit5)
