#Git #CoreCommand #Commit

>  `git commit` takes everything from the [[`git add`: Staging Your Changes|Staging Area]] and saves it as a new, permanent checkpoint (a commit) in your repository's history. Every commit **must** have a message describing the changes.

---

## 📝 Every Commit Needs a Message

Before you can run the command, you must understand that Git requires a message for every single commit.

> [!info] The Purpose of a Commit Message
> A commit message is a summary of the changes you made. It creates a human-readable history of your project, making it possible for you (and your collaborators) to understand what was changed and why, months or even years later. A project with well-written commit messages is vastly easier to maintain.

We will cover [[Writing Good Commit Messages|best practices for writing great commit messages]] later, but for now, just know that your message should be a short, descriptive summary of the work included in the commit.

---

## Two Ways to Run the Commit Command

There are two primary ways to make a commit. The one you choose determines *how* you provide the commit message.

### Method 1: `git commit` (Opens an Editor)

If you run `git commit` by itself, Git will open your system's default command-line text editor to let you write your message.

> [!warning] The Vim Trap for Beginners!
> For many new users who haven't configured a default editor, `git commit` will open a notoriously confusing editor called **Vim**. If you suddenly find yourself in a strange new interface and can't type or exit, you're likely in Vim.
>
> - **To get out of Vim without saving:** Press `Esc`, then type `:q!` and hit `Enter`.

Because of this, we will avoid this method for now.

### Method 2: `git commit -m "Your message here"` (⭐ Recommended)

This is the most straightforward and beginner-friendly way to make a commit. The `-m` flag (which stands for "message") allows you to provide your commit message directly in the command line.

-   Git will create the commit using the message you provide, without opening an external editor.
-   The message must be enclosed in quotes.

---

## Hands-On Demo: Making Your First Commit

Let's pick up where we left off with our "MyFirstNovel" project.

### Recap: Check the Status
First, let's remember our current state. We have already staged our two new files.

```bash
git status
```
**Output:**
```
On branch main
No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   characters.txt
        new file:   outline.txt
```
Our files are in the Staging Area, ready to be committed.

### Step 1: Run the Commit Command
Now, let's commit these staged changes with a descriptive message using the `-m` flag.

```bash
git commit -m "Start work on outline and main characters"
```

### Step 2: Analyze the Output
After hitting enter, Git will give you a summary of the commit it just created.

**Output:**
```
[main (root-commit) a1b2c3d] Start work on outline and main characters
 2 files changed, 5 insertions(+)
 create mode 100644 characters.txt
 create mode 100644 outline.txt
```
This confirms that a new commit was made on the `main` branch, two files were included, and it lists the files that were newly created.

### Step 3: Check the Status Again
This is the final and most satisfying step. Let's see what `git status` says now.

```bash
git status
```
**Output:**
```
On branch main
nothing to commit, working tree clean
```

---

## "Working Tree Clean": What It Means

> [!success] A State of Harmony
> The message **"nothing to commit, working tree clean"** is Git's way of telling you that your **Working Directory** is perfectly in sync with your latest commit. Git is not aware of any new, modified, or deleted files that haven't been saved to its history. It's the "all clear" signal.

As soon as you edit an existing file or create a new one, the working tree will no longer be "clean," and `git status` will report those new changes.

---

> [!summary] Key Takeaway
> You have now completed the fundamental Git workflow:
> 1.  You **changed** files in your working directory.
> 2.  You used **`git add`** to stage those changes.
> 3.  You used **`git commit`** to save those changes as a permanent snapshot in your repository.
>
> This **Change -> Add -> Commit** cycle is the bread and butter of working with Git.