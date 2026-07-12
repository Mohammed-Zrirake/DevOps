#Git #DevOps #Reflog #GitReset #CommitRecovery #GitInternals #CoreConcept

> [!abstract] Brief Description
> This note provides a step-by-step walkthrough for recovering lost commits after a destructive `git reset --hard` command. You will learn how to locate dangling commit hashes in the reflog ledger and safely restore them to your active branch.

---

> [!note] 📖 The Core Analogy: The Wastepaper Recovery Ledger
> Imagine working in a physical archive office:
> - **The Action (`git reset --hard`):** You decide the page you just wrote (your latest commit) is trash. You toss it into the office paper shredder, leaving your desk completely empty.
> - **The Recovery (Reflog):** The shredder is actually a delayed-disposal bin that files items into floorboard storage (the object database) and logs each action in a security ledger (the reflog). To get your page back, you look up its document number in the ledger, retrieve the page from the floorboard storage, and place it back on your desk.

---

## ⚠️ 1. The Danger of `git reset --hard`

The `git reset --hard <commit-hash>` command is highly destructive because it:
1.  Moves your current branch pointer backward to a historical commit.
2.  Discards all changes in your staging area and working directory.

Once executed, `git log` recalculates your history starting from the new, older branch tip. The commits you reset away are hidden from `git log` and appear to be permanently deleted.

---

## 🛠️ 2. Step-by-Step Recovery Walkthrough

Imagine we had a commit called `add summer seeds` that was accidentally wiped out by a hard reset:

```bash
# Lineage before reset:
# FB5072A (HEAD -> master) add summer seeds
# 9a8b7c6 plant winter veggies
# 2c3d4e5 add veggies file

git reset --hard 9a8b7c6
# Commit FB5072A is now gone from git log!
```

### Step 1: Query the Reflog
Run the `git reflog` command to inspect your local action history:

```bash
git reflog

# Output:
# 9a8b7c6 HEAD@{0}: reset: moving to 9a8b7c6
# FB5072A HEAD@{1}: commit: add summer seeds   <-- This is our target!
# 9a8b7c6 HEAD@{2}: commit: plant winter veggies
```

### Step 2: Inspect the Commit (Optional)
To verify that this commit contains the lost work, check it out into a detached HEAD state:

```bash
git checkout FB5072A
# Verify your files are restored.
```

### Step 3: Restore the Commit Permanently
Switch back to your active branch (`master`) and reset it forward to the target commit hash (or the reflog coordinate `master@{1}`):

```bash
git switch master
git reset --hard FB5072A
# Or: git reset --hard master@{1}
```

### Step 4: Verify Restoration
Run `git log` to confirm that the lost commit and all its associated files are back in your active timeline:

```bash
git log --oneline
# Output:
# FB5072A (HEAD -> master) add summer seeds
# 9a8b7c6 plant winter veggies
# 2c3d4e5 add veggies file
```

---

## 🧠 3. Behind the Scenes: Unreachable Objects

Why does this recovery work?
*   When you reset a commit, Git **does not** delete the commit or file objects immediately.
*   The objects continue to exist in `.git/objects` as **unreachable/dangling objects** (they have no active branch pointer pointing to them).
*   Git prevents these dangling objects from being permanently deleted by its background **Garbage Collection (GC)** process because they are still referenced in the `.git/logs/` ledger. They will only be deleted once their reflog entry expires (after 90 days).

---

> [!summary] Key Takeaways
> - **Core concept:** Commits discarded via `git reset --hard` remain in the Git database as dangling objects and are indexed in the reflog.
> - **Key implementation detail:** You can restore a branch to a lost commit by running `git reset --hard <hash>` or `git reset --hard branch@{index}`.
> - **Best practice:** Act quickly when you realize a reset was made in error, as garbage collection or manual database pruning can eventually clean up unreachable commits.
