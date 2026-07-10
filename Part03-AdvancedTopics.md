# Part III: Advanced Topics

## Exercise 1: App of Apps

So far you've created Applications using the ArgoCD UI or CLI. The **App of Apps** pattern lets ArgoCD manage Application definitions from using GitOps.

### How it works

A single "root" Application watches a directory in your repo. Any Application (or ApplicationSet) YAML you put in that directory gets automatically created by ArgoCD. Remove the file, and the Application is deleted.

### Set up the argocd-apps directory

```bash
cd ~/tutorial-argocd-tx2026
```

### Add podinfo-helm to the argocd-apps directory

```bash
cp p2-podinfo-helm/app.yaml.sample argocd-apps/podinfo-helm.yaml
git add argocd-apps/
git commit -m "Add podinfo-helm to argocd-apps"
```

### Create the root Application

```bash
cp root-app.yaml.sample root-app.yaml
# Edit root-app.yaml: replace <your-username> with your GitHub username
(editor) root-app.yaml
argocd app create -f root-app.yaml
```

> This is the **only** imperative `argocd app create` you'll need. Everything else goes through Git.

> **Note:** ArgoCD can access public Git repos and Helm chart repos without additional configuration. For private repositories, you would need to add credentials in ArgoCD Settings > Repositories.

### Verify

In the ArgoCD UI you should see `root-apps`. Click on it, its resources include the `podinfo-helm` Application.

Verify podinfo-helm is running:

```bash
curl localhost:30899 | jq '{message, color}'
```

You can also open `http://<your-vm-ip>:30899` in your browser -- you should see the green podinfo UI.

### Test the GitOps flow

Edit `argocd-apps/podinfo-helm.yaml`, change the `ui.message` value. Commit and push. Then manually sync using the Web UI or:

```bash
argocd app sync podinfo-helm
```

Verify the message changed:

```bash
curl localhost:30899 | jq .message
```

## Exercise 2: ApplicationSets

ApplicationSets generate multiple Applications from a template. Instead of writing one YAML file per app, you define a template with placeholders and a generator that fills them in.

### Review the sample

Look at `argocd-apps/appset-example.yaml.sample` in your repo. It uses a **List generator** with one element to deploy podinfo into a namespace.

### Deploy via App of Apps

Copy the sample to remove the `.sample` suffix, then commit and push:

```bash
cp argocd-apps/appset-example.yaml.sample argocd-apps/appset-example.yaml
git add argocd-apps/appset-example.yaml
git commit -m "Add podinfo ApplicationSet to root-apps"
git push origin main
```

`root-apps` picks up the new file, creates the ApplicationSet, and the ApplicationSet creates a `podinfo-podinfo-appset` Application automatically.

Verify:

```bash
curl localhost:30900 | jq '{message, color}'
```

You can also open `http://<your-vm-ip>:30900` in your browser -- you should see a red podinfo UI.

### See templating in action

Edit `argocd-apps/appset-example.yaml`, add a second element to the list, under the first:

```yaml
      - namespace: podinfo-appset-2
        message: "Second instance from AppSet!"
        color: "#ff6600"
        replicas: "1"
        nodePort: "30901"
```

```bash
git add argocd-apps/appset-example.yaml
git commit -m "Add second podinfo instance via ApplicationSet"
git push origin main
```

```bash
argocd app sync root-apps
```

A second Application appears. Verify both instances:

- Red: `http://<your-vm-ip>:30900`
- Orange: `http://<your-vm-ip>:30901`

Removing an element from the list deletes the corresponding Application.

### Clean up (Optional)

Remove the ApplicationSet from `argocd-apps/` and push , `root-apps` will prune it automatically:

```bash
git rm argocd-apps/appset-example.yaml
git commit -m "Remove podinfo ApplicationSet"
git push origin main
```

## Exercise 3: Rollbacks

The `podinfo-helm` Application has **no auto-sync** , rollbacks will stick.

### Confirm no auto-sync

```bash
argocd app get podinfo-helm | grep "Sync Policy"
```

### Break the application

Edit `argocd-apps/podinfo-helm.yaml` , change `targetRevision` to `99.99.99` (a version that doesn't exist):

```bash
(editor) argocd-apps/podinfo-helm.yaml
git add argocd-apps/podinfo-helm.yaml
git commit -m "Break podinfo-helm with invalid chart version"
git push origin main
```

Then trigger the sync:

```bash
argocd app get root-apps --refresh
argocd app sync podinfo-helm
```

The sync fails. Check the error:

```bash
argocd app get podinfo-helm
```

### Rollback via UI

1. Click on `podinfo-helm` in the ArgoCD UI
2. Click **HISTORY AND ROLLBACK**
3. Find the last successful sync
4. Click the three-dot menu -> **Redeploy**

The rollback **sticks** because there's no auto-sync to override it.

### Rollback via CLI (alternative)

```bash
argocd app history podinfo-helm
argocd app rollback podinfo-helm
```

### Fix Git

Even though the rollback worked, Git still has the broken version. Fix it:

```bash
git revert HEAD
git push origin main
```

### Key takeaway

- **Without auto-sync:** ArgoCD rollback sticks. You have manual control.
- **With auto-sync:** ArgoCD rollback is temporary , Git will re-apply the broken state. You must fix it in Git (`git revert`).

## Exercise 4: Sync Waves and Hooks

Sync waves and hooks give you control over **deployment order** and **lifecycle tasks**. The `p3-sync-waves-demo/` directory combines both into a single demo so you can see the full sync lifecycle in one shot.

### What's in the directory

The directory has six files. During a sync, ArgoCD processes them in this order:

1. **PreSync hook** : `pre-sync-job.yaml` (Job that simulates a database migration, runs before anything else)
2. **Wave -1** : `database.yaml` (ConfigMap deployed first among the regular resources)
3. **Wave 0** : `deployment.yaml` (Deployment deployed second)
4. **Wave 1** : `service.yaml` (Service deployed third)
5. **PostSync hook** : `post-sync-test.yaml` (Job that simulates a smoke test, runs after all resources are synced)

Waves are set via `argocd.argoproj.io/sync-wave` annotations. Hooks are set via `argocd.argoproj.io/hook` annotations (`PreSync`, `PostSync`). Hook Jobs are automatically deleted after success (`HookSucceeded` delete policy).

### Deploy via App of Apps

```bash
cp p3-sync-waves-demo/app-sync-wave-demo.yaml.sample argocd-apps/app-sync-wave-demo.yaml
# Edit: replace <your-username> with your GitHub username
(editor) argocd-apps/app-sync-wave-demo.yaml
git add argocd-apps/app-sync-wave-demo.yaml
git commit -m "Add sync-waves-demo to root-apps"
git push origin main
```

Then sync the demo:

```bash
argocd app sync root-apps
# after syncing root-apps, click on the sync-wave-demo in the Web UI
argocd app sync sync-waves-demo
```

### What to watch in the ArgoCD UI

Click on the `sync-waves-demo` Application during the sync. You should see:

1. The PreSync Job appears and runs to completion
2. The ConfigMap is created (wave -1)
3. The Deployment is created and pods start (wave 0)
4. The Service is created (wave 1)
5. The PostSync Job appears, runs the smoke test, and is cleaned up

The hook Jobs disappear after they succeed, that's the `HookSucceeded` delete policy in action.

---

**Next:** Part IV - Multi-Environment Deployments
