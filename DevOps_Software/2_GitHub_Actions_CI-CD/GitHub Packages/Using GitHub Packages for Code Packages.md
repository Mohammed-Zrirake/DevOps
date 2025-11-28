#GitHub #Packages #Registry #CI-CD #DevOps #npm #NuGet #Maven

>  To use [[What is GitHub Packages?|GitHub Packages]] for code like `npm` or `NuGet`, you must first authenticate your local package manager. This requires generating a **Personal Access Token (PAT)** and using it along with your username to log in to the specific package registry endpoint. Once authenticated, you can install and publish packages using your standard command-line tools.

---

While the previous unit covered container images, this guide focuses on the other package types supported by GitHub Packages. The service integrates directly with your project's existing ecosystem tooling (`npm`, `dotnet` CLI, `mvn`, etc.).

## 🔑 Authenticating to GitHub Packages

The way you authenticate depends on your package ecosystem, but every method requires three key pieces of information:

1.  **Your GitHub Username**
2.  **A Personal Access Token (PAT)**
3.  **The GitHub Packages Endpoint** for your specific package type.

### Step 1: Generate a Personal Access Token (PAT)
To install, publish, or delete packages, you need an access token. A PAT is the standard way to authenticate when using a package manager from your local machine or in scripts.

-   You can generate a PAT from your GitHub profile: **Settings > Developer settings > Personal access tokens**.
-   Ensure you grant the token the correct scopes (e.g., `read:packages`, `write:packages`, `delete:packages`).

> [!warning] Treat Your Tokens Like Passwords
> Your Personal Access Tokens are highly sensitive credentials. Keep them secret, store them securely (e.g., in a password manager), and never commit them to your source code.

### Step 2: Log In to the Correct Registry Endpoint
Before you can interact with packages, you must configure your local package manager to authenticate with the correct GitHub Packages endpoint. The endpoint URL follows a standard structure:

`https://<PACKAGE_TYPE>.pkg.github.com/OWNER`

The following table shows the specific command to run to authenticate, based on your package ecosystem:

| Your package ecosystem | Command line to authenticate to GitHub Packages |
| :--- | :--- |
| **NuGet** | `dotnet nuget add source https://nuget.pkg.github.com/OWNER/index.json -n github -u OWNER -p YOUR_PAT_TOKEN` |
| **npm** | `npm login --registry=https://npm.pkg.github.com` (You will be prompted for username, password [use your PAT], and email) |
| **RubyGems** | `echo ":github: Bearer YOUR_PAT_TOKEN" >> ~/.gem/credentials` (and configure your `.gemrc` to point to the GitHub source) |
| **Maven & Gradle** | Authentication is typically configured directly in your project's `pom.xml` or `build.gradle` file, not via a separate login command. |

> [!info]
> For more detailed, ecosystem-specific instructions, refer to the [official GitHub documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry).

---

## 📥 Installing a Package

Once you're authenticated, you can use published packages in your projects just like you would from a public registry. The home page for any package on GitHub provides the exact commands and configuration snippets needed to install it, specific to your project's environment.

---

## ⚙️ Managing the Package Lifecycle

GitHub Packages provides several ways to manage your packages and automate your workflows.

### 1. Programmatic Management (API & Webhooks)
You can manage your packages programmatically using the **GitHub REST API** and **GraphQL API**. This enables advanced integration scenarios.

For example, by using GitHub's **Webhook** feature, you can trigger custom actions whenever a new package version is published.

> [!tip] Webhook Example
> Imagine you are a maintainer of an open-source project. You could configure a webhook that, upon a new package publication, triggers a serverless function to:
> -   Automatically post a Tweet announcing the new version.
> -   Publish a new blog post with release notes.

### 2. Automated Management (GitHub Actions)
You can use [[GitHub Actions]] to fully automate your package management. A common and powerful use case is automatically pruning old versions.

The official `actions/delete-package-versions` action allows you to create a workflow that, for example, automatically deletes the oldest version of your package every time a new version is published, helping you stay within storage limits and keep your registry clean.