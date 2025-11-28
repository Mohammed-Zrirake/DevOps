#CI-CD #DevOps #GitHub #Automation #Workflow

>  GitHub Actions is an automation platform built directly into GitHub. It uses YAML files called **workflows** to define a series of **jobs** and **steps**. These workflows run automatically in response to events (like a `git push`), using pre-built or custom scripts called **actions** to automate tasks like building, testing, and deploying your code.

---

## 😫 The Problem: The Manual Development Grind

After code is written, a series of repetitive, error-prone tasks must be completed before it can be deployed.

> [!danger] The Manual Task List
> - Ensure code passes unit tests.
> - Perform code quality and compliance checks.
> - Scan code and dependencies for security vulnerabilities.
> - Build and integrate code from multiple contributors.
> - Run integration tests.
> - Version the new build.
> - Deliver binaries to a storage location.
> - Deploy the application to servers.
> - Report any failures to the correct team.

Performing these tasks manually is slow, inconsistent, and unsustainable. This is the perfect job for automation.

## ✨ The Solution: GitHub Actions

> [!info] Definition
> **GitHub Actions** are packaged scripts used to automate tasks within a software development lifecycle. You configure them within a **workflow** to trigger complex processes in response to events in your GitHub repository.

By automating these tasks, GitHub Actions dramatically decreases the time from idea to deployment.

---

## 🏛️ The Anatomy of GitHub Actions

### 1. What is a Workflow?
A **workflow** is the highest-level automated process. It is defined by a YAML file in the `.github/workflows/` directory of your repository. A workflow contains one or more jobs and is triggered by specific events.

#### Workflow Anatomy (`.yml` file)
- **`name`:** The name of the workflow, displayed on the Actions tab in GitHub.
- **`on` (The Trigger):** Specifies the event(s) that will trigger the workflow. This is a critical component.
  - **Single Event:** `on: push`
  - **Array of Events:** `on: [push, pull_request]`
  - **Configuration Map:** For more granular control (e.g., only run for pushes to the `main` branch).

   ```yaml
    on:
      push:
        branches: [ main ]
      pull_request:
        branches: [ main ]
    ```
- **`jobs`:** A workflow must have at least one job. Jobs run in parallel by default but can be configured to run sequentially.

#### Job Anatomy
- **A Job** is a set of steps that execute on a specific **runner**.
- **`runs-on`:** Specifies the type of machine to run the job on. `ubuntu-latest` is the most common for GitHub-hosted runners.
- **`steps`:** A sequence of tasks to be executed. Steps can run commands or use pre-packaged actions.

### 2. What is an Action?
An **action** is a reusable, packaged script that performs a specific task. It's the fundamental building block of a workflow.

#### Types of Actions
- **JavaScript Actions:** Can run on Linux, macOS, and Windows runners. The environment must be specified in the workflow.
- **Container Actions:** The environment is bundled with the code in a [[Container]]. Can only be run on Linux runners.
- **Composite Actions:** Allow you to bundle multiple workflow steps into a single, reusable action.

#### Action Anatomy (`action.yml` file)
This file defines the metadata for a custom action.
```yaml
name: "Hello Actions"
description: "Greet someone"
author: "octocat@github.com"

inputs:
  MY_NAME:
    description: "Who to greet"
    required: true
    default: "World"

runs:
  uses: "docker"
  image: "Dockerfile"

branding:
  icon: "mic"
  color: "purple"
```

---

## 🧩 Putting It All Together: An Example Workflow

Here's how these concepts fit together in a `main.yml` file.

```yaml
# .github/workflows/main.yml

# 1. WORKFLOW name
name: A simple CI workflow

# 2. TRIGGER for the workflow
on: push

# 3. JOBS to run
jobs:
  # The name of the first job
  build:
    name: Build and Test
    # 4. RUNNER for the job
    runs-on: ubuntu-latest
    # 5. STEPS to execute in the job
    steps:
      # Step 1: Use a pre-built ACTION to check out the code
      - name: Check out repository code
        uses: actions/checkout@v4

      # Step 2: Use another pre-built ACTION to set up Node.js
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      # Step 3: Run shell commands
      - name: Install dependencies and build
        run: |
          npm install
          npm run build
```

---

## 🔎 Finding and Using Actions

- **GitHub Marketplace:** The primary place to discover and share actions. Look for the "verified creator" checkmark.
- **Open Source Repositories:** Many actions are available in public GitHub repos (e.g., the official `actions` organization).
- **Write Your Own:** You can create custom actions specific to your project's needs.

> [!security] Best Practice: Pin Actions to a Commit SHA
> When using a public action, it's more secure to reference a specific commit SHA instead of a mutable tag (like `@v4`). This guarantees your workflow always uses the exact same version of the action's code.
>
> **Good:** `uses: actions/checkout@v4`
> **Better (More Secure):** `uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11`

---

## 🖥️ Runners: Where Workflows Execute

A **runner** is a server that runs your workflow jobs.

### GitHub-hosted Runners
- **What they are:** Virtual machines managed by GitHub. They provide a clean, fresh environment for every job.
- **Types:** `ubuntu-latest`, `windows-latest`, `macos-latest`.
- **Pros:** Quick, simple, no maintenance required.
- **Larger Runners:** GitHub also offers more powerful hosted runners (e.g., `ubuntu-latest-16core`) with more CPU, RAM, and disk space for resource-intensive jobs, available at an additional cost.

### Self-hosted Runners
- **What they are:** Your own machines (on-premises or in the cloud) that you register with GitHub to run jobs.
- **Pros:** Highly configurable, access to local network resources, custom hardware, choice of any OS, potentially lower cost for frequent, long-running jobs.
- **Cons:** You are responsible for maintaining and securing them.

---

> [!info] Usage Limits
> GitHub Actions has usage limits that vary based on your plan (Free, Pro, Team, Enterprise). Limits typically apply to job execution time and storage for artifacts. See the official documentation for current details.