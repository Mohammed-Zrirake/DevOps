#Git #CoreCommand #Workflow #StagingArea #UndoingChanges

>  Use `git restore --staged <file-name>` to remove a file from the **Staging Area**, moving it back to the Working Directory as an unstaged change. This is the modern way to "un-add" a file you've accidentally staged.

---

This is the second primary function of the `git restore` command. While the first use case affected your Working Directory, this one affects the Staging Area.

## 🤔 The Problem: Staging by Mistake

This is a very common scenario. You're getting ready to make a commit and you either:
-   Accidentally use `git add .` and stage a sensitive file (like `secrets.txt`) that should have been in your `.gitignore`.
-   Stage a file that you realize belongs in a separate, future commit, breaking your goal of [[Best Practice: Keep Your Commits Atomic|atomic commits]].

You need a way to remove that file from the "Changes to be committed" list without losing the actual changes you've made to the file.

## ✨ The Solution: `git restore --staged`

The `--staged` flag tells `git restore` to operate on the Staging Area instead of the Working Directory.

> [!info] The Command
> `git restore --staged <file-name>`

This command tells Git: "Take `<file-name>` out of the staging area. The file itself should not be changed, just move it back to being a modified/untracked file in my working directory."

---

## 🛠️ Hands-On Demo: Unstaging a Secret File

Let's see this in action.

### Step 1: Create a Situation
We'll make some valid changes and also create a sensitive file that we don't want to commit.
1.  In `cat.txt` and `dog.txt`, add a new line: `fourth commit`.
2.  Create a new file, `secrets.txt`, with an API key inside.

### Step 2: The Mistake
In a hurry, we use `git add .` to stage everything at once.

```bash
git add .
```

### Step 3: The "Oh No!" Realization
We run `git status` and see our mistake.

```bash
git status
```
**Output:**
```
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   cat.txt
        modified:   dog.txt
        new file:   secrets.txt
```
Our `secrets.txt` file is now staged and will be included in the next commit. We need to get it out of there!

### Step 4: The Fix - Unstaging the File
As the helpful message from `git status` suggests, we use `git restore --staged`.

```bash
git restore --staged secrets.txt
```

### Step 5: Verify the Result
Let's check `git status` again.
```bash
git status
```
**Output:**
```
Changes to be committed:
        modified:   cat.txt
        modified:   dog.txt

Untracked files:
        secrets.txt
```
> [!success] The File is Unstaged!
> `secrets.txt` has been moved from the "Changes to be committed" section back to "Untracked files." Crucially, **the file `secrets.txt` and its contents have not been deleted or altered**. It's just no longer part of the upcoming commit.

Now we can safely make our intended commit, and then add `secrets.txt` to our `.gitignore` file.
```bash
git commit -m "fourth commit"
```

---

## `git status`: Your Ever-Present Guide

You don't need to memorize these commands perfectly. `git status` is designed to be your guide. It provides the exact commands you need for the most common "undo" operations.

**When a file is staged, `git status` tells you how to unstage it:**
```
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   some-file.txt
```

**When a file is modified in the working directory, `git status` tells you how to discard the changes:**
```
Changes not staged for commit:
  (use "git restore <file>..." to discard changes in working directory)
        modified:   another-file.txt
```

### Comparing the Two `restore` Commands

| Command | Starting Point | Action | Result |
|---|---|---|---|
| `git restore <file>` | A modified file in the **Working Directory** | Discards local changes | File is reverted to match `HEAD` |
| `git restore --staged <file>`| A staged file in the **Staging Area** | "Unstages" the file | File is moved back to the Working Directory as a modified/untracked change |