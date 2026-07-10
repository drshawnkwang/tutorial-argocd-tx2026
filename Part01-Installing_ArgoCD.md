# Part I: Installing ArgoCD

## Connect to Your VM

```bash
ssh student@<your-vm-ip>
```

Verify the cluster is ready:

```bash
kubectl get nodes
kubectl get pods -n kube-system
```

You should see one node in `Ready` state.

## Fork and Clone the Tutorial Repository

> **Git setup:** Make sure you've reviewed the [GitHub Access Guide](GITHUB-ACCESS.md) and chosen how you'll push to GitHub before continuing.

This tutorial uses a Git repository containing exercise files, configuration, and sample manifests.

1. In your browser, fork `https://github.com/esnet/tutorial-argocd-tx2026` to your GitHub account
2. On your VM (Terminal 1):

```bash
git clone https://github.com/<your-username>/tutorial-argocd-tx2026.git
cd tutorial-argocd-tx2026
ls
```

> You will work inside this directory for the rest of the tutorial. The `argocd-values.yaml` file used in the next step comes from this repo.

## What is ArgoCD?

ArgoCD is a GitOps continuous delivery tool for Kubernetes. It watches a Git repository and automatically keeps your cluster in sync with what's declared in Git.

**Core concepts:**

- **Application** : a group of Kubernetes resources defined in Git
- **Sync** : making the live cluster match the desired state in Git
- **Health** : whether deployed resources are running correctly

## Install ArgoCD

### Step 1: Create the namespace

```bash
kubectl create namespace argocd
```

### Step 2: Install ArgoCD via Helm

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd \
  --namespace argocd \
  --values argocd-values.yaml \
  --wait
```

> This file is in the repo you just cloned. It is tuned for our single-node k3s environment.

### Step 3: Wait for pods

```bash
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s
kubectl get pods -n argocd
```

All pods should show `Running`.

## Access the ArgoCD UI

You are currently in **Terminal 1** (your SSH session on the VM). You will need to open **two additional terminals** on your laptop. Terminals 2 and 3 will be tied up for the rest of the tutorial, continue working in Terminal 1 for all remaining exercises.

**Terminal 2**  open a new terminal on your laptop, SSH to your VM, and start port-forward:

```bash
ssh student@<your-vm-ip>
kubectl port-forward svc/argocd-server -n argocd 8443:443
```

> Leave this running.

**Terminal 3**  open another terminal on your laptop for the SSH tunnel:

```bash
ssh -C -N -L 8443:localhost:8443 student@<your-vm-ip>
```

> Leave this running.

Open `https://localhost:8443` in your browser. Accept the self-signed certificate warning.

### Fallback: NodePort

If the SSH tunnel does not work, you can expose ArgoCD via a NodePort instead:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort", "ports": [{"port": 443, "targetPort": 8443, "nodePort": 30443}]}}'
```

Open `https://<your-vm-ip>:30443` in your browser. Accept the self-signed certificate warning (requires that port 30443 is open in your VM's firewall).

## Log In

Get the initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

- **Username:** `admin`
- **Password:** (output from above)

Log in via the browser UI. Optionally change the password under User Info.

## Verify

Run this in **Terminal 1** (your main VM session).

**If using SSH tunnel (Terminals 2 + 3):**

```bash
argocd login localhost:8443 --insecure
argocd cluster list
```

**If using NodePort:**

```bash
argocd login <your-vm-ip>:30443 --insecure
argocd cluster list
```

You should see one cluster (the local `in-cluster`). ArgoCD is ready.

---

**Next:** Part II - Core GitOps Exercises

---

## Appendix: Install ArgoCD with raw manifests

If you prefer not to use Helm, you can install ArgoCD by applying the upstream manifest directly:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

This installs the latest stable release with default settings. Note that the Helm-based install (Step 2 above) is recommended for this tutorial because it uses a values file tuned for our single-node k3s environment.
