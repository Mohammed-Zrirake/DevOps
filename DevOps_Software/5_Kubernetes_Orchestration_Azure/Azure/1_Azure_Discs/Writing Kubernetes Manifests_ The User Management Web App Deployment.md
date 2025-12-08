#DevOps #Containerization #Kubernetes #CoreConcept #YAML #Deployments #initContainers #StatefulSets #HandsOn #Tutorial

>  This guide details how to write a declarative YAML manifest for our **User Management Web Application**. This [[The Kubernetes Deployment|Deployment]] will connect to the [[Writing Kubernetes Manifests The MySQL Stateful Deployment|MySQL database]] we've provisioned. We will learn how to inject database connection details using **environment variables** and, most importantly, how to use an **`initContainer`** to ensure the web application only starts *after* the database is ready to accept connections.

---

This is the final piece of our stateful application architecture. We will deploy the frontend web app and expose it with a `LoadBalancer` Service.

## 😫 The Startup Dependency Problem

When we deploy all of our application's manifests at the same time (`kubectl apply -f kube-manifests/`), there's a race condition:
-   The MySQL database pod might take some time to start up and initialize its schema.
-   The User Management Web App pod might start up much faster.
-   If the web app starts and immediately tries to connect to the database before the database is ready, it will fail to connect and crash. Kubernetes will restart it, but this can lead to a crash-loop cycle until the database is finally available.

## ✨ The Solution: `initContainers`

> [!info] Definition
> **Init Containers** are special containers that run to completion in a specific order *before* the main application containers in a [[The Kubernetes Pod|Pod]] are started.

We can leverage an `initContainer` to solve our startup dependency problem. We will create a simple `initContainer` that runs a loop, continuously trying to establish a network connection to the MySQL service. The `initContainer` will only exit (and thus allow the main web app container to start) once it has successfully connected to the database.

---

## ✍️ Writing the `user-management-webapp-deployment.yaml` Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-mgmt-webapp
  labels:
    app: user-mgmt-webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: user-mgmt-webapp
  template:
    metadata:
      labels:
        app: user-mgmt-webapp
    spec:
      # 1. 'initContainers' Block: Runs before the main 'containers' block.
      initContainers:
      - name: init-db
        image: busybox:1.31
        # This command runs a loop that waits until it can successfully connect
        # to the 'mysql' service on port 3306.
        command: ['sh', '-c', 'echo -e "Checking for the availability of MySQL Server deployment"; while ! nc -z mysql 3306; do sleep 1; printf "-"; done; echo -e "  >> MySQL DB Server has started";']
        
      # 2. Main 'containers' Block
      containers:
      - name: user-management-webapp
        image: stacksimplify/kube-usermanagement-webapp:mysql-db
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        
        # 3. Injecting database connection details as environment variables.
        env:
        - name: DB_HOSTNAME
          value: "mysql" # The DNS name of our MySQL ClusterIP service
        - name: DB_PORT
          value: "3306"
        - name: DB_NAME
          value: "webappdb" # The database schema created by our ConfigMap
        - name: DB_USERNAME
          value: "root"
        - name: DB_PASSWORD
          value: "dbpassword11" # This should come from a Secret in production
```

### A Deep Dive into the `spec.template.spec` Section

#### 1. `initContainers`
-   **What it is:** A list of container definitions, just like the main `containers` block. These containers are run sequentially. Each one must exit successfully (exit code 0) before the next one is run. Only after all `initContainers` have completed successfully will Kubernetes start the main application containers.
-   **`image: busybox:1.31`:** We use `busybox`, a tiny, lightweight image that contains many common Linux utilities, including `netcat` (`nc`).
-   **`command`:** This is the crucial part. We run a shell script that:
    1.  Enters a `while` loop.
    2.  `! nc -z mysql 3306`: The `nc` (netcat) command tries to open a connection to the host `mysql` on port `3306`. The `-z` flag tells it to scan without sending any data. The `!` negates the result.
    3.  As long as the connection fails, the loop continues to `sleep 1`.
    4.  Once the connection succeeds (meaning the MySQL service is up and the pod is ready), the loop exits, and the `initContainer` completes successfully.

#### 2. `containers` (Main Application Container)
-   **`image: stacksimplify/kube-usermanagement-webapp:mysql-db`:** This is the pre-built Spring/JSP web application container.
-   **`imagePullPolicy: Always`:** This tells Kubernetes to always pull the image from the registry, even if it's already present on the node. This is useful for development to ensure you're always using the latest build.
-   **`ports`:** The web application listens on port `8080`.

#### 3. `env` (Environment Variables)
-   This section injects the database connection details into the web application container as environment variables.
-   **`DB_HOSTNAME: "mysql"`:** This is the most important part. The application will connect to the database using the DNS name `mysql`. Because the web app pod and the MySQL pod are in the same cluster, Kubernetes' internal DNS will automatically resolve the service name `mysql` to the IP address of the `ClusterIP` Service we created for the database.
-   The other variables provide the port, database name, and credentials.

---

> [!summary] Conclusion
> You have successfully written a `Deployment` manifest for a web application that depends on a database. This exercise demonstrates two critical Kubernetes patterns:
> 1.  Using **environment variables** to decouple an application's configuration from its code, allowing it to connect to other services within the cluster.
> 2.  Using **`initContainers`** to manage startup dependencies, ensuring that a dependent service (the web app) only starts after its dependency (the database) is fully available.
>
> In the next lecture, we will create the final `LoadBalancer` Service for this web app, deploy all the manifests together, and test the end-to-end functionality.