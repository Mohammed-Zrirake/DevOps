#Git #DevOps #GitTags #LightweightTags #AnnotatedTags #GitShow #CoreConcept

> [!abstract] Brief Description
> This note explains how to create Git tags on the current `HEAD` commit. It covers the syntax for both lightweight tags and annotated tags, explains how to write inline tag messages using the `-m` flag, and demonstrates how to inspect tag details using `git show`.

---

> [!note] 📖 The Core Analogy: The Warehouse Labeling System
> Imagine you are a warehouse manager labeling storage boxes of product versions:
> - **Lightweight Tag (Sharpie Scribble):** Writing a quick version number like "v1.2.0" directly on the cardboard. It is fast and simple, but stores no extra details.
> - **Annotated Tag (Shipping Invoice Sleeve):** Gluing a plastic sleeve onto the box containing a printed shipping invoice. It includes the manager's signature (author name/email), packaging date (date), a description of what is inside (tag message), and the box tracking ID (commit hash).

---

## 🏷️ 1. Creating Lightweight Tags

A lightweight tag is a simple reference that points directly to a commit. By default, running the `git tag` command without extra flags targets whatever commit `HEAD` is currently referencing (the tip of your active branch).

```bash
# Basic syntax to create a lightweight tag on HEAD
git tag <tag-name>

# Example: Tagging a quick README typo fix as v17.0.2
git commit -am "Fix typo in README"
git tag v17.0.2
```

Once created, you can see the tag associated with your latest commit in `git log --oneline` or by running `git tag` to view all references.

---

## 📝 2. Creating Annotated Tags

Annotated tags are stored as separate objects in the Git database and contain critical metadata. They are the standard for public releases.

### Method A: Using the Text Editor
To create an annotated tag, use the `-a` option. Git will open your default text editor and prompt you for a tag message, just like `git commit`:

```bash
# Creates an annotated tag, launching the text editor
git tag -a v17.1.0
```

### Method B: Passing an Inline Message
You can bypass the text editor by providing the message directly in the terminal using the `-m` option:

```bash
# Creates an annotated tag with a message directly
git tag -a v17.1.0 -m "Minor release including non-breaking new feature"
```

---

## 🔍 3. Inspecting Tag Details (`git show`)

To view the metadata stored inside an annotated tag, use the `git show` command followed by the tag name:

```bash
git show v17.1.0
```

### Example Metadata Output:
```text
tag v17.1.0
Tagger: Colt <colt@example.com>
Date:   Wed Jul 8 11:20:00 2026 +0100

Minor release including non-breaking new feature

commit 4e6f8d9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e (HEAD -> master)
Author: Colt <colt@example.com>
Date:   Wed Jul 8 11:15:00 2026 +0100

    add new feature

diff --git a/new_feature.js b/new_feature.js
...
```

> [!tip] Reviewing Public Releases
> You can run `git show` on tags created by other developers (e.g., `git show v17.0.0` in React) to inspect the tagger's identity, timestamp, custom release comments, and the final commit diff that marked the release.

---

> [!summary] Key Takeaways
> - **Core concept:** Lightweight tags (`git tag <name>`) are simple bookmarks, while annotated tags (`git tag -a <name> -m "<msg>"`) are complete Git objects containing metadata.
> - **Key implementation detail:** By default, creating a tag places it on the current `HEAD` commit of your active branch.
> - **Best practice:** Inspect the tagger details, messages, and associated code changes of any release tag using the `git show <tag-name>` command.
