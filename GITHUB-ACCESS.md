# GitHub Access for This Tutorial

Throughout this tutorial you'll edit YAML files, commit, and push to your forked GitHub repo. ArgoCD on your VM watches the repo and syncs changes automatically. Choose one of the three options below for how to do the Git work.

## Option A: Edit and Push from Your Laptop (Recommended)

You edit files and run Git commands on your laptop using your preferred editor and whatever GitHub authentication you already have (SSH keys, credential manager, GitHub Desktop, etc.). kubectl and argocd commands run on the VM.

- **Laptop:** edit files, `git add`, `git commit`, `git push`
- **VM:** `kubectl`, `argocd`, `curl`
- **When the VM needs updated files** (one time in Part III): run `git pull` on the VM

This is the recommended approach because you use your own editor and your existing GitHub auth, and there is no extra setup on the VM.

## Option B: Edit on GitHub.com with the Browser Editor

If you don't have Git installed locally, you can edit files directly on GitHub. Navigate to your forked repo and press `.` (period) to open a VS Code editor in your browser. Use the Source Control panel (Git icon in the left sidebar) to stage, commit, and push changes.

- **Browser:** edit files, commit, push (via github.dev VS Code editor)
- **VM:** `kubectl`, `argocd`, `curl`
- **When the VM needs updated files** (one time in Part III): run `git pull` on the VM

This assumes you are confortable with Github's VS code editor.

## Option C: Edit and Push from the VM Using a GitHub PAT

You do everything on the VM over SSH. You'll need a GitHub Personal Access Token (PAT) to push.

**Before the tutorial:**

1. On GitHub, click your profile icon (upper-right) > **Settings**. Then in the left sidebar, scroll to the bottom and click **Developer settings**. Go to **Personal access tokens** > **Tokens (classic)**.
2. Click **Generate new token (classic)**. Give it a name (e.g., "argocd-tutorial"), set the expiration to **7 days** (this is a short tutorial; there's no need for a long-lived token), and check the `repo` scope. This scope works on any repo you own, including repos you fork later.
3. Click **Generate token** and save the token somewhere safe. You won't be able to see it again.

**On the VM (one-time setup):**

```bash
git config --global credential.helper store
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

On the VM, on your first `git push`, enter your GitHub username and the PAT as the password. It will be cached for subsequent pushes.

## Option D: Edit and Push from the VM Using SSH Agent Forwarding

If you have SSH keys configured with GitHub on your laptop, you can forward your SSH agent to the VM:

```bash
# Connect with agent forwarding
ssh -A student@<your-vm-ip>

# Re-clone using the SSH URL
git clone git@github.com:<your-username>/tutorial-argocd-tx2026.git
```

Your laptop's SSH key is used for GitHub auth without storing credentials on the VM.

> **Note:** You must use `ssh -A` every time you connect for this to work. If you reconnect without `-A`, git push will fail.

## Summary

| | Option A: Laptop | Option B: GitHub.dev | Option C: VM + PAT | Option D: VM + SSH agent |
| --- | --- | --- | --- | --- |
| Edit files on | Laptop | Browser | VM | VM |
| Git push from | Laptop | Browser | VM | VM |
| Pre-setup needed | None (use existing Git auth) | None | Create a GitHub PAT | SSH keys configured with GitHub |
| Editor | Your choice | VS Code in browser | vim/nano on VM | vim/nano on VM |

Please pick whichever you're most comfortable with.
