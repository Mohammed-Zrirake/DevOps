#Cloud #Azure #Kubernetes #AKS #Networking #Ingress #AzureCLI #HandsOn #Tutorial

>  For a production-ready [[Hands-On A Basic Guide to Kubernetes Ingress|Ingress setup]], it's a best practice to use a **static public IP address** rather than a dynamic one that changes. This is done by creating a new Public IP resource within the special **node resource group** (also known as the infrastructure resource group) that [[Azure Kubernetes Service (AKS)|AKS]] automatically creates to hold all of its underlying cloud resources.

---

This is the first hands-on step in our Ingress implementation. We will provision the static public IP that our Nginx Ingress Controller will use as its single, stable entry point.

## 🏛️ Why a Static IP in the Node Resource Group?

-   **The Node Resource Group:** When you create an AKS cluster, Azure provisions a *second*, managed resource group. Its name typically starts with `MC_` (for "Managed Cluster"). This group contains all the actual infrastructure that powers your cluster, such as the VM Scale Sets for your node pools, the default load balancer, and managed disks.
-   **Co-location:** By creating our Ingress Controller's public IP inside this same managed resource group, we ensure it's co-located with the rest of the cluster's networking infrastructure, which is a requirement for the Ingress Controller's service to be able to use it.

---

## 🛠️ Step-by-Step Implementation

### Step 1: Get the Node Resource Group Name
First, we need to find the exact name of this auto-generated node resource group. We can do this with an Azure CLI command.

```bash
az aks show --resource-group aks-rg1 --name aksdemo1 --query nodeResourceGroup -o tsv
```
-   `az aks show`: Retrieves the details of a specific AKS cluster.
-   `--resource-group aks-rg1 --name aksdemo1`: Specifies our cluster.
-   `--query nodeResourceGroup`: This is a JMESPath query that filters the JSON output to show *only* the value of the `nodeResourceGroup` property.
-   `-o tsv`: Formats the output as a simple, tab-separated value (plain text) without quotes.

**Example Output:**
```
MC_aks-rg1_aksdemo1_centralus
```

### Step 2: Create the Static Public IP Address
Now we use another Azure CLI command to create the public IP resource inside the resource group we just identified.

```bash
# Store the node resource group name in a variable for clarity (optional)
NODE_RG="MC_aks-rg1_aksdemo1_centralus"

# Create the public IP
az network public-ip create \
    --resource-group $NODE_RG \
    --name myAKSPublicIPForIngress \
    --sku Standard \
    --allocation-method Static \
    --query publicIp.ipAddress -o tsv
```
**Command Breakdown:**
-   `az network public-ip create`: The Azure CLI command to create a public IP.
-   `--resource-group $NODE_RG`: **Crucially**, we are creating this resource in the **node resource group**, not our primary one.
-   `--name myAKSPublicIPForIngress`: A descriptive name for our new IP resource.
-   `--sku Standard`: We are creating a `Standard` SKU public IP, which is required for integration with the Standard SKU Azure Load Balancer that AKS uses.
-   `--allocation-method Static`: This ensures the IP address does not change over time.
-   `--query publicIp.ipAddress -o tsv`: After creation, this query filters the output to show only the newly created IP address itself.

**Example Output:**
```
52.123.45.67
```
> [!tip]
> Make a note of this IP address. We will need it in the next step when we install the Nginx Ingress Controller.

### Step 3: Verify Creation in the Azure Portal
1.  In the Azure Portal, search for and navigate to **Public IP addresses**.
2.  You will now see a new IP address resource listed with the name `myAKSPublicIPForIngress`.
3.  Clicking on it will show its details, including the static IP address that was assigned.

---

> [!summary] Conclusion
> We have successfully provisioned a static, Standard SKU public IP address inside our AKS cluster's managed node resource group. This provides a stable and predictable endpoint that we can now assign to our Nginx Ingress Controller, setting the foundation for a production-grade ingress setup.