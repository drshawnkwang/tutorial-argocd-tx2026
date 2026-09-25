# Part II: Core GitOps Exercises

## GitOps in a Nutshell

1. You declare your desired state in Git (YAML manifests)
2. ArgoCD watches the Git repo
3. ArgoCD syncs the cluster to match Git
4. If someone changes the cluster directly, ArgoCD detects the drift and corrects it

**Git is the single source of truth.**

> **Where do I run this?** Every command block below is labeled **Laptop (Git)** or (Student) **VM Terminal 1**. If you chose Option C or D in the [GitHub Access Guide](GITHUB-ACCESS.md), run everything on the Student VM.

## Exercise 1: Deploy Your First Application

### Verify your repo

You forked and cloned the tutorial repo in Part I. Make sure you are in the repo directory:

**VM Terminal 1:**

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

**Or via CLI, on the VM Terminal 1:**

```bash
argocd app create podinfo \
  --repo https://github.com/<your-username>/tutorial-argocd-tx2026 \
  --path p2-podinfo \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

### Sync the Application

The app shows **OutOfSync**, Git has manifests but nothing is deployed yet.

1. Click **SYNC** then **SYNCHRONIZE** in the UI

Or via CLI, on the **VM Terminal 1**:

```bash
argocd app sync podinfo
```

### Verify

**VM Terminal 1:**

```bash
kubectl get pods -l app=podinfo
curl localhost:32098 | jq .
```

You should see podinfo's JSON response with version `6.13.0`.

The podinfo service is exposed as a NodePort on port **32098**. You can also open `http://<your-vm-ip>:32098` in your browser to see the podinfo web UI.

## Exercise 2: Enable Auto-Sync and Self-Heal

### Enable auto-sync

In the ArgoCD UI: click on the `podinfo` app -> **DETAILS** -> **Edit** -> **Sync Policy** -> **ENABLE AUTO-SYNC**. Also Enable **Prune** and **Self Heal**.

Click **Save**.

Or via CLI, on the **VM Terminal 1**:

```bash
argocd app set podinfo --sync-policy automated --auto-prune --self-heal
```

### Test auto-sync

Edit `p2-podinfo/deployment.yaml`; change `replicas: 1` to `replicas: 3`:

**Laptop (Git):**

```bash
# Edit the file, then:
(editor) p2-podinfo/deployment.yaml
git add p2-podinfo/deployment.yaml
git commit -m "Scale podinfo to 3 replicas"
git push origin main
```

Now switch to the VM. Watch ArgoCD detect the change and create new pods (may take up to 3 minutes). Press `Ctrl-C` to stop watching:

**VM Terminal 1:**

```bash
kubectl get pods -l app=podinfo -w
```

### Test self-heal

Manually scale the deployment (drift from Git). This is a live cluster change, so there is no Git step here:

**VM Terminal 1:**

```bash
kubectl scale deployment podinfo --replicas=5
kubectl get pods -l app=podinfo
```

Watch ArgoCD detect the drift and scale back to 3. **Git always wins.**

## Exercise 3: Make Changes via Git

All the editing and Git work in this exercise happens in one place, then you switch to the VM once to verify.

Edit `p2-podinfo/deployment.yaml`; make two changes:

1. Upgrade the image tag from `6.13.0` to `6.14.0`
2. Add a custom message,  find the `containers` section and add an `env:` block at the same indentation level as `image:` and `ports:`:

   ```yaml
       containers:
       - name: podinfo
         image: ghcr.io/stefanprodan/podinfo:6.14.0   # <-- updated
         ports:
         - containerPort: 9898
           name: http
         env:                                          # <-- add from here
         - name: PODINFO_UI_MESSAGE
           value: "Hello from ArgoCD tutorial!"
   ```

Commit and push both changes together:

**Laptop (Git):**

```bash
git add p2-podinfo/deployment.yaml
git commit -m "Upgrade podinfo to 6.14.0 and set custom message"
git push origin main
```

Now switch to the VM and verify (after ArgoCD syncs):

**VM Terminal 1:**

```bash
curl localhost:32098 | jq '{version, message}'
# Should show version "6.14.0" and message "Hello from ArgoCD tutorial!"
```

You can also refresh `http://<your-vm-ip>:32098` in your browser, the message should appear in the podinfo web UI.

This demonstrates that a single Git push can make multiple changes atomically. ArgoCD syncs the full desired state, not individual edits.

## Exercise 4: Explore Application Details

In the ArgoCD UI, click on the `podinfo` application and explore:

- **Tree view** : visual map of Deployment -> ReplicaSet -> Pods, plus Service
- **Resource details** : click any resource to see its manifest, events, and logs
- **Diff tab** : compare Git state vs live cluster state
- **History** : click **HISTORY AND ROLLBACK** to see every sync, with Git commit SHAs and timestamps

## Exercise 5: Deploy a Helm Chart

ArgoCD can also deploy Helm charts from upstream repositories. The template repo includes a sample Application manifest that deploys podinfo via its official Helm chart.

### Review and activate the sample

This whole exercise runs on the Student VM. Unlike the earlier exercises, `app.yaml` is applied directly with `argocd app create -f`, so it never gets committed or pushed.

**VM Terminal 1:**

```bash
cd ~/tutorial-argocd-tx2026/p2-podinfo-helm
cat app.yaml.sample
```

This manifest points ArgoCD at the upstream podinfo Helm chart (not your Git repo). It uses **manual sync** (no auto-sync), we'll use this in Part III to practice rollbacks.

**VM Terminal 1:**

```bash
cp app.yaml.sample app.yaml
```

> This copy stays on the VM. Do not commit it. In Part III you'll copy the same sample into `argocd-apps/` instead, and that one **does** get committed.

### Create the Application

**VM Terminal 1:**

```bash
argocd app create -f app.yaml
argocd app sync podinfo-helm
```

### Verify (again)

**VM Terminal 1:**

```bash
kubectl get pods -n podinfo-helm
curl localhost:32899 | jq .message
# Should show "Helm-deployed podinfo!"
```

You now have two podinfo instances:

- Plain manifests: `http://<your-vm-ip>:32098`
- Helm chart: `http://<your-vm-ip>:32899`

Open both in your browser to compare.

### Modify Helm values

Edit `p2-podinfo-helm/app.yaml`; change the `ui.color` or `ui.message`, then re-apply. Because this file is VM-local, you edit it on the VM with `vim` or `nano`, with no Git step:

**VM Terminal 1:**

```bash
(editor) p2-podinfo-helm/app.yaml
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

## Cleanup

Before moving to Part III, delete both Applications from Part II:

**VM Terminal 1:**

```bash
argocd app delete podinfo --cascade
argocd app delete podinfo-helm --cascade
```

> The `--cascade` flag (default) deletes both the Application and the deployed resources (pods, service, etc.). We'll recreate `podinfo-helm` using the App of Apps pattern in Part III.

---

**Next:** Part III - Advanced Topics
