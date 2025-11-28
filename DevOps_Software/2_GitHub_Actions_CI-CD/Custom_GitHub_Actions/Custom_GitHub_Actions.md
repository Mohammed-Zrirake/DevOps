#CI-CD #GitHub #Automation #Actions #CoreConcept #Development #Debugging

>  [[GitHub Actions]] are individual, reusable tasks that form the building blocks of a workflow. You can create your own custom actions to encapsulate complex logic, interact with your repository, or set up specific tooling. There are three types: **Docker container**, **JavaScript**, and **Composite**.

---

## 🏛️ The Three Types of Custom Actions

### 1. 🐳 Docker Container Actions
- **Concept:** Packages the action's code and its entire environment into a [[Docker Image]].
- **Pros:**
  - **Consistent & Reliable:** The action runs in a predictable environment because all dependencies are bundled in the container.
  - **Customizable Environment:** You can use any operating system, tools, or language that can be containerized.
- **Cons:**
  - **Slower:** The job must build or pull the container image before running, which adds overhead compared to JavaScript actions.
- **How to Build:**
  1.  Create a `Dockerfile` to define the image.
  2.  Create an `action.yml` metadata file. Set `runs.using` to `docker` and `runs.image` to `Dockerfile`.
  3.  Create an `entrypoint.sh` script that will be executed when the container starts.
  4.  Commit the `action.yml`, `entrypoint.sh`, `Dockerfile`, and `README.md` to your repository.

### 2. 📜 JavaScript Actions
- **Concept:** Runs JavaScript code directly on the runner machine. The action code is separate from the execution environment.
- **Pros:**
  - **Fast:** Executes faster than container actions because there's no container build/pull step.
  - **Cross-Platform:** Can run on Linux, Windows, and macOS runners.
- **Cons:**
  - **Dependencies:** The code's dependencies (e.g., Node.js packages) must be checked into the repository (`node_modules`) or bundled into a single file.
- **How to Build:**
  1.  Prerequisites: Download and install Node.js (which includes `npm`). The [GitHub Actions Toolkit for Node.js](https://github.com/actions/toolkit) is highly recommended.
  2.  Create an `action.yml` metadata file to define inputs, outputs, and how to run the action (`runs.using: 'node16'`, `runs.main: 'index.js'`).
  3.  Create an `index.js` file containing the action's logic.
  4.  Commit the `action.yml`, `index.js`, `node_modules`, `package.json`, `package-lock.json`, and `README.md`.

### 3. 🧩 Composite Actions
- **Concept:** A powerful way to reuse a sequence of workflow steps by bundling them into a single, reusable action. You can mix multiple shell languages and other actions.
- **Pros:**
  - **Simple Reusability:** Perfect for turning a common sequence of `run` steps into a single, reusable unit.
  - **Reduces Duplication:** Centralizes logic, making workflows cleaner and easier to maintain.
  - **Modular:** Combines multiple commands or other actions into a single logical block.
- **How to Build:**
  1.  Define all the steps in a single `action.yml` file.
  2.  Set `runs.using` to `composite`.
  3.  The `steps` key contains the sequence of steps to execute.

---

## 🧩 Building a Composite Action: A Deep Dive

Composite actions are a fantastic way to simplify complex workflows without the overhead of JavaScript or Docker.

### 1. Directory Structure
A composite action must live in its own directory within the repository.
```
.github/
└── actions/
    └── my-composite-action/
        ├── action.yml
        └── scripts/
            └── my-script.sh (optional)
```

### 2. Define the `action.yml`
This file is the heart of the composite action.
```yaml
# .github/actions/my-composite-action/action.yml
name: "My Composite Action"
description: "A reusable composite action that checks out code and sets up Node.js"

inputs:
  node-version:
    description: "The Node.js version to use"
    required: true

runs:
  using: "composite"
  steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
```
- **`using: "composite"`:** This is the key that identifies it as a composite action.
- **`steps:`:** Contains a normal list of workflow steps.
- **`inputs:`:** You can define inputs that are passed from the calling workflow, accessed via the `inputs` context.

### 3. Use the Composite Action in a Workflow
You can reference the action using a relative path from the root of the repository.
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Use my composite action
        uses: ./.github/actions/my-composite-action
        with:
          node-version: '18'
```
> [!tip] To use a composite action from another repository, use the full `owner/repo/path@ref` syntax.

### 4. Adding Outputs to a Composite Action
Composite actions can define `outputs` to pass data back to the calling workflow.

**`action.yml` with an output:**
```yaml
# .github/actions/my-composite-action/action.yml
name: "Composite Action with Output"
description: "Runs a script and returns a result."

outputs:
  script-result:
    description: "Result from the script"
    value: ${{ steps.run-script.outputs.result }}

runs:
  using: "composite"
  steps:
    - id: run-script
      run: echo "result=Success" >> $GITHUB_OUTPUT
      shell: bash
```
- **`outputs:`:** Defines the output named `script-result`.
- **`value:`:** Maps the output's value to the output of a specific step (`run-script`) within the action.
- **`$GITHUB_OUTPUT`:** The `run` step sets an output value by writing to the `$GITHUB_OUTPUT` environment file.

**Using the output in the calling workflow:**
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run composite action and get output
        id: my-action # The step needs an ID to reference its outputs
        uses: ./.github/actions/my-composite-action

      - name: Display result
        run: echo "The script result was: ${{ steps.my-action.outputs.script-result }}"
```

---

## 📜 Building a JavaScript Action: CLI Setup Example

Let's create a practical JavaScript action that installs a custom CLI tool on a runner.

### Step 1: Set Up the Action Directory
Create a directory for your action: `.github/actions/my-cli-action/`.

### Step 2: Define `action.yml`
This file defines the action's metadata, inputs, and how it runs.
```yaml
# .github/actions/my-cli-action/action.yml
name: "Setup MyCLI"
description: "Installs MyCLI and adds it to the PATH"
author: "Your Name"

inputs:
  version:
    description: "The CLI version to install"
    required: false
    default: "latest"

runs:
  using: "node16" # Specifies the Node.js runtime
  main: "index.js"  # Specifies the entrypoint file
```

### Step 3: Create the `index.js` Script
This is the logic for installing the CLI. It uses the `@actions/core` toolkit package.
```javascript
// .github/actions/my-cli-action/index.js
const core = require('@actions/core');
const { execSync } = require('child_process');

async function run() {
  try {
    const version = core.getInput('version');
    
    console.log(`Installing MyCLI version: ${version}...`);
    // Example installation command
    execSync(`curl -fsSL https://cli.example.com/install.sh | sh`, { stdio: 'inherit' });

    console.log("MyCLI installed successfully.");
  } catch (error) {
    core.setFailed(`Installation failed: ${error.message}`);
  }
}

