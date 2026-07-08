#Git #DevOps #Rebasing #Troubleshooting #Conflicts #HandsOn

> [!abstract] Brief Description
> This note explains how to identify, resolve, and manage merge conflicts during a `git rebase` operation. You will learn how to handle the paused rebase state, resolve conflict markers, and use the `--continue` or `--abort` flags.

---

> [!note] 📖 The Core Analogy: Rearranging a Meeting Schedule
> Imagine you are an assistant updating a manager's calendar.
> - **The Rebase Goal:** Move a sequence of afternoon appointments (your feature commits) to follow a newly scheduled morning workshop (the master commits).
> - **The Conflict:** The workshop and your first afternoon meeting both booked the same conference room at the same time.
> - **Aborting (`git rebase --abort`):** Throwing up your hands, canceling the scheduling run, and returning the calendar to exactly how it was before.
> - **Resolving and Continuing (`git rebase --continue`):** Pausing the schedule run, updating the room booking for that meeting (resolving the conflict), logging the resolution (`git add`), and continuing to schedule the remaining afternoon appointments.

---

## ⚠️ 1. Simulating a Conflict Scenario

Conflicts occur during a rebase when changes on the target branch (e.g., `master`) and your branch (e.g., `feat`) edit the same lines of the same file.

### Step 1: Make a conflicting commit on the feature branch
```bash
git switch feat
# Open website.txt and change the navbar and footer descriptions
# "navbar added" -> "top navbar added"
# "footer added" -> "log out form added"
git commit -am "update website on feature branch"
```

### Step 2: Make a conflicting commit on master
```bash
git switch master
# Open website.txt and change the same lines
# "top navbar" -> "main navbar"
# "log out" -> "sign up form added"
git commit -am "update copy on master"
```

### Step 3: Switch back and make one more feature commit
```bash
git switch feat
echo "THREE" >> feature.txt
git commit -am "more work on feat"
```

---

## 🛑 2. The Paused Rebase State

When you run `git rebase master` from the `feat` branch, Git will apply commits sequentially. Once it hits the conflict on `website.txt`, the rebase will pause:

```text
Conflict: merge conflict in website.txt
error: Failed to merge in the changes.
Patch failed at 0001 update website on feature branch
```

If you check the repository status at this point, you will see a **partial rebase** in progress:
```bash
git status
# Output shows: "rebase in progress; onto <commit-hash>"
# website.txt is marked as "both modified"
```
In GitKraken or other GUI clients, you will see that some commits have been successfully rewritten, but the branch pointer is suspended mid-rebase.

---

## 🛠️ 3. How to Resolve the Conflicts

To successfully complete the rebase, follow Git's instructions:

### Step 1: Edit the conflicting files
Open the files containing conflict markers (e.g., `website.txt`) and resolve the code. You can choose to keep your branch's changes, incoming changes from master, or manually combine them into a hybrid version.

```text
<<<<<<< HEAD
main navbar
sign up form added
=======
top navbar added
log out form added
>>>>>>> update website on feature branch
```

### Step 2: Stage the resolved files
Mark the conflicts as resolved by staging them.

```bash
git add website.txt
```

> [!danger] Do Not Run `git commit`
> Running `git commit` in the middle of a rebase will break the rebase loop and create a corrupt repository state. Always stage with `git add` and let the rebase command handle the commit.

### Step 3: Continue the rebase
Tell Git to proceed with replaying the remaining commits:

```bash
git rebase --continue
```
If there are more conflicts in subsequent commits, Git will pause again. Repeat Steps 1–3 until the rebase is completed successfully.

---

## 🔄 4. Aborting a Rebase

If you get stuck, run into too many conflicts, or make a mistake during resolution, you can safely abort the entire rebase and return your branch to its exact pre-rebase state:

```bash
git rebase --abort
```
This is a safe, non-destructive escape hatch that erases all temporary commits made during the current rebase session.

---

> [!summary] Key Takeaways
> - **Core concept:** When a conflict occurs during a rebase, Git pauses the operation, allowing you to resolve conflicts commit-by-commit instead of all at once.
> - **Key implementation detail:** You resolve conflicts by manually editing conflict markers, running `git add <file>` to stage, and then running `git rebase --continue`.
> - **Best practice:** Use `git rebase --abort` as a safety net if a conflict resolution becomes too complex or confusing.
