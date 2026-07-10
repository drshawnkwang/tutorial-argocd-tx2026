# ArgoCD Tutorial

A hands-on tutorial for learning GitOps with ArgoCD on Kubernetes. Designed for instructor-led workshops using cloud VMs with single-node k3s clusters.

## Tutorial Parts

- [Part I: Installing ArgoCD](Part01-Installing_ArgoCD.md) - Set up ArgoCD on a k3s cluster using Helm
- [Part II: Core GitOps Exercises](Part02-Core_GitOps_Excercises.md) - Deploy apps, configure auto-sync, make changes via Git
- [Part III: Advanced Topics](Part03-AdvancedTopics.md) - App of Apps, ApplicationSets, rollbacks, sync waves and hooks
- [Part IV: Multi-Environment Deployments](Part04-MultiEnvironment.md) - Kustomize overlays for dev/staging/prod

## Student Template Repository

Students fork [esnet/tutorial-argocd-tx2026](https://github.com/esnet/tutorial-argocd-tx2026) and connect it to their ArgoCD instance. That repo contains the exercise files referenced throughout this tutorial.

## Prerequisites

- A cloud VM with k3s installed (provided by the instructor)
- SSH access to the VM
- A GitHub account (for forking the template repo)

## License

This project is licensed under the Apache License 2.0. See [LICENSE](LICENSE) for details.
