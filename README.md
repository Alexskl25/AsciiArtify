# AsciiArtify

AsciiArtify is a startup project that transforms images into ASCII art using **Machine Learning**. This repository contains the concept documentation, local Kubernetes PoC setup, and demo resources for development and testing.

---

## Getting Started

Clone the repository:

```bash
git clone https://github.com/Alexskl25/AsciiArtify.git
cd AsciiArtify
```

---

## Concept Document

The detailed concept and comparative analysis of local Kubernetes development tools is available in:

[doc/Concept.md](./doc/Concept.md)

It includes:

- Overview of **Minikube, Kind, and k3d**  
- Comparison table of features, pros, and cons  
- Recommendations for PoC deployment  
- Embedded demo of a “Hello World” app running on a local k3d cluster

## Deployment of ArgoCD on k3d

This section explains how to deploy **ArgoCD** on a lightweight **k3d (K3s)** Kubernetes cluster for proof-of-concept purposes.

[doc/POC.md](./doc/POC.md)

## Live demo of the **AsciiArtify**

ArgoCD MVP check with deploying of AsciiArtify

[doc/MVP.md](./doc/MVP.md)

---

## Demo

### Inline demo GIF from the PoC:

![Hello World Demo](https://raw.githubusercontent.com/Alexskl25/AsciiArtify/main/.data/demo.gif)

> The demo shows a simple Kubernetes deployment running locally on **k3d**.

### Recording from terminal:

[![Deploying ArgoCD on k3d](https://raw.githubusercontent.com/Alexskl25/AsciiArtify/main/.data/demoArgoCDDeploy.gif)](https://asciinema.org/a/oGGU7ZtkH4CjkxejlUyoMR3Xe)

> The demo shows an ArgoCD deployment.

### Click on the image to play the demo of the AsciiArtify runing on the k3d + ArgoCD

[![Demo video](https://github.com/Alexskl25/AsciiArtify/blob/main/.data/DemoOfArgoCD.JPG)](https://github.com/Alexskl25/AsciiArtify/blob/main/.data/DemoOfArgoCD.mp4)

---

