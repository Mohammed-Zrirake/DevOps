#DevOps #CI-CD #Jenkins #Docker #JenkinsAgent #JenkinsSlave #Architecture #HandsOn #Tutorial

> This is the modern evolution of the [[Jenkins Distributed Builds|master-slave architecture]]. Instead of maintaining permanent, static slave nodes, you configure Jenkins to **dynamically provision ephemeral build agents as Docker containers**. When a job starts, Jenkins automatically spins up a Docker container to act as a temporary agent, runs the build *inside* that container, and then terminates the container once the job is complete. This is incredibly resource-efficient, cost-effective, and provides a perfectly clean build environment every time.

---

## 😫 The Problem with Static Infrastructure

The traditional Jenkins architecture involves a master node and one or more static slave nodes (physical or virtual machines). As we've seen, this has significant drawbacks:

-   **Resource Inefficiency:** Static slaves are "always on," consuming resources and incurring costs even when no builds are running.
-   **Maintenance Hell:** You are responsible for manually installing, configuring, and maintaining the required software (Java, Maven, Docker, .NET, etc.) on every single slave node.
-   **Scalability Issues:** You have to pre-provision a fixed number of servers, which might be too few during peak load or too many during quiet periods.

## ✨ The Solution: On-Demand Docker Agents

The solution is to treat build agents not as pets to be cared for, but as cattle to be created and destroyed on demand. By integrating Docker with Jenkins, we can achieve this.

> [!success] The Docker Agent Workflow
> 1.  A developer triggers a Jenkins job.
> 2.  Jenkins sees that the job requires a specific agent type (e.g., a "maven" agent).
> 3.  It communicates with the Docker daemon on a host machine.
> 4.  It automatically spins up a **new Docker container** from a pre-defined image (e.g., the official `maven:latest` image).
> 5.  This container acts as a temporary Jenkins agent for this one build.
> 6.  The job's steps are executed *inside* this container.
> 7.  Once the job completes, Jenkins **automatically terminates and removes the container**.

This process is extremely beneficial, saving significant infrastructure cost and maintenance effort.

---

## 🛠️ Hands-On: Setting Up a Jenkins Docker Cloud

This guide follows the instructor's detailed, step-by-step process for setting up a new Jenkins instance (running in Docker) and configuring it to use other Docker containers as dynamic build agents.

### Step 1: Prepare the Host Machine
This is the machine that will run both the Jenkins master container and the ephemeral agent containers.

1.  **Provision a New Server:** Create a new Linux server (the instructor uses a CentOS droplet).
2.  **Install Docker Engine:** Install Docker on this host machine by following the official documentation. Verify the installation and ensure the Docker service is running.
    ```bash
    sudo systemctl status docker
    ```

### Step 2: Run the Jenkins Master in a Docker Container
We will run the Jenkins master itself as a container, but with a special configuration to allow it to control the host's Docker daemon.

1.  **The `docker run` Command (with Detailed Explanation):**
    ```bash
    docker run \
      -d \
      -u root \
      --privileged=true \
      --name jenkins-centos \
      -p 8080:8080 \
      -p 50000:50000 \
      -v /root/jenkins_home:/var/jenkins_home \
      -v /var/run/docker.sock:/var/run/docker.sock \
      jenkins/jenkins:lts
    ```
    -   `-d`: Run in detached (background) mode.
    -   `-u root`: Run the processes inside the Jenkins container as the `root` user. This helps avoid certain permission issues.
    -   `--privileged=true`: Grants the container extended privileges on the host. This is a powerful but necessary setting for this use case.
    -   `--name jenkins-centos`: Assigns a name to the container.
    -   `-p 8080:8080`: Maps port 8080 on the host to port 8080 in the container for the Jenkins UI.
    -   `-p 50000:50000`: Maps port 50000, which is used for communication between Jenkins agents and the master.
    -   `-v /root/jenkins_home:/var/jenkins_home`: **Crucial for persistence.** This [[Bind Mounts|bind mounts]] a directory on the host (`/root/jenkins_home`) to the Jenkins home directory inside the container. All Jenkins configuration, jobs, and data will be saved on the host, surviving container restarts.
    -   `-v /var/run/docker.sock:/var/run/docker.sock`: **This is the magic.** This bind mounts the Docker daemon's Unix socket from the host machine into the Jenkins container. This allows the Jenkins container to send commands to the host's Docker daemon, enabling it to start and stop other ("sibling") containers.
    -   `jenkins/jenkins:lts`: The Docker image to use for the Jenkins master.

2.  **Initial Jenkins Setup:** Access the Jenkins UI, get the initial admin password from the mounted volume (`cat /root/jenkins_home/secrets/initialAdminPassword`), install the suggested plugins, and create an admin user.

