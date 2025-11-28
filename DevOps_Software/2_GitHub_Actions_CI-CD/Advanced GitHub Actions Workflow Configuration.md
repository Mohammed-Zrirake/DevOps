#CI-CD #GitHub #Automation #CoreConcept #Workflow #Security

>  Go beyond simple `on: push` triggers by using scheduled events (`cron`), manual triggers (`workflow_dispatch`), and external webhooks (`repository_dispatch`). Use conditionals (`if:`) to control job execution and follow security best practices by pinning actions to specific versions.

---

## Triggering Workflows: Beyond `push` and `pull_request`

A workflow's power comes from its ability to run automatically in response to a wide variety of events.

### 1. 🕒 Scheduled Events (`schedule`)
The `schedule` event allows you to trigger a workflow at a specific UTC time using POSIX `cron` syntax.

- **Cron Syntax:** `[minute] [hour] [day of month] [month] [day of week]`

**Examples:**
```yaml
on:
  schedule:
    # Run every 15 minutes
    - cron: '*/15 * * * *'
    # Run every Sunday at 3:00 AM UTC
    - cron: '0 3 * * SUN'
```

> [!info] The shortest interval for scheduled workflows is every 5 minutes. They run on the latest commit of the default branch.

### 2. ▶️ Manual Events (`workflow_dispatch` & `repository_dispatch`)
Sometimes you need to trigger a workflow manually.

#### `workflow_dispatch` (UI Trigger)
This event adds a "Run workflow" button to the Actions tab in the GitHub UI, allowing you to trigger it manually. You can also define `inputs` that are presented as form fields in the UI.

```yaml
on:
  workflow_dispatch:
    inputs:
      logLevel:
        description: 'Log Level'
        required: true
        default: 'warning'
        type: choice
        options:
        - info
        - warning
        - debug
      tags:
        description: 'Test scenario tags'
```

#### `repository_dispatch` (API Trigger)
This powerful event allows you to trigger a workflow from an **external** source by sending a `POST` request to the GitHub API. It's the key to integrating GitHub Actions with other systems.

- **Use Cases:**
  - Triggering a workflow from an external CI/CD tool.
  - Coordinating multi-repo deployments (Repo A finishes build → triggers Repo B).
  - Chaining workflows together.

- **Workflow Configuration:**

  ```yaml
  on:
    repository_dispatch:
      # Optional: only listen for specific event types
      types: [run-tests, deploy-to-prod]
  ```

- **API Request:**
  ```bash
  curl -X POST \
    -H "Accept: application/vnd.github+json" \
    -H "Authorization: token YOUR_GITHUB_TOKEN" \
    https://api.github.com/repos/OWNER/REPO/dispatches \
    -d '{"event_type":"run-tests","client_payload":{"env":"staging"}}'
  ```

- **Accessing Data in the Workflow:**
  - `github.event.action`: Contains the `event_type` (e.g., "run-tests").
  - `github.event.client_payload`: An object containing any custom JSON data you sent.

### 3.  webhook Events
You can configure a workflow to run in response to almost any webhook event on GitHub, with fine-grained control over the specific activity `types`.

```yaml
on:
  check_run:
    # Only run when a check run is re-requested or a custom action is requested
    types: [rerequested, requested_action]
```

---

## ⚙️ Controlling Job Execution with Conditionals

You can use the `if` conditional to control whether a job or step should run. GitHub evaluates the expression to determine the outcome.

- **Syntax:** `if: <expression>`
- **Context:** You can use any supported [context](https://docs.github.com/en/actions/learn-github-actions/contexts) to build your expression. `github` is the most common.

**Example:** Only run a job if the workflow was triggered by a push to the `main` branch.
```yaml
jobs:
  prod-check:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      ...
```

> [!tip]
> For simple `if` conditionals, you can omit the `${{ }}` syntax. For more complex expressions or use in other places (like `with:`), you must include it.

---

## (Management) & Best Practices

### 1. Disabling and Deleting Workflows
- **Disable:** You can temporarily disable a workflow in the GitHub UI (under the Actions tab) or via the API. This is useful for pausing a broken, noisy, or resource-intensive workflow without deleting the file from your repository.
- **Cancel:** You can cancel a workflow run that is currently in progress.

### 2. Using Organization Workflow Templates
To promote consistency across an organization, you can define workflow templates in a central `.github` repository. These templates will then appear as starting points for all other repositories within the organization, preventing teams from reinventing the same CI/CD process.

### 3. 🔒 Pinning Action Versions (Security Best Practice)
When referencing an action, a tag like `@v4` is mutable and can change. This could introduce unexpected or malicious changes into your build.

For maximum security and stability, **pin the action to a specific commit SHA**.

```yaml
steps:    
  # Good: References a major version tag
  - uses: actions/setup-node@v4
  
  # Better (More Secure): References an immutable commit SHA
  - uses: actions/setup-node@60edb5dd545a775199f52541f313c978b1b5be58
```
This ensures your workflow *always* executes the exact same code, every single time.