# Adding a New Kubernetes Cluster to ArgoCD

## Overview

This guide explains how to:

- Add a new Kubernetes cluster to an existing ArgoCD setup
- Configure private container registry (e.g., Harbor) access
- Deploy applications to the new cluster

---

## Prerequisites

Make sure you have:

- ArgoCD CLI installed and authenticated
- `kubectl` access to:
    - ArgoCD cluster
    - Target (new) cluster
- Network connectivity from ArgoCD to the new cluster (API port 6443)
- Container registry credentials (if using private images)

---

## Step 1: Login to ArgoCD

```
argocd login <argocd-server>
```

---

## Step 2: Export Kubeconfig from Target Cluster

Run on the target cluster:

```
kubectl config view--flatten--minify > new-cluster-config.yaml
```

### Important Fix

Edit the kubeconfig:

```
nano new-cluster-config.yaml
```

Replace:

```
server: https://127.0.0.1:6443
```

With:

```
server: https://<CLUSTER-IP>:6443
```

---

## Step 3: Transfer Kubeconfig

Copy the file to the ArgoCD machine:

```
scp new-cluster-config.yaml user@argocd-server:/path/
```

---

## Step 4: Add Cluster to ArgoCD

```
argocd cluster add default \
--kubeconfig new-cluster-config.yaml \
--name <cluster-name> \
--insecure
```

### What Happens Internally

- Creates `argocd-manager` ServiceAccount
- Assigns cluster-admin role
- Generates authentication token
- Registers cluster in ArgoCD

---

## Step 5: Verify Cluster

```
argocd cluster list
```

Expected:

```
SERVER                      NAME             STATUS
https://<cluster-ip>:6443   <cluster-name>   Unknown/Successful
```

Note: `Unknown` is normal until applications are deployed.
