#CI-CD #GitHub #Packages #Registry #Containerization #Workflow

>  Use [[GitHub Actions]] to automate publishing versioned artifacts. For packages like `npm`, use the `setup-node` action with a `registry-url`. For container images, use actions like `docker/login-action` to authenticate to the GitHub Container Registry (`ghcr.io`) and then `docker push`. In both cases, the built-in `${{ secrets.GITHUB_TOKEN }}` provides seamless authentication.

---

## 📦 Publishing Packages (e.g., npm)

[[GitHub Packages]] allows you to publish and consume packages like `npm` or `Maven` directly from your repository. You can automate this process with a workflow.

A common pattern is to trigger the publish workflow only when a new release is created, ensuring that only properly versioned and tagged code is published.

### Example Workflow: Publish an `npm` Package on Release
This workflow has two jobs: `build` to validate the code, and `publish-gpr` to publish the package to the GitHub Package Registry (GPR).

```yaml
name: Publish Node.js Package to GPR

on:
  release:
    types: [created] # This workflow runs only when a new release is published

jobs:
  # First, a job to build and test the code to ensure it's valid
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18 # Use a specific version
      - run: npm ci
      - run: npm test

  # Second, a job to publish the package, which only runs if 'build' succeeds
  publish-gpr:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write # Grant write permissions to the packages scope
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          # This configures the .npmrc file to use the GitHub Packages registry
          registry-url: https://npm.pkg.github.com/
      - run: npm ci
      - name: Publish to GitHub Packages
        run: npm publish
        env:
          # The GITHUB_TOKEN is used to authenticate with the registry
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Key Steps Explained:**
1.  **`needs: build`**: The `publish-gpr` job will not start until the `build` job completes successfully.
2.  **`permissions`**: This block is crucial. It grants the temporary `GITHUB_TOKEN` the necessary `packages: write` permission to publish the package.
3.  **`registry-url`**: The `actions/setup-node` action has a special parameter that automatically configures the `.npmrc` file in the runner, pointing the `npm` client to the correct GitHub Packages registry.
4.  **`NODE_AUTH_TOKEN`**: The `npm publish` command requires an authentication token. You provide the built-in `GITHUB_TOKEN`, which is a secure, automatically generated secret with the permissions you defined above.

---

## 🐳 Publishing to GitHub Container Registry (`ghcr.io`)

The **GitHub Container Registry** is specifically designed to host and manage container images. It offers several advantages:
-   **Org/User Scoped:** Images are stored within your organization or user account, not tied to a single repository.
-   **Fine-Grained Permissions:** You can configure precisely who can manage and access your container images.
-   **Anonymous Public Access:** Public container images can be pulled without authentication.

### The Manual Process (for context)
Before automating, it's helpful to understand the manual Docker commands for publishing to `ghcr.io`.

1.  **Log in to the registry:**
    ```bash
    # Replace USERNAME with your GitHub username and provide a PAT with package permissions
    echo $YOUR_PAT | docker login ghcr.io -u USERNAME --password-stdin
    ```
2.  **Tag the image correctly:** The image tag must be prefixed with the registry and your username/org.
    ```bash
    # Format: ghcr.io/OWNER/IMAGE_NAME:tag
    docker tag YOUR_IMAGE_ID ghcr.io/your-username/my-image:latest
    ```
3.  **Push the image:**
    ```bash
    docker push ghcr.io/your-username/my-image:latest
    ```

### The Automated Workflow (The Best Practice)
In a GitHub Actions workflow, you don't need to manually manage Personal Access Tokens (PATs). You can use specialized actions and the built-in `GITHUB_TOKEN`.

**Example Steps to Build and Push a Docker Image to `ghcr.io`:**
```yaml
jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write # Grant permissions to push to the container registry

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Log in to the Container registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          # 'github.actor' is the username of the person who initiated the workflow
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          # 'github.repository' is in the format 'owner/repo'
          tags: ghcr.io/${{ github.repository }}:latest
```

> [!info] The Power of `GITHUB_TOKEN`
> As demonstrated, the automatically generated `GITHUB_TOKEN` is the key to seamless authentication within your workflows. By granting it the appropriate `packages: write` permission, it can be used to securely authenticate with both the standard package registries (`npm.pkg.github.com`, etc.) and the GitHub Container Registry (`ghcr.io`), eliminating the need for long-lived PATs in your repository secrets.