#CI-CD #GitHub #Automation #Workflow #Security #Artifacts #BestPractice

>  Maintain a clean and secure CI/CD pipeline by managing [[#🧹 Removing Workflow Artifacts from GitHub|artifact retention]], increasing visibility with [[#📛 Adding a Workflow Status Badge|status badges]], and securing your deployments by configuring [[#🛡️ Adding Workflow Environment Protections|environment protection rules]] like required reviewers and wait timers.

---

## 🧹 Removing Workflow Artifacts from GitHub

By default, GitHub stores build logs and uploaded [[#📦 Working with Artifacts|artifacts]] for **90 days** before automatically deleting them. As you approach your account's storage limits, you may need to manage these artifacts more proactively to prevent your workflows from being blocked.

> [!warning]
> Once an artifact is deleted, it cannot be restored.

### 1. Manually Deleting an Artifact
You can delete individual artifacts directly from the GitHub UI. This requires write access to the repository.

1.  Navigate to your repository's **Actions** tab.
2.  Select the workflow and then the specific run containing the artifact.
3.  Under the **Artifacts** section of the run summary, find the artifact you want to remove and click the trash can icon to delete it.

You can also programmatically delete artifacts using the [Artifacts REST API](https://docs.github.com/en/rest/actions/artifacts).

### 2. Changing the Retention Period
Instead of manual deletion, you can configure the default retention period at the repository, organization, or enterprise level.

-   **Note:** Changing the default only applies to *new* artifacts and logs; it does not affect existing ones.

You can also override the default retention period for a *specific artifact* directly within your workflow file using the `retention-days` option in the `actions/upload-artifact` action.

**Example: Upload an artifact that will only be stored for 10 days.**
```yaml
- name: 'Upload Artifact with custom retention'
  uses: actions/upload-artifact@v4
  with:
    name: my-artifact
    path: my_file.txt
    retention-days: 10
```

---

## 📛 Adding a Workflow Status Badge

A status badge provides a quick, visual indicator of whether your workflow is passing or failing, directly in your `README.md` file or any other web page.

### Creating a Status Badge
1.  Navigate to your repository's **Actions** tab.
2.  Select a workflow from the left sidebar.
3.  Click the **"..."** menu and select **Create status badge**.
4.  GitHub will provide you with the Markdown snippet to copy.

### Customizing the Badge
By default, the badge shows the status of the workflow on your repository's default branch. You can customize this by adding query parameters to the badge URL.

-   **`branch`:** Show the status for a specific branch.
-   **`event`:** Show the status for a specific event (e.g., `pull_request`).

**Example Markdown for a status badge:**
```markdown
<!-- Badge for the default branch -->
![CI Workflow Status](https://github.com/OWNER/REPO/actions/workflows/main.yml/badge.svg)

<!-- Badge for a specific branch named 'my-feature-branch' -->
![CI Workflow Status for my-feature-branch](https://github.com/OWNER/REPO/actions/workflows/main.yml/badge.svg?branch=my-feature-branch)
```
This allows you to create a table in your `README.md` showing the build status for all your important branches.

---

## 🛡️ Adding Workflow Environment Protections

For sensitive environments like `production` or `staging`, you can configure protection rules. A job targeting a protected environment will not run until all of its protection rules have passed.

> [!note]
> For most plans, these protection rules are only available for **public repositories**.

### How to Configure Environment Protections
1.  In your repository, go to **Settings > Environments**.
2.  Create a new environment or edit an existing one.
3.  Add protection rules.

### Available Protection Rules
-   **Required reviewers:** Require a specific person or team to manually approve a deployment job before it can proceed. This is a critical control for production deployments.
-   **Wait timer:** Delay a job for a specified amount of time (in minutes) after it has been triggered. This can provide a window to cancel a deployment if a problem is discovered.

### Using an Environment in a Workflow
To subject a job to these rules, reference the environment by name in your workflow file.
```yaml
jobs:
  deploy-to-prod:
    runs-on: ubuntu-latest
    # This job is now protected by the rules of the 'production' environment
    environment: production
    steps:
      - name: Deploy
        run: ./deploy-to-prod.sh
```
When this workflow runs, the `deploy-to-prod` job will pause and wait for a required reviewer to approve it before it is sent to a runner and gains access to any environment-specific secrets.