run();
```

### Step 4: Test the Action in a Workflow
Create a test workflow file to use your new local action.
```yaml
# .github/workflows/test-cli-action.yml
name: Test MyCLI Setup

on:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install MyCLI using our custom action
        uses: ./.github/actions/my-cli-action
        with:
          version: '1.2.3'

      - name: Verify CLI Installation
        run: mycli --version
```

> [!tip] **Optimization: Caching**
> To speed up subsequent runs, use the `actions/cache` action to cache the installed CLI tool.
> ```yaml
> - name: Cache MyCLI
>   uses: actions/cache@v4
>   with:
>     path: ~/.mycli # Path where the CLI is installed
>     key: mycli-${{ runner.os }}-${{ inputs.version }}
> ```

---

## 🐞 Troubleshooting and Debugging Custom Actions

### Common Issues
| Problem | Possible Cause | Suggested Fix |
|---|---|---|
| Action fails to start | `ENTRYPOINT`/`CMD` misconfigured (Docker). Syntax/runtime error (JS). | Confirm `Dockerfile` `ENTRYPOINT`. Use `console.log()` in `index.js`. |
| Inputs are undefined | Incorrect input name or not passed in `with:`. | Verify `action.yml` input names and the calling workflow's `with:` block. |
| File not found | Incorrect relative path. | Use absolute paths (`$GITHUB_WORKSPACE/path`) or `path.resolve()`. |
| Permission denied | Script lacks execute permission (`+x`). | In `Dockerfile`, add `RUN chmod +x <script>`. |
| Cache not restored | Wrong key or path values. | Double-check `path` and `key` for the `actions/cache` step. |

### Enabling Debug Logging
To get more verbose logs, set repository secrets:
- `ACTIONS_STEP_DEBUG` set to `true` for detailed step diagnostics.
- `ACTIONS_RUNNER_DEBUG` set to `true` for runner-level diagnostics.

### Testing Locally with `act`
The `act` CLI tool lets you run your GitHub Actions workflows locally, providing a much faster feedback loop for debugging.
```bash
act -j your-test-job
```

### Best Practices for Debugging
| Practice | Description |
|---|---|
| **Use `core.debug()` (JS)** | Hide verbose logs unless debugging is enabled. |
| **Validate `action.yml`** | Ensure inputs, outputs, and `runs` configuration are correct. |
| **Bundle code (JS)** | Use a tool like `@vercel/ncc` to compile your JavaScript and `node_modules` into a single file. This prevents "module not found" errors. |
| **Use `try/catch` (JS)** | Catch exceptions and use `core.setFailed()` to provide clear error messages and fail the workflow gracefully. |
| **Generous Logging (Docker)** | Use `echo` statements in your `entrypoint.sh` to trace execution flow and variable values. |

---

### Understanding the Docker Action Lifecycle
1.  **Workflow Trigger:** An event starts the workflow.
2.  **Runner Setup:** GitHub provisions a runner VM.
3.  **Action Resolution:** GitHub sees `runs.using: docker`.
4.  **Image Build/Pull:** GitHub builds the `Dockerfile` or pulls the specified image.
5.  **Container Execution:** The runner launches the container, mounting the workspace and injecting environment variables (inputs become `INPUT_<NAME>`).
6.  **Entrypoint Runs:** The `ENTRYPOINT` from the `Dockerfile` is executed.
7.  **Result Handling:** The runner captures outputs and the container is discarded.