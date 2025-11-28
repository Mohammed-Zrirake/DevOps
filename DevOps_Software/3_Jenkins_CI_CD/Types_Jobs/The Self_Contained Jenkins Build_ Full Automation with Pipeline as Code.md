#DevOps #CI-CD #Jenkins #PipelineAsCode #Groovy #Automation #HandsOn #Tutorial

>  This is the ultimate goal of modern Jenkins: a **self-contained build process**. This approach automates everything—plugin installation, OS-level software setup, and the build/test/publish pipeline itself—through Jenkins jobs. The result is a portable, version-controlled, and fully reproducible CI/CD environment that can be set up on any new Jenkins server with minimal manual effort.

---

## 😫 The Problem: The Fragility of Manual Setups

In previous demonstrations, we built a complete CI/CD pipeline using a [[Hands-On A Standard Jenkins Build with Manual Plugins|manual, Freestyle approach]]. While it worked, it was deeply flawed:

-   **Manual Plugin Installation:** Required a human to log in to the UI, search for plugins, and install them. Migrating to a new server meant maintaining a separate checklist of required plugins.
-   **Silent Server Dependencies:** The pipeline depended on tools (`.NET SDK`, `Docker`) being manually installed on the Jenkins server's operating system. This configuration was not tracked in version control and made moving the pipeline to a new node a difficult, error-prone process.
-   **UI-Based Job Configuration:** The entire build logic was trapped in the Jenkins UI, stored in a `config.xml` file. It was not version-controlled, hard to review, and difficult to manage at scale.

## ✨ The Solution: The Self-Contained Build

The "self-contained" philosophy aims to automate the entire setup and execution process, with Jenkins itself driving the automation. Everything is defined as code.

