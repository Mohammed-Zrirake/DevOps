#DevOps #CI-CD #Jenkins #Advanced #Plugins #Development #Java #Maven

>  Creating a custom [[Jenkins Plugins|Jenkins plugin]] is an advanced task that requires significant Java and Maven knowledge. It involves setting up a development toolkit (Java, Maven), generating a plugin skeleton using a Maven archetype, writing Java code to implement the desired functionality (like a custom build step), packaging it into an `.hpi` file, and deploying it to Jenkins.

---

## ⚠️ A Word of Caution: Should You Build a Plugin?

Before diving into custom plugin development, it's crucial to consider the alternatives.

> [!danger] Instructor's Advice: Try NOT to Build a Jenkins Plugin
> 1.  **Vast Ecosystem:** There are over 1800+ community plugins available. Before writing your own, thoroughly search the [Jenkins Plugin site](https://plugins.jenkins.io) to see if an existing plugin already meets your needs.
> 2.  **Significant Effort:** Writing a plugin from scratch is not an easy task. It requires a strong, hands-on understanding of Java programming, as Jenkins itself is a Java application.
> 3.  **Complexity:** While simple plugins (like printing a message) are achievable, creating complex functionality (like a custom Docker Hub publisher) requires deep knowledge of Jenkins' extension points and Java APIs.

Creating a custom plugin should be considered a last resort when no existing solution fits your specific, unique use case.

---

## 🏛️ The Three Pillars of Plugin Development

The process of creating a plugin can be broken down into three main phases.

1.  **Development Tools:** Setting up your local environment with the necessary tools to write, compile, and build the plugin code.
2.  **Jenkins Integration:** Writing the Java code that hooks into Jenkins' "extension points" to add new functionality (e.g., a new build step, a post-build action, a new SCM option). This also includes creating the Web UI components.
3.  **Plugin Publishing:** Packaging your plugin into an `.hpi` file (Jenkins Plugin file) and deploying it to a Jenkins instance for testing and use.

---

## 🛠️ Part 1: Setting Up the Jenkins Development Toolkit

This section details the manual setup of the required tools on a Jenkins server (or any development machine).

### 1. Verify Java Installation
Jenkins requires Java to run, so it should already be installed.
```bash
java -version
```
(The instructor's machine had OpenJDK version 11).

### 2. Install Apache Maven
Maven is the build tool used for compiling and packaging Jenkins plugins.

#### Step 2a: Download Maven
1.  Go to the [Apache Maven download page](https://maven.apache.org/download.cgi).
2.  Find the latest binary `.tar.gz` archive.
3.  Copy the link address and download it to your server using `wget`.
    ```bash
    wget https://dlcdn.apache.org/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.tar.gz
    ```

#### Step 2b: Extract and Move Maven
1.  Extract the archive:
    ```bash
    tar -xvzf apache-maven-3.9.6-bin.tar.gz
    ```
2.  Move the extracted directory to a standard location, like `/opt`.
    ```bash
    sudo mv apache-maven-3.9.6 /opt/
    ```

#### Step 2c: Configure Environment Variables
The system needs to know where to find the Maven executable (`mvn`).
1.  Open your shell's profile file (e.g., `~/.profile` for Ubuntu).
    ```bash
    vi ~/.profile
    ```
2.  Add the following lines to the end of the file, adjusting the path to your Maven installation directory.
    ```bash
    export M2_HOME=/opt/apache-maven-3.9.6
    export PATH=$M2_HOME/bin:$PATH
    ```
3.  Apply the changes to your current terminal session. (Alternatively, log out and log back in).
    ```bash
    source ~/.profile
    ```

#### Step 2d: Verify Maven Installation
```bash
mvn -version
```
You should now see the installed Apache Maven version and the Java version it's using.

---

## (Skeleton): Generating a Jenkins Plugin Skeleton

With the toolkit ready, you can now use a Maven Archetype (a project template) to generate the basic file structure for a new plugin.

### 1. The Maven Archetype Command
This command tells Maven to generate a new project based on the Jenkins plugin archetype.
```bash
mvn -U archetype:generate -Dfilter=io.jenkins.archetypes:
```

### 2. Choosing a Template
After downloading necessary metadata, Maven will present a list of available Jenkins plugin templates:
1.  **`empty-plugin`:** A minimal skeleton with a `pom.xml` and an empty source tree.
2.  **`global-configuration-plugin`:** A skeleton for a plugin that adds a new global configuration page in "Manage Jenkins".
3.  **`global-shared-library`:** A template for testing a Jenkins Pipeline Global Shared Library.
4.  **`hello-world-plugin`:** **(The one we will use)** A skeleton that includes a complete example of a custom **Build Step**.
5.  **`scripted-pipeline`:** A template for testing scripted pipeline logic.

### 3. Generating the "Hello World" Plugin
1.  Run the `archetype:generate` command from above.
2.  When prompted to "Choose a number," enter `4` for the `hello-world-plugin`.
3.  When prompted for the archetype version, choose the latest available (e.g., `21`).
4.  **Define artifact details:**
    -   `groupId`: (Leave default or specify your own).
    -   `artifactId`: This is your plugin's name. Enter `hello-world`.
    -   `version`: (Leave default `1.0-SNAPSHOT`).
    -   `package`: (Leave default).
5.  Confirm with `Y`.

Maven will generate a new directory named `hello-world` containing the complete plugin project structure.

### 4. Anatomy of the Plugin Skeleton
The generated project has a specific structure:

-   `pom.xml`: The "heart of the Maven." This XML file defines the project's dependencies, build process, and metadata.
-   `Jenkinsfile`: A sample `Jenkinsfile` for building the plugin itself via a Jenkins pipeline.
-   `src/main/java/.../HelloWorldBuilder.java`: The core Java source code for the plugin. This class `extends Builder` and implements `SimpleBuildStep`, defining the logic for our new build step.
-   `src/main/resources/.../config.jelly`: The UI definition file. This Jelly (a Java-based XML templating language) file defines the HTML form elements that appear on the job configuration page (e.g., the text box for our "name" parameter).
-   `src/test/java/.../HelloWorldBuilderTest.java`: The Java source code for the unit tests for our plugin.

> [!warning] This structure highlights the complexity. You need to understand Java (for logic), Maven (for building), and Jelly (for UI) to be effective.

---

## 📦 Building and Deploying the Custom Plugin

### 1. Verify and Package the Plugin
1.  Navigate into the generated `hello-world` directory.
2.  **(Optional) Verify the code:** Run `mvn verify`. This command downloads dependencies, compiles the code, and runs the unit tests to ensure everything is correct. This can take several minutes on the first run.
3.  **Package the plugin:** Run `mvn package`. This command does everything `verify` does, and then packages the final plugin into a Jenkins-installable format.

The output will show the tests running and, upon success, will create a file in the `target/` directory named `hello-world.hpi`. **The `.hpi` file is your Jenkins plugin.**

### 2. Deploy the Plugin to Jenkins
1.  Copy the generated `.hpi` file into your Jenkins server's plugins directory.
    ```bash
    sudo mv target/hello-world.hpi /var/lib/jenkins/plugins/
    ```
2.  **Restart the Jenkins service** for it to discover and load the new plugin.
    ```bash
    sudo systemctl restart jenkins
    ```

### 3. Use the Custom Plugin in a Job
1.  Log in to your Jenkins dashboard.
2.  Create a new **Freestyle project**.
3.  Go to the **Build Steps** section and click **Add build step**.
4.  You will now see a new option: **"Say hello world"**. This is the custom build step defined by your plugin!
5.  Select it. You will see a text box for the **Name** parameter, which was defined in the `config.jelly` file.
6.  Enter a message, like `This is a custom message from my custom plugin`.
7.  Save the job and click **Build Now**.
8.  Check the **Console Output**. You will see the message you entered, prefixed with "Hello, ".

**Result:** You have successfully created, built, deployed, and used a custom Jenkins plugin that adds a new build step to the Jenkins UI. You can now also find your plugin listed in **Manage Jenkins > Plugins > Installed plugins**.

> [!info] Extending to Other Step Types
> The `HelloWorldBuilder.java` class `extends Builder`. To create other types of plugin functionality, you would extend different base classes provided by the Jenkins API, such as `Publisher` for a post-build action. The fundamental process of generating, coding, and packaging remains the same.