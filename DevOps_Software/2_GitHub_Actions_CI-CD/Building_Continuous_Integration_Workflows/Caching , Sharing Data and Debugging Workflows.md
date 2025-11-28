#CI-CD #GitHub #Automation #Workflow #Debugging #Performance #Artifacts

>  Optimize your workflows for speed and efficiency by [[#⚡ Caching Dependencies for Faster Workflows|caching dependencies]]. Share data between jobs using [[#📦 Passing Artifact Data Between Jobs|artifacts]]. Troubleshoot failures by [[#🐞 Enabling Debug Logging|enabling debug logging]] and accessing detailed [[#📊 Accessing Workflow Logs|workflow logs]] through the UI or the REST API.

---
## ⚡ Caching Dependencies for Faster Workflows

Jobs on GitHub-hosted runners start in a clean virtual environment every time, which means they have to re-download dependencies (like `npm` packages or Maven `.jar` files) on every run. Caching these dependencies can dramatically reduce workflow execution time.

To cache dependencies, use the official `actions/cache` action.

### `actions/cache` Configuration

The action requires two key parameters to identify and restore a cache:

| Parameter | Description | Required |
|---|---|---|
| `key` | A unique identifier for the cache. The action saves a cache with this key on a successful run and searches for this key on subsequent runs. | Yes |
| `path` | The file path on the runner to cache (e.g., the directory where `npm` stores packages). | Yes |
| `restore-keys`| (Optional) An ordered list of alternative keys to use for finding a cache if an exact match for `key` isn't found. This allows for partial cache hits. | No |

**Example: Caching `npm` dependencies**

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Cache NPM dependencies
    uses: actions/cache@v3
    with:
      # The path to the npm cache directory
      path: ~/.npm
      # A unique key for the cache. It combines the OS, a static prefix, and a hash of the lock file.
      # The cache will only be restored if the lock file is identical.
      key: ${{ runner.os }}-npm-cache-${{ hashFiles('**/package-lock.json') }}
      # If no exact match is found, restore the most recent cache for this OS.
      restore-keys: |
        ${{ runner.os }}-npm-cache-
```

-   The `key` is crucial. By including `hashFiles('**/package-lock.json')`, you ensure that a new cache is created only when your dependencies actually change.

---

## 📦 Passing Artifact Data Between Jobs

Jobs run in isolated environments and cannot directly share files. To pass data (like a compiled binary or test report) from one job to another, you must use [[#📦 Working with Artifacts|artifacts]].

The process involves two actions: `actions/upload-artifact` and `actions/download-artifact`.

**Example: A `build` job creates a file, and a `test` job uses it.**
```yaml
name: Share data between jobs
on: push
jobs:
  job_1:
    name: Upload File
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello World" > file.txt
      - name: Upload the file as an artifact
        uses: actions/upload-artifact@v4
        with:
          name: my-file # The name of the artifact
          path: file.txt   # The path to the file to upload

  job_2:
    name: Download File
    runs-on: ubuntu-latest
    # 'needs' ensures job_2 waits for job_1 to complete successfully
    needs: job_1
    steps:
      - name: Download the artifact
        uses: actions/download-artifact@v4
        with:
          name: my-file # Must match the upload name
      - name: Read the file's contents
        run: cat file.txt
```

---

## 🐞 Debugging and Logging

### Enabling Step Debug Logging
If the standard logs aren't detailed enough, you can enable more verbose diagnostic logging. This is done by setting repository secrets (requires admin access).

-   **Runner Diagnostic Logging:** Set the repository secret `ACTIONS_RUNNER_DEBUG` to `true`.
-   **Step Diagnostic Logging:** Set the repository secret `ACTIONS_STEP_DEBUG` to `true`. This provides detailed logs for each step's execution, including inputs and outputs.

### Accessing Workflow Logs in GitHub
1.  In your repository, go to the **Actions** tab.
2.  In the left sidebar, select the workflow you want to inspect.
3.  In the list of workflow runs, click on the specific run you need to debug.
4.  On the summary page, click on the job you want to view.
5.  You can expand individual steps to see their log output. Failed steps will be automatically expanded.
6.  You can also download the complete raw logs for offline analysis.

### Accessing Workflow Logs from the REST API

You can programmatically retrieve logs, rerun workflows, or cancel runs via the GitHub REST API.

- **Endpoint:** `GET /repos/{owner}/{repo}/actions/runs/{run_id}/logs`
- **Authentication:** Requires read access. For private repositories, an access token with the `repo` scope is needed.

---

## 🔑 Using Installation Tokens from a GitHub App

When you build a GitHub App, you can authenticate it to perform API requests on behalf of an installation (i.e., a user or organization that has installed your app). This is done using a short-lived **installation access token**.

### Using the Token for API Requests
Once you generate an installation access token, you include it in the `Authorization` header of your API calls.
```bash
curl --request GET \
--url "https://api.github.com/meta" \
--header "Accept: application/vnd.github+json" \
--header "Authorization: Bearer INSTALLATION_ACCESS_TOKEN" \
--header "X-GitHub-Api-Version: 2022-11-28"
```

These requests are attributed to the GitHub App, not a specific user.

### Using the Token for Git Access

You can also use an installation access token for HTTP-based Git operations, provided your app has the `Contents` repository permission. The token acts as the password.

```bash
git clone https://x-access-token:INSTALLATION_ACCESS_TOKEN@github.com/owner/repo.git
```