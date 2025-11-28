#DevOps #CI-CD #Jenkins #Architecture #CoreConcept #DistributedBuilds #JenkinsAgent #JenkinsSlave #HandsOn #Tutorial

>  Jenkins uses a **master-slave** (also known as master-agent) architecture to handle heavy workloads and support multi-platform builds. This is called **Distributed Builds**. The **master** node schedules and dispatches jobs, while multiple **slave** nodes perform the actual build executions. This allows Jenkins to scale, run jobs in parallel, and build on different operating systems simultaneously.

---

## 😫 The Problem with a Single Jenkins Node

Up to this point, we have been using a single Jenkins node that acts as both the master (for management) and the slave (for execution).

> [!danger] The Single-Server Bottleneck
> In a large organization with hundreds or thousands of jobs running multiple times a day, a single server cannot handle the load. It will "start melting down," leading to slow or failed builds and an unresponsive UI. It also cannot support building for multiple operating systems (e.g., Linux and Windows) from a single machine.

## ✨ The Solution: Distributed Builds

Jenkins solves this problem with its built-in capability for **distributed builds**, which is based on a master-slave architecture.

-   **Concept:** You can have one master node and connect hundreds of slave nodes to it.
-   **Master:** The machine where the main Jenkins application is installed. Its job is to manage the system and orchestrate the builds.
-   **Slaves:** Remote machines that are connected to the master. Their only job is to execute the builds assigned to them.

### Key Benefits of the Master-Slave Architecture
-   **Scalability:** Handle a massive number of builds by adding more slave nodes.
-   **Multi-Platform Testing:** You can have slaves with different operating systems (Linux, Windows, macOS). This allows you to build and test your application across all target environments in parallel from a single Jenkins instance.
-   **Centralized Reporting:** Even though builds run on different slaves, all the results, logs, and artifacts are sent back to the master for centralized monitoring and reporting.

---

## 🏛️ The Roles Explained

### The Jenkins Master
The main Jenkins server where the application is installed. It connects to slaves via a TCP connection.
-   **Primary Tasks:**
    1.  **Scheduling Build Jobs:** The master is responsible for deciding which jobs need to run and when.
    2.  **Dispatching Builds to Slaves:** It sends the scheduled jobs to an available slave for execution. If a slave has a capacity of 20 parallel jobs and you send 50, the master will queue the remaining 30 and dispatch them as slots become free.
    3.  **Monitoring Slaves:** It keeps track of the health and status of all connected slaves.
    4.  **Recording and Presenting Build Results:** All console logs, test results, and artifacts from the slaves are collected by the master and displayed in the web UI.
-   The master can also execute jobs directly (as we have been doing), but in a distributed setup, this is generally discouraged to keep the master responsive.

### The Jenkins Slave
A Jenkins slave is a Java executable that runs on a remote machine.
-   **Primary Characteristics:**
    1.  **Hears Requests from the Master:** The slave's job is to do as it's told by the master.
    2.  **Runs on a Variety of OS:** Slaves can be any OS (Linux, Windows, macOS), independent of the master's OS.
    3.  **Executes Build Jobs:** Its sole purpose is to execute the build jobs dispatched by the master.
    4.  **Can be Targeted:** You can configure a specific job to *always* run on a particular slave machine or a group of slaves. This is achieved using **labels**.

---

## 🛠️ Hands-On: Creating and Configuring a Jenkins Slave

This guide follows the instructor's detailed, step-by-step process for setting up a new Linux slave node and connecting it to the Jenkins master.

### Step 1: Create the Slave Machine
1.  **Provision a New Server:** Create a new virtual machine (the instructor uses a new Ubuntu Droplet on DigitalOcean). This will be our slave node.
2.  **Initial Login:** SSH into the new slave machine as `root` and set a new password.

### Step 2: Configure Passwordless SSH from Master to Slave
The Jenkins master will start the slave agent process on the slave machine via SSH. For this to be automated, it must be passwordless.

1.  **Switch to `jenkins` user on Master:** On the master machine's terminal, switch to the `jenkins` user. This is the user that the Jenkins service runs as and the one that will initiate the SSH connection.
    ```bash
    # On the MASTER machine
    sudo -iu jenkins
    ```
2.  **Generate SSH Keys on Master:** If they don't already exist, generate an SSH key pair for the `jenkins` user.
    ```bash
    # On the MASTER machine, as the 'jenkins' user
    ssh-keygen -t rsa
    ```
    Accept the default file locations by pressing Enter. This creates `~/.ssh/id_rsa` (private key) and `~/.ssh/id_rsa.pub` (public key).
3.  **Create `.ssh` directory on Slave:** From the master, create the `.ssh` directory on the slave machine for the `root` user.
    ```bash
    # On the MASTER machine, as the 'jenkins' user
    ssh root@<slave_ip> 'mkdir -p .ssh'
    ```
    You will be prompted for the slave machine's `root` password.
