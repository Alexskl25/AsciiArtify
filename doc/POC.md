# Deploying ArgoCD on k3d

This guide explains how to deploy **ArgoCD** on a lightweight **k3d (K3s)** Kubernetes cluster.

---

## Prerequisites

Before starting, make sure you have:

- **Docker** installed
- **k3d** installed
- **kubectl** installed
- A working shell environment

---
## Recording from terminal:

[![Deploying ArgoCD on k3d](https://raw.githubusercontent.com/Alexskl25/AsciiArtify/main/.data/demoArgoCDDeploy.gif)](https://asciinema.org/a/oGGU7ZtkH4CjkxejlUyoMR3Xe)

---

## Step by step *instruction*:
## Step 0: Create an alias for kubectl (optional)

```sh
alias k=kubectl
```

## Step 1: Create a k3d Cluster
```sh
k3d cluster create argo
```
Verify that the cluster is running:
```sh
k cluster-info
```

## Step 2: Create the ArgoCD Namespace
```sh
k create namespace argocd
```
## Step 3: Install ArgoCD
```sh
k apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Step 4: Check ArgoCD Components
List all resources in the **argocd** namespace:
```sh
k get all -n argocd
```
Watch the pods as they start:
```sh
k get pod -n argocd -w
```
Wait until all pods are in the Running or Completed state.

## Step 5: Port Forward ArgoCD API/UI
Forward your local port **8080** to the ArgoCD server service on port **443**:
```sh
k port-forward svc/argocd-server -n argocd 8080:443 &
```
Test the connection:
```sh
curl 127.0.0.1:8080
```
You should receive an HTTP response from ArgoCD.
> **Tip:** The **&** puts the port-forward process in the background.

## Step 6: Retrieve the Initial Admin Password
ArgoCD stores the default admin password in a Kubernetes secret.
Get the password (encoded in base64):
```sh
k -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
```
Decode it:
```sh
k -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
This is your **ArgoCD admin password.**

---

## Step 7: Access the ArgoCD UI (GUI PART)

![ArgoCD Login Page](https://raw.githubusercontent.com/Alexskl25/AsciiArtify/main/.data/Argo_login.PNG)

Open your browser and go to:
```sh
https://localhost:8080
```
Log in with:
* Username: **admin**
* Password: **(the value you decoded in the previous step)**

---

## Now ArgoCD is Ready for MVP check out next step in **[doc/MVP.md](./doc/MVP.md)**

![ArgoCD Login Page](https://raw.githubusercontent.com/Alexskl25/AsciiArtify/main/.data/ArgoCD_index.PNG)

---
