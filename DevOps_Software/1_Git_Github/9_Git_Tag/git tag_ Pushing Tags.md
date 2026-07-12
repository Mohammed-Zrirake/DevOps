#Git #DevOps #GitTags #Push #Remote #GitHub #CoreConcept

> [!abstract] Brief Description
> This note explains how to share Git tags with collaborators by pushing them to remote repositories like GitHub. It details why tags are omitted from default push commands and outlines the syntax for pushing single tags and bulk tags.

---

> [!note] 📖 The Core Analogy: Shipping Box Manifests
> Imagine shipping products from your local workshop to a central distribution warehouse:
> - **Default Shipping (Normal `git push`):** You ship the products (commits/branches) to the warehouse, but you leave your local office inventory sticky-notes (tags) behind on your desk.
> - **Shipping a Single Label:** Filling out a custom order to ship a specific label ("v1.0") to stick on a box that is already at the warehouse.
> - **Shipping All Labels (`--tags`):** Loading all local labels onto a carrier truck and syncing them with the warehouse so both locations match perfectly.

---

## ☁️ 1. Why Tags Aren't Pushed by Default

When you run `git push origin <branch-name>`, Git uploads your commits and updates the branch reference on the remote host (e.g., GitHub). However, **Git does not push tags by default.**

*   **Why?** Tags are often used for internal notes, release candidates, or experimental labels. Omitting them from default pushes keeps the remote repository clean of local developer clutter.
*   **The Symptom:** If you create a tag locally and run `git push`, the remote repository will show "0 tags" or show that the tag is missing, even though all code changes are present.

---

## 🏷️ 2. Pushing a Single Tag

To share a specific tag with your team, you must specify the remote name and the tag name explicitly in the push command.

```bash
# General syntax
git push <remote-name> <tag-name>

# Example: Pushing v17.0.2 to origin
git push origin v17.0.2
```

---

## 🚀 3. Pushing All Tags (The `--tags` Flag)

If you have created multiple tags locally and want to upload them all at once, use the `--tags` flag.

```bash
# Push all local tags to the remote repository
git push <remote-name> --tags

# Example:
git push origin --tags
```

This command scans your local tags and pushes any that do not already exist on the remote repository.

---

> [!summary] Key Takeaways
> - **Core concept:** Default push commands do not upload tags. You must push tags explicitly to remote hosts.
> - **Key implementation detail:** Use `git push <remote> <tag>` to upload one tag, or `git push <remote> --tags` to upload all local tags.
> - **Best practice:** Always double-check that your version tags have been pushed to GitHub so that CI/CD pipelines can detect release points and trigger builds.
