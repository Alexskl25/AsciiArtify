# AsciiArtify — Local Kubernetes for PoC

## Executive summary

AsciiArtify is a small startup building an ML-powered image to ASCII-art service. For a quick, repeatable PoC and local development workflow, here we compared three popular local Kubernetes options: **minikube**, **kind** and **k3d**. This document summarizes the comparison, highlights licensing/runtime risks around Docker Desktop, shows a recommended quick demo (k3d `Hello World`) and gives a clear recommendation for which tool to use where.

---

## Table of contents

1. Introduction
2. Characteristics (OS/arch, runtimes, automation)
3. Feature comparison table
4. Pros & cons
5. Demo — recommended option (k3d) with a minimal `Hello World`
6. Recommendations & next steps

---

## 1. Introduction

**minikube**: a single-node local Kubernetes distribution meant mainly for development and learning. It can run using different drivers (VM or container runtime) and exposes helpful developer features such as `minikube dashboard` and built-in addons.

**kind (Kubernetes IN Docker)**: runs Kubernetes nodes as containers. It is optimized for fast lifecycle (create/delete) and is widely used in CI pipelines and local integration testing.

**k3d**: a wrapper that runs the lightweight distribution **k3s** inside Docker containers. It is fast, small-footprint and friendly for multi-node local clusters and PoC scenarios.

---

## 2. Characteristics

- **Supported OS / Architectures**
  - **minikube**: Linux, macOS, Windows — supports x86_64 and ARM variants depending on a driver.
  - **kind**: Linux, macOS, Windows (requires a container runtime) — supports x86_64 and ARM where the runtime/containers are available.
  - **k3d**: Linux, macOS, Windows (Docker required) — supports x86_64 and ARM through k3s.

- **Container runtime**
  - All three commonly use **Docker**. `minikube` and `kind` have options and partial support for **Podman** and other runtimes.

- **Automation / CI suitability**
  - **kind**: best-in-class for CI (fast, ephemeral clusters, reproducible images and node configuration).
  - **k3d**: very good for automated PoC and CI when you need a lightweight, multi-node cluster.
  - **minikube**: oriented to interactive development; can be automated but less common in ephemeral CI jobs.

- **Addons & tooling**
  - **minikube**: has built-in addons and `dashboard` command.
  - **kind**: minimalist; integrate external tooling (Prometheus, Grafana, local ingress controllers) as needed.
  - **k3d**: integrates well with Traefik (k3s default), Helm and common dev tooling.

---

## 3. Feature comparison table

| Feature / Aspect | minikube | kind | k3d |
|---|---:|---:|---:|
| Primary use case | Interactive dev, learning | CI, ephemeral test clusters | Lightweight PoC, fast multi-node local clusters |
| Startup speed | Moderate (VMs slower) | Fast | Very fast |
| Multi-node support | Yes (VMs or container nodes) | Yes (containers) | Yes (k3s agents) |
| Resource footprint | Medium (VMs) | Low-to-medium | Low |
| Podman support | Partial/experimental | Partial (with configuration) | Works with Docker primarily |
| Ease of automation | Medium | High | High |
| Similarity to production k8s | High (upstream components) | High (upstream components) | Slight differences (k3s simplifications) |
| Best for CI | No | Yes | Yes |
| Best for local PoC | Yes | Good | Best |

