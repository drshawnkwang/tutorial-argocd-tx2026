# GitHub Access for This Tutorial

Throughout this tutorial you'll edit YAML files, commit, and push to your forked GitHub repo. ArgoCD on your student VM watches the repo and syncs changes automatically. Choose one of the four options below for how to do the Git work.

## How to Read the Command Blocks

Every command block in Parts II-IV is labeled with where to run it:

- **Laptop (Git)** : editing files, `git add`, `git commit`, `git push`
- **(Student ) VM Terminal 1** : `kubectl`, `argocd`, `curl`

If you chose **Option A or B** below, these are two different machines and you will switch between them. If you chose **Option C or D**, you do everything on the VM, so run the **Laptop (Git)** blocks on the student VM as well.

### Which files are Git-managed?

The tutorial has you copy several `.sample` files. Two different things are going on, and the label on each block tells you which:

| File | Where it lives | Committed to Git? |
| --- | --- | --- |
| Anything in `argocd-apps/` | Laptop (your clone) | **Yes** : this is how ArgoCD sees it |
| `p2-podinfo-helm/app.yaml` | VM only | No |
| `root-app.yaml` | VM only | No |

Files under `argocd-apps/` are read by the `root-apps` Application, so they only take effect once pushed to GitHub. The other two are applied directly with `argocd app create -f`, so they never need to be committed. If you are working entirely on the VM (Option C or D), `git status` will list them as untracked; that is expected, just leave them alone.

## Option A: Edit and Push from Your Laptop (Recommended)

You edit files and run Git commands on your laptop using your preferred editor and whatever GitHub authentication you already have (SSH keys, credential manager, GitHub Desktop, etc.). kubectl and argocd commands run on the VM.

- **Laptop:** edit files, `git add`, `git commit`, `git push`
- **Student VM:** `kubectl`, `argocd`, `curl`
- **When the VM needs updated files** (one time in Part III): run `git pull` on the VM

This is the recommended approach because you use your own editor and your existing GitHub auth, and there is no extra setup on the VM.

## Option B: Edit on GitHub.com with the Browser Editor

If you don't have Git installed locally, you can edit files directly on GitHub. Navigate to your forked repo and press `.` (period) to open a VS Code editor in your browser. Use the Source Control panel (Git icon in the left sidebar, looks like three circles connceted by lines, like the letter 'Y') to stage, commit, and push changes.

- **Browser:** edit files, commit, push (via github.dev VS Code editor)
- **Student VM:** `kubectl`, `argocd`, `curl`
- **When the VM needs updated files** (one time in Part III): run `git pull` on the VM

This assumes you are confortable with Github's VS code editor.

## Option C: Edit and Push from the Student VM Using a GitHub PAT

You do everything on the Student VM over SSH. You'll need a GitHub Personal Access Token (PAT) to push.

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

## Option D: Edit and Push from the Student VM Using SSH Agent Forwarding

If you have SSH keys configured with GitHub on your laptop, you can forward your SSH agent to the Student VM:

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
| Edit files on | Laptop | Browser | Student VM | Student VM |
| Git push from | Laptop | Browser | Student VM | Student VM |
| Pre-setup needed | None (use existing Git auth) | None | Create a GitHub PAT | SSH keys configured with GitHub |
| Editor | Your choice | VS Code in browser | vim/nano on Student VM | vim/nano on Student VM |

Please pick whichever you're most comfortable with.
