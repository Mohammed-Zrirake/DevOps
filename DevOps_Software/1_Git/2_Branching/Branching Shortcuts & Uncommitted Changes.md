#Git #CoreCommand #Branching #Workflow #Troubleshooting

>  Use `git switch -c <name>` to create and switch to a new branch in one command. Be aware that the older `git checkout` command is often used for switching branches. Crucially, always commit or stash your changes before switching branches to avoid errors or losing work.

---

## ⚡ The One-Step Shortcut: Create & Switch

The two-step process of `git branch <name>` followed by `git switch <name>` is so common that Git provides a convenient shortcut.

> [!success] The `-c` Flag for Create
> The `git switch` command has a `-c` flag (for "create") that creates a new branch and immediately switches to it.

```bash
git switch -c <new-branch-name>
```

This is the modern and recommended way to create and move to a new branch in a single step.

**Example:**
Instead of two commands...
```bash
# We are currently on the 'master' branch
git branch recentish-music
git switch recentish-music
```
...you can do it in one:
```bash
# We are currently on the 'master' branch
git switch -c recentish-music
# Output: Switched to a new branch 'recentish-music'
```

---

## `git switch` vs. `git checkout`: Old vs. New

For a long time, the `git checkout` command was used for switching branches. You will see it in countless tutorials, articles, and codebases.

> [!info] The "Swiss Army Knife" Problem
> `git checkout` is a powerful and versatile command that does many things, including switching branches, restoring files, and more. Because it was overloaded with so many functions, it could be confusing for new users.

In recent versions of Git, new, more specific commands were introduced to make these actions clearer:
-   **`git switch`** is now the dedicated command for switching branches.
-   **`git restore`** is the dedicated command for discarding changes to files.

While `git switch` is preferred, `git checkout` still works perfectly for switching branches, and you should know how to recognize it.

### Command Comparison

| Action                  | Modern Command (`git switch`)         | Old Command (`git checkout`)            |
| ----------------------- | ------------------------------------- | --------------------------------------- |
| **Switch to a branch**  | `git switch oldies`                   | `git checkout oldies`                   |
| **Create & switch**     | `git switch -c recentish-music`       | `git checkout -b recentish-music`       |

> [!tip] Notice the confusing flag difference: `-c` for **c**reate in `switch`, but `-b` for **b**ranch in `checkout`. This is one of the reasons `git switch` is now preferred for its clarity.

---

## ⚠️ The Golden Rule: Commit or Stash Before You Switch

What happens if you try to switch branches while you have uncommitted changes in your working directory? Git will try to protect you from losing your work.

> [!danger] Do Not Switch with a "Dirty" Working Directory
> Before switching branches, your working directory should be "clean" (meaning `git status` shows `nothing to commit`). If you have uncommitted changes, you should either **commit** them or temporarily save them with `git stash` (which we will learn later).

### Scenario A: Git Stops You (Conflict)
This is the most common case. You have uncommitted changes in a file (`playlist.txt`), and you try to switch to another branch where that same file is different.

```bash
# On the 'oldies' branch, we add new songs to playlist.txt but don't commit them.
git status
# Output shows 'modified: playlist.txt'

git switch master
```
**Git will block the switch with an error:**
```
error: Your local changes to the following files would be overwritten by checkout:
        playlist.txt
Please commit your changes or stash them before you switch branches.
Aborting
```
Git is protecting you. If it allowed the switch, your uncommitted changes (the new Van Morrison songs) would be wiped out and lost forever. The solution is to commit your work first, *then* switch.

### Scenario B: Git Brings the Changes With You (No Conflict)
This is a more subtle and sometimes confusing case. You create a **brand new file** (`chickens.txt`) that doesn't exist on any other branch, but you don't commit it.

```bash
# On a new 'chicken' branch, we create chickens.txt but don't commit it.
git status
# Output shows 'untracked file: chickens.txt'

git switch oldies
```
**This time, the switch succeeds!** But something strange happens. The `chickens.txt` file is now present in your working directory while you are on the `oldies` branch.

Git allowed this because there was no conflict—the `oldies` branch didn't have a file called `chickens.txt` to overwrite. So, it brings the "homeless" uncommitted file along with you. This can get confusing, as you now have changes in your working directory that don't belong to the branch you're on.

---

> [!best-practice] The Safest Workflow
> The simplest and safest rule to follow until you learn `git stash` is: **Always have a clean working tree before you switch branches.**
>
> 1. Run `git status`.
> 2. If you see changes, `git add` and `git commit` them.
> 3. Once `git status` says `nothing to commit, working tree clean`, you are safe to `git switch` to another branch.