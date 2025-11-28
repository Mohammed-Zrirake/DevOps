#CI-CD #GitHub #Automation #Workflow #Deployment #Azure

>  Automate your Continuous Deployment (CD) pipeline to Microsoft Azure using [[GitHub Actions]]. Use event triggers like pull request labels to control deployments, secure your credentials with [[#🔒 Storing Credentials with GitHub Secrets|GitHub Secrets]], and leverage pre-built actions from the marketplace to log in, build, and deploy to Azure services like Web Apps.

---

## ▶️ Options for Triggering a CD Workflow

While CI workflows are often triggered by a `push` to a branch, Continuous Deployment (CD) workflows often require more deliberate triggers.

-   **ChatOps:** Trigger a workflow by posting a specific command or comment in a chat client or a pull request. For example, a bot could be configured to initiate a deployment when a `/deploy` comment is posted.
-   **Pull Request Labels:** This is a very common and powerful pattern. You can configure your workflow to run only when a specific label (e.g., `stage`, `deploy-to-prod`) is added to a pull request.

**Example: Triggering a workflow when a pull request is labeled:**
```yaml
on:
  pull_request:
    types: [labeled]
```

---

## ⚙️ Controlling Execution with Job Conditionals

Often, you only want to run a job if a certain condition is met. The `if` conditional is perfect for this, allowing you to evaluate an expression at runtime.

**Example: Only run this job if the `stage` label was the one added to the pull request.**
```yaml
jobs:
  deploy-to-staging:
    runs-on: ubuntu-latest
    if: contains(github.event.pull_request.labels.*.name, 'stage')
    steps:
      ...
```
-   `github.event.pull_request.labels.*.name`: This expression accesses an array of all label names on the pull request that triggered the event.
-   `contains(...)`: This is a built-in function that checks if the array contains the string 'stage'.

---

## 🔒 Storing Credentials with GitHub Secrets

You should **never** store sensitive information like passwords, API keys, or cloud credentials directly in your workflow file. Use **GitHub Secrets** to store them securely.

1.  **Create the Secret:** In your repository, go to **Settings > Secrets and variables > Actions**. Click **New repository secret** to add your credential.
2.  **Reference the Secret:** Use the `${{ secrets.SECRET_NAME }}` syntax in your workflow to access the secret's value. GitHub ensures the value is never printed in the logs.

**Example: Using Azure credentials stored in a secret.**
```yaml
- name: "Login via Azure CLI"
  uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}
```

---

## 🚀 Deploying to Microsoft Azure

The GitHub Marketplace provides a rich ecosystem of actions for interacting with Azure, simplifying your deployment pipeline.

### Common Azure Actions
-   `azure/login@v1`: Authenticates with your Azure account using the credentials stored in secrets.
-   `azure/docker-login@v1`: Logs in to a container registry, such as Azure Container Registry or GitHub Packages.
-   `azure/webapps-deploy@v1`: Deploys a containerized web application to Azure Web Apps.

### Example: A Job to Deploy a Container to Azure Web Apps
This job depends on a previous `Build-Docker-Image` job (which would have built and pushed the container image artifact).

```yaml
jobs:
  Deploy-to-Azure:
    runs-on: ubuntu-latest
    needs: Build-Docker-Image
    name: Deploy app container to Azure
    steps:
      # Step 1: Log in to Azure
      - name: "Login via Azure CLI"
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      # Step 2: Log in to your container registry (e.g., GitHub Packages)
      - uses: azure/docker-login@v1
        with:
          login-server: ${{ env.IMAGE_REGISTRY_URL }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # Step 3: Deploy the container image to the Azure Web App
      - name: Deploy web app container
        uses: azure/webapps-deploy@v1
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          images: ${{ env.IMAGE_REGISTRY_URL }}/${{ github.repository }}/${{ env.DOCKER_IMAGE_NAME }}:${{ github.sha }}

      # Step 4: Best practice - log out of Azure
      - name: Azure logout
        run: az logout
```

---

## 🏗️ Managing Azure Resources with GitHub Actions (Infrastructure as Code)

A true CD process includes automating the creation and destruction of infrastructure. You can use GitHub Actions to manage your Azure resources using an Infrastructure as Code (IaC) approach.

> [!warning]
> It's critical to tear down ephemeral resources (like those for pull request reviews) as soon as they are no longer needed to avoid unnecessary cloud costs.

You can create a single workflow with separate jobs for "spin up" and "destroy," controlled by different pull request labels.

**Example Workflow Structure:**
```yaml
on:
  pull_request:
    types: [labeled]

jobs:
  set-up-azure-resources:
    runs-on: ubuntu-latest
    if: contains(github.event.pull_request.labels.*.name, 'spin up environment')
    steps:
      - uses: actions/checkout@v2
      - name: Azure login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      - name: Create Azure resource group
        run: az group create --location ... --name ...
      - name: Create Azure app service plan
        run: az appservice plan create ...
      - name: Create webapp resource
        run: az webapp create ...

  destroy-azure-resources:
    runs-on: ubuntu-latest
    if: contains(github.event.pull_request.labels.*.name, 'destroy environment')
    steps:
      - name: Azure login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      - name: Delete Azure resource group
        run: az group delete --name ... --yes --no-wait
```

This workflow uses the Azure CLI (`az`) to directly create and destroy resources, providing a powerful and flexible way to manage your infrastructure as part of your automated CI/CD pipeline.