#CI-CD #GitHub #Automation #Actions #BestPractice #Development #Community

>  To share your custom action, you must decide on its [[#🌍 Choosing a Location and Visibility for Your Action|visibility]], create clear [[#📝 Documenting Your Action|documentation]], implement a robust [[#릴리즈 Releasing and Versioning Your Action|versioning strategy]], and, if it's public, [[#🏪 Publishing an Action to the GitHub Marketplace|publish it to the GitHub Marketplace]].

---

## 🌍 Choosing a Location and Visibility for Your Action

Before you publish, you must decide where the action will live and who can access it.

### Public Actions
-   **Visibility:** Can be used by workflows in *any* repository on GitHub.
-   **Best Practice:** A public action should exist in its **own dedicated repository**.
-   **Why a dedicated repo?**
    -   It allows you to version, track, and release the action independently, just like any other piece of software.
    -   It makes the action easier for the community to discover.
    -   It narrows the scope of the code base, making it easier for others to contribute or fix issues.
    -   It separates the action's versioning from any specific application's code.

### Private Actions
-   **Visibility:** Can only be used by workflows in the *same* repository where the action is defined.
-   **Location:** You can store the action's files anywhere in your repository, but the recommended practice is to place them in the `.github/actions/` directory (e.g., `.github/actions/my-action`).

---

## 📝 Documenting Your Action

Good documentation is critical for usability, whether your action is public or private. The `README.md` file is the most important piece of documentation.

> [!tip] Key Information to Include in Your `README.md`
> -   A detailed description of what the action does.
> -   **Required** input and output arguments.
> -   **Optional** input and output arguments.
> -   Any `secrets` the action needs to function.
> -   Any `environment variables` the action uses.
> -   A clear, copy-pasteable example of how to use your action in a workflow.

---

## (Releasing) and Versioning Your Action

A clear release-management strategy is essential for providing stability to your users.

> [!danger] Do Not Reference the Default Branch
> Users of your action should **never** reference the default branch (e.g., `main`). This branch contains the latest, and potentially unstable, code, which could break their workflows at any time.

### Versioning Best Practices
-   **Use Semantic Versioning:** Follow the `Major.Minor.Patch` (e.g., `v1.2.3`) versioning scheme.
-   **Use a Release Branch:** Create and validate your release on a dedicated branch (e.g., `release/v1`) before creating the release tag.
-   **Move Major Version Tags:** After creating a new patch or minor release (e.g., `v1.1.0`), move the major version tag (e.g., `v1`) to point to the Git ref of this new, current release. This allows users who pin to a major version (`@v1`) to get non-breaking updates automatically.
-   **Introduce New Major Versions for Breaking Changes:** If you make a change that will break existing workflows, create a new major version tag (e.g., `v2`).
-   **Use Beta Tags for Pre-releases:** You can release a major version with a beta tag (e.g., `v2-beta`) to allow for community testing before the official release.

### Ways for Users to Reference Your Action
Based on the strategy above, users can choose their desired level of stability.

```yaml
steps:
    # Recommended: Use a major version tag to get non-breaking updates
    - uses: actions/your-action@v1

    # More specific: Pin to a minor or patch version
    - uses: actions/your-action@v1.2

    # Most specific and secure: Pin to a full commit SHA
    # This guarantees the action's code will never change.
    - uses: actions/your-action@2522385f6f7ba04fe7327647b213799853a8f55c
```

---

## 🏪 Publishing an Action to the GitHub Marketplace

When you're ready to share your action with the entire GitHub community, you can publish it to the GitHub Marketplace.

### Requirements for Publishing
1.  The action must be in a **public repository**.
2.  The repository must contain **only one action**.
3.  The action's metadata file (`action.yml` or `action.yaml`) must be in the **root directory** of the repository.
4.  The `name` in the metadata file must be **globally unique** on the GitHub Marketplace.
5.  The `name` cannot match an existing GitHub user or organization (unless you are that user/org).
6.  The `name` cannot match an existing Marketplace category or GitHub feature.

### How to Publish
The process is straightforward:
1.  Ensure all the requirements above are met.
2.  Navigate to your action's repository on GitHub.
3.  Draft a new release.
4.  On the release page, you will see a checkbox to **"Publish this action to the GitHub Marketplace"**.
5.  Check the box, fill out the required category and other information, and publish the release. If all requirements are met, your action will be live on the Marketplace immediately.