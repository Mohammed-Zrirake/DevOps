#Git #CoreCommand #Branching #Workflow

>  Use `git branch -d <name>` to safely delete a branch and `git branch -m <new-name>` to rename your **current** branch. Use `git branch -D <name>` to forcefully delete a branch, even if it has unmerged work.

---

While creating and switching branches is a daily activity, you'll also need to clean up and organize them. This involves deleting old branches and occasionally renaming them.

## 🗑️ Deleting Branches

It's common practice to delete a feature branch after its work has been successfully merged into your main branch. This keeps your repository clean and easy to navigate.

### The "Safe" Delete: `git branch -d`

The `-d` flag (for `--delete`) is the standard, safe way to delete a branch.

```bash
git branch -d <branch-name-to-delete>
```

> [!info] A Built-in Safety Net
> This command will only succeed if the branch you are deleting has been **fully merged** into your current branch. If the branch contains any unique commits (work that doesn't exist on your current branch), Git will refuse to delete it, protecting you from accidentally losing work.

### The "Force" Delete: `git branch -D`

If you are certain you want to delete a branch and discard all of its unique work, you can use the `-D` flag (an uppercase D).

```bash
git branch -D <branch-name-to-delete>
```

> [!danger] Use with Caution!
> The `-D` flag is a combination of `--delete --force`. It tells Git, "Delete this branch, even if it has unmerged commits." If this work isn't backed up anywhere else (like on GitHub), it will be lost. Only use this if you are absolutely sure you want to discard the work on that branch.

### 🛠️ Hands-On Workflow: Deleting a Branch
Let's walk through the process and its safety checks.

1.  **Create a branch to be deleted.** You can't delete the branch you're currently on, so we'll need to switch away from it first.
    ```bash
    # Create and switch to a new branch
    git switch -c delete-me
    
    # Now, switch back to master so we can delete 'delete-me'
    git switch master
    ```

2.  **Try the "Safe" Delete (and fail).** Since `delete-me` has no new commits, it's technically already "merged" and can be deleted safely. But let's pretend it had unique work. If it did, this is the error you'd see:
    ```bash
    git branch -d delete-me
    ```
    **Potential Error:**
    ```
    error: The branch 'delete-me' is not fully merged.
    If you are sure you want to delete it, run 'git branch -D delete-me'.
    ```
    This is Git's safety mechanism kicking in.

3.  **Force Delete the Branch.** Since we are sure we want to get rid of this branch, we use the `-D` flag.
    ```bash
    git branch -D delete-me
    ```
    **Output:**
    ```
    Deleted branch delete-me (was a1b2c3d).
    ```
    Running `git branch` will now show that `delete-me` is gone.

---

## ✏️ Renaming Branches

You might need to rename a branch if you made a typo or if the scope of the feature has changed.

### The Command: `git branch -m`

The command to rename a branch uses the `-m` flag, which stands for **"move"** (a historical quirk, but it's how you rename).

```bash
git branch -m <new-branch-name>
```

> [!tip] The Opposite Rule
> Unlike deleting, to rename a branch, you **must be on the branch you want to rename**.

### 🛠️ Hands-On Workflow: Renaming a Branch
Let's rename our `recentish-music` branch to a more accurate name, `2000s`.

1.  **Switch to the target branch.** We must be on the branch we intend to rename.
    ```bash
    git switch recentish-music
    ```

2.  **Run the Rename Command.**
    ```bash
    git branch -m 2000s
    ```

3.  **Verify the Change.** Your terminal prompt will likely update immediately. You can also verify with `git status` or `git branch`.
    ```bash
    git status
    # Output: On branch 2000s...
    
    git branch
    # Output will now show '* 2000s' instead of 'recentish-music'
    ```
Any new commits you make will now be on the `2000s` branch.

---

### Summary Table of Commands

| Action | Command | Key Rule |
|---|---|---|
| **Safe Delete** | `git branch -d <name>` | Cannot be on the branch. Fails if unmerged. |
| **Force Delete** | `git branch -D <name>` | Cannot be on the branch. Deletes regardless of merge status. |
| **Rename** | `git branch -m <new-name>` | **Must** be on the branch you want to rename. |