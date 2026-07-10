# Part II: Core GitOps Exercises

## GitOps in a Nutshell

1. You declare your desired state in Git (YAML manifests)
2. ArgoCD watches the Git repo
3. ArgoCD syncs the cluster to match Git
4. If someone changes the cluster directly, ArgoCD detects the drift and corrects it

**Git is the single source of truth.**

## Exercise 1: Deploy Your First Application

### Verify your repo

You forked and cloned the tutorial repo in Part I. Make sure you are in the repo directory:

```bash
cd ~/tutorial-argocd-tx2026
```

The repo contains a `p2-podinfo/` directory with a Deployment and Service for [podinfo](https://github.com/stefanprodan/podinfo), a demo microservice with a web UI.

### Create the Application in ArgoCD

**Via the UI:**

1. Click **+ NEW APP**
2. Fill in:
   - Application Name: `podinfo`
   - Project: `default`
   - Sync Policy: `Manual`
   - Repository URL: `https://github.com/<your-username>/tutorial-argocd-tx2026`
   - Path: `p2-podinfo`
   - Cluster URL: `https://kubernetes.default.svc`
   - Namespace: `default`
3. Click **CREATE**

**Or via CLI:**

```bash
argocd app create podinfo \
  --repo https://github.com/<your-username>/tutorial-argocd-tx2026 \
  --path p2-podinfo \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

### Sync the Application

The app shows **OutOfSync** -- Git has manifests but nothing is deployed yet.

1. Click **SYNC** then **SYNCHRONIZE** in the UI

Or via CLI:

```bash
argocd app sync podinfo
```

### Verify

```bash
kubectl get pods -l app=podinfo
kubectl port-forward svc/podinfo 9898:9898 &
curl localhost:9898 | jq .
```

You should see podinfo's JSON response with version `6.13.0`.

## Exercise 2: Enable Auto-Sync and Self-Heal

### Enable auto-sync

In the ArgoCD UI: click on the `podinfo` app -> **APP DETAILS** -> **ENABLE AUTO-SYNC**. Enable **Prune** and **Self Heal**.

Or via CLI:

```bash
argocd app set podinfo --sync-policy automated --auto-prune --self-heal
```

### Test auto-sync

Edit `p2-podinfo/deployment.yaml` -- change `replicas: 1` to `replicas: 3`:

```bash
# Edit the file, then:
git add p2-podinfo/deployment.yaml
git commit -m "Scale podinfo to 3 replicas"
git push origin main
```

Watch ArgoCD detect the change and create new pods (may take up to 3 minutes):

```bash
kubectl get pods -l app=podinfo -w
```

### Test self-heal

Manually scale the deployment (drift from Git):

```bash
kubectl scale deployment podinfo --replicas=5
kubectl get pods -l app=podinfo
```

Watch ArgoCD detect the drift and scale back to 3. **Git always wins.**

## Exercise 3: Make Changes via Git

### Change 1: Upgrade the image

Edit `p2-podinfo/deployment.yaml` -- change the image tag from `6.13.0` to `6.14.0`:

```bash
git add p2-podinfo/deployment.yaml
git commit -m "Upgrade podinfo to 6.14.0"
git push origin main
```

Verify:

```bash
curl localhost:9898 | jq .version
# Should show "6.14.0"
```

### Change 2: Add resource limits

Edit `p2-podinfo/deployment.yaml` -- add resource requests and limits to the container spec. Commit and push.

### Change 3: Set a custom message

Add an environment variable to the container:

```yaml
env:
- name: PODINFO_UI_MESSAGE
  value: "Hello from ArgoCD tutorial!"
```

Commit, push, then verify:

```bash
curl localhost:9898 | jq .message
```

## Exercise 4: Explore Application Details

In the ArgoCD UI, click on the `podinfo` application and explore:

- **Tree view** -- visual map of Deployment -> ReplicaSet -> Pods, plus Service
- **Resource details** -- click any resource to see its manifest, events, and logs
- **Diff tab** -- compare Git state vs live cluster state
- **History** -- click **HISTORY AND ROLLBACK** to see every sync, with Git commit SHAs and timestamps

## Exercise 5: Deploy a Helm Chart

ArgoCD can also deploy Helm charts from upstream repositories. The template repo includes a sample Application manifest that deploys podinfo via its official Helm chart.

### Review and activate the sample

```bash
cd ~/tutorial-argocd-tx2026/p2-podinfo-helm
cat app.yaml.sample
```

This manifest points ArgoCD at the upstream podinfo Helm chart (not your Git repo). It uses **manual sync** (no auto-sync) -- we'll use this in Part III to practice rollbacks.

```bash
cp app.yaml.sample app.yaml
```

### Create the Application

```bash
argocd app create -f app.yaml
argocd app sync podinfo-helm
```

### Verify

```bash
kubectl get pods -n podinfo-helm
kubectl port-forward svc/podinfo-helm-podinfo 9899:9898 -n podinfo-helm &
curl localhost:9899 | jq .message
# Should show "Helm-deployed podinfo!"
```

You now have two podinfo instances: one from plain manifests (port 9898), one from Helm (port 9899).

### Modify Helm values

Edit `p2-podinfo-helm/app.yaml` -- change the `ui.color` or `ui.message`, then re-apply:

```bash
argocd app create -f app.yaml --upsert
argocd app sync podinfo-helm
```

### Key takeaway

| | Plain Manifests | Helm Chart |
| --- | --- | --- |
| Source | YAML in your Git repo | Upstream chart repo |
| Customization | Edit YAML directly | Set Helm values |
| Use case | Custom apps | Third-party software |

ArgoCD handles both. In production you'll use a mix.

---

**Next:** Part III -- Advanced Topics
