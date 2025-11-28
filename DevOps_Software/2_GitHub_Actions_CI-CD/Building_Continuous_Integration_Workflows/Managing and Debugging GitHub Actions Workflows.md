#CI-CD #GitHub #Automation #Workflow #Debugging #Artifacts

>  Effectively manage your CI/CD pipelines by understanding what [[#🔍 Identifying the Event That Triggered a Workflow|triggered a workflow]], knowing how to access and interpret [[#📊 Accessing and Understanding Workflow Logs|workflow logs]], using [[#📦 Working with Artifacts|artifacts]] to share data between jobs, and automating processes like pull request reviews.

---

## 🔍 Identifying the Event That Triggered a Workflow

Understanding what kicked off a workflow is the first step in debugging and auditing.

### What is a Workflow Trigger?
A workflow trigger is an event that causes a workflow to run. Common triggers include:
-   `push` or `pull_request` (code changes)
-   `workflow_dispatch` (manual UI trigger)
-   `schedule` (cron jobs)
-   `repository_dispatch` (external API calls)
-   `issues`, `discussion`, `pull_request_review` (repository activity)

### How to Identify the Trigger
1.  **Use the GitHub Actions UI:**
    -   Go to your repository's **Actions** tab.
    -   Select a specific workflow run.
    -   The triggering event (e.g., `push`, `pull_request`) is displayed at the top of the summary.
2.  **Use the `github` Context:**
    -   The `github.event_name` variable is available within any workflow run. You can print it in a step for easy debugging.
      ```yaml
      - name: Show event trigger
        run: echo "Triggered by a ${{ github.event_name }} event."
      ```
3.  **Infer from Repository Activity:**
    -   You can often deduce the trigger by correlating the workflow's timestamp with recent activity like commits, pull request updates, or new issue comments.

---

## 📊 Accessing and Understanding Workflow Logs

When a workflow runs, it produces detailed logs for each step. When a run fails, these logs are your primary tool for diagnosis.

### How to Access Logs
1.  Navigate to the **Actions** tab in your repository.
2.  Select the workflow run you want to inspect (failed runs are marked with a red ❌).
3.  On the run summary page, you'll see a list of jobs.
4.  Click on the failed job to expand it.
5.  Click on the specific step that failed to view its detailed log output.

### How to Diagnose a Failed Run
1.  **Review the Error:** In the failed step's log, look for specific error messages, stack traces, or non-zero exit codes. For example, a failed test might show `npm ERR! Test failed` or `exit code 1`.
2.  **Check the Workflow Configuration:** Review your `.yml` file.
    -   What was the step trying to do?
    -   Are there any `if` conditionals that might have caused unexpected behavior?
    -   Are all necessary `env` variables or `secrets` correctly configured?
3.  **Common Failure Scenarios:**
    -   `command not found`: A required dependency was not installed in a previous step.
    -   `npm install` fails: Often due to a corrupt `package-lock.json` or network issues.
    -   `Permission denied`: The workflow token might be missing permissions, or a required secret is not set.

---

## 📦 Working with Artifacts

> [!info] Definition
> An **artifact** is a file or collection of files produced during a workflow run (e.g., a compiled binary, a test coverage report, a Docker image).

Since each job runs in a fresh, clean virtual machine, you cannot pass files between jobs directly. Artifacts solve this problem by allowing you to save files at the end of one job and download them at the beginning of another.

### Storing and Retrieving Artifacts
-   **Uploading:** Use the `actions/upload-artifact` action. You provide a `name` for the artifact and a `path` to the file(s) you want to upload.
-   **Downloading:** Use the `actions/download-artifact` action. You specify the `name` of the artifact to download.

**Example Workflow:** A `build` job creates an artifact, and a `test` job downloads and uses it.
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          npm install
          npm run build
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: webpack-artifacts
          path: public/

  test:
    # 'needs' ensures that 'test' only runs after 'build' succeeds
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: webpack-artifacts
          path: public
      # ... now you can run tests against the files in the 'public' directory
```

> GitHub stores artifacts for 90 days (for public repos and within the storage limits of private repos).

---

## 🤖 Automating Reviews and Other Processes

Workflows can be triggered by events beyond just code changes, allowing for powerful repository automation.

**Example:** Automatically add an "approved" label to a pull request after it receives one review approval.

This can be achieved with the `pull_request_review` trigger and a community action.
```yaml
name: Label PR when approved

on:
  pull_request_review:
    types: [submitted]

jobs:
  labeling:
    runs-on: ubuntu-latest
    steps:
     - name: Label when approved
       if: github.event.review.state == 'approved'
       uses: pullreminders/label-when-approved-action@main
       env:
         # Number of approvals required
         APPROVALS: "1"
         # The GITHUB_TOKEN is required to allow the action to modify the PR
         GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
         # The name of the label to add
         ADD_LABEL: "approved"
```

This simple automation can, in turn, trigger another workflow. For example, a separate deployment workflow could be configured to run only when a pull request with the "approved" label is merged.