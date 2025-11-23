# AsciiArtify — Local Kubernetes for PoC

**File:** `doc/Concept.md`
**Branch:** `main`

---

## Executive summary

AsciiArtify is a small startup building an ML-powered image→ASCII-art service. For a quick, repeatable PoC and local development workflow we compared three popular local Kubernetes options: **minikube**, **kind** and **k3d**. This document summarizes the comparison, highlights licensing/runtime risks around Docker Desktop, shows a recommended quick demo (k3d `Hello World`) and gives a clear recommendation for which tool to use where.

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
  - **minikube**: Linux, macOS, Windows — supports x86_64 and ARM variants depending on driver.
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

> Note: The table is a summary. Each project evolves — check official docs for the most current options for Podman, Windows and ARM.

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

---

## 6. Demo — recommended option: `k3d` (Hello World)

This short demo demonstrates a full PoC lifecycle: create cluster → deploy a small service → test → destroy. Replace `docker` with your chosen runtime if needed.

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

3. Apply a minimal `Hello World` deployment and service (file: `hello-deploy.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 2
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
        image: hashicorp/http-echo:0.2.3
        args:
          - "-text=Hello from AsciiArtify PoC!"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: hello-svc
spec:
  selector:
    app: hello
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5678
  type: NodePort
```

Apply & test:
```bash
kubectl apply -f hello-deploy.yaml
kubectl get pods -l app=hello
kubectl port-forward svc/hello-svc 8080:80 &
curl http://127.0.0.1:8080/  # should return "Hello from AsciiArtify PoC!"
```

4. Cleanup:
```bash
k3d cluster delete asciiartify
```

> For CI-focused demos, replace k3d with kind and embed cluster creation inside a job (GitHub Actions/GitLab CI). A minimal `kind` job will `kind create cluster` → `kubectl apply` → `kind delete cluster`.

---

## 7. Embedded demo & references

- Example `demo` style: see project `wagoodman/dive` for demo structure and repo layout conventions.
- Example comparison table style: see `den-vasyliev/SRE-Competency-Matrix` for a clean markdown table style and layout.

(These examples are referenced as layout inspiration only — the content and commands here are tailored for AsciiArtify.)

---

## 8. Recommendations and next steps for AsciiArtify

1. **Local PoC**: use **k3d** for day-to-day PoC and local acceptance testing (fast cycles, minimal resources).
2. **CI**: use **kind** in GitHub Actions for reproducible integration tests.
3. **Developer learning**: keep `minikube` available for team members who prefer a more full-featured developer environment with a dashboard.
4. **Container runtime**: if you want to avoid Docker Desktop licensing implications, adopt Podman for development machines and standardize CI on Linux runners with Docker Engine or containerd.
5. **Repository structure**: create `doc/Concept.md` (this file), a `demo/` folder containing `hello-deploy.yaml`, and `ci/` folder with a `kind` GitHub Actions workflow example.

---

## 9. Appendix — quick checklist to prepare PoC

- [ ] Choose runtime (Docker or Podman)
- [ ] Create `demo/hello-deploy.yaml`
- [ ] Add `Makefile` targets: `make cluster-up`, `make deploy`, `make test`, `make cluster-down`
- [ ] Add GitHub Actions workflow that uses `kind` for CI tests
- [ ] Add README with developer quickstart

---

*End of Concept.md*