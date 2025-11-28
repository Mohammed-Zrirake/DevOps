#CI-CD #GitHub #Automation #Workflow #Scripting #API #Octokit

>  `GitHub Script` (`actions/github-script`) is a powerful action that gives you an authenticated **Octokit client** directly within a workflow step. This allows you to write JavaScript to interact with the GitHub API to automate almost any repository task, like creating comments, labeling issues, or managing pull requests.

---

## ❓ What is GitHub Script?

GitHub Script is an official action that provides a sandboxed JavaScript environment within your workflow. Its primary purpose is to simplify interaction with the GitHub API.

It handles all the complex setup for you:
-   **Authentication:** It provides a pre-authenticated client, so you don't need to manually create and manage tokens for most operations.
-   **Configuration:** No need to install or configure any SDKs or dependencies.
-   **Environment:** It runs in a Node.js environment, giving you access to its powerful standard library.

### What is Octokit?
**Octokit** is the official collection of clients for the GitHub API. GitHub Script specifically uses `rest.js`, the Octokit client for the GitHub REST API. This means you can use it to automate virtually any action you could perform manually on GitHub:
-   Manage issues, pull requests, and commits.
-   Interact with users, projects, and organizations.
-   Render Markdown.
-   Retrieve common files like `.gitignore` templates or popular licenses.

### How is GitHub Script Different from Using `octokit/rest.js` Manually?
The key difference is convenience. GitHub Script provides a set of pre-configured global variables in your script's context.

-   `github`: A pre-authenticated `octokit/rest.js` client. Instead of `octokit.issues.createComment({...})`, you simply use `github.issues.createComment({...})`.
-   `context`: An object containing the full context of the workflow run (e.g., information about the triggering event, repository, issue number, etc.).
-   `core`: A reference to the `@actions/core` package for logging, setting outputs, and other workflow interactions.
-   `io`: A reference to the `@actions/io` package for file system utilities.

---

## 🚀 Creating a Workflow with GitHub Script

Using GitHub Script is like using any other action. You add it as a step in your job.

**Example: A workflow that automatically posts a comment on newly created issues.**

#### 1. The Trigger and Job Setup
The workflow is named and configured to run whenever an issue is opened.
```yaml
name: Learning GitHub Script

on:
  issues:
    types: [opened]

jobs:
  comment:
    runs-on: ubuntu-latest
    steps:
      # ... GitHub Script action will go here ...
```

#### 2. The GitHub Script Step
This is the core of the workflow. We use the `actions/github-script` action.
```yaml
      - name: Create a comment on the new issue
        uses: actions/github-script@v6 # Use a recent, stable version
        with:
          # The GITHUB_TOKEN is automatically provided and has permissions for the repo
          github-token: ${{ secrets.GITHUB_TOKEN }}
          # The JavaScript code to execute
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: "🎉 Thank you for creating this issue! We'll get back to you shortly."
            })
```
**Explanation:**
-   `github-token`: This provides the necessary authentication. The default `${{ secrets.GITHUB_TOKEN }}` has sufficient permissions for most repository-level tasks.
-   `script`: The inline JavaScript to run.
    -   We call `github.rest.issues.createComment`, the modern syntax for the Octokit REST client.
    -   `context.issue.number`, `context.repo.owner`, and `context.repo.repo` are used to get the details of the specific issue that triggered the workflow, making the action dynamic.

When this workflow runs, GitHub Script will log the code it executed for easy debugging in the Actions tab.

---

## 📂 Running from a Separate File

For more complex scripts, writing them inline in your YAML file can become messy. You can instead store your script in a separate `.js` file and reference it from your workflow.

**Example Script (`.github/scripts/my-script.js`):**
```javascript
// This script is a Node.js module that exports a function
module.exports = async ({ github, context }) => {
  // You have access to the same global variables
  const issueBody = context.payload.issue.body;
  
  // Do something complex with the issue body...
  const newBody = `### Acknowledged!\n\n> ${issueBody}`;

  return github.rest.issues.createComment({
    issue_number: context.issue.number,
    owner: context.repo.owner,
    repo: context.repo.repo,
    body: newBody
  });
};
```

**Workflow File (`.github/workflows/main.yml`):**
```yaml
- name: Run script from file
  uses: actions/github-script@v6
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    script: |
      const script = require('./.github/scripts/my-script.js');
      await script({github, context});
```
This approach keeps your workflow file clean and allows you to version, lint, and test your automation script just like any other piece of code in your repository.