# 🚀 Git Workflow Guide (Cloning and commiting a repository)

### Table of content:
      1. Cloning a Repositor
      2. Commiting a Repositor


# 📌 1. Cloning a Repository
To start working on an existing project, clone the repository:
```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
```
- This **automatically initializes Git** and sets up `origin` as the remote.
- You do **not** need to run `git init` and `git remote` after cloning.

### 🔹 Using a Personal Access Token (PAT)
If GitHub asks for a password while cloning, use a **Personal Access Token (PAT)** instead of your password.  

1. Generate a PAT from **GitHub → Settings → Developer Settings → Personal Access Tokens**.
2. Copy the generated token (**you won’t see it again**).
3. Use this command to clone with your PAT:
   ```bash
   git clone https://YOUR_USERNAME:<YOUR_PERSONAL_ACCESS_TOKEN>@github.com/YOUR_USERNAME/YOUR_REPO.git
   ```
4. If you don’t want to enter the token every time, use:
   ```bash
   git config --global credential.helper store
   ```


## 📝 A. Making Changes and Committing
After making changes, stage and commit them:
```bash
git add .
git commit -m "Your commit message"
```

### 🔹 Fix "Author Identity Unknown" Error
If you see this error:
```
Author identity unknown
```
Set your Git username and email **for this repository only**:
```bash
git config user.name "Your GitHub Display Name"
git config user.email "your-email@example.com"
```
the name should be the name i want to appear as the author of your commits.

The email should match the email associated with your GitHub account.

you can add `--global` for all repositories.


## ⬆️ B. Pushing Changes to GitHub
Push your changes to the repository:
```bash
git push origin main  # or your current branch
```

### ✅ If your branch is already tracked:
You can simply use:
```bash
git push
```

### ❌ If you see "no upstream branch" error:
Set the upstream branch with:
```bash
git push -u origin main
```
Now, `git push` will work automatically in the future.

---

## 🔄 C. Avoiding Force Push Issues
If Git asks you to use `--force`, first try pulling the latest changes:
```bash
git pull --rebase
git push
```
Use `git push --force` **only if you're sure** you want to overwrite remote changes.

---

# 📌 2. Commiting a Repository

# Git Workflow Guide

```bash
# Initialize a new Git repository
$ git init

# Add remote origin
$ git remote add origin https://tokenName:Token@URL.git

# Check branches
$ git branch
# Output example:
# * master (you may want to rename it to main)

# Fetch latest updates from origin
$ git fetch origin

# Show all remote branches (e.g., main, master)
$ git branch -r

# Switch to main branch
$ git checkout main

$ git add .

# you should to configure you name and email
$ git config user.name "username"
$ git config user.email "email"

$ gitcommit -m "your message"

# If my code was pushed to master and needs to be merged into main
$ git merge master  # Merge master into main

# Delete the master branch locally
$ git branch -d master

# Delete the master branch from GitLab
$ git push origin --delete master
```

## 📜 Summary of Key Commands

| Action | Command |
|---------|------------------|
| Clone a repository | `git clone <repo-url>` |
| Check branch | `git branch -vv` |
| Stage changes | `git add .` |
| Commit changes | `git commit -m "message"` |
| Set username & email (repo-only) | `git config user.name "Your Name"` <br> `git config user.email "your-email@example.com"` |
| Push changes | `git push origin main` (or just `git push` if upstream is set) |
| Set upstream branch | `git push -u origin main` |
| Pull latest changes | `git pull --rebase` |
| Set push default to current branch | `git config --global push.default current` |

---

## 📌 Notes
- **`main` vs `master`**: Some repositories use `master` instead of `main`. Run `git branch -a` to check your branch name.
- **Avoid `git push --force`** unless necessary to prevent overwriting remote changes.
- Always check your branch tracking with:
  ```bash
  git branch -vv
  ```

---

This guide provides the essential Git workflow from cloning a repository to pushing changes while avoiding common errors. 🚀
