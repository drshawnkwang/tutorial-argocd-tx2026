# ArgoCD Tutorial

A hands-on tutorial for learning GitOps with ArgoCD on Kubernetes. Designed for instructor-led workshops using cloud VMs with single-node k3s clusters.

## Getting Started

1. Review the [GitHub Access Guide](GITHUB-ACCESS.md) and choose how you'll push to GitHub
1. **Fork** this repository to your GitHub account
1. **SSH** into your assigned VM: `ssh student@<your-vm-ip>`
1. **Clone** your fork on the VM:

   ```bash
   git clone https://github.com/<your-username>/tutorial-argocd-tx2026.git
   cd tutorial-argocd-tx2026
   ```

1. Follow the tutorial parts in order, starting with Part I

## Tutorial Parts

- [Part I: Installing ArgoCD](Part01-Installing_ArgoCD.md) - Set up ArgoCD on a k3s cluster using Helm
- [Part II: Core GitOps Exercises](Part02-Core_GitOps_Excercises.md) - Deploy apps, configure auto-sync, make changes via Git
- [Part III: Advanced Topics](Part03-AdvancedTopics.md) - App of Apps, ApplicationSets, rollbacks, sync waves and hooks
- [Part IV: Multi-Environment Deployments](Part04-MultiEnvironment.md) - Kustomize overlays for dev/staging/prod

## Prerequisites

- A Kubernetes cluster. For the instructor-led tutorial, a cloud VM with k3s installed (provided by the instructor).
- However, a [kind](https://kind.sigs.k8s.io/) or [minikube](https://minikube.sigs.k8s.io/docs/) cluster may be used for self-instruction. Alternates include [k3s](https://k3s.io/).
- SSH access to the VM. You will need to have a ssh public-key for the instructor-led session.
- A GitHub account (for forking this repo).

## License

This project is licensed under the Apache License 2.0. See [LICENSE](LICENSE) for details.
