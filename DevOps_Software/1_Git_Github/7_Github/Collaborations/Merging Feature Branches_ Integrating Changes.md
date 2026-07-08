#Git #GitHub #Collaboration #Merging #FeatureBranches

> "Integrating your changes involves more than just code; it involves communication."

---

## 🎯 The Goal: Integration
Working on **Feature Branches** isolates your work, but none of that work is valuable until it is integrated back into the `main` (or `master`) branch.

---

## 🛣️ Options for Merging
How do we get code from `feature-branch` -> `main`?

### 1. Merge at Will
*   **Method**: Any developer merges their branch into `main` whenever they feel ready.
*   **Risk**: High. No testing, no code review, potential for breaking builds ("Fraught with problems").
*   **Use Case**: Hobby projects or solo work.

### 2. Discuss, Then Merge
*   **Method**: Merge your own branch, but only **after** discussing it (email, chat, verbal agreement).
*   **Risk**: Medium. Relies on manual communication.
*   **Use Case**: Small, trusted teams.

### 3. Pull Requests (PRs)
*   **Method**: A formal request to merge code, allowing for review and discussion *before* integration.
*   **Status**: **The Industry Standard** (Covered in the next note).

---

## 🧪 Scenario: Merging the Pricing Table
(Continuing with Colt and Stevie)

### 1. The Setup
*   **Colt**: Has finished the `pricing-table` branch.
*   **Stevie**: Has finished the `navbar` branch (locally).
*   **Main**: Currently empty on GitHub (no features yet).

### 2. Colt Merges (The "Owner" Role)
Colt decides to merge his `pricing-table` into `main`.

1.  **Switch to Main**:
    ```bash
    git switch main
    ```
2.  **Pull Latest**: (Always check for updates first!)
    ```bash
    git pull origin main
    ```
3.  **Merge**:
    ```bash
    git merge pricing-table
    ```
    *   *Result*: **Fast-Forward Merge**. Since `main` hadn't changed, Git just moved the pointer forward.
    *   *Now*: `main` has the pricing table code locally.
4.  **Push to GitHub**:
    ```bash
    git push origin main
    ```
    *   *Result*: Now `origin/main` has the pricing table.

### 3. Stevie Updates (The "Collaborator" Role)
Stevie wants to merge his `navbar`, but first, he needs Colt's new code.

1.  **Update Main**:
    ```bash
    git switch main
    git pull origin main
    ```
    *   *Result*: Stevie downloads the `pricing-table` code into his local `main`.
2.  **Merge Navbar**:
    ```bash
    git merge navbar
    ```
    *   *Result*: Combines `navbar` code with the `pricing-table` code on `main`.
3.  **Push**:
    ```bash
    git push origin main
    ```

---

## 🧹 Cleanup
After a successful merge, the feature branch is obsolete.

*   **Delete Local**: `git branch -d pricing-table`
*   **Delete Remote**: `git push origin --delete pricing-table`

> **Next**: The more robust workflow using **Pull Requests**.
