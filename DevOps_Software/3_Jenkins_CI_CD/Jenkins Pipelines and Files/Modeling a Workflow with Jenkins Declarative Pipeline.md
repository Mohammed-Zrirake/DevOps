#DevOps #CI-CD #Jenkins #PipelineAsCode #DeclarativePipeline #Jenkinsfile #HandsOn #Tutorial

>  This is a deep dive into the practical structure of a [[The Jenkins Declarative Pipeline|Declarative Pipeline]] (`Jenkinsfile`). It covers how to define and scope [[#Scope of Environment Variables|environment variables]], assign [[#Agent Declarations (Where to Run)|agents]] at different levels, add [[#Manual Approvals with the input Step|manual approval gates]], and execute stages in [[#Executing Stages in Parallel|parallel]].

This guide follows a series of hands-on demos that build upon each other, revealing key features and concepts of the `Jenkinsfile` syntax.

---

## 🏛️ Core `Jenkinsfile` Concepts: A Review

A `Jenkinsfile` has a specific structure:
-   `pipeline { ... }`: The root block.
-   `agent any`: Defines the default execution node for the entire pipeline.
-   `environment { ... }`: Defines global environment variables.
-   `stages { ... }`: A container for all the sequential phases of your workflow.
-   `stage('Stage Name') { ... }`: A specific phase, like 'Build' or 'Test'.
-   `steps { ... }`: The actual commands to run within a stage.

---

## Demo 1: Scoping Agents and Environment Variables

This demo explores how `agent` and `environment` blocks can be defined at different levels of the pipeline and what the scoping rules are for variables.

### The `jenkinsfile1`
```groovy
pipeline {
    agent any

    // 1. Pipeline-level (Global) Environment Variable
    environment {
        RELEASE = '20.04'
    }

    stages {
        stage('Build') {
            // 2. Stage-level Agent Declaration
            agent any

            // 3. Stage-level (Local) Environment Variable
            environment {
                LOG_LEVEL = 'info'
            }

            steps {
                echo "Building release ${env.RELEASE} with log level ${env.LOG_LEVEL}"
            }
        }
        stage('Test') {
            steps {
                // 4. Attempting to access both variables
                echo "Testing release ${env.RELEASE} with log level ${env.LOG_LEVEL}"
            }
        }
    }
}
```

### The Experiment and Key Learnings

1.  **Create the Job:** A new Pipeline job (`Modeling_Demo1`) is created, configured to use `Pipeline script from SCM` pointing to the repository and the path of `jenkinsfile1`.
2.  **Run and Fail:** The job is executed.
    -   The `Build` stage succeeds, printing: `Building release 20.04 with log level info`.
    -   The `Test` stage **FAILS**.
3.  **Diagnosis:** The console output shows the error: `no such property: LOG_LEVEL for class: groovy.lang.Binding`.
4.  **The Learning -> Scope of Environment Variables:**
    -   **Pipeline-level variables** (like `RELEASE`) are **global**. They are accessible in *all* stages of the pipeline.
    -   **Stage-level variables** (like `LOG_LEVEL`) are **local**. Their scope is limited *only* to the stage in which they are defined. The `Test` stage has no knowledge of `LOG_LEVEL`, causing the failure.

5.  **Agent Declarations (Where to Run):**
    -   The `agent` directive can be defined at the top `pipeline` level, making it the default for all stages.
    -   It can also be defined within a specific `stage`. This allows you to execute different stages on different types of agents (e.g., run a build on a Linux agent and a UI test on a Windows agent).

### Bonus: Installing the Pipeline Stage View Plugin
-   **Problem:** By default, the pipeline job's UI shows a simple log. There's no visual breakdown of the stages.
-   **Solution:** Install the **`Pipeline Stage View`** plugin (**Manage Jenkins > Plugins > Available plugins**).
-   **Result:** After installation, the job's main page displays a graphical table showing each stage, its status (success/failure), and its duration, providing a much clearer and more intuitive view of the pipeline's execution.

---

## Demo 2: Manual Approvals with the `input` Step

This demo shows how to pause a pipeline and require manual human confirmation before proceeding, a critical feature for production deployments.

### The `jenkinsfile2`
A new `Deploy` stage is added with an `input` directive. A `post` block is also introduced.
```groovy
pipeline {
    agent any
    environment { RELEASE = '20.04' }
    stages {
        stage('Build') { /* ... as before ... */ }
        stage('Test') { /* ... as before, but with LOG_LEVEL removed ... */ }

        // New 'Deploy' stage with manual approval
        stage('Deploy') {
            steps {
                // The 'input' step pauses the pipeline
                input(
                    message: "Deploy to production?",
                    ok: "Do It!", // Custom text for the proceed button
                    parameters: [
                        string(
                            name: 'TARGET_ENVIRONMENT',
                            defaultValue: 'PROD',
                            description: 'Target deployment environment'
                        )
                    ]
                )
                // This step only runs after approval
                echo "Deploying release ${env.RELEASE} to environment ${params.TARGET_ENVIRONMENT}"
            }
        }
    }
    // New 'post' block for cleanup/notification actions
    post {
        always {
            echo "This message prints whether deploy happened or not, success or failure."
        }
    }
}
```

### The Experiment and Key Learnings
1.  **Create the Job:** A new Pipeline job (`Modeling_Demo2`) is created, pointing to `jenkinsfile2`.
2.  **Run and Pause:** The job is executed.
    -   The `Build` and `Test` stages complete successfully.
    -   The `Deploy` stage **pauses**. The Stage View shows it as a flashing, waiting stage. A popup appears on the build's page asking for input.
3.  **The Input Prompt:** The UI prompt shows:
    -   The `message`: "Deploy to production?"
    -   The custom `ok` button text: "Do It!"
    -   The `parameters` defined: a text field for `TARGET_ENVIRONMENT` pre-filled with the `defaultValue` 'PROD'.
4.  **Proceeding:** The user can change the parameter (e.g., to `dev`) and click "Do It!".
    -   The `Deploy` stage resumes and completes. The console output for the step shows: `Deploying release 20.04 to environment dev`.
    -   The `post { always { ... } }` block then executes.
5.  **Aborting:** If the job is run again and the user clicks "Abort" at the input prompt:
    -   The `Deploy` stage is marked as `Aborted` (not failed).
    -   The `post { always { ... } }` block **still executes**, because `always` runs regardless of the pipeline's status.

**Key Learnings:**
-   The `input` step is a powerful way to add manual gates to a pipeline.
-   You can pass parameters from the user at the time of approval, which are then accessible via the `params` object (e.g., `params.TARGET_ENVIRONMENT`).
-   The `post` block is for defining actions that run at the end of a pipeline, with conditions like `always`, `success`, `failure`, and `aborted`.

---

## Demo 3: Executing Stages in Parallel

This demo introduces the `parallel` directive, which allows you to run multiple stages simultaneously to speed up your pipeline.

### The `jenkinsfile3`
The `Build` stage is refactored to contain a `parallel` block with nested stages.
```groovy
pipeline {
    agent none // Set a global agent of 'none' when stages define their own agents

    stages {
        stage('Build') {
            parallel {
                // These three stages will run at the same time
                stage('Build for Linux ARM64') {
                    agent any // In a real scenario, this would be 'label linux-arm'
                    steps {
                        echo "Building for Linux ARM64..."
                        // sh './build-for-arm.sh'
                    }
                }
                stage('Build for Linux AMD64') {
                    agent any // In a real scenario, this would be 'label linux-amd'
                    steps {
                        echo "Building for Linux AMD64..."
                        // sh './build-for-amd.sh'
                    }
                }
                stage('Build for Windows') {
                    agent any // In a real scenario, this would be 'label windows'
                    steps {
                        echo "Building for Windows..."
                        // bat 'build.bat'
                    }
                }
            }
        }
        stage('Test') { /* ... as before ... */ }
        stage('Deploy') { /* ... as before ... */ }
    }
}
```

### The Experiment and Key Learnings
1.  **Create the Job:** A new Pipeline job (`Modeling_Demo3`) is created, pointing to `jenkinsfile3`.
2.  **Run and Observe:** The job is executed.
    -   In the Stage View, you will see the three nested build stages (`Build for Linux ARM64`, etc.) appear and run **at the same time** under the parent `Build` stage.
    -   Once all three parallel stages complete successfully, the pipeline proceeds sequentially to the `Test` stage.

**Key Learnings:**
-   The `parallel` directive is used to run a set of stages simultaneously.
-   This is a powerful optimization technique. A common use case is to build for multiple platforms (Linux, Windows, macOS) or run different types of tests (unit, integration, linting) in parallel to reduce total pipeline execution time.
-   You can nest `stages` inside a `stage` to group related parallel tasks.

---

> [!summary] Summary of Concepts Learned
> 1.  **Environment Variables** have scope: pipeline-level variables are global, while stage-level variables are local to that stage.
> 2.  **Agents** can be defined globally or per-stage, allowing different parts of your pipeline to run on different machines.
> 3.  The **`input` step** is used to create manual approval gates and gather user parameters at runtime.
> 4.  The **`post` block** is for defining cleanup or notification actions that run at the end of a pipeline.
> 5.  The **`parallel` directive** is used to execute multiple stages concurrently, significantly speeding up workflows.