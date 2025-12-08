#Cloud #Azure #Kubernetes #AKS #Networking #Services #ExternalName #Database #HandsOn #Tutorial

>  This guide details how to write a declarative YAML manifest for an **`ExternalName` [[Kubernetes Services A Deep Dive|Service]]**. This special service type acts as a DNS alias *within* the Kubernetes cluster, allowing your applications to connect to an external service (like a managed cloud database) using a simple, consistent, internal service name, rather than hardcoding the long, external DNS hostname.

---

This is a hands-on guide to creating an `ExternalName` service to connect our application in [[Azure Kubernetes Service (AKS)|AKS]] to the [[Provisioning an Azure Database for MySQL|Azure Database for MySQL]] instance we created previously.

## 🏛️ The `ExternalName` Service Manifest

The manifest for an `ExternalName` service is simpler than other service types because it doesn't deal with ports or selectors. Its sole purpose is to create a CNAME-like DNS record.

```yaml
# 1. API Version for Service is 'v1'
apiVersion: v1
# 2. The kind of object is 'Service'
kind: Service
# 3. Metadata to identify the Service
metadata:
  # This is the simple, internal DNS name our applications will use
  name: mysql
spec:
  # 4. Specification of the desired state for this Service
  
  # 'type' is the key that defines this as an ExternalName service.
  type: ExternalName
  
  # 'externalName' is the fully qualified domain name (FQDN) of the external service.
  externalName: aks-webapp-db.mysql.database.azure.com
```

### A Deep Dive into the `spec` Section
-   **`type: ExternalName`**: This is the defining characteristic of this service type.
-   **`externalName`**: This is the critical value. You must provide the exact, fully qualified domain name (FQDN) of the external resource you want to alias. In this case, it's the **Server name** from the Overview page of our Azure Database for MySQL instance.
-   **No `selector` or `ports`:** An `ExternalName` service does not select pods and does not proxy traffic, so it has no `selector` or `ports` section. The DNS resolution for the port happens at the client level.

---

## 🛠️ Hands-On: Deploying the Service and Testing Connectivity

### Step 1: Deploy the `ExternalName` Service Manifest
1.  Navigate to the directory containing your `01-mysql-external-name-service.yaml` file.
2.  Apply the manifest to the cluster:
    ```bash
    kubectl apply -f 01-mysql-external-name-service.yaml
    ```
    **Output:** `service/mysql created`

This creates a DNS entry within the cluster. Now, any pod in the same namespace that tries to connect to the hostname `mysql` will have its request redirected to `aks-webapp-db.mysql.database.azure.com`.

### Step 2: Test Connectivity from Inside the Cluster
We need to verify that our applications inside the cluster can now reach the Azure MySQL database. We will also create the initial database schema that our web application will use.

1.  **Run a temporary MySQL client Pod:** Use `kubectl run` to start a temporary container with a MySQL client, get a shell, and connect to our new Azure database.
    ```bash
    kubectl run -it --rm --image=mysql:5.6 mysql-client -- mysql -h aks-webapp-db.mysql.database.azure.com -u "dbadmin@aks-webapp-db" -p"Redhat@1449"
    ```
    **Command Breakdown:**
    -   `run -it --rm --image=mysql:5.6 mysql-client`: Creates an interactive, temporary pod named `mysql-client` using the `mysql:5.6` image.
    -   `--`: Separator for the command to run inside the pod.
    -   `mysql`: The MySQL client executable.
    -   `-h aks-webapp-db.mysql.database.azure.com`: The **host** to connect to. We are using the full external FQDN here to test direct connectivity.
    -   `-u "dbadmin@aks-webapp-db"`: The **username**. For Azure Database for MySQL, the username is in the format `admin_user@server_name`.
    -   `-p"Redhat@1449"`: The **password** we configured when creating the database.

2.  **Create the Database Schema:** Once you have a `mysql>` prompt, create the database that our user management application expects.
    ```sql
    mysql> CREATE DATABASE webappdb;
    ```
3.  **Verify Schema Creation and Exit:**
    ```sql
    mysql> SHOW DATABASES;
    -- You should see 'webappdb' in the list.
    mysql> exit;
    ```
    The `mysql-client` pod will be automatically removed because we used the `--rm` flag.

This test confirms that our AKS cluster has network access to the Azure Database for MySQL instance, thanks to the firewall rules we configured previously.

---

> [!summary] Conclusion
> You have successfully created an `ExternalName` Service, providing a clean abstraction layer for your applications to connect to an external, managed database. This is a powerful pattern that:
> -   **Decouples your application from the infrastructure:** Your application code only needs to know the simple service name (`mysql`), not the long, environment-specific cloud hostname.
> -   **Simplifies configuration management:** If the database hostname ever changes, you only need to update the `ExternalName` service manifest in one place, without having to reconfigure and redeploy all of your applications.
>
> In the next lecture, we will update our User Management Web App's Deployment to use this new external database and deploy the final application.