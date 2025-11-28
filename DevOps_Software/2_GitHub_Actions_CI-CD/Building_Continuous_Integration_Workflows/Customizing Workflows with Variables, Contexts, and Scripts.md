#CI-CD #GitHub #Automation #CoreConcept #Workflow #Security

>  Make your workflows dynamic and adaptable by using **environment variables** and **contexts**. Variables provide simple key-value data, while contexts are rich objects containing information about the workflow run, runner, and more. Use these to control job logic, pass data between steps, and run custom scripts. Secure your deployments using **Environments** to add protection rules like manual approvals.

---

## 🏛️ Default Environment Variables vs. Contexts

GitHub Actions provides two primary ways to access information about the workflow's environment, each with a distinct purpose and scope.

### Default Environment Variables

- **What they are:** A set of default, case-sensitive variables (e.g., `GITHUB_REF`, `RUNNER_OS`) available to any command running on the runner. They typically refer to system and user configuration values.
- **Scope:** They can **only** be used within a `run` step on the runner machine.
- **Syntax:** Accessed using standard shell syntax (e.g., `$GITHUB_REF` on Linux/macOS).
- **Recommendation:** Use these to reference the filesystem rather than using hard-coded paths.


  ```yaml
  jobs:
    prod-check:
      steps:
        - run: echo "Deploying to production server on branch $GITHUB_REF"
  ```

### Contexts

- **What they are:** Objects that provide a rich, structured set of read-only information about the workflow run, jobs, steps, secrets, etc.
- **Scope:** They can be used **anywhere** in a workflow file, including in `if` conditionals that are evaluated *before* a job even starts on a runner.
- **Syntax:** Accessed using expression syntax: `${{ <context>.<property> }}`. Context names are lowercase, while default environment variables are uppercase.

  ```yaml
  name: CI
  on: push
  jobs:
    prod-check:
      if: github.ref == 'refs/heads/main'
      runs-on: ubuntu-latest
      steps:
        - run: echo "Deploying to production server on branch $GITHUB_REF"
  ```

In this example, the `github.ref` context is used in the `if` conditional to check the branch *before* the job runs. Then, inside the `run` step, the `$GITHUB_REF` environment variable is used to print the same information.

---

## 🔎 A Deep Dive into Contexts

### Available Contexts
Contexts are your primary way to access information about the state of your workflow.

