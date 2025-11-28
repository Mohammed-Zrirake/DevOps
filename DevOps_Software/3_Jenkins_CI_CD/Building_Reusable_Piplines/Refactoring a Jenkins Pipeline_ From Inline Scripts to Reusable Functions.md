#DevOps #CI-CD #Jenkins #PipelineAsCode #DeclarativePipeline #Jenkinsfile #HandsOn #Refactoring #Tutorial

>  This is a deep dive into refactoring a [[The Jenkins Declarative Pipeline|Declarative Pipeline]] to make it cleaner, more maintainable, and more powerful. Key techniques include adding [[#Step 2 Dynamically Versioning Builds|parameters]] to differentiate between release and integration builds, using [[#Step 3 Conditionally Publishing Artifacts and Cleaning the Workspace|conditionals]] to control stages, and abstracting complex logic into reusable [[#Step 4 Simplifying with Pipeline Functions|Groovy functions]] within the `Jenkinsfile`.

---

This guide follows a series of hands-on demos that progressively refactor a `Jenkinsfile` for a Java/Maven application, introducing advanced concepts and best practices along the way. All code is located in the `reusable-pipeline/refactoring-pipeline/` directory of the `Jenkins_Upgradev3` repository.

## Step 1: Building a Basic Application Pipeline (`jenkinsfile1`)

The starting point is a simple pipeline to build a Java application (`java-tomcat-sample`) using Maven.

### The Initial Problem: Missing Dependencies
-   **The `Jenkinsfile`:** A basic pipeline with stages for `Audit Tools`, `Unit Test`, and `Build`. The `Audit Tools` stage is a crucial first step that verifies the necessary build tools (Git, Java, Maven) are installed on the runner.
    ```groovy
    // jenkinsfile1 (simplified)
    pipeline {
        agent any
        stages {
            stage('Audit Tools') {
                steps {
                    sh 'git --version'
                    sh 'java -version'
                    sh 'mvn -version'
                }
            }
            stage('Unit Test') { steps { sh 'mvn test' } }
            stage('Build') { steps { sh 'mvn clean package' } }
        }
    }
    ```
-   **The Failure:** The first run of the job **fails**. The console output shows `mvn: not found`.
-   **The Lesson:** This highlights the importance of the `Audit Tools` stage. It immediately tells you that the Jenkins node executing the job is missing a required dependency (Maven). The solution is to manually install Maven on the runner (`sudo apt install maven`).
-   **The Second Run:** After installing Maven, the job is run again. It succeeds, but it is slow on the first run because Maven has to download all of its own plugins and project dependencies. Subsequent runs are much faster as these dependencies are cached locally on the runner.

### Viewing the Results
-   After a successful build, the console output shows that a `.war` file was created.
-   You can navigate to the job's **Workspace** in the Jenkins UI to find the generated artifact (e.g., inside the `java-tomcat-sample/target/` directory).

---

## Step 2: Dynamically Versioning Builds (`jenkinsfile2`)

A key requirement for any real-world CI pipeline is to manage build versions. This demo introduces a method to version the application artifact differently for integration builds vs. release builds.

### The `jenkinsfile2`
```groovy
pipeline {
    agent any
    environment {
        VERSION = '1.1.1' // Example integration version
        RELEASE_VERSION = '2.1.0' // Example release version
    }
    stages {
        // ... (Audit and Test stages) ...
        stage('Build') {
            steps {
                echo "Building with Version: ${env.VERSION}"
                // Use the 'dir' step to change directory for a block of commands
                dir('java-tomcat-sample') {
                    // Use Maven Versions Plugin to set the project version dynamically
                    sh "mvn versions:set -DnewVersion=${env.VERSION}"
                    sh "mvn clean package"
                }
            }
        }
    }
}
```

### Key Concepts Introduced
-   **`dir` Step:** Instead of chaining `cd` commands, the `dir` step provides a cleaner way to execute a block of commands within a specific directory. This is the recommended approach in pipelines.
-   **Dynamic Versioning with Maven:** The `mvn versions:set -DnewVersion=...` command uses the Maven Versions Plugin to dynamically change the `<version>` tag in the `pom.xml` file before building. We pass the version defined in our `environment` block to this command.

### The Experiment
1.  The `VERSION` in the `environment` block is set to `1.1.1`.
2.  The job is run. The console output shows Maven updating the project version to `1.1.1`. The final artifact in the workspace is now `java-tomcat-sample-1.1.1-SNAPSHOT.war`.
3.  The `VERSION` is changed to `2.1.0` in the `Jenkinsfile`, and the code is pushed.
4.  The job is run again. The build artifact is now correctly versioned as `java-tomcat-sample-2.1.0-SNAPSHOT.war`.

**The Lesson:** The build process is now driven by the configuration in the `Jenkinsfile`. The application's version is no longer hardcoded in the `pom.xml` but is controlled by the pipeline, which is a core tenet of Pipeline as Code.

---

## Step 3: Conditional Execution and Artifact Management (`jenkinsfile3`)

This demo introduces more advanced logic: asking the user if a build is a "release" build and then behaving differently based on their input.

### The `jenkinsfile3`
```groovy
pipeline {
    agent any
    // 1. 'parameters' block to ask for user input
    parameters {
        booleanParam(name: 'RELEASE', defaultValue: false, description: 'Is this a Release version?')
    }
    environment {
        RELEASE_VERSION = '1.1.0'
        INT_VERSION = 'R2'
    }
    stages {
        // ... (Audit and Test stages) ...
        stage('Build') {
            steps {
                script {
                    // 2. Complex logic to determine the version suffix
                    def versionSuffix = ""
                    if (params.RELEASE == false) {
                        versionSuffix = "${env.INT_VERSION}ci:${env.BUILD_NUMBER}"
                    } else {
                        versionSuffix = "${env.RELEASE_VERSION}.${env.BUILD_NUMBER}"
                    }
                    env.VERSION_SUFFIX = versionSuffix
                }
                echo "Building with suffix: ${env.VERSION_SUFFIX}"
                dir('java-tomcat-sample') {
                    sh "mvn versions:set -DnewVersion=${env.VERSION_SUFFIX}"
                    sh "mvn clean package"
                }
            }
        }
        stage('Publish') {
            // 3. Conditional stage execution
            when {
                expression { params.RELEASE == true }
            }
            steps {
                // 4. Archiving the build artifact
                archiveArtifacts artifacts: 'java-tomcat-sample/target/*.war'
            }
        }
    }
    post {
        // 5. Post-build cleanup step
        always {
            cleanWs()
        }
    }
}
```

### Key Concepts Introduced
1.  **`parameters` Block:** Defines parameters that Jenkins will ask the user for when a build is triggered manually. After the first run, the "Build Now" button changes to **"Build with Parameters"**.
2.  **`script` Block:** Allows you to run more complex, imperative Groovy script within a Declarative Pipeline. Here, it's used to implement `if/else` logic.
3.  **`when` Directive:** A powerful way to conditionally execute a `stage`. The `Publish` stage will be completely **skipped** if the condition is not met.
4.  **`archiveArtifacts` Step:** A built-in step that saves specified files from the workspace and attaches them to the build record. This is how you persist your final build artifacts.
5.  **`post { always { cleanWs() } }` Block:** The `post` section defines actions to run at the end of the pipeline. `always` ensures the step runs regardless of success or failure. `cleanWs()` is a step (provided by the **Workspace Cleanup** plugin) that deletes the job's workspace.

### The Experiment and Debugging
1.  **Run with `RELEASE=false` (Default):**
    -   The `VERSION_SUFFIX` becomes something like `R2ci:6`.
    -   The build artifact is versioned `R2ci6-SNAPSHOT.war`.
    -   The `Publish` stage is **skipped** due to the `when` condition.
    -   The workspace is cleaned up by `cleanWs()`.
2.  **Run with `RELEASE=true`:**
    -   The `VERSION_SUFFIX` becomes `1.1.0.7`.
    -   The build artifact is versioned `1.1.0.7-SNAPSHOT.war`.
    -   The `Publish` stage **runs**, and the `.war` file is archived. A "Last successful artifacts" link now appears on the job's main page.
3.  **Failure and Debugging (`cleanWs()`):** The instructor demonstrates that the `cleanWs()` step fails initially. The reason is that the **Workspace Cleanup** plugin is not installed. This reinforces the lesson that many useful steps in Jenkins are provided by plugins that must be installed first.

---

## Step 4: Simplifying with Pipeline Functions (`jenkinsfile4`)

The final refactoring step addresses the complexity that has crept into the `Build` stage. The `if/else` logic in the `script` block is messy and hard to read. The solution is to abstract this logic into reusable Groovy functions.

### The `jenkinsfile4` (Refactored)
```groovy
// Define all functions outside the 'pipeline' block
def auditTools() {
    sh 'git --version'
    sh 'java -version'
    sh 'mvn -version'
}

String getBuildVersion() {
    if (params.RELEASE) { // Simplified boolean check
        return "${env.RELEASE_VERSION}.${env.BUILD_NUMBER}"
    } else {
        return "${env.INT_VERSION}ci:${env.BUILD_NUMBER}"
    }
}

def packageApplication(String version) {
    dir('java-tomcat-sample') {
        sh "mvn versions:set -DnewVersion=${version}"
        sh "mvn clean package"
    }
}

pipeline {
    agent any
    parameters { booleanParam(...) }
    environment { ... }
    stages {
        stage('Audit Tools') {
            steps {
                // Call the function
                auditTools()
            }
        }
        stage('Unit Test') { ... }
        stage('Build') {
            steps {
                script {
                    // Call the function and store the result
                    def versionSuffix = getBuildVersion()
                    echo "Building with suffix: ${versionSuffix}"
                    // Call another function, passing the result as an argument
                    packageApplication(versionSuffix)
                }
            }
        }
        // ... (Publish and post stages) ...
    }
}
```

### The Benefit: Clean, Reusable, and Readable Code
-   **Clean Pipeline:** The `stages` block is now incredibly clean and easy to read. It describes *what* the pipeline is doing, not *how* it's doing it.
-   **Abstracted Logic:** All the complex, messy implementation details are hidden away in well-named functions.
-   **Reusability:** A function like `auditTools()` could be called in multiple stages or even shared across pipelines (leading to the concept of Shared Libraries).
-   **Maintainability:** If the logic for getting the build version needs to change, you only have to edit the `getBuildVersion()` function in one place.

The final execution of this job produces the exact same result as `jenkinsfile3`, but the underlying `Jenkinsfile` is now far more mature, maintainable, and readable, which is the ultimate goal of refactoring.