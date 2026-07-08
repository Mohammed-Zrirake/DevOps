#Git #CoreCommand #Workflow #History


> **TL;DR:** Use `git diff` for unstaged changes, `git diff --staged` for staged changes, and `git diff HEAD` to see everything since your last commit. Add a filename to any command to narrow its scope.

---

The `git diff` command is a powerful tool for inspecting changes between the three "zones" of Git: the Working Directory, the Staging Area, and your last commit (`HEAD`). While we've learned `[[`git diff`: Viewing Changes|how to read a diff]]`, let's now master the different comparisons we can make.

## 1. `git diff` (Unstaged Changes)

This is the default behavior we've seen before.

-   **Command:** `git diff`
-   **Compares:** Working Directory **vs.** Staging Area.
-   **Shows you:** All modifications you have made that you have **not yet staged** with `git add`. It answers the question, "What have I changed that I haven't prepared for my next commit?"

## 2. `git diff --staged` (Staged Changes)

This is the logical counterpart to the default `diff`. It lets you see the changes you *have* staged.

-   **Command:** `git diff --staged` (or its alias `git diff --cached`)
-   **Compares:** Staging Area **vs.** Last Commit (`HEAD`).
-   **Shows you:** All the changes you have successfully added to the staging area. It's a **preview of your next commit**.

## 3. `git diff HEAD` (All Changes)

This command gives you the big picture of everything you've done since your last save point.

-   **Command:** `git diff HEAD`
-   **Compares:** Working Directory **vs.** Last Commit (`HEAD`).
-   **Shows you:** **All changes** since your last commit, regardless of whether they are staged or unstaged. It answers the question, "What is the total difference between my last commit and my current work?"

---

### Summary Table

This table summarizes the three main comparisons:

| Command | Compares A... | ...With B | Shows You... |
|---|---|---|---|
| **`git diff`** | Working Directory | Staging Area | **Unstaged** changes. |
| **`git diff --staged`** | Staging Area | Last Commit (`HEAD`) | **Staged** changes (a "preview" of your next commit). |
| **`git diff HEAD`** | Working Directory | Last Commit (`HEAD`) | **All changes** (staged and unstaged). |

---

## 🛠️ Hands-On Demo: Seeing the Difference

Let's create a scenario with both staged and unstaged changes to see each command in action.

1.  **Create a starting point:** A file `colors.txt` with some initial content, and a new file `numbers.txt` with the number "ONE". Commit this work.
2.  **Make some changes:**
    -   Add "TWO" to `numbers.txt`.
    -   Add "VIOLET" to `colors.txt`.
3.  **Stage only one change:**
    ```bash
    git add numbers.txt
    ```
4.  **Check `git status`:**
    ```
    Changes to be committed:
      (use "git restore --staged <file>..." to unstage)
            new file:   numbers.txt

    Changes not staged for commit:
      (use "git restore <file>..." to discard changes in working directory)
            modified:   colors.txt
    ```
    We have one staged change and one unstaged change. Perfect.

Now let's run the `diff` commands:

-   **`git diff`** will **only** show the addition of "VIOLET" in `colors.txt`, because that is the only unstaged change.
-   **`git diff --staged`** will **only** show the creation of `numbers.txt` with "ONE" and "TWO" inside, because that is the only staged change.
-   **`git diff HEAD`** will show **both** sets of changes: the creation of `numbers.txt` AND the addition of "VIOLET" to `colors.txt`.

---

## 🎯 Narrowing the Scope to a Specific File

You can make any of the above commands more specific by adding a file path (or multiple paths) to the end. This is incredibly useful in large projects where running a full `diff` would produce too much output.

### The Syntax
Simply append the file path(s) to any `diff` command.

-   **See unstaged changes in just one file:**
    ```bash
    git diff styles/main.css
    ```
-   **See all changes (staged and unstaged) since the last commit in a specific file:**
    ```bash
    git diff HEAD js/app.js
    ```
-   **See only the staged changes for a specific file:**
    ```bash
    git diff --staged index.html
    ```
-   **See the diff for multiple specific files:**
    ```bash
    git diff HEAD index.html styles/main.css
    ```