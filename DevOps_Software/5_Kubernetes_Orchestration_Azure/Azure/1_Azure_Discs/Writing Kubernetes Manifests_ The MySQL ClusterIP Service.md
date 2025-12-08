#DevOps #Containerization #Kubernetes #CoreConcept #YAML #Services #ClusterIP #Deployments #StatefulSets #Volumes #PVC #ConfigMap #Manifests #HandsOn #Tutorial

>  This guide details how to write a declarative YAML manifest for a **`ClusterIP` [[Kubernetes Services A Deep Dive|Service]]** to provide a stable, internal network endpoint for our [[Writing Kubernetes Manifests The MySQL Stateful Deployment|MySQL Deployment]]. For single-replica stateful services like a database, we will create a special type of service known as a **Headless Service** by setting `clusterIP: None`.

---

This is the final manifest we need to create to get our MySQL database operational.

## 🏛️ What is a Headless Service?

Normally, when you create a `ClusterIP` service, Kubernetes assigns it a single, stable virtual IP address. The service then load balances requests sent to this IP across all the backend pods it selects.

However, for some stateful applications (especially those with a single replica or those that manage their own clustering), you might not want or need this load balancing proxy. You might want to connect directly to the Pod's IP address.

> [!info] The Headless Service (`clusterIP: None`)
> By explicitly setting `clusterIP: None` in the service's `spec`, you are telling Kubernetes **not** to allocate a virtual IP for the service. Instead, the DNS system will be configured to return the **IP addresses of the individual Pods** that back the service. Since we are running a single MySQL pod, this effectively means the service's DNS name (`mysql`) will resolve directly to our MySQL Pod's IP address.

## ✍️ Writing the `mysql-clusterip-service.yaml` Manifest

```yaml
# 1. API Version for Service is 'v1'
apiVersion: v1
# 2. The kind of object is 'Service'
kind: Service
metadata:
  name: mysql
spec:
  # This is the key that makes it a "Headless Service".
  # It tells Kubernetes not to assign a virtual ClusterIP.
  # The service's DNS name will resolve directly to the Pod's IP.
  clusterIP: None
  
  ports:
  - port: 3306 # The port the service will be available on
    # targetPort is omitted, so it defaults to the same value as 'port' (3306)
  
  # This selector tells the Service to find the Pod with the label 'app: mysql'.
  selector:
    app: mysql
```

### A Deep Dive into the `spec` Section
-   **`name: mysql`:** We are naming our service `mysql`. This is important, as our web application will be configured to connect to the database using this DNS name.
-   **`clusterIP: None`:** This is the defining characteristic of our headless service.
-   **`ports`:** We expose port `3306`, the standard MySQL port.
-   **`selector`:** `app: mysql`. This is the crucial link that tells the service to target the Pod created by our MySQL deployment, which has the matching label.

---

## 🛠️ Hands-On: Deploying and Verifying the Entire MySQL Stack

Now we will deploy all the manifests we have created (`StorageClass`, `PVC`, `ConfigMap`, `Deployment`, and `Service`) and verify each step of the process.

### Step 1: Deploy All Manifests
We can apply all the YAML files in our `kube-manifests` directory with a single command.
```bash
kubectl apply -f kube-manifests/
```
**Output:**
-   The `StorageClass` and `PVC` will be `unchanged` if they already exist.
-   The `ConfigMap`, `Deployment`, and `Service` will be `created`.

### Step 2: Verify the Storage Provisioning
1.  **Check the `PVC`:**
    ```bash
    kubectl get pvc
    ```
    The `STATUS` of our `azure-managed-disk-pvc` has now changed from `Pending` to **`Bound`**. This is because the MySQL Pod (the "first consumer") was created, which triggered the `WaitForFirstConsumer` `StorageClass` to provision the storage.

2.  **Check the `PV`:**
    ```bash
    kubectl get pv
    ```
    You will now see a `PersistentVolume` has been automatically created and is in the `Bound` state, linked to our PVC. Its `RECLAIM POLICY` is `Retain`.

3.  **Check the Azure Disk:**
    -   Go to the **Disks** service in the Azure Portal.
    -   You will see a **new managed disk** has been created. Its name will match the name of the `PersistentVolume`. This confirms that the physical storage has been successfully provisioned.

### Step 3: Verify the MySQL Pod and Initialization
1.  **Check the Pod:**
    ```bash
    kubectl get pods
    ```
    The `mysql-....` pod should be in the `Running` state.

2.  **Describe the Pod:**
    ```bash
    kubectl describe pod <mysql-pod-name>
    ```
    In the `Events` section, you can see the entire lifecycle:
    -   The Pod was successfully assigned to a node.
    -   The `azure-managed-disk-pvc` volume was successfully attached.
    -   The `mysql:5.6` image was pulled.
    -   The container was created and started.

3.  **Check the Logs:**
    ```bash
    kubectl logs -f <mysql-pod-name>
    ```
    The logs will show the standard MySQL server startup process. You should see messages indicating that the initialization was successful and the server is ready for connections. There should be no errors or warnings.

### Step 4: Connect to the Database and Verify Schema Creation
The final test is to connect to the database *from within the cluster* and verify that our `ConfigMap` script ran correctly.

1.  **Run a temporary MySQL client Pod:** We can use `kubectl run` to start a temporary container with a MySQL client, get a shell into it, and connect to our new database service.
    ```bash
    kubectl run -it --rm --image=mysql:5.6 mysql-client -- mysql -h mysql -pdb_password
    ```
    -   `-it`: Get an interactive terminal.
    -   `--rm`: Automatically remove the client pod when we exit.
    -   `--image=mysql:5.6`: Use the same MySQL image, which includes the client tools.
    -   `mysql-client`: The name of our temporary pod.
    -   `--`: Separator.
    -   `mysql -h mysql -pdb_password`: The command to run inside the container. It connects to the host `mysql` (our service name!) with the password `db_password`.

2.  **Verify the Schema:** Once you have a `mysql>` prompt, run the `SHOW DATABASES;` command.
    ```sql
    SHOW DATABASES;
    ```
    You should see the `webappdb` database in the list. This confirms that the `.sql` script from our `ConfigMap` was successfully mounted and executed by the MySQL entrypoint script during initialization.

### Step 5: Clean Up and Test the `Retain` Policy
Let's test our custom `StorageClass`'s `Retain` policy.

1.  **Delete All Kubernetes Resources:**
    ```bash
    kubectl delete -f kube-manifests/
    ```
    This will delete the Deployment, Service, ConfigMap, Pod, and PVC.

2.  **Check the `PV` and Azure Disk:**
    -   `kubectl get pv`: The `PersistentVolume` will still exist, but its `STATUS` will have changed from `Bound` to **`Released`**.
    -   **Azure Portal:** The Azure Managed Disk **still exists**. It has not been deleted. This is the `Retain` policy in action, protecting our critical data.

3.  **Final Cleanup:** To fully clean up and avoid costs, you must now manually delete the `PersistentVolume` in Kubernetes and the `Disk` in the Azure Portal.
    ```bash
    # Manually delete the PV object
    kubectl delete pv <pv-name>
    
    # Manually delete the disk in the Azure UI
    ```

---

> [!summary] Conclusion
> You have successfully deployed a complete, stateful MySQL application using a series of declarative YAML manifests. This exercise demonstrates the powerful interplay between Deployments, ConfigMaps, PersistentVolumeClaims, and StorageClasses. We have also proven the effectiveness of our custom `StorageClass` with its `Retain` policy, providing a critical safety net for our database's data.