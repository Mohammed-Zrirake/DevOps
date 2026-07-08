#Git #DevOps #Rebasing #Squash #Fixup #BestPractice #Workflow

> [!abstract] Brief Description
> This guide details how to combine multiple commits into a single commit using the `squash` and `fixup` options during an interactive rebase. You will learn the difference between combining messages and discarding redundant messages to produce a clean commit history.

---

> [!note] 📖 The Core Analogy: Consolidating Journal Entries
> Imagine you are polishing daily entries in your travel diary before printing it.
> - **Primary Entry (Parent Commit):** "Visited the Eiffel Tower" (detailed story).
> - **Follow-up Entry (Child Commit):** "Whoops, forgot to mention I bought crepes at the base."
> - **Squash:** Merging the two entries together and keeping both titles: "Visited the Eiffel Tower & bought crepes at the base."
> - **Fixup:** Merging the entries but deleting the second title card because it was just a minor correction, keeping only the main title: "Visited the Eiffel Tower" (with the crepes information now neatly integrated inside it).

---

## 🏗️ 1. Squash vs. Fixup: The Key Distinction

When you want to combine several local commits into a single commit, Git provides two interactive commands: **Squash** and **Fixup**. Both commands meld the commit's code changes into the commit directly *above* it in the rebase list.

| Command | Action | Commit Message Behavior |
| :--- | :--- | :--- |
| **`squash`** (or `s`) | Melds code changes into the preceding commit. | Git pauses to open an editor, prompting you to combine or rewrite the commit messages of all squashed commits. |
| **`fixup`** (or `f`) | Melds code changes into the preceding commit. | Git automatically discards the squashed commit's log message, keeping only the parent commit's message. |

> [!tip] Which to Use?
> In daily development, `fixup` is often preferred over `squash` because follow-up commits are usually minor fixes, typo resolutions, or additions that do not need to clutter the final commit message log.

---

## 🔑 2. Hands-On Walkthrough: Fixing Up Bootstrap Commits

### The Scenario:
You are adding Bootstrap to a website. You create two commits:
1.  `2b3c4d5 add bootstrap` (adds the CSS file)
2.  `3c4d5e6 whoops forgot to add bootstrap js script` (adds the JS script)

Since both changes should be part of a single "add bootstrap" task, we combine them.

### Step 1: Launch Interactive Rebase
```bash
git rebase -i HEAD~9
```

### Step 2: Configure the Rebase File
Change the instruction for the follow-up commit from `pick` to `fixup` (or `f`):
```text
pick 2b3c4d5 add bootstrap
fixup 3c4d5e6 whoops forgot to add bootstrap js script
```
*Note: Git will meld `3c4d5e6` into `2b3c4d5`.*

### Step 3: Save and Exit
Save and close the editor. Git will complete the rebase.
If you check `git log --oneline`, the messy `"whoops..."` commit is gone, replaced by a single, clean `"add bootstrap"` commit. However, the JavaScript script tag is still fully preserved in your index files.

---

## 🔄 3. Multi-Commit Consolidation: Collapsing Typos

You can stack multiple `fixup` commands sequentially. All subsequent `fixup` commits will merge upward into the nearest preceding `pick` or `reword` commit.

### The Scenario:
You have a series of typo fixes on top of a navbar feature:
1.  `de1a2b3 add top navbar`
2.  `4c5d6e7 fix navbar typos`
3.  `8f9a0b1 fix another navbar typo`

### The Configuration:
```text
pick de1a2b3 add top navbar
fixup 4c5d6e7 fix navbar typos
fixup 8f9a0b1 fix another navbar typo
```

### The Result:
All code changes (fixing double letters like `Navbarr` to `Navbar` and `Linkk` to `Link`) are applied directly into the original `add top navbar` commit. The two typo commits are completely erased from the history.

---

> [!summary] Key Takeaways
> - **Core concept:** Squashing and fixing up merge the contents of younger commits into their older parent commits, rewriting local history to group changes logically.
> - **Key implementation detail:** While `squash` prompts you to merge commit logs, `fixup` discards the log message of the combined commit, saving time.
> - **Best practice:** Use `fixup` to consolidate small incremental commits (like lint fixes, typos, or forgotten imports) into their main feature commits before opening a pull request.
