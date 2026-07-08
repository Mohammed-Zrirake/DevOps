


> **TL;DR:** `git diff` is a powerful, read-only command that shows the exact line-by-line differences between commits, branches, or your working files. The output highlights lines that have been added (`+`) and removed (`-`).

---

## 🤔 What is `git diff`?

`git diff` is your primary tool for seeing changes in your repository. It's a purely informational command, just like `git status` and `git log`, meaning it will never change your files or history.

It can answer many questions, such as:
-   What changes have I made in my working directory since my last commit?
-   What changes have I staged that are different from my last commit?
-   What has changed between two different branches?
-   What changed in a specific commit?

### The First Use Case: Unstaged Changes

The simplest and most common use of `git diff` is to run it with no arguments.

> [!info] `git diff` (with no options)
> This command shows the differences between your **Working Directory** and your **[[`git add`: Staging Your Changes|Staging Area]]**. In simpler terms, it shows all of your changes that you **have not yet staged**.

**Example:**
Imagine you've made a bunch of changes to your HTML, CSS, and JS files. You can't remember exactly what you changed everywhere. Running `git diff` will give you a detailed, line-by-line report of every single modification.

---

## 🏛️ Anatomy of the Diff Output

The output of `git diff` can look complex and intimidating, but it follows a consistent, logical pattern. Let's break it down using a simple example.

**The Scenario:**
-   **Old Version (Last Commit):** A file `rainbow.txt` contains `red, orange, yellow, green, blue, purple`.
-   **New Version (Working Directory):** We've edited `rainbow.txt` to contain `red, orange, yellow, green, blue, indigo, violet`.

Running `git diff` on this change produces the following output:

```diff
diff --git a/rainbow.txt b/rainbow.txt
index 3d623b1..827b409 100644
--- a/rainbow.txt
+++ b/rainbow.txt
@@ -3,4 +3,5 @@
 yellow
 green
 blue
-purple
+indigo
+violet
```

Let's dissect this piece by piece.

### 1. The Header Lines
```diff
diff --git a/rainbow.txt b/rainbow.txt
index 3d623b1..827b409 100644
--- a/rainbow.txt
+++ b/rainbow.txt
```
-   `diff --git a/rainbow.txt b/rainbow.txt`: This line simply states that `git diff` is comparing two files. Git labels the "before" version as `a` and the "after" version as `b`.
-   `index ...`: This is internal metadata about the file hashes. **You can safely ignore this line.**
-   `--- a/rainbow.txt`: The "before" file (`a`) will be represented by lines starting with a `-` (minus sign).
-   `+++ b/rainbow.txt`: The "after" file (`b`) will be represented by lines starting with a `+` (plus sign).

### 2. The "Chunk" Header
```diff
@@ -3,4 +3,5 @@
```
Git doesn't show the whole file, only the "chunks" where changes occurred, plus a few lines of context.
-   `@@ ... @@`: This line marks the beginning of a chunk of changes.
-   `-3,4`: This refers to the `a` file (old version). It means "this chunk starts at line **3** and is **4** lines long."
-   `+3,5`: This refers to the `b` file (new version). It means "this chunk starts at line **3** and is **5** lines long."

### 3. The Changes
```diff
 yellow
 green
 blue
-purple
+indigo
+violet
```
This is the most important part. It shows the actual content changes.
-   **Lines with no prefix** (like `yellow`, `green`, `blue`): These are unchanged lines of context. They exist in both versions.
-   **Lines with a `-` prefix** (like `-purple`): This line **existed in the old version** (`a`) but was removed. It's colored red in most terminals.
-   **Lines with a `+` prefix** (like `+indigo`, `+violet`): These lines **exist only in the new version** (`b`). They were added. They're colored green in most terminals.

> [!success] Reading the Story
> By reading this diff, you can tell a clear story: "In `rainbow.txt`, I replaced the line 'purple' with two new lines, 'indigo' and 'violet'."

### A More Complex Example with Multiple Chunks

```diff
diff --git a/main.css b/main.css
index ...
--- a/main.css
+++ b/main.css
@@ -22,4 +22,8 @@
   display: flex;
   justify-content: center;
   align-items: center;
+
+  /* New styles added */
+  border: 1px solid #ddd;
+  border-radius: 5px;
 }
@@ -63,7 +67,6 @@
 .score {
-  font-size: 2em;
-  font-weight: bold;
-  color: steelblue;
+  font-size: 2.5em;
+  color: tomato;
 }
```
-   This diff shows two separate chunks of changes in the same file, `main.css`.
-   **First Chunk:** Starting around line 22, four new lines were added.
-   **Second Chunk:** Starting around line 63, three old lines were removed and two new lines were added to change the font size and color.