#Git #Workflow #HandsOn #Tutorial #CoreCommand

>  This is a hands-on walkthrough of the fundamental `change -> add -> commit` Git workflow, introducing `[[git log]]` to view history and `git add .` as a useful shortcut for staging all changes.

---

## The Basic Loop (Recap)

The fundamental rhythm of working with Git is a three-step process that you will repeat constantly:

1.  **Change:** Modify files in your **Working Directory**.
2.  **Stage:** Use [[git add]] to move those changes to the **Staging Area**.
3.  **Commit:** Use [[git commit]] to save the staged changes permanently to your **Repository History**.

Let's get more practice with this cycle.

---

## 🚀 Hands-On: Committing More Changes

It's the afternoon of Day 1 of writing our novel. We've made our first commit, and now we're back to do more work.

### Step 1: Make Changes (New and Modified Files)
We'll start writing `chapter1.txt` and also update our `outline.txt` file with more detail.

### Step 2: Check the Status
Let's see what [[git status]] tells us now.

```bash
git status
```
**Output:**
```
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   outline.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        chapter1.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

> [!tip] "Modified" vs. "Untracked"
> This output shows a crucial difference:
> - **Untracked:** `chapter1.txt` is brand new. Git sees it, but has never tracked its history before.
> - **Modified:** `outline.txt` was part of our first commit. Git is already tracking it and has detected that the file has changed since that last commit.

### Step 3: Stage the Changes
Since the work on both files is related, we'll group them into a single commit. You can `git add` multiple files at once by listing them with spaces.

```bash
git add outline.txt chapter1.txt
```

### Step 4: Commit the Changes
Now we commit the staged files with a descriptive message.

```bash
git commit -m "Begin work on Chapter 1"
```
After committing, `git status` will once again report `nothing to commit, working tree clean`.

---

## 📜 Viewing History with `git log`

How can we prove that our commits are being saved? The `git log` command shows you the history of all commits in the repository.

```bash
git log
```
**Output:**
```
commit b81b3a3a417e2e3d74f28329a1a2b2781d4e744c (HEAD -> main)
Author: Your Name <you@example.com>
Date:   Wed Oct 26 10:30:00 2023 -0700

    Begin work on Chapter 1

commit a1b2c3d9f8e7c6b5a4d3c2b1a0f9e8d7c6b5a4d3
Author: Your Name <you@example.com>
Date:   Wed Oct 26 09:15:00 2023 -0700

    Start work on outline and main characters
```
`git log` displays each commit in reverse chronological order. For each commit, you can see:
-   **Commit Hash:** A unique 40-character ID for the commit.
-   **Author:** The name and email configured in your Git settings.
-   **Date:** When the commit was made.
-   **Commit Message:** The descriptive message you provided.

---

## ⚡ The `git add .` Shortcut

Let's continue our work. We finish writing Chapter 1 and update the outline accordingly.

### Step 1: Make More Changes
We add a lot more text to `chapter1.txt` and `outline.txt`. `git status` will show that both files are `modified`.

### Step 2: Stage All Changes at Once
Instead of typing out every filename, you can use a shortcut to stage all modified and untracked files in the current directory and its subdirectories.

```bash
git add .
```
The `.` represents the current directory.

> [!warning] Be Surgical with Your Commits!
> `git add .` is very convenient, but use it with care. If you've worked on multiple unrelated features, this command will lump them all together. It's often better practice to add files individually to create clean, focused commits. For this tutorial, since all our changes are related to "finishing Chapter 1," it's appropriate to use.

### Step 3: Commit Again

```bash
git commit -m "Finish Chapter 1"
```

## 🛠️ A More Complex Change: Splitting a Chapter

After sleeping on it, we decide Chapter 1 is too long and needs to be split.

1.  **Change:**
    -   Modify `outline.txt` to reflect a new Chapter 2.
    -   Cut the second half of `chapter1.txt`.
    -   Create a new file, `chapter2.txt`, and paste the content into it.
2.  **Status:** `git status` now shows two `modified` files (`outline.txt`, `chapter1.txt`) and one `untracked` file (`chapter2.txt`).
3.  **Stage:** Since all these changes are part of one logical action ("splitting the chapter"), we can use `git add .` to stage all of them at once.
4.  **Commit:**
    ```bash
    git commit -m "Split Chapter 1 into two parts"
    ```
5.  **Log:** Running `git log` will now show our complete history of four commits.