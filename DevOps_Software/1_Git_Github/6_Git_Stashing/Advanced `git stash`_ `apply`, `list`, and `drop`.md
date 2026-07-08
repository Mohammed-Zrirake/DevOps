
#Git #CoreCommand #Workflow #Advanced

>  `git stash apply` keeps the stash after applying it, `git stash list` shows all your stashes, `git stash apply <id>` applies a specific one, and `git stash drop` or `clear` removes stashes. Even with these options, `git stash` and `git stash pop` will be your main tools.

---

> [!info] The 90% Use Case
> While Git provides a rich set of commands for managing the stash, it's important to remember that for the vast majority of day-to-day scenarios, the simple [[`git stash`: Temporarily Shelving Your Work|git stash]] and `git stash pop` workflow is all you will ever need. The following commands are for more complex or specific situations.

## `pop` vs. `apply`: The Key Difference

The two primary commands for retrieving stashed changes have one critical difference.

-   **`git stash pop`**: Applies the most recent stash AND **removes it** from the stash stack. It's a "use once and discard" operation.
-   **`git stash apply`**: Applies the most recent stash BUT **keeps it** in the stash stack. This allows you to reuse the same set of stashed changes on multiple branches.

> [!tip] When is `apply` useful?
> Imagine you have some temporary debugging code (e.g., a bunch of `console.log` statements) that you don't want to commit. You can `stash` it, then `apply` it to a couple of different feature branches to test them, without having to re-stash each time.

Just like `pop`, `git stash apply` can result in a [[Resolving Merge Conflicts|merge conflict]] if the stashed changes conflict with the current state of your branch.

---

## Managing Multiple Stashes: The Stash Stack

You are not limited to a single stash. You can run `git stash` multiple times, and Git will save each set of changes in a stack (Last-In, First-Out).

### `git stash list`: Viewing the Stack
To see all the sets of changes you have stashed, use the `list` command.

```bash
git stash list
```
**Output:**
```
stash@{0}: WIP on rainbow: d3a4b1c Remove background color
stash@{1}: WIP on rainbow: d3a4b1c Remove background color
stash@{2}: WIP on rainbow: d3a4b1c Remove background color
stash@{3}: WIP on goodbye: e7f8g9h change to goodbye
```
-   **`stash@{n}`**: This is the identifier for each stash. `stash@{0}` is always the *most recent* one.
-   **`WIP on <branch>`**: Tells you which branch you were on when you created the stash.
-   **`<hash> <message>`**: The last commit on that branch when the stash was made.

### `git stash apply <stash-id>`: Applying a Specific Stash
If you want to apply a stash other than the most recent one, you can specify its identifier.

```bash
# This will apply the 'red' background color from our demo, not the 'yellow' one.
git stash apply stash@{2}
```
This is useful if you need to retrieve an older set of stashed changes without first applying and dropping the newer ones.

---

## 🧹 Cleaning Up: `drop` and `clear`

If you use `git stash apply`, your stashes will accumulate. You need a way to clean them up.

### `git stash drop <stash-id>`: Removing a Single Stash
To remove a specific stash from the stack without applying it, use the `drop` command with its identifier.

```bash
# After applying and testing the 'yellow' background from stash@{0}, we can delete it.
git stash drop stash@{0}
```
Running `git stash list` again will show that it has been removed.

### `git stash clear`: Removing All Stashes
To completely empty the stash stack, use the `clear` command.

> [!danger] This is a Destructive Operation!
> `git stash clear` will permanently delete **all** of your stashed work. This action cannot be undone.

```bash
git stash clear
```
Running `git stash list` after this command will produce no output.

---

### Summary of Stash Commands

| Command | Description |
|---|---|
| **`git stash`** | Saves uncommitted changes to the stash and cleans the working directory. |
| **`git stash pop`** | **Applies** the most recent stash and **removes** it from the stack. |
| **`git stash apply`** | **Applies** the most recent stash but **keeps** it on the stack. |
| **`git stash list`** | Shows all sets of changes currently in the stash. |
| **`git stash apply <id>`** | Applies a specific stash (e.g., `stash@{2}`) but keeps it on the stack. |
| **`git stash drop <id>`** | Permanently removes a specific stash from the stack. |
| **`git stash clear`** | Permanently removes **all** stashes from the stack. |