# Part IV: Multi-Environment Deployments

## Why Multiple Environments?

Real-world applications run in multiple environments:

- **Dev** -- rapid iteration, testing new features
- **Staging** -- pre-production validation
- **Prod** -- live, serving users

Each environment differs in replicas, resources, image tags, and configuration. We use **Kustomize** to manage these differences without duplicating YAML.

## What is Kustomize?

Kustomize lets you define a **base** configuration and **overlays** that customize it per environment. It's built into `kubectl` -- no extra tools needed.

```text
p4-multi-env-demo/
├── base/                 # Common config (shared)
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/              # Dev: 1 replica, minimal resources
    ├── staging/          # Staging: 2 replicas, more memory
    └── prod/             # Prod: 3 replicas, high resources
```

## Exercise 1: Explore the Multi-Environment Structure

Your forked repo already includes a fully populated `p4-multi-env-demo/` directory. Explore it:

```bash
cd ~/tutorial-argocd-tx2026
find p4-multi-env-demo -type f | sort
```

### Walk through the files

**Base** (`p4-multi-env-demo/base/`):

- `deployment.yaml` -- nginx with 1 replica and base resource limits
- `service.yaml` -- ClusterIP on port 80
- `kustomization.yaml` -- lists the resources

**Overlays** customize the base:

- **Dev** -- namespace `dev`, prefix `dev-`, 1 replica
- **Staging** -- namespace `staging`, prefix `staging-`, 2 replicas, more memory
- **Prod** -- namespace `prod`, prefix `prod-`, 3 replicas, much higher resources

### Test Kustomize locally

Preview what each environment produces:

```bash
cd ~/tutorial-argocd-tx2026/p4-multi-env-demo

kubectl kustomize overlays/dev
kubectl kustomize overlays/staging
kubectl kustomize overlays/prod
```

Compare replica counts:

```bash
kubectl kustomize overlays/dev | grep replicas
kubectl kustomize overlays/staging | grep replicas
kubectl kustomize overlays/prod | grep replicas
```

You should see 1, 2, and 3.

## Exercise 2: Deploy All Environments

We'll deploy all three environments with a single ApplicationSet, managed by the App of Apps pattern from Part III.

### Activate the ApplicationSet

```bash
cd ~/tutorial-argocd-tx2026

# Copy the sample into argocd-apps/ so root-apps picks it up
cp p4-multi-env-demo/app-multi-env-demo.yaml.sample argocd-apps/myapp-environments.yaml

# Edit: replace <your-username> with your GitHub username

git add argocd-apps/myapp-environments.yaml
git commit -m "Add multi-environment ApplicationSet via App of Apps"
git push origin main
```

**What happens:**

1. `root-apps` detects the new file
2. It creates the ApplicationSet `myapp-environments`
3. The ApplicationSet generates three Applications: `myapp-dev`, `myapp-staging`, `myapp-prod`
4. Each deploys the corresponding Kustomize overlay
5. Namespaces are created automatically

One git push. Three environments. No `kubectl create namespace`. No `argocd app create`.

### Verify

```bash
argocd app list

kubectl get all -n dev
kubectl get all -n staging
kubectl get all -n prod

kubectl get deployment -n dev
kubectl get deployment -n staging
kubectl get deployment -n prod
```

Expected: dev = 1 replica, staging = 2, prod = 3.

## Exercise 3: Environment-Specific Changes

### Scenario 1: Update image in staging only

Edit `p4-multi-env-demo/overlays/staging/kustomization.yaml` -- add an `images` section:

```yaml
images:
- name: nginx
  newTag: 1.26-alpine
```

```bash
git add p4-multi-env-demo/overlays/staging/kustomization.yaml
git commit -m "Update staging to nginx 1.26-alpine"
git push origin main
```

Watch the ArgoCD UI -- staging updates, dev and prod are unchanged.

### Scenario 2: Add a ConfigMap to production

Create `p4-multi-env-demo/overlays/prod/configmap.yaml` with environment-specific settings, then add it to the prod `kustomization.yaml` resources list. Commit and push.

Only prod gets the ConfigMap.

### Scenario 3: Promote a base change

Edit `p4-multi-env-demo/base/deployment.yaml` -- change the image to `nginx:1.27`:

```bash
git add p4-multi-env-demo/base/deployment.yaml
git commit -m "Update base nginx to 1.27"
git push origin main
```

**What happens:**

- Dev updates to 1.27 (uses base directly)
- Staging stays on 1.26-alpine (overlay overrides the base)
- Prod updates to 1.27 (uses base)

To promote to staging: remove or update the image override in the staging overlay.

## Environment Promotion Strategies

| Strategy | How | Best for |
| --- | --- | --- |
| Manual | Edit overlay, commit, push | Learning, controlled deployments |
| Automated | CI/CD updates overlays | Fast teams with good test coverage |
| Branch-based | Merge dev -> staging -> main | Familiar Git workflow |

For this tutorial we use **manual promotion** -- you control exactly when each environment changes.

## Key Takeaways

- Kustomize base + overlays = DRY multi-environment config
- ApplicationSets generate one Application per environment from a template
- App of Apps manages the ApplicationSet from Git
- Changes propagate through Git commits -- pure GitOps
- The full arc: imperative -> declarative -> App of Apps -> ApplicationSets

---

**Congratulations!** You've completed the ArgoCD GitOps tutorial.
