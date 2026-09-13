# Git Multiple Account Setup Guide for Linux/Ubuntu

Using two GitHub accounts on the same Linux/Ubuntu PC is completely possible. The cleanest approach is to use **separate SSH keys**, **SSH host aliases**, and **Git user configuration per repository**.

This guide assumes:

- Ubuntu/Linux
- Git is installed
- OpenSSH is installed
- You have two GitHub accounts
- You want to use SSH instead of HTTPS

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Check Git and SSH](#2-check-git-and-ssh)
3. [Create the SSH Directory](#3-create-the-ssh-directory)
4. [Generate an SSH Key for the First Account](#4-generate-an-ssh-key-for-the-first-account)
5. [Generate an SSH Key for the Second Account](#5-generate-an-ssh-key-for-the-second-account)
6. [Start and Configure ssh-agent](#6-start-and-configure-ssh-agent)
7. [Add SSH Keys to GitHub](#7-add-ssh-keys-to-github)
8. [Configure SSH Host Aliases](#8-configure-ssh-host-aliases)
9. [Test Both GitHub Accounts](#9-test-both-github-accounts)
10. [Clone a Repository](#10-clone-a-repository)
11. [Configure Git Identity Per Repository](#11-configure-git-identity-per-repository)
12. [Create a New Project](#12-create-a-new-project)
13. [Change an Existing Repository to the Second Account](#13-change-an-existing-repository-to-the-second-account)
14. [Check Current Configuration](#14-check-current-configuration)
15. [Common Problems and Fixes](#15-common-problems-and-fixes)
16. [Recommended Ubuntu Setup](#16-recommended-ubuntu-setup)

---

# 1. Prerequisites

Install Git and OpenSSH if they are not already installed:

```bash
sudo apt update
sudo apt install git openssh-client
```

Check the installed versions:

```bash
git --version
ssh -V
```

You should see output similar to:

```text
git version 2.x.x
OpenSSH_9.xp1 Ubuntu-...
```

---

# 2. Check Git and SSH

Before configuring multiple accounts, check whether SSH keys already exist:

```bash
ls -la ~/.ssh
```

Typical files may include:

```text
id_rsa
id_rsa.pub
id_ed25519
id_ed25519.pub
config
known_hosts
```

> **Important:** Do not overwrite an existing SSH private key unless you are sure it is no longer needed.

For new setups, **Ed25519** is recommended instead of RSA.

---

# 3. Create the SSH Directory

If the `.ssh` directory does not exist:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

---

# 4. Generate an SSH Key for the First Account

If you already have a working SSH key for your first GitHub account, you can skip this section.

Generate an Ed25519 key:

```bash
ssh-keygen -t ed25519 -C "your_first_email@example.com" -f ~/.ssh/id_ed25519
```

When prompted:

```text
Enter passphrase (empty for no passphrase):
```

Using a passphrase is recommended.

This creates:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

Where:

- `id_ed25519` → private key; **never share this**
- `id_ed25519.pub` → public key; this can be added to GitHub

---

# 5. Generate an SSH Key for the Second Account

Generate a separate key for the second GitHub account:

```bash
ssh-keygen -t ed25519 -C "your_second_email@example.com" -f ~/.ssh/id_ed25519_second
```

This creates:

```text
~/.ssh/id_ed25519_second
~/.ssh/id_ed25519_second.pub
```

The two accounts now have separate keys:

```text
First account:
~/.ssh/id_ed25519

Second account:
~/.ssh/id_ed25519_second
```

---

# 6. Start and Configure ssh-agent

Start the SSH agent:

```bash
eval "$(ssh-agent -s)"
```

Add the first key:

```bash
ssh-add ~/.ssh/id_ed25519
```

Add the second key:

```bash
ssh-add ~/.ssh/id_ed25519_second
```

Check which keys are loaded:

```bash
ssh-add -l
```

You should see both keys listed.

## Optional: Automatically Start ssh-agent

On Ubuntu, your desktop session may already manage an SSH agent. If `ssh-add` works without manually running `eval`, you generally do not need to configure anything else.

If you use a custom shell/session setup, you can start the agent from your shell configuration, but avoid starting a new agent unnecessarily every time a terminal opens.

---

# 7. Add SSH Keys to GitHub

You need to add each public key to the corresponding GitHub account.

## First Account

Display the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output.

Then, while logged into your **first GitHub account**:

**GitHub → Settings → SSH and GPG keys → New SSH key**

Paste the public key and save it.

---

## Second Account

Display the second public key:

```bash
cat ~/.ssh/id_ed25519_second.pub
```

Copy the entire output.

Then, while logged into your **second GitHub account**:

**GitHub → Settings → SSH and GPG keys → New SSH key**

Paste the public key and save it.

---

# 8. Configure SSH Host Aliases

This is the most important part of the setup.

Create or edit:

```bash
nano ~/.ssh/config
```

Add:

```sshconfig
# First GitHub account
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

# Second GitHub account
Host github-second
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_second
    IdentitiesOnly yes
```

Save the file.

If using `nano`:

```text
Ctrl + O
Enter
Ctrl + X
```

Set the correct permissions:

```bash
chmod 600 ~/.ssh/config
```

## Why `IdentitiesOnly yes`?

It tells SSH to use the identity specified for that host instead of trying unrelated keys from the SSH agent.

This is especially useful when multiple GitHub keys are loaded.

---

# 9. Test Both GitHub Accounts

## Test the First Account

Run:

```bash
ssh -T git@github.com
```

You should receive a message similar to:

```text
Hi FirstUsername! You've successfully authenticated, but GitHub does not provide shell access.
```

The username should belong to your **first GitHub account**.

---

## Test the Second Account

Run:

```bash
ssh -T git@github-second
```

You should receive:

```text
Hi SecondUsername! You've successfully authenticated, but GitHub does not provide shell access.
```

The username should belong to your **second GitHub account**.

---

## If SSH Asks About the Host

The first time you connect, you may see:

```text
The authenticity of host 'github.com' can't be established.
```

or:

```text
The authenticity of host 'github-second' can't be established.
```

SSH may ask whether you want to continue.

After verifying that you are connecting to GitHub, accept the host key.

The host information is stored in:

```bash
~/.ssh/known_hosts
```

---

# 10. Clone a Repository

The SSH alias determines which GitHub account is used.

## Clone From the First Account

Use the normal GitHub hostname:

```bash
git clone git@github.com:FirstUsername/repo-name.git
```

Example:

```bash
git clone git@github.com:myusername/my-project.git
```

---

## Clone From the Second Account

Use the `github-second` alias:

```bash
git clone git@github-second:SecondUsername/repo-name.git
```

Example:

```bash
git clone git@github-second:myworkaccount/my-project.git
```

The important difference is:

```text
First account:
git@github.com:...

Second account:
git@github-second:...
```

Once cloned, Git will continue using the configured remote URL.

---

# 11. Configure Git Identity Per Repository

SSH authentication and Git commit identity are **two different things**.

SSH determines:

> Which GitHub account can authenticate to the repository?

Git configuration determines:

> Which name and email are written into the commit?

For every repository, configure the appropriate identity.

Enter the project:

```bash
cd my-project
```

Set the name:

```bash
git config user.name "Second User"
```

Set the email:

```bash
git config user.email "your_second_email@example.com"
```

These settings are stored only for this repository.

Check them:

```bash
git config user.name
git config user.email
```

Or:

```bash
git config --local --list
```

---

## Why Use `--local`?

This:

```bash
git config user.email "your_second_email@example.com"
```

is equivalent to setting the repository-level configuration.

You can explicitly write:

```bash
git config --local user.name "Second User"
git config --local user.email "your_second_email@example.com"
```

Repository-level configuration is recommended when you work with multiple accounts.

---

# 12. Create a New Project

Suppose you want to create a new project under your second GitHub account.

## Step 1: Create the Repository on GitHub

Log into your **second GitHub account** and create a new repository.

For example:

```text
Repository name: my-project
```

For a new local project, you can create the GitHub repository without initializing it with a README.

---

## Step 2: Initialize Git Locally

```bash
mkdir my-project
cd my-project
git init
```

---

## Step 3: Configure the Second Account

```bash
git config --local user.name "Second User"
git config --local user.email "your_second_email@example.com"
```

---

## Step 4: Add the Remote

Use the second-account SSH alias:

```bash
git remote add origin git@github-second:SecondUsername/my-project.git
```

Check the remote:

```bash
git remote -v
```

You should see:

```text
origin  git@github-second:SecondUsername/my-project.git (fetch)
origin  git@github-second:SecondUsername/my-project.git (push)
```

---

## Step 5: Add and Commit

```bash
git add .
git commit -m "Initial commit"
```

---

## Step 6: Set the Main Branch

```bash
git branch -M main
```

---

## Step 7: Push

```bash
git push -u origin main
```

The repository will use the second GitHub account because the remote uses:

```text
github-second
```

---

# 13. Change an Existing Repository to the Second Account

Suppose an existing project currently uses:

```text
git@github.com:SecondUsername/my-project.git
```

but you want it to use the second SSH key.

First, enter the project:

```bash
cd my-project
```

Check the current remote:

```bash
git remote -v
```

Change it:

```bash
git remote set-url origin git@github-second:SecondUsername/my-project.git
```

Verify:

```bash
git remote -v
```

Then configure the commit identity:

```bash
git config --local user.name "Second User"
git config --local user.email "your_second_email@example.com"
```

Test the connection:

```bash
ssh -T git@github-second
```

Then push:

```bash
git push
```

---

# 14. Check Current Configuration

When working with multiple accounts, these commands are useful.

## Check Remote URL

```bash
git remote -v
```

Example:

```text
origin  git@github-second:SecondUsername/my-project.git (fetch)
origin  git@github-second:SecondUsername/my-project.git (push)
```

---

## Check Repository Git Identity

```bash
git config --local user.name
git config --local user.email
```

---

## Check All Repository Configuration

```bash
git config --local --list
```

---

## Check Global Git Configuration

```bash
git config --global --list
```

---

## Check Which SSH Key Is Being Used

For the second account:

```bash
ssh -vT git@github-second
```

For the first account:

```bash
ssh -vT git@github.com
```

The verbose output can show which identity files SSH is attempting to use.

---

# 15. Common Problems and Fixes

## Problem 1: GitHub Shows the Wrong Username

Run:

```bash
ssh -T git@github.com
```

and:

```bash
ssh -T git@github-second
```

Make sure each command returns the expected GitHub username.

If the second account returns the first username, check:

```bash
cat ~/.ssh/config
```

Make sure you have:

```sshconfig
Host github-second
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_second
    IdentitiesOnly yes
```

---

## Problem 2: Permission Denied (publickey)

You may see:

```text
Permission denied (publickey).
```

Check whether the key is loaded:

```bash
ssh-add -l
```

If necessary:

```bash
ssh-add ~/.ssh/id_ed25519_second
```

Then test again:

```bash
ssh -T git@github-second
```

---

## Problem 3: Wrong Remote URL

Check:

```bash
git remote -v
```

If the repository belongs to the second account but the remote uses:

```text
git@github.com:...
```

change it to:

```bash
git remote set-url origin git@github-second:SecondUsername/repo-name.git
```

---

## Problem 4: Wrong Commit Email

Check:

```bash
git config user.email
```

If it is incorrect:

```bash
git config --local user.email "your_second_email@example.com"
```

Remember that changing the Git email affects **new commits**. It does not automatically change the author information of existing commits.

---

## Problem 5: GitHub Does Not Associate the Commit With the Account

GitHub associates commits with an account when the commit email matches an email verified on that GitHub account.

Check:

```bash
git config user.email
```

Make sure it matches an email address added and verified in the appropriate GitHub account.

---

## Problem 6: `ssh-add` Says the Agent Is Not Available

If you see something like:

```text
Could not open a connection to your authentication agent.
```

start the agent:

```bash
eval "$(ssh-agent -s)"
```

Then:

```bash
ssh-add ~/.ssh/id_ed25519
ssh-add ~/.ssh/id_ed25519_second
```

---

# 16. Recommended Ubuntu Setup

For a two-account Ubuntu setup, the following structure is simple and reliable:

```text
~/.ssh/
├── config
├── known_hosts
├── id_ed25519
├── id_ed25519.pub
├── id_ed25519_second
└── id_ed25519_second.pub
```

SSH configuration:

```sshconfig
# Personal GitHub account
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

# Work/second GitHub account
Host github-second
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_second
    IdentitiesOnly yes
```

Then use:

```text
Personal repository:
git@github.com:username/repository.git

Second repository:
git@github-second:username/repository.git
```

And configure the Git identity separately in each repository:

```bash
git config --local user.name "Your Name"
git config --local user.email "your_email@example.com"
```

---

# Quick Reference

## SSH Keys

```bash
# First account
ssh-keygen -t ed25519 -C "first@example.com" -f ~/.ssh/id_ed25519

# Second account
ssh-keygen -t ed25519 -C "second@example.com" -f ~/.ssh/id_ed25519_second
```

## Add Keys

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add ~/.ssh/id_ed25519_second
```

## Test Accounts

```bash
ssh -T git@github.com
ssh -T git@github-second
```

## First Account Remote

```bash
git remote add origin git@github.com:FirstUsername/repo.git
```

## Second Account Remote

```bash
git remote add origin git@github-second:SecondUsername/repo.git
```

## Set Repository Identity

```bash
git config --local user.name "Your Name"
git config --local user.email "your_email@example.com"
```

## Check Everything

```bash
git remote -v
git config --local --list
ssh-add -l
ssh -T git@github.com
ssh -T git@github-second
```

---

# Important Security Notes

- **Never share your private SSH keys.**
- Never upload `id_ed25519` or `id_ed25519_second` to GitHub or anywhere else.
- Only add the `.pub` files to GitHub.
- Keep `~/.ssh/config` permissions at `600`.
- Keep the `.ssh` directory permissions at `700`.
- Use passphrases for SSH keys when possible.
- Do not commit `.env` files, private keys, passwords, or API secrets to Git repositories.

Recommended permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/id_ed25519
chmod 600 ~/.ssh/id_ed25519_second
chmod 644 ~/.ssh/id_ed25519.pub
chmod 644 ~/.ssh/id_ed25519_second.pub
```

---

# Final Workflow

Once everything is configured, you only need to remember one rule:

> **The SSH host in the Git remote determines which GitHub account/key is used.**

For example:

```bash
# Personal account
git clone git@github.com:personal-user/project.git

# Second account
git clone git@github-second:work-user/project.git
```

Then configure the commit identity inside each repository:

```bash
git config --local user.name "Your Name"
git config --local user.email "your_email@example.com"
```

After that, normal Git commands work as usual:

```bash
git add .
git commit -m "Your commit message"
git pull
git push
```

You do **not** need to switch accounts manually before every `git push` or `git pull`. Git automatically uses the SSH key associated with the host alias in that repository's remote URL.