> | Official links to the docs pages: |
> | [https://minikube.sigs.k8s.io/docs/](https://minikube.sigs.k8s.io/docs/) |
> | [https://kind.sigs.k8s.io/](https://kind.sigs.k8s.io/) |
> | [https://k3d.io/stable/](https://k3d.io/stable/) |

---

## 4. Pros & cons (short)

### minikube
**Pros**: Great developer UX; GUI (dashboard), many drivers; closer to full upstream k8s.
**Cons**: VM mode can be slow; Podman driver sometimes experimental; less ideal for ephemeral CI.

### kind
**Pros**: Excellent for automated testing and CI; quick create/delete; full control of node images.
**Cons**: Node-as-container networking differs from real cloud clusters; Podman/rootless requires extra setup.

### k3d
**Pros**: Fast, tiny footprint; great for PoC and repeated local cycles; easy multi-node.
**Cons**: k3s is a lightweight distribution — some control plane components differ from upstream k8s.

---

## 5. Docker licensing risk and Podman alternative

- **Docker Desktop** licensing changed for some commercial scenarios and some organizations opt to avoid Docker Desktop for proprietary-license concerns. For small personal use it is usually fine, but for companies you should review Docker's license terms.

- **Mitigation**: Use **Podman** (open, upstream-friendly) or run Docker Engine on Linux hosts (no Desktop). `minikube` and `kind` have documented ways to use Podman, but expect some extra configuration (rootless cgroups, networking differences).

> The primary Docker licensing risk stems from the 2021 update that requires a paid subscription for commercial use in large enterprises (over 250 employees or $10 million in annual revenue)

---

## 6. Demo — recommended option: `k3d` (Hello World)

This short demo demonstrates a full PoC lifecycle: create cluster → deploy a small service → test → destroy. Replace `docker` with your chosen runtime if needed.

[![Hello World Demo](https://raw.githubusercontent.com/Alexskl25/AsciiArtify/main/.data/demo.gif)](https://asciinema.org/a/p0wTyacilZOT88ZeVyHNzIaFu)

1. Install k3d (example):
```bash
# Linux/macOS (quick installer)
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
# verify
k3d version
kubectl version --client
```

2. Create a lightweight multi-node cluster:
```bash
k3d cluster create asciiartify --servers 1 --agents 1 --wait
kubectl get nodes
```

3. Prepare you basic container application

**Create Dockerfile**
```Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```
**Create index.html**
```html
Hello from AsciiArtify PoC!
```
**Build docker image**
```bash
# Build locally
docker build -t hello-world:local .
```

4. Apply a minimal `Hello World` deployment and service (file: `hello-deploy.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deployment
  labels:
    app: hello
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: hello
          image: hello-world:local
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hello-service
  labels:
    app: hello
spec:
  type: ClusterIP
  selector:
    app: hello
  ports:
    - port: 80
      targetPort: 80
```

Apply & local test:
```bash
# Import into k3d
k3d image import hello-world:local --cluster asciiartify
kubectl apply -f hello-deploy.yaml
kubectl get pods -l app=hello
kubectl port-forward svc/hello-service 8080:80 &
curl http://127.0.0.1:8080/  # should return "Hello from AsciiArtify PoC!"
```

5. Cleanup:
```bash
k3d cluster delete asciiartify
```

> It is easy to replace k3d with kind for CI-focused demos.

---

## 7. Recommendations and next steps for AsciiArtify

1. **Local PoC**: use **k3d** for day-to-day PoC and local acceptance testing (fast cycles, minimal resources).
2. **CI**: use **kind** in GitHub Actions for reproducible integration tests.
3. **Developer learning**: keep `minikube` available for team members who prefer a more full-featured developer environment with a dashboard.
4. **Container runtime**: if you want to avoid Docker Desktop licensing implications, adopt Podman for development machines and standardize CI on Linux runners with Docker Engine or containerd.
5. **Repository structure**: create `doc/Concept.md` (this file), a `demo/` folder containing `hello-deploy.yaml`, and `ci/` folder with a `kind` GitHub Actions workflow example.

---

## 8. Appendix — quick checklist to prepare PoC

- [x] Choose runtime (Docker or Podman)
- [x] Create `demo/hello-deploy.yaml`
- [ ] Add `Makefile` targets: `make cluster-up`, `make deploy`, `make test`, `make cluster-down`
- [ ] Add GitHub Actions workflow that uses `kind` for CI tests
- [ ] Add README with developer quickstart

