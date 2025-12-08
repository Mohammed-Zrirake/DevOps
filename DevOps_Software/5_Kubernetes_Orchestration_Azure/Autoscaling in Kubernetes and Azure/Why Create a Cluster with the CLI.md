#Cloud #Azure #Kubernetes #AKS #Scaling #Autoscaling #ClusterAutoscaler #AzureCLI #HandsOn #Tutorial

>  The **Cluster Autoscaler** is a Kubernetes component that automatically adjusts the number of **worker nodes** in a [[Azure Kubernetes Service (AKS)#Customer-Managed Worker Nodes (Node Pools)|node pool]] based on the resource demands of your workloads. If there are pending pods that cannot be scheduled due to insufficient resources, the Cluster Autoscaler will **add new nodes** (scale-out). If nodes are underutilized for a period of time, it will **remove nodes** (scale-in) to save costs.

---

This is a hands-on guide to creating a new AKS cluster with the Cluster Autoscaler feature enabled from the start using the Azure CLI.

## 🏛️ Why Create a Cluster with the CLI?

While creating clusters through the Azure Portal is convenient for learning, using the **Azure CLI** is a best practice for several reasons:
-   **Automation:** CLI commands can be scripted, making the cluster creation process repeatable and automatable.
-   **Version Control:** You can check your cluster creation scripts into source control.
-   **CI/CD Integration:** CLI commands are easily integrated into CI/CD pipelines for creating and managing infrastructure.

## 🛠️ Step 1: Set Up the Environment and Create a Resource Group

We will start by setting up some environment variables in our terminal for clarity and reusability, and then create a new resource group for our cluster.

1.  **Set Environment Variables:**
    ```bash
    export RESOURCE_GROUP="aks-rg1-autoscaling"
    export REGION="centralus"
    export CLUSTER_NAME="aks-autoscaling-demo"
    ```
2.  **Create the Resource Group:**
    ```bash
    az group create --name $RESOURCE_GROUP --location $REGION
    ```

---

## 🚀 Step 2: Create the AKS Cluster with Autoscaler Enabled

Now we will use the `az aks create` command with specific flags to enable the Cluster Autoscaler.

### The `az aks create` Command
```bash
az aks create \
  --resource-group $RESOURCE_GROUP \
  --name $CLUSTER_NAME \
  --enable-managed-identity \
  --generate-ssh-keys \
  --node-count 1 \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 5
```

**Command Breakdown:**
-   `--resource-group` and `--name`: Specify the resource group and name for our new cluster.
-   `--enable-managed-identity`: A best practice for securely authenticating the cluster to other Azure services.
-   `--generate-ssh-keys`: Automatically creates SSH keys for accessing the nodes if needed.
-   `--node-count 1`: We are starting the cluster with only **one** initial worker node to save costs during setup. (Production clusters should start with at least 2 or 3).
-   `--enable-cluster-autoscaler`: **This is the critical flag.** It enables the Cluster Autoscaler feature on the cluster's default system node pool.
-   `--min-count 1`: The minimum number of nodes the autoscaler will maintain in this node pool.
-   `--max-count 5`: The maximum number of nodes the autoscaler can scale out to. You can set this to a higher number (e.g., 20, 40, 100) based on your expected workload.

> [!info] Cluster creation can take 3-5 minutes.

---

## 🔌 Step 3: Connect to the New Cluster

Once the cluster is created, configure `kubectl` to connect to it.
```bash
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME

# Verify the connection
kubectl get nodes
kubectl cluster-info
```
You should see a single worker node in the `Ready` state.

---

## 🔬 Step 4: Verifying the Autoscaler Configuration in the Azure Portal

While we configured the cluster via the CLI, we can verify the settings in the Azure Portal.

1.  **Navigate to your Cluster:** Go to **Kubernetes services** and select your new cluster (`aks-autoscaling-demo`).
2.  **Go to Node Pools:** In the left sidebar, under **Settings**, click on **Node pools**.
3.  **Select the Node Pool:** Click on the default node pool (e.g., `agentpool`).
4.  **View Scale Settings:** A new pane will open. Click on **Scale**.
5.  **Confirm Configuration:** You will see that the **Scale method** is set to **`Autoscale`**, and the **Node count range** is correctly set to a minimum of `1` and a maximum of `5`.

### Autoscaling is a Per-Node Pool Setting
-   The instructor emphasizes that the Cluster Autoscaler is configured on a **per-node pool basis**.
-   You can have multiple node pools in a single cluster. For example, you could have:
    -   A system node pool with autoscaling disabled.
    -   A user node pool for general workloads with autoscaling enabled from 1 to 10 nodes.
    -   Another user node pool with GPU-enabled VMs for machine learning tasks, with autoscaling enabled from 0 to 3 nodes.
-   This gives you fine-grained control over the scaling behavior and cost of different parts of your cluster's infrastructure.

---

> [!summary] Conclusion
> We have successfully provisioned a new AKS cluster with the **Cluster Autoscaler** enabled and configured for the default system node pool. The cluster is now capable of automatically scaling its infrastructure (adding or removing VMs) in response to application workload demands.
>
> In the next lecture, we will:
> 1.  Deploy a sample application.
> 2.  Use `kubectl scale` to dramatically increase the number of pod replicas.
> 3.  Observe as the Cluster Autoscaler detects the pending pods, realizes it doesn't have enough node capacity, and automatically provisions new worker nodes to meet the demand.