### Step 3: Configure the Docker Cloud in Jenkins

This is where you tell Jenkins how to create dynamic Docker agents.

#### 3a: Install the Docker Plugin
Go to **Manage Jenkins > Plugins > Available plugins** and install the **Docker** plugin. This plugin and its dependencies are essential. Restart Jenkins after installation.

#### 3b: The First (Failed) Cloud Configuration
1.  Go to **Manage Jenkins > Clouds**.
2.  Click **Add a new cloud** and select **Docker**.
3.  **`Name`:** Give your cloud a name (e.g., `Docker-General`).
4.  **`Docker Host URI`:** For a Linux host, this is the path to the Docker socket.
    ```
    unix:///var/run/docker.sock
    ```
5.  **Test Connection:** Click **Test Connection**. It will **FAIL**.
6.  **Diagnosis:** The instructor shows that you need to examine the logs of the Jenkins container (`docker logs jenkins-centos`). Exceptions and errors there point to permission issues.
7.  **The Fix (Host Machine Configuration):**
    -   On the host machine, you need to set the correct permissions on the Docker socket.
        ```bash
        # On the HOST machine
        sudo chmod 666 /var/run/docker.sock
        ```
    -   It's also a good practice to disable the firewall and ensure the `jenkins` user (if it exists on the host) is in the `docker` group.
    -   After making these changes, **restart the Jenkins container** with the full, corrected `docker run` command from Step 2.

#### 3c: The Second (Successful) Cloud Configuration
1.  After restarting the container, go back to **Manage Jenkins > Clouds** and re-configure the Docker cloud.
2.  **Test Connection:** Click **Test Connection** again. This time, it will succeed, showing the host's Docker Engine version.

#### 3d: Configure the Docker Agent Template
Now, you need to define a "template" for the agents that this cloud can create.

1.  Click **Add Docker Template**.
2.  **`Labels`:** This is the most important field. Give the template a unique label (e.g., `maven-agent`). This is how jobs will request an agent of this type.
3.  **`Name`:** A descriptive name for the template.
4.  **`Enabled`:** Check this box.
5.  **`Docker Image`:** Specify the Docker image to use for this agent template. For a Maven build, we'll use the official `maven:latest` image from Docker Hub.
6.  **`Instance Capacity`:** The maximum number of concurrent containers of this type that Jenkins can spin up.
7.  **`User`:** In the container settings, specify the user to run as. The instructor uses `root` to avoid permission issues inside the container.
8.  Click **Save**.

### Step 4: Using the Docker Agent in a Job
1.  Create a new **Freestyle project** (e.g., `Build-Maven-Application`).
2.  Configure SCM to pull a Maven-based Java project.
3.  Add `Execute shell` build steps to check the environment and build the application:
    ```bash
    # Step 1: Audit Tools
    echo "Checking runtime environment..."
    java -version
    mvn -version

    # Step 2: Run Tests
    echo "Executing Unit Tests..."
    mvn -f java-tomcat-sample/pom.xml test

    # Step 3: Build Application
    echo "Executing Application Build..."
    mvn -f java-tomcat-sample/pom.xml clean package
    ```
4.  **The Key Step:** In the job's main configuration, check the box for **`Restrict where this project can be run`**.
5.  **`Label Expression`:** Enter the label you defined in your Docker Agent Template (e.g., `maven-agent`).
6.  Click **Save**.

### Step 5: Run the Job and Observe the Magic
1.  Click **Build Now**.
2.  The job will enter a "pending" state with a message like "pending—Waiting for agent to provision."
3.  **On the Host Machine:** If you run `docker ps`, you will see a **new container** being created from the `maven:latest` image.
4.  **In Jenkins:** The job will start running. The console output will show "Building remotely on maven-agent...". The `java -version` and `mvn -version` commands will show the versions *from inside the Docker container*, not from the Jenkins master or the host.
5.  The build proceeds, downloading Maven dependencies *inside* the ephemeral container.
6.  Once the job completes successfully, the artifact is archived.
7.  **On the Host Machine:** If you run `docker ps` again a few moments later, you will see that the agent container has been **automatically terminated and removed**.

---

> [!summary] Conclusion
> By configuring a Docker Cloud, you have transformed Jenkins from a system that requires static, manually-managed infrastructure into a dynamic, on-demand CI/CD platform. You can now create multiple agent templates for different technology stacks (Maven, Node.js, Python, etc.) by simply specifying a different Docker image. This is the modern, scalable, and cost-effective way to manage Jenkins build environments, providing unparalleled flexibility and consistency.