4.  **Copy Public Key to Slave:** Append the master's public key to the `authorized_keys` file on the slave. This is the step that authorizes the master to log in.
    ```bash
    # On the MASTER machine, as the 'jenkins' user
    cat ~/.ssh/id_rsa.pub | ssh root@<slave_ip> 'cat >> ~/.ssh/authorized_keys'
    ```
    You will again be prompted for the slave's password.
5.  **Verify Passwordless Login:** Test the connection.
    ```bash
    # On the MASTER machine, as the 'jenkins' user
    ssh root@<slave_ip>
    ```
    You should now be logged into the slave machine's shell without being asked for a password.

### Step 3: Prepare the Slave Environment
1.  **Install Java:** The slave agent is a Java program, so Java must be installed on the slave machine.
    ```bash
    # On the SLAVE machine
    sudo add-apt-repository ppa:webupd8team/java # To add a repository for older Java versions if needed
    sudo apt-get update
    sudo apt-get install openjdk-8-jdk # The instructor installs Java 8
    java -version # Verify installation
    ```
2.  **(Optional but shown) Download the slave agent manually:** While Jenkins can push this automatically, the instructor shows how to download it. This is not the primary method used in the final configuration.
    ```bash
    # On the SLAVE machine
    mkdir -p /root/bin
    cd /root/bin
    wget http://<master_ip>:8080/jnlpJars/slave.jar
    ```

### Step 4: Configure the Node in the Jenkins UI
1.  In the Jenkins UI, go to **Manage Jenkins > Nodes**.
2.  Click **New Node**.
3.  **Node Name:** Enter a name (e.g., `Jenkins-slave-one`).
4.  Select **Permanent Agent** and click **Create**.
5.  **Configuration Page:**
    -   **`# of executors`:** This defines how many concurrent builds this slave can run. A good rule of thumb is 2 executors per CPU core. The instructor uses `2` for a 1-core machine.
    -   **`Remote root directory`:** A directory on the slave where Jenkins will store workspaces and other data (e.g., `/home/jenkins`).
    -   **`Labels`:** This is a crucial field for targeting jobs. Add a space-separated list of labels (e.g., `linux build-server`).
    -   **`Usage`:** Set to `Use this node as much as possible` (initially).
    -   **`Launch method`:** This is the most important setting. Select **Launch agents via execution of command on the master**.
        > [!warning] Plugin May Be Required
        > In newer Jenkins versions, this launch method is moved to the **Command Launcher** plugin. If you don't see this option, you must install that plugin first.
    -   **`Command`:** Enter the command that the master will run to start the agent on the slave.
        ```bash
        ssh root@<slave_ip> "java -jar /root/bin/slave.jar"
        ```
        (Note: The instructor downloads the `slave.jar` manually. In many SSH setups, Jenkins can push this file automatically, and you might only need `ssh root@<slave_ip>`).
6.  Click **Save**. Jenkins will attempt to connect to the slave. After a few moments, the red 'X' next to the node name should disappear, indicating a successful connection.

---

## 🚀 Using the Distributed Build System

### Concurrent Builds
The instructor creates three simple Freestyle jobs (`concurrent-job-1`, `2`, `3`) with a `sleep` command to simulate a long-running task.
-   When all three jobs are run simultaneously, Jenkins automatically distributes them across all available executors on both the master and the slave. For example, two jobs might run on the slave, and one might run on the master.

### Targeting Jobs to Specific Slaves with Labels
This is the power feature of a distributed setup.
1.  **Configure the Slave:** In the slave node's configuration, change the **`Usage`** setting to **`Only build jobs with label expressions matching this node`**. This tells Jenkins not to send any random job to this slave.
2.  **Configure the Job:** Open the configuration page for each of the three concurrent jobs.
    -   Find the **`Restrict where this project can be run`** option.
    -   Check the box.
    -   In the **`Label Expression`** field, enter the label you assigned to your slave (e.g., `linux`).
    -   Save the job.
3.  **Run the Jobs:** Trigger all three jobs again.
4.  **Observe the Behavior:**
    -   The Jenkins slave has only **two** executors.
    -   Jenkins immediately schedules `concurrent-job-1` and `concurrent-job-2` to run on the slave.
    -   `concurrent-job-3` is now in a **pending** state.
    -   If you inspect the pending job, the message will say: **"pending—Waiting for next available executor on Jenkins-slave-one"**.
    -   As soon as one of the first two jobs finishes, freeing up an executor, Jenkins immediately starts `concurrent-job-3` on the slave.

**Conclusion:** By using labels, you gain precise control over your build infrastructure, ensuring that specific jobs run on the correct machines with the required environment, and allowing Jenkins to efficiently manage the queue for a large number of concurrent builds.