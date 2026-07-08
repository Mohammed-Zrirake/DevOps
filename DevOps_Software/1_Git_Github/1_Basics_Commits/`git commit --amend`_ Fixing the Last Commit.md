#Git #CoreCommand #Workflow #Troubleshooting

>  The `git commit --amend` command is a powerful shortcut to modify the **most recent commit**. It's perfect for quickly adding forgotten files or fixing a typo in your last commit message.

---

## ❓ When Should You Amend a Commit?

Occasionally, you'll make a commit and immediately realize you made a mistake. The two most common scenarios are:

1.  **You forgot to include a file:** You made a logical change across three files, but only staged and committed two of them.
2.  **You made a typo in the commit message:** You wrote a great message but misspelled a key word.

`git commit --amend` is the perfect tool to fix these immediate errors.

> [!warning] The Golden Rule of Amending
> You should only amend commits that **have not yet been pushed** to a shared repository (like GitHub). Amending rewrites commit history, which can cause major problems for collaborators who have already pulled the original commit. We will cover this in more detail when we get to `git push`. For now, only use it on your local commits.

## The Workflow: How to Amend

The process is straightforward:

1.  You make a commit and realize your mistake.
2.  **If you forgot files:** Stage the additional files using `git add <forgotten-file>`.
3.  Run the command `git commit --amend`.
4.  Your configured editor will open, pre-filled with your last commit message.
    -   You can either edit the message or leave it as-is.
5.  Save and close the editor to finalize the amended commit.

---

## 🛠️ Hands-On Example: Forgetting a File

Let's walk through a scenario where we forget to include a file in a commit.

### Step 1: The Initial (Flawed) Work
We'll add a simple heading to the top of our four main text files: `outline.txt`, `characters.txt`, `chapter1.txt`, and `chapter2.txt`.

### Step 2: The Mistake
We're ready to commit, but we accidentally forget to stage the `outline.txt` file.

```bash
# We add three files, but forget the fourth
git add chapter1.txt chapter2.txt characters.txt
```

### Step 3: The Flawed Commit
Not realizing our mistake, we make the commit. Notice the message is now technically incorrect.

```bash
git commit -m "Add headings to ALL files"
```
The output will show `3 files changed`. A moment later, we realize we forgot `outline.txt`!

### Step 4: The Fix - Staging the Forgotten File
First, we need to stage the file we missed. `git status` will show `modified: outline.txt`.

```bash
git add outline.txt
```

### Step 5: Amending the Commit
Now, instead of creating a *new* commit, we'll run the `amend` command.

```bash
git commit --amend
```

### Step 6: The Editor
Your configured text editor (e.g., VS Code) will open. The `COMMIT_EDITMSG` file will contain your previous commit message: `"Add headings to ALL files"`.

-   Since our message is now accurate, we don't need to change it.
-   If we had a typo, this is where we would fix it.

Simply **save and close the file** to complete the process.

### Step 7: The Result
Your terminal will now show the output for the *new*, amended commit, which correctly states `4 files changed`.

If you run `git log --oneline`, you will see **one single commit** with the message "Add headings to ALL files." The original, flawed commit has been replaced by the new, corrected one.

---

## The Two Main Use Cases Summarized

1.  **Adding Forgotten Files:**
    -   `git add <the-forgotten-file>`
    -   `git commit --amend`
    -   Save and close the editor (without changing the message).

2.  **Fixing Only the Commit Message:**
    -   (You don't need to `git add` anything).
    -   `git commit --amend`
    -   Fix the typo in your editor, then save and close.

You can also do both at the same time: stage forgotten files, run `amend`, and fix the message in the editor.