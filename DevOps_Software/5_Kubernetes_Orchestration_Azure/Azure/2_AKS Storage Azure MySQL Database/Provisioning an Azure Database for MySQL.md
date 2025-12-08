#Cloud #Azure #Kubernetes #AKS #Database #AzureMySQL #ManagedServices #HandsOn #Tutorial

>  This is a hands-on guide to creating and configuring a new **Azure Database for MySQL** instance through the Azure Portal. The process involves defining the server details, selecting the compute and storage tier, setting administrative credentials, and crucially, configuring the connection security settings to allow access from [[Azure Kubernetes Service (AKS)|AKS]] and other Azure services.

---

This is the first step in migrating our stateful application from a self-hosted database to a fully managed service.

## 🛠️ Step 1: Create the MySQL Server Instance

This process is done entirely through the Azure Portal (`portal.azure.com`).

1.  **Navigate to the Service:**
    -   In the Azure Portal's top search bar, search for "Azure MySQL".
    -   Select **Azure Database for MySQL servers**.
2.  **Start Creation:**
    -   Click the **Add** or **Create** button.
3.  **Configure Server Details:**
    -   **`Subscription`:** Select your desired subscription.
    -   **`Resource group`:** Select the same resource group where your AKS cluster is located (e.g., `aks-rg1`). This keeps all related resources organized together.
    -   **`Server name`:** Provide a **globally unique** name for your database server (e.g., `aks-webapp-db-123`). The portal will validate if the name is available. This name becomes part of the server's public DNS endpoint.
    -   **`Data source`:** Select `None` to create a new, empty server.
    -   **`Location`:** Select the **same region** as your AKS cluster (e.g., `Central US`). Co-locating resources minimizes network latency.
    -   **`Version`:** Select `5.7`. The instructor uses this version for compatibility with the demo web application.
4.  **Configure Compute + Storage:**
    -   Click **Configure server**.
    -   For this demo, we will select the most basic and cost-effective options to minimize cost.
    -   **`Compute Generation`:** (Select the appropriate generation).
    -   **`Tier`:** Select **Basic**.
    -   **`vCore`:** Bring it down to **1 vCore**.
    -   **`Storage`:** Bring it down to the minimum, **5 GB**.
    -   **`Storage auto-growth`:** Leave enabled (`Yes`).
    -   **`Backup Redundancy Options`:** `Locally Redundant`.
    -   Click **OK**.
5.  **Configure Administrator Account:**
    -   **`Admin username`:** `dbadmin`.
    -   **`Password`:** Set and confirm a strong password (e.g., `Redhat@1449` or `dbpassword@11`). Note that Azure has complexity requirements.
6.  **Review and Create:**
    -   Click through the **Additional settings** and **Tags** tabs, leaving them as default.
    -   Click **Review + create**. The portal will display an estimated monthly cost.
    -   Click **Create**.

> [!info] Deployment Time
> The provisioning process for a new Azure Database for MySQL server can take a significant amount of time, often around **10 minutes**.

---

## ⚙️ Step 2: Configure Connection Security

Once the database server is created, you must configure its firewall and security settings to allow your AKS cluster (and optionally, your local machine) to connect to it.

1.  **Navigate to the Resource:** Go to your newly created MySQL server in the Azure Portal.
2.  **Find Connection Security:** In the left sidebar, under the **Settings** category, click on **Connection security**.
3.  **Apply Three Critical Settings:**
    -   **A. Allow Access to Azure Services:**
        -   Set the **`Allow access to Azure services`** toggle to **`YES`**.
        -   **Why is this critical?** This is the firewall rule that allows other services running within the Azure network—most importantly, your AKS cluster—to connect to this database. Without this, your application pods will not be able to reach the database.
    -   **B. (Optional) Allow Your Local IP:**
        -   To allow you to connect to the database from your local machine using a tool like MySQL Workbench or the `mysql` CLI, click **`Add current client IP address`**.
        -   This will create a firewall rule that allows traffic from your current public IP address.
    -   **C. Disable Enforce SSL Connection:**
        -   Set the **`Enforce SSL connection`** toggle to **`DISABLED`**.
        -   **Why?** By default, Azure MySQL requires SSL-encrypted connections. While this is a best practice for production, it requires additional configuration in the application's connection string (e.g., `?useSSL=true&requireSSL=true`). To simplify our demo setup, we are disabling this requirement.
4.  **Save the Changes:**
    -   Click the **Save** button at the top.
    -   Applying these security settings can also take a few moments. The portal will show a notification when the update is successful.

### Verifying the Configuration
After saving, you can re-open the **Connection security** page to confirm that all three settings have been correctly applied:
-   `Allow access to Azure services`: `YES`
-   A firewall rule exists for your client IP.
-   `Enforce SSL connection`: `DISABLED`

---

## 🔗 The Database Endpoint

On the **Overview** page for your MySQL server, you will find the **Server name**. This is the fully qualified domain name (FQDN) that your applications will use to connect to the database.

-   **Example:** `aks-webapp-db.mysql.database.azure.com`

---

> [!summary] Conclusion
> You have successfully provisioned a fully managed Azure Database for MySQL instance and configured its security settings. The database is now ready to accept connections from services within Azure.
>
> In the next lecture, we will create a Kubernetes **`ExternalName` Service** manifest. This will create a simple, stable DNS alias within our AKS cluster (`mysql`) that points to this long, external database hostname, providing a clean abstraction layer for our application.