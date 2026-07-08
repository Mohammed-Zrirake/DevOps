#Git #GitHub #Remote #Workflow #CoreCommand

> The `git push` command is the bridge between your **local** repository and the **remote** (GitHub) repository. It uploads your commits and creates or updates branches on the remote to match your local history.

---

## 🚀 The Basics: Uploading Code

Before you push, your commits exist **only** on your machine. To share them or back them up, you use `git push`.

### Syntax
```bash
git push <remote> <branch>
```

-   **`<remote>`**: The name of the remote connection (usually `origin`).
-   **`<branch>`**: The local branch you want to upload (e.g., `master`, `main`, `feature-x`).

### Example: Pushing Master
```bash
git push origin master
```
This tells Git: "Take my local `master` branch and push its commits to the `master` branch on the remote `origin`."

> [!NOTE] Master vs. Main
> You might see `master` or `main` used as default branch names.
> *   **Master**: The traditional default name in Git.
> *   **Main**: GitHub changed their default to `main` in late 2020.
> The commands are identical regardless of the name: `git push origin main` works just like `git push origin master`.

### Authentication
When you push, GitHub needs to know who you are.
*   **Password/Personal Access Token**: You may be prompted to enter your username and password.
*   **SSH**: Configuring SSH keys allows you to push without entering credentials every time. (Recommended)

---

## 🌿 Branching Workflows

You are not limited to pushing the default branch. You can push **any** local branch to GitHub.

### Creating New Remote Branches & Deleting
If you create a local branch called `empty` and work on it:
```bash
# 1. Switch to new branch
git switch -c empty

# 2. Make commits (e.g., deleting a "mood board" folder)
git rm -r mood_board
git commit -m "Delete mood board"

# 3. Push the new branch
git push origin empty
```
Git will create a **new branch** on GitHub called `empty` and upload your commits.

> **Crucial Concept:** GitHub does not automatically sync.
> *   If you make new commits locally on `master`, GitHub's `master` doesn't change.
> *   If you push `empty`, it doesn't affect `master` on GitHub.
> You must explicitly push every branch you want to share.

---

## 🔗 Local vs. Remote Branches

It is important to understand that your local branches and remote branches are **distinct objects**.

-   **Local**: `master` (on your machine).
-   **Remote**: `origin/master` (the version GitHub has).
-   **Connection**: They are often connected, but can diverge.

### Advanced: The Colon Syntax
You *can* push a local branch to a remote branch with a different name.

**Syntax**: `git push <remote> <local-name>:<remote-name>`

For example, if you have a local branch `pancake` but want it to be named `waffle` on GitHub:
```bash
git push origin pancake:waffle
```
*   **Source**: Local `pancake` branch.
*   **Destination**: Remote `waffle` branch.
*(This is rare and not recommended for daily use, but it proves that local and remote branches are separate things.)*

---

## ✨ The `-u` Flag (Set Upstream)

The **first time** you push a new branch, you should use the `-u` flag (or `--set-upstream`).

```bash
git push -u origin <branch-name>
```

### What it does:
1.  **Pushes the code**: Uploads the branch to the remote (creating it if it doesn't exist).
2.  **Sets the Upstream**: Tells Git to **remember** the link between your local branch and the remote branch.

### The Payoff: Shortcuts!
Once the upstream is set, you can simply run:
```bash
git push
```
Git remembers where to send your code!

-   **Without `-u`**: Git might complain: `fatal: The current branch has no upstream branch.`
-   **With `-u`**: Git knows exactly where to go.

---

## 🛠️ Hands-On: The Full Workflow

1.  **Create a branch & commit:**
    ```bash
    git switch -c dogs
    touch dogs.txt
    git add dogs.txt
    git commit -m "Add dogs file"
    ```

2.  **Push for the first time (with upstream):**
    ```bash
    git push -u origin dogs
    ```
    *Output: Branch 'dogs' set up to track remote branch 'dogs' from 'origin'.*

3.  **Make more changes:**
    ```bash
    echo "Poodle" >> dogs.txt
    git commit -am "Update dogs"
    ```

4.  **Push subsequent changes easily:**
    ```bash
    git push
    ```

## 🏷️ The "Master" vs. "Main" Naming Shift

In 2020, GitHub changed the default branch name for new repositories from `master` to `main`. While functional identical, this can cause confusion if your local Git creates `master` but GitHub expects `main`.

### The Scenario
1.  **Locally**: You run `git init`, creating a `master` branch (unless configured otherwise).
2.  **Remote**: You create a repo on GitHub, which now defaults to `main`.
3.  **Conflict**: You try to push `master`, but GitHub shows `main`.

### How to Rename to `main`
If you are on `master` and want to switch to `main`:

```bash
# 1. Rename the current branch to main
git branch -M main

# 2. Push and set upstream
git push -u origin main
```
*   The `-M` flag moves/renames the branch even if `main` exists (force rename).

### Changing the Default on GitHub
If you pushed `master` but want `main` to be the default:
1.  Push the `main` branch to GitHub.
2.  Go to your repository **Settings** on GitHub.
3.  Click **Branches** on the left sidebar.
4.  Switch the **Default branch** to `main`.

> **Takeaway:** It is just a name! Git doesn't care if it's `master`, `main`, or `banana`. However, `main` is the modern standard.

> [!summary] Best Practice
> Always use `git push -u origin <branch>` the **first time** you push a new branch. It configures Git to track the relationship, enabling simpler commands (`git push`, `git pull`, `git status`) for all future work on that branch.
