#Git #DevOps #Rebasing #Merging #HandsOn #Demo

> [!abstract] Brief Description
> This hands-on guide walks through a local demo comparing a merge-based workflow with a rebase-based workflow. You will learn how to set up a dummy repository, simulate parallel team development, and run `git rebase` to clean up branch history and align commits linearly.

---

> [!note] 📖 The Core Analogy: The Construction Site Log
> Imagine building a house where a framing crew and a painting crew work in parallel.
> - **Main Crew (Master Branch):** Pours foundation, builds walls, installs windows.
> - **Painting Crew (Feature Branch):** Mixes colors, paints trim.
> - **Merging:** The painters write in their logs daily: "Main crew installed windows; we paused to read their update." The painters' log becomes cluttered with structural updates instead of painting progress.
> - **Rebasing:** The painters rewrite their logs at the end, making it look as though they only arrived on site and started painting *after* the main crew completed all foundations, walls, and windows.

---

## 🏗️ 1. Setting Up the Demo Repository

To visualize the difference, we create a simple local repository called `RebaseDemo` and establish our base commits on `master`.

```bash
# Create and enter the directory
mkdir RebaseDemo
cd RebaseDemo

# Initialize a new Git repository
git init

# Create the initial website file
echo "WEBSITE CONTENT" > website.txt
git add website.txt
git commit -m "initial commit"

# Simulating work on master (e.g., adding a navbar)
echo "NAV BAR ADDED" >> website.txt
git commit -am "add navbar"
```

---

## 🔄 2. The Merge Workflow (The Cluttered Path)

Next, we branch off to work on a feature, while changes continue to happen on the `master` branch. We resolve this divergence using a standard merge.

### Step 1: Create a Feature Branch and Make Commits
```bash
git switch -c feat

# Create a new feature file to avoid direct conflicts
echo "ONE" > feature.txt
git add feature.txt
git commit -m "begin feature"
```

### Step 2: Simulate Parallel Work on Master
```bash
git switch master

# A coworker adds a footer in the meantime
echo "FOOTER ADDED" >> website.txt
git commit -am "add footer"
```

### Step 3: Merge Master into Feature (Merge Commit 1)
To bring the footer change onto our feature branch, we merge `master` into `feat`:
```bash
git switch feat
git merge master
# Close the editor prompt to accept the default merge commit message
```
This creates our first merge commit: `"Merge branch 'master' into feat"`.

### Step 4: Repeat the Cycle
We make more progress on the feature, and another developer adds a login form to `master`:
```bash
# More feature work
echo "TWO" >> feature.txt
git commit -am "more work on feature"

# Switch to master to simulate coworker work
git switch master
echo "LOGIN FORM ADDED" >> website.txt
git commit -am "add login form"

# Merge master into feat again (Merge Commit 2)
git switch feat
git merge master
```
Running `git log --oneline` on the `feat` branch now shows a cluttered history mixed with merge commits and master's history:
```text
* (HEAD -> feat) Merge branch 'master' into feat
* More work on feature
* Merge branch 'master' into feat
* work on feature
* begin feature
* (master) add login form
* add footer
* add navbar
* initial commit
```

---

## 🔑 3. The Rebase Workflow (The Linear Path)

Rather than keeping the cluttered history, we can use `git rebase` to rewrite the branch history. Rebasing takes our feature commits, sets them aside, moves our branch pointer to the tip of `master`, and replays our commits one-by-one.

> [!important] Rebase command syntax
> You must first switch to the branch you want to move (the feature branch), then rebase it **onto** the target branch (the master branch):
> ```bash
> git switch feat
> git rebase master
> ```

### The Rebase Execution Output
When running `git rebase master`, Git outputs:
```text
Successfully rebased and updated refs/heads/feat.
```
*(Under the hood, Git rewinds HEAD to replay your work on top of master, applying each commit sequentially.)*

### The New Linear History
If we inspect the commit graph now, we see a straight line:
```text
* (HEAD -> feat) work on feature (Rewritten)
* begin feature (Rewritten)
* (master) add login form
* add footer
* add navbar
* initial commit
```

---

## 🧬 4. Inspecting the Rewritten History

> [!warning] Commit Hashes Have Changed
> If you look closely at the commit hashes before and after the rebase, you will see they are completely different. 
> - **Why?** Git has created brand new commits based on the original content, giving them new parent references and generating new SHA-1 hashes.
> - **Metadata:** The author name, email, commit messages, and author dates are preserved, but they are technically entirely new Git objects.

### Master Remains Untouched
A common point of confusion is thinking that `git rebase master` modifies the `master` branch. It does not. The `master` branch pointer is unchanged. We are rebasing the `feat` branch **onto** the tip of `master`.

---

> [!summary] Key Takeaways
> - **Core concept:** Running `git rebase master` on a feature branch detaches your custom commits, updates the base of your branch to the tip of `master`, and replays your commits linearly.
> - **Key implementation detail:** While merging adds merge commits with multiple parents, rebasing generates a linear history where every commit has a single parent.
> - **Best practice:** Use rebasing locally to keep your history clean and readable before sharing. Avoid rebasing if other developers have already cloned or pulled your feature branch.
