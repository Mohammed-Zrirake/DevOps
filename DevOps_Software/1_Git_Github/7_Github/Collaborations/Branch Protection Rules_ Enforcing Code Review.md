#GitHub #Collaboration #Safety #Settings #CodeReview

> "Branch protection rules ensure every change is reviewed before it enters the main branch."

---

## 🛡️ Why Use Branch Protection?
When collaborating on a shared repository (especially on the `main` or `master` branch), you want to prevent accidental breakage or low-quality code from being merged without oversight.

*   **Prevent Force Pushes**: Stop history from being rewritten.
*   **Prevent Deletion**: Stop the branch from being deleted.
*   **Enforce Code Review**: Require human approval via Pull Requests before merging.
*   **Require Status Checks**: Ensure tests pass (CI/CD) before merging.

---

## ⚙️ Setting Up Protection Rules
(Requires Admin access to the repository)

1.  **Go to Settings**: Navigate to the repository on GitHub -> **Settings** tab.
2.  **Branches**: On the left sidebar, click **Branches**.
3.  **Add Rule**: Click **"Add branch protection rule"**.

### Configuration
*   **Branch name pattern**: `main` (Apply this rule to the main branch).
*   **Require pull request reviews before merging**:
    *   **Required approving reviews**: Set to `1` (or more).
    *   *Effect*: No one can merge code into `main` until at least one other person approves the PR.
*   **Create**: Click the **Create** button to save.

---

## 🚫 The New Workflow (Blocked!)
Once protection is active, you generally **cannot** push directly to `main` anymore.

### Scenario: Stevie Tries to Push
1.  Stevie edits `README.md` directly on `main` (locally or via GitHub UI).
2.  **Commit/Push Attempt**:
    > "You can't commit to main because it's a protected branch."
3.  **Solution**: Stevie *must* create a new branch (e.g., `patch-1`), push that branch, and open a Pull Request.

---

## 👀 The Review Process
Now that a PR is open, merging is **Blocked** until a review happens.

### 1. Requesting a Review
*   On the right side of the PR, click **Reviewers** and select a teammate (e.g., Colt).

### 2. Performing a Review (Colt)
1.  **Files Changed**: Colt views the diff.
2.  **Review Changes**: Click the "Review changes" button.
3.  **Options**:
    *   **Comment**: Just leaving feedback (e.g., *"Always with the Bock Bock, Stevie?"*).
    *   **Approve**: Validates the code and unblocks merging.
    *   **Request Changes**: Blocks merging until the author fixes issues.

### 3. Merging
Once Colt selects **Approve** and submits:
*   The PR status changes to **Approved**.
*   The **"Merge pull request"** button turns green.
*   Stevie (or Colt) can now merge the code into `main`.

---

## 🏆 Summary
| Action | Without Protection | With Protection |
| :--- | :--- | :--- |
| **Push to Main** | Allowed (Risky) | **Blocked** 🛑 |
| **Merge PR** | Anyone, anytime | **Blocked** until Approved ✅ |
| **Code Quality** | Depends on trust | Enforced by process |
