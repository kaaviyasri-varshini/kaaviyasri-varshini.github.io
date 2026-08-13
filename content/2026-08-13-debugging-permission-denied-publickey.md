Title: Debugging "Permission Denied (publickey)": A Git SSH Troubleshooting Story
Date: 2026-08-13
Category: Developer Tools
Tags: git, ssh, github, debugging, developer-tools, troubleshooting
Slug: debugging-permission-denied-publickey-git-ssh

## If You've Ever Run `git push` and Been Greeted With This...

```
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

...you know the particular flavor of dread it brings. The good news: this error is almost always fixable in a few minutes once you know where to look. Here's a walkthrough based on a real debugging session, including a couple of red herrings along the way.

## Step 1: Rule Out "No SSH Key At All"

The most common cause is exactly what it sounds like — there's no SSH key registered with GitHub yet, or the key isn't loaded. The fix is the standard SSH setup dance:

1. Check for an existing key with `ls -al ~/.ssh`
2. If none exists, generate one: `ssh-keygen -t ed25519 -C "your_email@example.com"`
3. Load it into the agent: `ssh-add ~/.ssh/id_ed25519`
4. Copy the public key: `cat ~/.ssh/id_ed25519.pub`
5. Paste it into GitHub under **Settings → SSH and GPG keys**
6. Test it: `ssh -T git@github.com`

In this case, though, SSH already worked fine for other repositories. That ruled out a missing key entirely — the key existed, and GitHub trusted it. So the hunt moved elsewhere.

## Step 2: When It Works Everywhere Else, Suspect the Repo

If SSH auth is healthy in general but one specific repo throws "permission denied," the usual suspects are:

- **You don't actually have push access** to that repo (wrong account, no collaborator rights)
- **The remote URL is wrong** — check with `git remote -v`
- **Multiple GitHub accounts** with different keys, where the wrong key is being offered for that particular repo

Running `git remote -v` confirmed the URL was correct and the repo belonged to the user's own account. So that wasn't it either.

## Step 3: The Real Culprit — It Was the Terminal Session, Not GitHub

Here's the twist: two diagnostic commands cracked it open.

```bash
ssh-add -l
```

This asks the **ssh-agent** — a background helper that holds your decrypted keys in memory — "which keys do you currently have loaded?" It came back with *"The agent has no identities."* Alarming at first glance, but it turned out to be a non-issue, because...

```bash
ssh -T git@github.com
```

This one actually opens a connection to GitHub as the `git` user (GitHub's generic SSH username — it identifies you by *which key* you present, not by this username) and asks it to confirm authentication, without opening a full interactive shell (that's what the `-T` flag does). It came back with:

```
Hi kaaviyasri-varshini! You've successfully authenticated, but GitHub does not provide shell access.
```

That was the real signal. It meant SSH was authenticating fine — just not via the agent, but by reading a key file directly, likely configured through `~/.ssh/config`. The empty agent was a red herring; the actual auth path was working the whole time.

Once that was confirmed, a fresh `git push` in that terminal session went through without issue.

## The Takeaways

A few lessons worth keeping in your back pocket for next time:

**"Permission denied (publickey)" doesn't always mean you have no key.** It can also mean the *wrong* key is being offered, or that a specific repo needs different access.

**`ssh-add -l` and `ssh -T git@github.com` are your two best diagnostic commands.** The first tells you what the agent is currently holding; the second tells you whether authentication actually succeeds end-to-end — and the second one matters more, since SSH doesn't strictly require the agent to work.

**When something works in one terminal but not another,** the difference is often environment-specific — a shell that didn't load your SSH agent, a different `~/.ssh/config`, or a stale session — rather than anything wrong with GitHub or the repo itself.

**Sometimes the fix is just retrying** once you've confirmed the underlying auth is actually fine. Not every red flag in the output is the actual problem.

Debugging SSH issues can feel like chasing shadows, but methodically ruling out "no key," then "wrong repo/account," then "this specific session's config" gets you there fast — and mostly it's a five-minute detour, not a crisis.
