#Git #CoreCommand #UndoingChanges #Workflow #Advanced

>  `git reset` is a powerful command that moves your current branch pointer to a previous commit, effectively "undoing" one or more commits. It has two main modes: a **default reset** that keeps your changes, and a **hard reset** that permanently deletes them.

---

`git reset` is the tool you reach for when you realize you've made one or more commits that you want to completely remove from your local branch history. Perhaps you committed them to the wrong branch, or they represent a failed experiment.

> [!danger] The Golden Rule of Reset
> `git reset` rewrites history. Like `git commit --amend`, you should **NEVER** use `git reset` on commits that you have already pushed to a shared repository (like GitHub). It's a tool for cleaning up your *local* history before you share it.

## The Default Reset: "Uncommit" Your Changes

The default `git reset` (officially called a "mixed" reset) is the safer of the two options. It undoes your commits but preserves the work you did.

> [!info] The Command
> `git reset <commit>`

-   **Effect on Commits:** It moves the branch pointer back to the specified `<commit>`, effectively removing all subsequent commits from the branch's history.
-   **Effect on Files:** The changes from those removed commits are **kept** in your **Working Directory** as unstaged modifications.

You lose the commits, but you don't lose the code.

### 🛠️ Hands-On: The Default Reset

1.  **Make some mistakes:** In our demo repo, make a couple of bad commits.
    ```bash
    # Make a messy change in your files...
    git commit -a -m "Mistake commit"
    
    # Make another messy change...
    git commit -a -m "Another bad commit"
    ```
    Your `git log` now shows these two unwanted commits at the top.

2.  **Find your target:** Identify the hash of the last "good" commit you want to return to.
    ```bash
    git log --oneline
    # 4661abc (HEAD -> master) Another bad commit
    # d3a4b1c Mistake commit
    # e7f8g9h fourth commit  <-- This is our target!
    ```

3.  **Perform the reset:**
    ```bash
    git reset e7f8g9h
    ```
    **Output:** `Unstaged changes after reset: M cat.txt M dog.txt`

4.  **Verify the result:**
    -   `git log --oneline` will now show that "fourth commit" is the latest commit. The two mistake commits are gone from the history.
    -   `git status` will show that `cat.txt` and `dog.txt` are `modified`.
    -   If you open the files, you'll see the messy changes from the "mistake" commits are still there in your working directory.

### A Powerful Use Case: Moving Commits
The default reset is perfect for when you've made commits on the wrong branch.

1.  **`git reset <last-good-commit>`**: Removes the commits from the wrong branch (e.g., `master`), but keeps the file changes.
2.  **`git switch -c <correct-branch-name>`**: Create and switch to the correct branch.
3.  **`git add .` & `git commit`**: Re-commit the changes, now on the correct branch. Your `master` branch is now clean.

---

## The Hard Reset (`--hard`): "Nuke It From Orbit"

This is the destructive, "no-going-back" version of reset. It undoes your commits AND deletes all the work associated with them.

> [!danger] Use With Extreme Caution!
> `git reset --hard` will permanently delete your commits AND discard all changes in your working directory. Any uncommitted work will be lost forever.

> [!info] The Command
> `git reset --hard <commit>`

-   **Effect on Commits:** Same as the default reset—moves the branch pointer back.
-   **Effect on Files:** All changes in your working directory and staging area are **permanently deleted**, and your files are reverted to match the state of the specified `<commit>`.

### 🛠️ Hands-On: The Hard Reset

1.  **Make another mistake:** Let's make one more bad commit on `master`.
    ```bash
    # Make some unwanted changes...
    git commit -a -m "Undo this commit please"
    ```
2.  **Find your target:** We'll reset back to the "third commit."
    ```bash
    git log --oneline
    # 1a2b3c4 (HEAD -> master) Undo this commit please
    # e7f8g9h fourth commit
    # 4j5k6l7 third commit <-- This is our target!
    ```

3.  **Perform the hard reset:**
    ```bash
    git reset --hard 4j5k6l7
    ```
    **Output:** `HEAD is now at 4j5k6l7 third commit`

4.  **Verify the result:**
    -   `git log --oneline` will show "third commit" as the latest commit. The "fourth commit" and "Undo this..." commit are gone.
    -   `git status` will show a `clean working tree`.
    -   If you open your files, you'll see they have been reverted to their "third commit" state. All the later changes have been completely erased.

### Reset is Branch-Specific
Remember, `git reset` only affects the current branch. If another branch also contained the commits you just reset, those commits will still exist on that other branch. You are only rewriting the history of the branch you are currently on.

### Summary Table

| Command | Effect on Commits | Effect on Working Directory | Primary Use Case |
|---|---|---|---|
| **`git reset <commit>`** | **Removes** commits after `<commit>` | **Keeps** the changes as unstaged modifications | Moving commits to a different branch. |
| **`git reset --hard <commit>`** | **Removes** commits after `<commit>` | **Deletes** all changes, reverting files to match `<commit>` | Completely and permanently deleting bad work. |