#DevOps #Containerization #Kubernetes #CoreConcept #YAML #Deployments #StatefulSets #Volumes #PVC #ConfigMap #Manifests #HandsOn #Tutorial

>  This guide details how to write a declarative YAML manifest for a stateful application, using a **MySQL database** as the example. This is a critical pattern that combines a [[The Kubernetes Deployment|Deployment]] with two types of **volumes**: one for **persistent data** using a [[Writing Kubernetes Manifests The PersistentVolumeClaim (PVC)|PersistentVolumeClaim (PVC)]] and another for **configuration** using a [[Writing Kubernetes Manifests The ConfigMap|ConfigMap]].

---

This is a hands-on guide to creating the `mysql-deployment.yaml` file, which will define our database workload.

## 🏛️ The `mysql-deployment.yaml` Manifest

```yaml
# API Version for Deployment is 'apps/v1'
apiVersion: apps/v1
# The kind of object is 'Deployment'
kind: Deployment
metadata:
  name: mysql
spec:
  # We only want a single instance of our database
  replicas: 1
  
  # 'selector' tells the Deployment which pods its ReplicaSet should manage
  selector:
    matchLabels:
      app: mysql
      
  # This strategy ensures that when updating, the old pod is killed before the new one is created.
  # This is important for stateful apps with a single replica to avoid conflicts with volume mounts.
  strategy:
    type: Recreate
    
  # 'template' is the blueprint for the pods that will be created.
  template:
    metadata:
      labels:
        # This pod label MUST match the 'selector.matchLabels' above.
        app: mysql
    spec:
      # 'containers' is a list defining the containers to run in the pod
      containers:
      - name: mysql
        image: mysql:5.6
        
        # 'env' defines environment variables for the container
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "db_password" # In a real scenario, this should come from a Secret
        
        # 'ports' is a list of network ports this container exposes
        ports:
        - name: mysql
          containerPort: 3306
          
        # 'volumeMounts' makes volumes available inside the container's filesystem
        volumeMounts:
        # Mount for the persistent database files
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
        # Mount for the initialization SQL script from the ConfigMap
        - name: user-management-db-creation-script
          mountPath: /docker-entrypoint-initdb.d
          
      # 'volumes' defines the storage volumes that are available to the pod
      volumes:
      # This volume is for persistent data, linked to our PVC
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: azure-managed-disk-pvc
      # This volume is for configuration data, linked to our ConfigMap
      - name: user-management-db-creation-script
        configMap:
          name: user-management-db-creation-script
```

---

### A Deep Dive into the `spec` Section

#### `strategy`
-   **`type: Recreate`**: The default deployment strategy is `RollingUpdate`, which is great for stateless apps as it ensures zero downtime. However, for a single-replica stateful application using persistent storage like an Azure Disk, a rolling update can cause issues. The `Recreate` strategy tells Kubernetes to **kill the old Pod first** before creating the new one. This ensures that the persistent volume is fully detached from the old Pod before the new Pod tries to mount it, preventing "volume in use" errors.

#### `spec.template.spec.containers` (The Container Definition)
-   **`env`:** This is how we pass environment variables to a container. The official `mysql` image requires `MYSQL_ROOT_PASSWORD` to be set on its first run.
    > [!security] In a real-world scenario, you would **never** hardcode a password like this. You would store it in a **Kubernetes `Secret`** object and reference it here.
-   **`ports`:** We declare that our MySQL container listens on the standard port `3306`.
-   **`volumeMounts`:** This is a critical section. It is a list that makes the volumes (defined below) accessible *inside* the container's filesystem.
    -   **Mount 1 (Persistent Data):** We mount the volume named `mysql-persistent-storage` to the path `/var/lib/mysql`. This is the directory where the MySQL image stores its database files.
    -   **Mount 2 (Init Script):** We mount the volume named `user-management-db-creation-script` to the path `/docker-entrypoint-initdb.d`.

#### `spec.template.spec.volumes` (The Pod's Volumes)
-   **What it is:** This is a list that defines all the storage volumes that are available to this **Pod**. These volumes are then "mounted" into specific containers using `volumeMounts`.
-   **Volume 1 (`persistentVolumeClaim`):**
    -   We define a volume named `mysql-persistent-storage`.
    -   We specify that this volume's source is a `persistentVolumeClaim`.
    -   `claimName: azure-managed-disk-pvc`: This is the crucial link. It tells Kubernetes to use the `PersistentVolumeClaim` we created earlier. This is how our Pod gets access to the persistent Azure Disk.
-   **Volume 2 (`configMap`):**
    -   We define a volume named `user-management-db-creation-script`.
    -   We specify that this volume's source is a `configMap`.
    -   `name: user-management-db-creation-script`: This links the volume to the `ConfigMap` we created. Kubernetes will create a temporary filesystem from the ConfigMap's data.

---

### The Magic of `/docker-entrypoint-initdb.d`
The instructor highlights a special feature of the official `mysql` Docker image:
-   When a MySQL container is started for the very first time (i.e., when `/var/lib/mysql` is empty), its entrypoint script will automatically execute any files it finds in the `/docker-entrypoint-initdb.d` directory that have extensions like `.sh` or `.sql`.
-   By mounting our `mysql-user-mgmt.sql` file from the `ConfigMap` into this exact directory, we are leveraging this built-in feature to **automatically initialize our database schema** (`webappdb`) on the first run.

---

> [!summary] Conclusion
> You have successfully written a complex `Deployment` manifest for a stateful application. This is a powerful pattern that demonstrates:
> -   How to use a `Recreate` strategy for single-replica stateful apps.
> -   How to define volumes at the Pod level using different sources (`persistentVolumeClaim` for data, `configMap` for configuration).
> -   How to mount those volumes into a container at specific paths using `volumeMounts` to provide persistent storage and initialization scripts.
>
> In the next lecture, we will create the final piece—the `ClusterIP` Service for this MySQL deployment—and then deploy all the manifests to the cluster.