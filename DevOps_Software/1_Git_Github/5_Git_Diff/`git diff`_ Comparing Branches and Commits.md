#Git #CoreCommand #Workflow #Branching #History


> **TL;DR:** Use `git diff <branch1>..<branch2>` to see all changes between two branches. The same syntax works for comparing any two commit hashes. For complex comparisons, a Git GUI is often much easier to use than the command line.

---

Beyond comparing your working states, `git diff`'s real power lies in its ability to compare any two points in your repository's history. This is essential for code reviews, understanding feature development, and debugging.

## 1. Comparing Branches

This is one of the most common and powerful uses of `git diff`. It answers the question: "What work has been done on this feature branch that isn't on my main branch yet?"

> [!info] The Command
> `git diff <branch1>..<branch2>`
>
> This shows all the changes needed to transform `<branch1>` into `<branch2>`. You can also use a space instead of `..`.

### The Order Matters!
The order in which you list the branches is critical.
-   `git diff master..feature`: Shows changes on `feature` that are **not** on `master`. (`master` is `a`, `feature` is `b`).
-   `git diff feature..master`: Shows changes on `master` that are **not** on `feature`. (`feature` is `a`, `master` is `b`).

### 🛠️ Hands-On Demo:
Let's create an `odd-numbers` branch in our demo repo and make some changes.

1.  **Create the branch and make commits:**
    ```bash
    git switch -c odd-numbers
    # Edit numbers.txt to only contain 1, 3, 5
    git commit -a -m "Remove even numbers and add 5"
    ```
2.  **Compare `master` to `odd-numbers`:**
    ```bash
    git diff master..odd-numbers
    ```
    **Output:**
    ```diff
    --- a/numbers.txt
    +++ b/numbers.txt
    @@ -1,4 +1,3 @@
     ONE
    -TWO
     THREE
    -FOUR
    +FIVE
    ```
    This shows that to get from `master` (`a`) to `odd-numbers` (`b`), we need to remove "TWO" and "FOUR" and add "FIVE".

---

## 2. Comparing Specific Commits

You can use the exact same syntax to compare any two commits in your repository's history. This is perfect for seeing what changed in a specific range of work.

> [!tip] The Command
> `git diff <commit-hash-1>..<commit-hash-2>`

### 🛠️ Hands-On Demo:
1.  **Find two commit hashes** using `git log --oneline`.
    ```
    a1b2c3d Add red
    d4e5f6g Add blue and purple
    ```
2.  **Run the diff:**
    ```bash
    git diff a1b2c3d..d4e5f6g
    ```
The output will show you the cumulative changes that occurred between those two specific points in time.

---

## 3. Narrowing the Scope to Specific Files

For large projects, a full diff between branches can be very noisy. You can append file paths to any `diff` command to limit its output to only those files.

```bash
# See changes in just the CSS file between master and a feature branch
git diff master..feature/new-layout -- styles/main.css

# See what changed in two specific files between two commits
git diff <hash1>..<hash2> -- index.html js/app.js
```

---

## Beyond the Terminal: Visualizing Diffs with a GUI

While the command line is powerful, reading complex diffs in a terminal can be difficult. This is an area where Git GUIs (Graphical User Interfaces) like GitKraken, Sourcetree, or the integrated source control in VS Code truly shine.

> [!success] The Advantages of a GUI for Diffs
> - **Visual Clarity:** GUIs present changes in a side-by-side or inline view with clear color highlighting, which is often much easier to read.
> - **Easy Navigation:** You can simply click on files to see their diffs, rather than typing long commands.
> - **Context at a Glance:** Easily switch between viewing staged, unstaged, and committed changes.
> - **Comparing Commits/Branches:** Most GUIs allow you to simply click one commit, then `Shift+Click` another to instantly see the diff for the entire range.
> - **Hunk vs. File View:** Easily toggle between viewing only the changed "hunks" (chunks) of code and seeing the changes within the context of the entire file.

For tasks like code reviews or analyzing a complex history, using a GUI's diff viewer is often a more productive and pleasant experience.

### Recap of `diff` Commands

| Command | Shows You... |
|---|---|
| `git diff` | **Unstaged** changes in the working directory. |
| `git diff --staged` | **Staged** changes (a preview of your next commit). |
| `git diff HEAD` | **All** changes (staged and unstaged) since the last commit. |
| `git diff <A>..<B>` | Changes between any two branches or commits. |

...and you can add `-- <file>` to any of them to narrow the scope.