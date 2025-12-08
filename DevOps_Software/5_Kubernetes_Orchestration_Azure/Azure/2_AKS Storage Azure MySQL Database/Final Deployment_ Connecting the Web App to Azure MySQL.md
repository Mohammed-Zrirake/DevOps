#DevOps #Containerization #Kubernetes #CoreConcept #YAML #Deployments #Services #AzureMySQL #HandsOn #Tutorial

>  This is the final hands-on guide where we deploy our complete, multi-tier application. We will update the **User Management Web App's `Deployment` manifest** with the correct credentials for our [[Provisioning an Azure Database for MySQL|Azure Database for MySQL]] instance. Then, we will deploy all the manifests—including the [[Writing Kubernetes Manifests The ExternalName Service|`ExternalName` Service]]—and test the end-to-end functionality, from the user's browser all the way to the managed database.

---

This exercise brings together all the concepts from the previous lectures to create a fully functional, cloud-native application.

## ✍️ Step 1: Updating the Web App Deployment Manifest

The core task in this step is to update the environment variables in our `user-management-webapp-deployment.yaml` to point to the new, external Azure MySQL database instead of the self-hosted one.

### The `user-management-webapp-deployment.yaml` (Updated)
We will copy the deployment manifest from the previous section (`05-03`) and make the following critical changes to the `env` section.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-mgmt-webapp
# ... (labels, replicas, selector, template metadata, initContainers are all the same) ...
    spec:
      initContainers:
      - name: init-db
        image: busybox:1.31
        # The command now checks for the availability of the 'mysql' ExternalName service
        command: ['sh', '-c', 'echo "Checking for availability of MySQL Server"; while ! nc -z mysql 3306; do sleep 1; printf "-"; done; echo "  >> MySQL DB Server has started";']
        
      containers:
      - name: user-management-webapp
        image: stacksimplify/kube-usermanagement-webapp:mysql-db
        # ... (imagePullPolicy, ports are the same) ...
        
        # --- CRITICAL CHANGES ARE HERE ---
        env:
        - name: DB_HOSTNAME
          value: "mysql" # This now points to our ExternalName Service
        - name: DB_PORT
          value: "3306"
        - name: DB_NAME
          value: "webappdb"
        - name: DB_USERNAME
          # This MUST match the admin user for your Azure MySQL instance
          value: "dbadmin@aks-webapp-db" 
        - name: DB_PASSWORD
          # This MUST match the password for your Azure MySQL instance
          value: "Redhat@1449"
```
**Key Changes Explained:**
-   **`DB_HOSTNAME`**: This remains `mysql`. Our application doesn't need to know that the database is external. It simply connects to the internal `mysql` service name, and the `ExternalName` service handles the DNS redirection to the Azure database FQDN. This is a key abstraction.
-   **`DB_USERNAME`**: This is updated to the full username required by Azure Database for MySQL, which is in the format `admin_user@server_name`.
-   **`DB_PASSWORD`**: This is updated to the password you set when provisioning the Azure MySQL instance.

The `LoadBalancer` Service manifest for the frontend (`user-management-webapp-service.yaml`) requires no changes.

---

## 🛠️ Hands-On: Deploying and Testing the Full Application

### Step 1: Deploy All Manifests
1.  Navigate to the directory for this section (`06-azure-mysql-for-aks-storage/`).
2.  Apply all the manifests in the `kube-manifests/` directory. This will create the `ExternalName` Service, the `user-mgmt-webapp` Deployment, and the `user-mgmt-webapp-service`.
    ```bash
    kubectl apply -f kube-manifests/
    ```

### Step 2: Verify the Deployment
1.  **Check the Pods:**
    ```bash
    kubectl get pods
    ```
    You should see the `user-mgmt-webapp` pod starting up. The `initContainer` will run, checking for connectivity to the `mysql` service, which now points to our external Azure database. Once the connection is established, the main container will start.

2.  **Check the Logs:** Verify that the web application started successfully and connected to the database without any errors.
    ```bash
    kubectl logs -f <user-mgmt-webapp-pod-name>
    ```

### Step 3: Access and Test the Web Application
1.  **Get the Public IP:** Get the external IP address of the frontend service.
    ```bash
    kubectl get svc
    ```
    Copy the `EXTERNAL-IP` for the `user-mgmt-webapp-service`.

2.  **Log in:**
    -   Navigate to the IP address in your browser.
    -   Log in with the default credentials:
        -   Username: `admin101`
        -   Password: `password101`

3.  **Test the Application:**
    -   Click **List Users**. You will see the default `admin101` user.
    -   Click **Add User** and create a new user (e.g., `admin102`). The user should be created successfully.
    -   Log out and log back in with the new `admin102` credentials to confirm it works.

### Step 4: Verify Data Persistence in the Azure Database
Let's connect directly to the Azure MySQL database to confirm that the new user data was persisted in the managed service.
1.  **Run a temporary MySQL client pod:**
    ```bash
    kubectl run -it --rm --image=mysql:5.6 mysql-client -- mysql -h aks-webapp-db.mysql.database.azure.com -u "dbadmin@aks-webapp-db" -p"Redhat@1449"
    ```
2.  **Query the Database:**
    ```sql
    mysql> USE webappdb;
    mysql> SELECT * FROM user;
    ```
    The output of the `SELECT` query will now show both the `admin101` and `admin102` users. This confirms that our application, running in AKS, successfully connected to and wrote data to the external, fully managed Azure Database for MySQL instance.

---

## 🧹 Step 5: Full Cleanup

It is a critical best practice to delete all resources after a demo to avoid ongoing cloud costs.

1.  **Delete Kubernetes Resources:**
    ```bash
    kubectl delete -f kube-manifests/
    ```
    This will delete the Deployment and the two Services.

2.  **Delete the Azure Database for MySQL Server:**
    -   Navigate to the Azure Portal.
    -   Find the Azure Database for MySQL server resource (`aks-webapp-db`).
    -   Click the **Delete** button.
    -   You will be prompted to type the name of the server to confirm the deletion. Type the name and click **Delete**.

This completes the full cleanup of all resources created during this multi-part section.

---

> [!summary] Conclusion
> This end-to-end demonstration successfully showcases the power and flexibility of a cloud-native architecture. We have deployed a multi-tier application where:
> -   The **stateless frontend** runs efficiently within an AKS cluster, managed by a Kubernetes Deployment.
> -   The **stateful backend** is offloaded to a fully managed, robust, and scalable **Azure Database for MySQL** service.
> -   The connection between them is cleanly abstracted using a Kubernetes **`ExternalName` Service**.
>
> This pattern is a common and highly recommended best practice, as it allows you to leverage the best of both worlds: the powerful orchestration of Kubernetes for your application logic and the reliability and operational ease of a managed service for your database.