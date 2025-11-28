#DevOps #CI-CD #Jenkins #HandsOn #Tutorial #FreestyleJob #Plugins #Docker #DotNet

>  This is a detailed, step-by-step tutorial demonstrating the **traditional** way to build a CI/CD pipeline in Jenkins. It involves manually installing plugins, creating a [[The Jenkins Job#🆚 Freestyle vs. Pipeline A Detailed Comparison|Freestyle Job]], and configuring each stage (Fetch, Build, Test, Publish) through the Jenkins UI and shell commands. This guide also highlights the challenges and limitations of this manual approach.

---

## 🏛️ The Goal: A Manual CI/CD Pipeline

The objective of this demonstration is to construct a complete pipeline for a .NET application by performing the following steps within a single Freestyle Job:
1.  **Fetch:** Clone the source code from a GitHub repository.
2.  **Build:** Compile the .NET application.
3.  **Test:** Run the automated test suites for the application.
4.  **Publish (Test Reports):** Parse and display the test results in a user-friendly format on the Jenkins UI.
5.  **Publish (Artifacts):** Build a Docker image from the application and push it to Docker Hub.

This entire process will be configured using the **"Standard"** or **"Traditional"** Jenkins method: manual plugin installation and UI-based job configuration.

---

## Setup 1: A Fresh Jenkins Installation

The demo begins with a fresh installation of Jenkins, intentionally set up without any default plugins to demonstrate the process from scratch.

1.  After the initial Jenkins startup, provide the admin password.
2.  On the "Customize Jenkins" screen, choose **Select plugins to install**.
3.  On the "Install Plugins" screen, click **None** to deselect all suggested plugins, and then click **Install**.
4.  Create the `admin` user or skip and continue as admin.

This results in a barebones Jenkins instance, ready for manual configuration.

---

## Stage 1: Fetching Code with the GitHub Plugin

The first step of any CI pipeline is to get the source code.

### The Problem Without Plugins
-   Upon creating a new Freestyle Job (`Std-Project1`), navigating to the **Source Code Management** section reveals only one option: `None`. Jenkins, out of the box, does not know how to speak to Git.

### The Solution: Manually Installing the GitHub Plugin
1.  **Find the Plugin:** Go to the official Jenkins Plugin site (`https://plugins.jenkins.io`) and search for "GitHub". This is the central repository for all plugins. The plugin page provides details like version, dependencies, and documentation.
2.  **Install in Jenkins:**
    -   In your Jenkins UI, go to **Manage Jenkins > Plugins**.
    -   Go to the **Available plugins** tab.
    -   Search for "GitHub".
    -   Select the checkbox next to the "GitHub Plugin".
    -   Click **Install**.
3.  **Dependency Hell (A First Glimpse):** Observe the installation process. Jenkins will automatically download not only the GitHub plugin but also all of its **dependent plugins**. A single plugin installation can result in 15, 50, or even 100+ total plugins being downloaded. This highlights a key challenge of manual management: upgrading or managing one plugin can have a wide-ranging impact.
4.  **Verify Installation:** After installation, go to the **Installed plugins** tab and confirm the GitHub plugin is listed.

### Configuring the Job to Use Git
1.  Go back to your Freestyle Job and click **Configure**.
2.  In the **Source Code Management** section, a new option, **Git**, is now available. Select it.
3.  **`Repository URL`**: Enter the URL of the public repository (e.g., `https://github.com/anshulc55/Jenkins_Upgradev3.git`). No credentials are needed for a public repo.
4.  **`Branch Specifier`**: Set to `master` (or whichever branch contains the code).
5.  **Advanced Clone Behaviors (Optional but Recommended):**
    -   **`Shallow clone`**: Check this and set the `depth` to `1`. This tells Git to only download the latest version of the files, not the entire commit history, which significantly speeds up the checkout process and saves disk space.
    -   **`Clean before checkout`**: Check this. It ensures that the workspace is wiped clean before a new checkout, preventing old files from interfering with the build.
6.  **Build Triggers:**
    -   Select **`Poll SCM`**.
    -   Set the schedule to `H/5 * * * *`. This tells Jenkins to check the Git repository for new commits every five minutes. If a change is detected, a new build will be triggered.
7.  **Build Steps:**
    -   Add an `Execute shell` step.
    -   Enter the command `ls -la` to list the files and verify that the checkout was successful.
8.  **Save and Build:** Click **Save** and then **Build Now**. Check the **Console Output** of the build. You should see the log of the `git clone` operation followed by the file listing, confirming the "Fetch" stage is working.

---

## Stage 2: Building the .NET Application

Now that we have the code, the next step is to compile it.

### The Problem: Missing Server-Side Dependencies
1.  **Configure:** Add a new `Execute shell` step to your job.
2.  **Command:** Add the command to build the .NET project: `dotnet build jenkins-plugin-model/src/Pi.Web/Pi.Web.csproj`.
3.  **Run and Fail:** Save and run the job. The build will **fail**.
4.  **Diagnosis:** Checking the console output reveals the error: `dotnet: command not found`. The Jenkins server itself does not have the .NET SDK installed, so it doesn't know what the `dotnet` command means.

### The Solution: Manually Installing the .NET SDK
1.  **Attempt Plugin Installation:** First, search for a ".NET" plugin in **Manage Jenkins > Plugins**. You might find a `.NET SDK support` plugin. Install it.
2.  **Run and Fail Again:** Rerun the job. It will likely fail again. This demonstrates a key lesson: **not all plugins are sufficient for your use case, and sometimes a plugin doesn't exist for what you need.**
3.  **The Real Solution (Server-Side Setup):** The only option is to manually install the dependency on the machine where Jenkins is running.
    -   SSH into the Jenkins server.
    -   Run the necessary commands to install the .NET 7.0 SDK (e.g., `sudo apt-get update`, `sudo apt-get install dotnet-sdk-7.0`).
    -   Verify the installation with `dotnet --version`.
    -   Export the necessary `PATH` variables so the system can find the `dotnet` executable.
4.  **Run and Succeed:** Go back to Jenkins and run the job again. This time, the `dotnet build` step will succeed, as Jenkins can now find and execute the command.

---

## Stage 3 & 4: Running Tests and Publishing Reports

The build is successful; now we need to run tests and visualize the results.

### The Problem: Raw Test Results are Hard to Read
1.  **Configure:** Add two more `Execute shell` steps to run the two test projects:
    ```bash
    dotnet test jenkins-plugin-model/src/Pi.Math.Test/Pi.Math.Test.csproj
    dotnet test jenkins-plugin-model/src/Pi.Runtime.Test/Pi.Runtime.Test.csproj
    ```
2.  **Run and Analyze:** Run the job. The tests will execute successfully, and you'll see a summary in the console log (e.g., "Total tests: 2. Passed: 2."). However, the detailed results are buried in `.trx` (XML) files deep within the job's workspace. This is not user-friendly.

### The Solution: Manually Installing the MSTest Plugin
1.  **Find the Plugin:** The `.trx` file format is specific to Microsoft's test runner. Search `plugins.jenkins.io` for "TRX". This leads to the **MSTest** plugin.
2.  **Install in Jenkins:** Go to **Manage Jenkins > Plugins > Available plugins**, search for "MSTest", and install it.
3.  **Configure Post-build Action:** This is a critical step. Installing a plugin often doesn't do anything by itself; you must configure your job to *use* it.
    -   Go to your job's configuration.
    -   Scroll down to **Post-build Actions**.
    -   Click **Add post-build action** and select **Publish MSTest test result report**.
    -   Leave the default path (`**/*.trx`) as is. This tells the plugin to search recursively for all `.trx` files in the workspace.
4.  **Run and See the Magic:** Save and run the job again. Now, two new things will appear on your job's page:
    -   A **Test Result** link, which takes you to a detailed, browsable report of all test cases, classes, and their durations.
    -   A **Test Result Trend** graph, which shows the history of test runs over time (pass vs. fail vs. skipped). This provides an at-a-glance view of your project's code quality.

---

## Stage 5: Publishing the Docker Artifact

The final step is to package our application as a Docker image and publish it.

### The Problem: Missing Docker and Permission Errors
1.  **Find and Install Plugin:** Search for and install the "Docker Build and Publish" plugin.
2.  **Configure Build Step:** Add a new `Docker build and publish` build step to your job.
    -   **`Repository Name`:** Set this to your Docker Hub repository (e.g., `anshuldevops/jenkins-demo-two`).
    -   **`Registry Credentials`:** Add your Docker Hub credentials (username and an access token as the password) to the Jenkins Credentials store.
    -   **`Dockerfile path`:** Provide the path to the `Dockerfile` within your checked-out code.
3.  **Run and Fail (Error 1: Docker not found):** The build fails with `Cannot run program "docker"`. The Jenkins server doesn't have Docker installed.
    -   **Solution:** SSH into the server and manually install the Docker engine.
4.  **Run and Fail (Error 2: Permission denied):** The build fails again with a `permission denied` error when trying to connect to the Docker daemon socket. The `jenkins` user, which runs the Jenkins process, does not have permission to use Docker.
    -   **Solution:** Add the `jenkins` user to the `docker` group (`sudo usermod -aG docker jenkins`) and **restart the Jenkins service** (`sudo systemctl restart jenkins`).
5.  **Run and Fail (Error 3: Authentication/Plugin Issue):** The build may fail again with a `denial: requested access to the resource is denied` error during the `docker push` step. This indicates that even though you provided credentials to the plugin, it is not successfully authenticating with Docker Hub. This can be a common plugin bug or misconfiguration.
    -   **Solution (Workaround):** Instead of relying on the plugin's built-in authentication, add a manual `Execute shell` step *before* the Docker step to perform the login. Use the **Credentials Binding** plugin (usually installed as a dependency) to securely inject the Docker Hub credentials as environment variables (`DOCKER_USER`, `DOCKER_PASS`), and then run `echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin`.

### Configuring Image Tagging
-   By default, the plugin pushes to the `latest` tag, overwriting the previous image.
-   To version your images, use the **`Tag`** field in the plugin's configuration. You can set this to a static value (like `v2`) or, more powerfully, use a Jenkins environment variable like `$BUILD_NUMBER` to tag each build uniquely.

---

## 📉 Summary of the Manual Process and Its Drawbacks

Although we successfully built a complete CI/CD pipeline, the process was fraught with challenges that highlight the limitations of the "traditional" Freestyle approach.

1.  **Silent Tool Dependencies:** The pipeline's success depends on tools (`.NET SDK`, `Docker`) being manually installed and configured on the Jenkins server. This configuration is "silent"—it's not documented anywhere in the source code or the job itself. This makes moving the job to a new Jenkins server a nightmare, as you have to remember and reproduce all these manual server-side steps.
2.  **Plugin Management Hell:** The pipeline relies on a specific set of manually installed plugins and their many dependencies. Migrating this job requires a checklist of all plugins that must be installed on the new server.
3.  **Fragile Configuration:** The entire job logic is stored in a complex `config.xml` file on the Jenkins master, mixed in with build logs and artifacts. It's not version-controlled, hard to review, and difficult to back up or restore cleanly.
4.  **Mix of UI and Shell:** The logic is a mix of UI configurations (in post-build actions) and raw shell commands (in build steps), making it hard to read and maintain.

This manual, UI-driven process is brittle and does not scale well. The upcoming sections will explore how to automate and codify this entire pipeline to solve these problems.