#Git #GitHub #Collaboration #PullRequests #Workflow

> "Pull Requests are not a native Git feature. They are built on top of platforms like GitHub to facilitate discussion."

---

## 🤝 What is a Pull Request (PR)?
A request to merge code from a **feature branch** into a **base branch** (usually `main`), but with a critical intermediate step: **Discussion & Review**.

*   **Goal**: To review code, discuss changes, and ensure quality *before* integration.
*   **The Workflow**:
    1.  Push feature branch to GitHub.
    2.  Open a PR on GitHub.
    3.  Team reviews/discusses/approves.
    4.  Merge functionality on GitHub.

---

## 🔄 The Standard PR Workflow
(Example: Stevie adding a Navbar)

### 1. The Setup
*   **Stevie**: Creates `navbar` branch, pushes code to GitHub.
*   **GitHub**: Shows a "Compare & pull request" button.

### 2. Opening the PR
*   **Base**: `main` <- **Compare**: `navbar`
*   **Title/Description**: Explain *what* changed and *why*.
*   **Reviewers**: Tag team members (e.g., `@Colt`) to review.

### 3. Review & Merge
*   **Reviewer (Colt)**: Checks the files, leaves comments ("Looks good!").
*   **Merge**: Colt clicks **"Merge pull request"** on GitHub.
*   **Result**: `main` now contains the navbar code.

### 4. Sync Local
Everyone (including Stevie) must update their local `main`:
```bash
git switch main
git pull origin main
# Now local main has the merged PR code
```

---

## ⚔️ Dealing with Merge Conflicts in PRs
Sometimes GitHub says: **"Can't automatically merge."**
This means the `main` branch has changed in a way that conflicts with your feature branch.

### �️ Option 1: Resolve in the Browser (GitHub UI)
*   **Best for**: Small, simple text conflicts (e.g., a single line change).
*   **How**:
    1.  Click the **"Resolve conflicts"** button on the PR page.
    2.  Find the conflict markers (`<<<<`, `====`, `>>>>`).
    3.  Delete the markers and keep the code you want.
    4.  Click **"Mark as resolved"** and **"Commit merge"**.
*   **Note**: Not recommended for complex conflicts across multiple files.

### �🛑 Option 2: The "Right" Way to Fix It (Locally)
Don't rely on the browser for big conflicts. Fix them on your machine.

#### Step 1: Update Local Main
Always make sure your local `main` is up to date before starting.
```bash
git switch main
git pull origin main
```

#### Step 2: Switch to Feature Branch
Go to the branch with the conflict (e.g., `new-heading`).
```bash
git switch new-heading
```

#### Step 3: Merge Main INTO Feature
Bring the latest `main` code into your feature branch to trigger the conflict locally.
```bash
git merge main
# This will stop and tell you there is a CONFLICT.
```

#### Step 4: Resolve & Commit
1.  Open files, fix the conflict markers.
2.  Add and commit.
    ```bash
    git add .
    git commit -m "Resolve merge conflicts"
    ```

#### Step 5: Finalize the Merge
You have two choices now:
1.  **Push and Merge on GitHub**: `git push origin new-heading` (Update the PR).
2.  **Merge Locally & Close PR** (As seen in the video):
    ```bash
    git switch main
    git merge --no-ff new-heading
    git push origin main
    ```
    *Result*: GitHub detects the merge and automatically closes the PR as "Merged".

---

## 🎨 The `--no-ff` Flag (No Fast-Forward)
When merging a PR (especially via command line or some GUI settings), you might see:

```bash
git merge --no-ff new-heading
```

*   **Default Merge (Fast-Forward)**: If possible, Git just moves the `main` pointer forward. History looks flat.
*   **No Fast-Forward (`--no-ff`)**: Forces Git to create a **Merge Commit**, even if it didn't strictly need one.
    *   **Why?**: It creates a "bubble" in the history, clearly showing where a feature branch started and ended.
    *   **GitHub**: Typically uses `--no-ff` behavior by default when you click the "Merge" button, preserving one commit for the merge itself.

---

## 🧹 Cleanup
Once the PR is merged:
1.  **Delete Remote Branch** (GitHub usually offers a button).
2.  **Delete Local Branch**:
    ```bash
    git branch -d new-heading
    git fetch --prune (cleans up remote references)
    ```