| Context | Description |
|---|---|
| `github` | Information about the workflow run and the event that triggered it. See [github context](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context). |
| `env` | Contains variables that you have set in a workflow, job, or step. See [env context](https://docs.github.com/en/actions/learn-github-actions/contexts#env-context). |
| `vars` | Contains configuration variables set at the repository, organization, or environment level. See [vars context](https://docs.github.com/en/actions/learn-github-actions/contexts#vars-context). |
| `job` | Information about the currently running job. See [job context](https://docs.github.com/en/actions/learn-github-actions/contexts#job-context). |
| `jobs` | For reusable workflows only, contains outputs of jobs from the reusable workflow. See [jobs context](https://docs.github.com/en/actions/learn-github-actions/contexts#jobs-context). |
| `steps` | Information about the steps that have already run in the current job. See [steps context](https://docs.github.com/en/actions/learn-github-actions/contexts#steps-context). |
| `runner` | Information about the runner executing the job. See [runner context](https://docs.github.com/en/actions/learn-github-actions/contexts#runner-context). |
| `secrets` | Contains the names and values of secrets available to the workflow run. See [secrets context](https://docs.github.com/en/actions/learn-github-actions/contexts#secrets-context). |
| `strategy`| Information about the matrix execution strategy for the current job. See [strategy context](https://docs.github.com/en/actions/learn-github-actions/contexts#strategy-context). |
| `matrix` | Contains the specific matrix properties that apply to the current job. See [matrix context](https://docs.github.com/en/actions/learn-github-actions/contexts#matrix-context). |
| `needs` | Contains the outputs of all jobs that are defined as a dependency of the current job. See [needs context](https://docs.github.com/en/actions/learn-github-actions/contexts#needs-context). |
| `inputs` | Contains the inputs of a reusable (`workflow_call`) or manually triggered (`workflow_dispatch`) workflow. See [inputs context](https://docs.github.com/en/actions/learn-github-actions/contexts#inputs-context). |

### Context Availability and Restrictions

Not all contexts or special functions are available everywhere in a workflow file. The following table details where specific contexts can be used.

| Workflow key                                       | Available Contexts                                                                                    | Special Functions                                       |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `run-name`                                         | `github`, `inputs`, `vars`                                                                            | None                                                    |
| `concurrency`                                      | `github`, `inputs`, `vars`                                                                            | None                                                    |
| `env`                                              | `github`, `secrets`, `inputs`, `vars`                                                                 | None                                                    |
| `jobs.<job_id>.concurrency`                        | `github`, `needs`, `strategy`, `matrix`, `inputs`, `vars`                                             | None                                                    |
| `jobs.<job_id>.container`                          | `github`, `needs`, `strategy`, `matrix`, `vars`, `inputs`                                             | None                                                    |
| `jobs.<job_id>.container.credentials`              | `github`, `needs`, `strategy`, `matrix`, `env`, `vars`, `secrets`, `inputs`                           | None                                                    |
| `jobs.<job_id>.container.env.<env_id>`             | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `secrets`, `inputs`          | None                                                    |
| `jobs.<job_id>.container.image`                    | `github`, `needs`, `strategy`, `matrix`, `vars`, `inputs`                                             | None                                                    |
| `jobs.<job_id>.continue-on-error`                  | `github`, `needs`, `strategy`, `vars`, `matrix`, `inputs`                                             | None                                                    |
| `jobs.<job_id>.defaults.run`                       | `github`, `needs`, `strategy`, `matrix`, `env`, `vars`, `inputs`                                      | None                                                    |
| `jobs.<job_id>.env`                                | `github`, `needs`, `strategy`, `matrix`, `vars`, `secrets`, `inputs`                                  | None                                                    |
| `jobs.<job_id>.environment`                        | `github`, `needs`, `strategy`, `matrix`, `vars`, `inputs`                                             | None                                                    |
| `jobs.<job_id>.environment.url`                    | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `steps`, `inputs`            | None                                                    |
| `jobs.<job_id>.if`                                 | `github`, `needs`, `vars`, `inputs`                                                                   | `always`, `canceled`, `success`, `failure`              |
| `jobs.<job_id>.name`                               | `github`, `needs`, `strategy`, `matrix`, `vars`, `inputs`                                             | None                                                    |
| `jobs.<job_id>.outputs.<output_id>`                | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `secrets`, `steps`, `inputs` | None                                                    |
| `jobs.<job_id>.runs-on`                            | `github`, `needs`, `strategy`, `matrix`, `vars`, `inputs`                                             | None                                                    |
| `jobs.<job_id>.secrets.<secrets_id>`               | `github`, `needs`, `strategy`, `matrix`, `secrets`, `inputs`, `vars`                                  | None                                                    |
| `jobs.<job_id>.services`                           | `github`, `needs`, `strategy`, `matrix`, `vars`, `inputs`                                             | None                                                    |
| `jobs.<job_id>.services.<service_id>.credentials`  | `github`, `needs`, `strategy`, `matrix`, `env`, `vars`, `secrets`, `inputs`                           | None                                                    |
| `jobs.<job_id>.services.<service_id>.env.<env_id>` | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `secrets`, `inputs`          | None                                                    |
| `jobs.<job_id>.steps.continue-on-error`            | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `secrets`, `steps`, `inputs` | `hashFiles`                                             |
| `jobs.<job_id>.steps.env`                          | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `secrets`, `steps`, `inputs` | `hashFiles`                                             |
| `jobs.<job_id>.steps.if`                           | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `steps`, `inputs`            | `always`, `canceled`, `success`, `failure`, `hashFiles` |
| `jobs.<job_id>.steps.name`                         | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `secrets`, `steps`, `inputs` | `hashFiles`                                             |
| `jobs.<job_id>.steps.run`                          | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `secrets`, `steps`, `inputs` | `hashFiles`                                             |
| `jobs.<job_id>.steps.timeout-minutes`              | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `secrets`, `steps`, `inputs` | `hashFiles`                                             |
| `jobs.<job_id>.steps.with`                         | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `secrets`, `steps`, `inputs` | `hashFiles`                                             |
| `jobs.<job_id>.steps.working-directory`            | `github`, `needs`, `strategy`, `matrix`, `job`, `runner`, `env`, `vars`, `secrets`, `steps`, `inputs` | `hashFiles`                                             |
| `jobs.<job_id>.strategy`                           | `github`, `needs`, `vars`, `inputs`                                                                   | None                                                    |
| `jobs.<job_id>.timeout-minutes`                    | `github`, `needs`, `strategy`, `matrix`, `vars`, `inputs`                                             | None                                                    |
| `jobs.<job_id>.with.<with_id>`                     | `github`, `needs`, `strategy`, `matrix`, `inputs`, `vars`                                             | None                                                    |
| `on.workflow_call.inputs.<inputs_id>.default`      | `github`, `inputs`, `vars`                                                                            | None                                                    |
| `on.workflow_call.outputs.<output_id>.value`       | `github`, `jobs`, `vars`, `inputs`                                                                    | None                                                    |

---

## 🔧 Using Custom Environment Variables

You can define your own environment variables at different scopes within your workflow.

### Scoping Custom Variables

- **Workflow Scope:** Defined at the top level with `env`, available to all jobs.
- **Job Scope:** Defined under a job with `jobs.<job_id>.env`, available to all steps in that job.
- **Step Scope:** Defined under a step with `jobs.<job_id>.steps[*].env`, available only to that specific step.

```yaml
name: Greeting on variable day
on: workflow_dispatch

# 1. Workflow-scoped variable
env:
  DAY_OF_WEEK: Monday

jobs:
  greeting_job:
    runs-on: ubuntu-latest
    # 2. Job-scoped variable
    env:
      Greeting: Hello
    steps:
      - name: "Say Hello Mona it's Monday"
        # 3. Step-scoped variable
        env:
          First_Name: Mona
        run: echo "$Greeting $First_Name. Today is $DAY_OF_WEEK!"
```

### Passing Variables Between Steps
A step cannot directly access an environment variable set by a previous step. To pass data between steps within the same job, you must write the variable to the special `GITHUB_ENV` environment file.

```yaml
steps:
  - name: Set the value in Step 1
    id: step_one
    run: |
      echo "action_state=yellow" >> "$GITHUB_ENV"

  - name: Use the value in Step 2
    id: step_two
    run: |
      # The 'action_state' variable is now available in this step and all subsequent steps
      printf '%s\n' "$action_state" # This will output 'yellow'
```

> [!info] The step that creates or updates the variable via `GITHUB_ENV` does not have access to the new value, but all subsequent steps in the same job do.

---

## 🛡️ Environment Protections for Deployments

**Environments** are used to describe a general deployment target like `production`, `staging`, or `development`. They are a powerful feature for adding security and control to your deployment process.

### Configuring an Environment

1.  In your repository, go to **Settings**.
2.  On the left pane, select **Environments**.
3.  Select **New environment** to create and configure one.

### Environment Protection Rules

These rules must pass before a job referencing the environment is sent to a runner.
-   **Required reviewers:** Require a specific person or team to manually approve a deployment job. You can also prevent self-reviews, forcing a second person to approve a deployment.
-   **Wait timer:** Delay a job for a specific amount of time (1 to 43,200 minutes) after it's triggered.
-   **Deployment branches and tags:** Restrict which branches or tags can deploy to the environment (e.g., only allow branches matching `releases/*`).
-   **Custom deployment protection rules:** Integrate with external partner services (via GitHub Apps) to gate deployments based on checks from observability, change management, or code quality systems.

> [!note] Plan Availability
> For GitHub Free, Pro, or Team plans, most environment protection rules are only available for public repositories. Branch and tag protection rules are available for private repositories on Pro and Team plans.

### Using an Environment in a Workflow

A job referencing an environment gains access to that environment's secrets and is subject to its protection rules.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: ./deploy-to-prod.sh
```

---

## 📜 Using Custom Scripts in Your Workflow

The `run` keyword can execute single commands, multi-line scripts, or even script files stored in your repository.

- **Multi-line Script:** Use a pipe (`|`) for a multi-line `run` step.

  ```yaml
  - name: Install and test
    run: |
      npm install -g bats
      bats -v
  ```

- **Script File:** Store your script in your repository (e.g., in `.github/scripts/`) and execute it.

  ```yaml
  - name: Run my custom build script
    run: ./.github/scripts/build.sh
    shell: bash # Optionally specify which shell to use
  ```