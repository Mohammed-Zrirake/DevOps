#Git #GitHub #Collaboration #Workflow #FeatureBranches

> "Don't work on master, you silly goose."
> The **Feature Branch Workflow** is the industry standard. It treats `master` (or `main`) as the sacred, stable history, while all work happens on separate, dedicated branches.

---

## 🛑 The Golden Rule
**Never commit directly to `master`/`main`.**
*   **Centralized Workflow**: Everyone pushes to `master`. (Chaos, broken builds).
*   **Feature Branch Workflow**: Everyone pushes to `feature-branches`. `master` is only updated via merges when code is ready.

---

## 🌳 The Core Concept
1.  **Source of Truth**: `master` is the official project history. It should **never** be broken.
2.  **Isolation**: Every new feature, bug fix, or experiment gets its own branch (e.g., `navbar`, `add-dark-theme`).
3.  **Collaboration**: You can push broken/incomplete code to a feature branch to get help without breaking the app for everyone else.

---

## 🎭 Scenario 1: The "Add Dark Theme" Collaboration
*   **Characters**: David, Pamela, Forrest.
*   **David**: Starts working on a branch `add-dark-theme`.
    *   He is 30% done. The code is messy.
    *   **Goal**: He wants Pamela's help.
    *   **Action**: He pushes `add-dark-theme` to GitHub. (He does **NOT** merge to master).
*   **Pamela**: working on her own `animated-scroll` branch.
    *   She pauses her work.
    *   `git fetch` (sees David's branch).
    *   `git switch add-dark-theme` (Downloads David's code).
    *   She fixes things, adds commits, and pushes back to `add-dark-theme`.
*   **Result**: They collaborated on a feature without touching `master` or disturbing Forrest.

---

## 🧪 Scenario 2: Hands-On Demo (Colt & Stevie The Chicken)

### 1. The Setup
*   **Repo**: `feature-branch-workflow`.
*   **Base**: Basic Bootstrap site.
*   **Stevie**: Wants to build a **Navbar**.
*   **Colt**: Wants to build a **Pricing Table**.

### 2. Stevie's Broken Navbar
1.  **Branch**: Stevie creates a branch.
    ```bash
    git switch -c navbar
    ```
    *   *Note on Naming*: Companies often use prefixes like `feat/navbar`, `bug/login`, `refactor/header`. Stevie uses simple `navbar`.
2.  **Work**: Copies a massive Bootstrap navbar into `index.html`.
    *   **Bug**: It doesn't collapse on mobile! It's "broken".
3.  **Sharing**: Stevie wants help.
    ```bash
    git commit -m "add broken navbar"
    git push origin navbar
    ```
    *   SAFE! `main` is still clean. Only the `navbar` branch has the broken code.

### 3. Colt's Pricing Table (Independent Work)
1.  **Branch**: Colt works on a separate feature.
    ```bash
    git switch -c pricing-table
    ```
2.  **Work**: Adds a container and pricing table HTML.
3.  **Push**:
    ```bash
    git push origin pricing-table
    ```
    *   Now GitHub has 2 feature branches: `navbar` and `pricing-table`.

### 4. The Assist (Colt Fixes Stevie's Bug)
Stevie asks: *"Hey, why is my navbar broken?"*
1.  **Fetch**: Colt gets latest info.
    ```bash
    git fetch
    # Output: [new branch] navbar -> origin/navbar
    ```
2.  **Switch**: Colt checks out Stevie's code.
    ```bash
    git switch navbar
    # Setup to track remote branch 'navbar' from 'origin'.
    ```
3.  **The Fix**: Colt realizes Stevie forgot the Bootstrap JS script tag.
    *   Adds `<script>` tag.
    *   Refreshes -> **It works!**
4.  **Push**:
    ```bash
    git add index.html
    git commit -m "Fix navbar bug (missing JS)"
    git push origin navbar
    ```

### 5. Stevie Updates
Stevie is back at his computer.
```bash
git pull origin navbar
```
*   Fast-foward merge.
*   Stevie now has the working code.

---

## 🏆 Benefits Summary
| Feature | Benefit |
| :--- | :--- |
| **Isolation** | You can experiment radically. If it fails, just delete the branch. `main` is never harmed. |
| **Collaboration** | Share incomplete/broken work for feedback by pushing the *branch*, not merging to master. |
| **Traffic Control** | Merging into `main` becomes a deliberate event (often via Pull Requests), not a side effect of saving your work. |

> **Next**: Merging these branches back into `master` (The endgame).
