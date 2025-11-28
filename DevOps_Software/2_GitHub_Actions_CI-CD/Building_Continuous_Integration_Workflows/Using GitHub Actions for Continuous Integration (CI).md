#CI-CD #GitHub #Automation #Workflow #BestPractice

>  [[GitHub Actions]] is a powerful tool for implementing Continuous Integration (CI). You can start quickly with pre-built templates, test your code against multiple platforms using a build [[#Testing Against Multiple Targets with a Build Matrix|matrix]], and reduce duplication across your projects by creating [[#Avoiding Duplication with Reusable Workflows|reusable workflows]].

**Continuous Integration (CI)** is the practice of automating the build and testing of code every time a developer commits a change to version control. This helps teams discover and fix issues early.

---

## 🚀 Creating a Workflow from a Template

The easiest way to get started with CI is to use a template. GitHub provides pre-configured workflows for many common languages and frameworks.

1.  In your GitHub repository, go to the **Actions** tab.
2.  Select **New workflow**.
3.  Search for a relevant template, like `Node.js` or `Python package`.
4.  Select **Configure** to add the template's YAML file to your repository.

### Example: The Default Node.js Template
This template provides a solid starting point for a typical Node.js project.

```yaml
name: Node.js CI

# Run on pushes and pull requests to the main branch
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    # Define a build matrix to test against multiple Node.js versions
    strategy:
      matrix:
        node-version: [14.x, 16.x, 18.x]

    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: npm run build --if-present
    - run: npm test
```
This single `build` job checks out the code, sets up Node.js, installs dependencies, builds the project, and runs tests. It does this three times in parallel, once for each Node version specified in the `matrix`.

---

## ♻️ Avoiding Duplication with Reusable Workflows

As projects scale, you'll find the same CI steps (linting, testing, deploying) repeated across many repositories. Reusable workflows solve this by allowing you to define a common process once and call it from other workflows.

> [!info] Definition
> A **reusable workflow** is a special workflow that can be called by other workflows, much like a function in programming. It centralizes and standardizes automation logic.

### Why Use Reusable Workflows?
-   **Consistency:** Enforce the same CI standards across all projects.
-   **Efficiency:** Drastically reduce code duplication.
-   **Easy Updates:** Update a process (e.g., add a new security scan) in one place, and all calling workflows inherit the change automatically.
-   **Scalability:** Ideal for platform and DevOps teams managing many services.

### How to Implement Reusable Workflows
1.  **Create the Reusable Workflow:** In a central repository, create a workflow file (e.g., `ci-standard.yml`).
2.  **Enable it for Calling:** Add the `workflow_call` event trigger to this file. This is what makes it "reusable".
    ```yaml
    # In the reusable workflow file (e.g., .github/workflows/reusable-ci.yml)
    on:
      workflow_call:
        # Define any inputs the workflow expects
        inputs:
          node-version:
            required: true
            type: string
        # Define any secrets the workflow needs
        secrets:
          NPM_TOKEN:
            required: true
    ```
3.  **Call it from Another Workflow:** In your main (caller) workflow, use the `uses:` key under a job to reference the reusable workflow.
    ```yaml
    # In the main application's workflow file
    jobs:
      call-reusable-workflow:
        uses: your-org/your-central-repo/.github/workflows/reusable-ci.yml@v1
        with:
          node-version: '18'
        secrets:
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
    ```

---

## 🛠️ Customizing Workflows for Your Needs

The default templates are a great start, but you'll often need to customize them.

### 1. Testing Against Multiple Targets with a Build Matrix
The `strategy.matrix` key allows you to define a set of configurations. GitHub will then create a job for each possible combination. This is perfect for testing against different OS versions, language versions, or other variables.

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node-version: [16.x, 18.x]
```
This matrix will generate **four parallel jobs**:
1.  `ubuntu-latest` with Node `16.x`
2.  `ubuntu-latest` with Node `18.x`
3.  `windows-latest` with Node `16.x`
4.  `windows-latest` with Node `18.x`

### 2. Separating Build and Test Jobs
For clarity and better organization of logs, it's a good practice to separate building and testing into distinct jobs. This also opens the door for more complex workflows, like saving build artifacts in one job and using them in another.

```yaml
jobs:
  # The first job just builds the application
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18.x'
      - run: npm ci
      - run: npm run build

  # The second job runs the tests, and can have its own matrix
  test:
    # This job needs the 'build' job to complete successfully first
    needs: build
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [16.x, 18.x]
    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```