This guide covers the three core automation steps to achieve this:
1.  **Automated Plugin Installation:** Using a Groovy script to install all required plugins and their dependencies in one go.
2.  **Automated Node/Server Setup:** Using a [[The Jenkins Job#🆚 Freestyle vs Pipeline A Detailed Comparison|Pipeline Job]] to install and configure OS-level software (`.NET`, `Docker`) on the execution node.
3.  **Automated Build Pipeline:** Using a `Jenkinsfile` from SCM to define the entire CI/CD workflow (Fetch, Build, Test, Publish) as code.

---

## Part 1: Automating Plugin Installation with a Groovy Script

The first challenge on a fresh Jenkins server is installing the necessary plugins. Instead of manual UI clicks, we can use a Groovy script executed in the Jenkins Script Console.

### The Cleanup (Simulating a Fresh Install)
To demonstrate the process, the instructor first simulates a fresh server by deleting all existing plugins:
1.  SSH into the Jenkins server.
2.  Navigate to the plugins directory: `cd /var/lib/jenkins/plugins`.
3.  Delete all plugins: `sudo rm -rf *`.
4.  Restart the Jenkins service: `sudo systemctl restart jenkins`.
5.  After restarting, the Jenkins UI shows no plugins are installed. Existing jobs are broken because the plugins they depend on (like the Git plugin) are gone.

### The Groovy Script (`install_plugins.groovy`)
This script automates the installation of a list of plugins and all their transitive dependencies.

```groovy
// Import necessary Jenkins classes
import jenkins.model.Jenkins
import hudson.model.UpdateCenter

// Get instances of the Jenkins Plugin Manager and Update Center
def jenkins = Jenkins.get()
def pluginManager = jenkins.getPluginManager()
def updateCenter = jenkins.getUpdateCenter()

// 1. Force a refresh of the plugin catalog from the update center
updateCenter.updateAllSites()

// 2. Define the list of plugins you want to install by their short ID
def plugins = [
    "github", 
    "mstest", 
    "workflow-aggregator", // This is the ID for the core Pipeline plugin
    "docker-build-publish"
]

// 3. Iterate through the list and install each plugin with its dependencies
plugins.each { pluginName ->
    println "--> Installing plugin: ${pluginName}"
    def plugin = updateCenter.getPlugin(pluginName)
    if (plugin) {
        // The 'true' flag ensures all dependencies (and their dependencies) are also deployed
        plugin.deploy(true)
    } else {
        println "--> !! Plugin not found in update center: ${pluginName}"
    }
}

// 4. (Optional but recommended) Restart Jenkins after installation is complete
// This ensures all plugins are loaded correctly.
// jenkins.safeRestart()
```

> [!tip] **How to find a plugin's ID?**
> Go to the official Jenkins Plugin site (`https://plugins.jenkins.io`), search for your plugin (e.g., "GitHub"), open its page, and find the **Plugin ID** in the information section. This is the ID you use in the script.

### Executing the Script
1.  In the Jenkins UI, go to **Manage Jenkins > Script Console**.
2.  Copy and paste the entire Groovy script into the console.
3.  Click **Run**.

The script will begin downloading and installing all specified plugins and their dependencies. This process can take several minutes. Once complete (and after the restart), all plugins will be installed and ready, and your broken jobs will be restored.

**Benefit:** This script is now a version-controlled, repeatable way to provision any Jenkins server with the exact set of plugins required.

---

## Part 2: Automating Node/Server Setup with a Pipeline Job

The next challenge is automating the installation of OS-level software like the `.NET SDK` and `Docker`. This is crucial for environments with multiple, dynamically created slave nodes.

### The Pipeline Job (`SetUp_Node`)
Instead of SSHing into each machine, we create a **Pipeline Job** in Jenkins to run the setup commands.

1.  **Create the Job:** Go to **New Item**, name it `SetUp_Node`, and select the **Pipeline** job type.
2.  **The Pipeline Script:** In the job configuration, under the `Pipeline` section, select `Pipeline script` and paste the following `Jenkinsfile` code.

```groovy
pipeline {
    agent any
    stages {
        stage('Install .NET 7.0 SDK') {
            steps {
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y dotnet-sdk-7.0
                    dotnet --version
                '''
            }
        }
        stage('Install Docker Engine') {
            steps {
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y ca-certificates curl gnupg
                    # ... (full Docker installation script for Ubuntu) ...
                    sudo service docker start
                    sudo systemctl enable docker.service
                    sudo docker --version
                '''
            }
        }
        stage('Configure Docker Permissions for Jenkins user') {
            steps {
                sh '''
                    sudo usermod -aG docker jenkins
                    sudo systemctl restart docker
                '''
            }
        }
    }
}
```

> [!warning] OS-Specific Commands
> The shell commands (`sh`) in this script are specific to an Ubuntu/Debian-based OS. If your Jenkins nodes are Windows, CentOS, etc., you would need to replace these commands with the equivalent commands for that OS (e.g., using `yum` instead of `apt-get`). The pipeline *structure* remains the same.

3.  **Run the Job:** Save and run the `SetUp_Node` job. It will execute each stage sequentially, installing and configuring all the necessary software on the node where it runs.

**Benefit:** Whenever you spin up a new Jenkins slave node, the first thing you do is run this job on it. This automates the entire node provisioning process, ensuring a consistent and correctly configured environment for your application builds.

---

## Part 3: The Fully Automated Build Pipeline (The `Jenkinsfile`)

With plugins and server dependencies automated, the final step is to automate the application build pipeline itself. We do this by moving all the build logic from a Freestyle job's UI into a `Jenkinsfile`.

### Supporting Shell Scripts

First, the build, test, and push logic is extracted from the Freestyle job's `Execute shell` steps into reusable shell scripts, which are checked into the Git repository (e.g., in a `ci/` folder).

-   **`ci/build.sh`**: Contains the `dotnet build` command.
-   **`ci/unit-test.sh`**: Contains the `dotnet test` commands.
-   **`ci/push.sh`**: Contains the logic to build the Docker image, log in to Docker Hub, and push the image. This script is parameterized to accept a build number as an argument for tagging.

### The `Jenkinsfile`
This file defines the entire multi-stage CI/CD pipeline as code. It lives in the root of the application's Git repository.

```groovy
pipeline {
    agent any // Run on any available agent

    stages {
        stage('Verify Shell Environment') {
            steps {
                echo "Verifying node setup..."
                sh 'docker --version'
                sh 'dotnet --version'
            }
        }
        stage('Checkout Source Code') {
            steps {
                // Jenkins' built-in Git step
                git branch: 'master', 
                    url: 'https://github.com/anshulc55/Jenkins_Upgradev3.git',
                    changelog: true,
                    poll: true
            }
        }
        stage('Build Application') {
            steps {
                sh 'chmod +x ./ci/build.sh'
                sh './ci/build.sh'
            }
        }
        stage('Run Unit Tests') {
            steps {
                sh 'chmod +x ./ci/unit-test.sh'
                sh './ci/unit-test.sh'
            }
        }
        stage('Push Docker Image') {
            steps {
                // Use the Credentials Binding plugin to securely access secrets
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'chmod +x ./ci/push.sh'
                    // Pass the Jenkins BUILD_NUMBER as an argument to the script for tagging
                    sh "./ci/push.sh ${env.BUILD_NUMBER}"
                }
            }
        }
    }
}
```
### Creating and Running the Pipeline Job
1.  **Create the Job:** Go to **New Item**, name it `application-build-workflow`, and select **Pipeline**.
2.  **Configure:** In the job configuration page, scroll down to the **Pipeline** section.
    -   Change the **Definition** dropdown to **Pipeline script from SCM**.
    -   **SCM:** Select `Git`.
    -   **Repository URL:** Enter the URL of your application's repository.
    -   **Script Path:** Enter the path to your `Jenkinsfile` within the repository (e.g., `jenkins-plugin-model/Jenkinsfile`).
3.  **Save and Run:** Save the job and click **Build Now**.

**The Execution Flow:**
1.  Jenkins checks out the specified repository.
2.  It finds the `Jenkinsfile` at the specified path.
3.  It then executes the stages defined in the `Jenkinsfile` sequentially.
4.  The `Push Docker Image` stage securely accesses the 'dockerhub-creds' from the Jenkins credential store.
5.  It executes the `push.sh` script, passing the current build number (`${env.BUILD_NUMBER}`) as an argument.
6.  The script builds the Docker image and tags it with the build number (e.g., `anshuldevops/jenkins-demo:3`).
7.  The script logs in to Docker Hub and pushes the newly tagged image.

**Benefit:** The entire CI/CD process is now defined as code, living alongside the application. It is version-controlled, reusable, and fully automated. Any new Jenkins server can run this pipeline simply by creating a job that points to this repository's `Jenkinsfile`.

---

## ✅ Conclusion: The Power of the Self-Contained Build

By automating these three key areas—plugin installation, node setup, and the build pipeline itself—we have created a **fully-fledged, self-contained, and portable CI/CD system**. This "as code" approach is the cornerstone of modern DevOps practices, providing a level of automation, consistency, and reliability that is impossible to achieve with a manual, UI-driven process.