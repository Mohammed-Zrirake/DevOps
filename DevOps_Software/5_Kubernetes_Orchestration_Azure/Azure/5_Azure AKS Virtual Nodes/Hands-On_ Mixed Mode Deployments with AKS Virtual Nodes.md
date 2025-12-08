#Cloud #Azure #Kubernetes #AKS #Serverless #ACI #VirtualKubelet #VirtualNodes #MixedMode #nodeSelector #HandsOn #Tutorial


>  This is a comprehensive, hands-on guide to implementing a **mixed-mode deployment** on [[Azure Kubernetes Service (AKS)|AKS]]. We will deploy a multi-tier application where the **stateful MySQL database pod** is scheduled on a standard, VM-based **node pool**, and the **stateless User Management Web App pod** is scheduled on a serverless **[[Azure AKS Virtual Nodes Serverless Kubernetes|Virtual Node]]**. This is achieved by using the `nodeSelector` field in our Deployment manifests to explicitly tell the Kubernetes scheduler where to place each workload.

---

This guide follows the practical implementation of the concepts discussed in the previous lecture, demonstrating a powerful pattern for optimizing cost and resource management.

## ✍️ Step 1: Reviewing the Manifests for a Mixed-Mode Deployment

The core of this exercise lies in the modifications made to the `Deployment` manifests to control where the pods are scheduled.

### MySQL Manifests (`01` to `05`)
-   The manifests for `StorageClass`, `PVC`, `ConfigMap`, `Secret`, and the `mysql-clusterip-service` are largely the same as in our previous [[Hands-On Using Azure Files for Shared Storage in AKS|Azure Disks]] section.
-   **Crucially, in `04-mysql-deployment.yaml`, there is no `nodeSelector` defined.**
    -   **Effect:** When a `nodeSelector` is absent, the Kubernetes scheduler is free to place the pod on any available, compatible node. In our mixed-mode cluster, this means it will be scheduled on the **standard, VM-based node pool**, as Virtual Nodes have taints that prevent pods from being scheduled on them by default unless explicitly targeted. This is the desired behavior, as our MySQL pod requires an Azure Disk, which Virtual Nodes do not support.

### User Management Web App Manifests (`06` and `07`)
This is where the key changes are made.

#### `06-usermanagement-webapp-deployment.yaml` (Key Modifications)
```yaml
apiVersion: apps/v1
kind: Deployment
# ... (metadata, replicas, selector) ...
  template:
    # ... (pod metadata) ...
    spec:
      # --- CRITICAL CHANGES ARE HERE ---
      
      # 1. 'nodeSelector' to target the Virtual Node
      nodeSelector:
        kubernetes.io/role: agent
        beta.kubernetes.io/os: linux
        type: virtual-kubelet
      
      # 'tolerations' to allow scheduling on the Virtual Node
      tolerations:
      - key: virtual-kubelet.io/provider
        operator: Exists
      - key: azure.com/aci
        operator: Exists

      # 2. 'initContainers' are commented out or removed
      # initContainers:
      #   ...

      containers:
      - name: user-management-webapp
        # ... (image, ports) ...
        env:
        - name: DB_HOSTNAME
          # 3. Using the full FQDN for the service
          value: "mysql.default.svc.cluster.local"
        # ... (other DB env vars) ...
```
1.  **`nodeSelector` and `tolerations`**:
    -   This is the most important part. We add a `nodeSelector` to explicitly tell the scheduler to place this pod on a node that has the label `type: virtual-kubelet`. AKS automatically applies this label to the virtual node.
    -   `tolerations` are added to allow the pod to be scheduled on the virtual node, which has specific taints.
2.  **`initContainers` Limitation**:
    -   The instructor highlights a key limitation of Virtual Nodes: they **do not support `initContainers`**.
    -   Our previous logic to wait for the MySQL database must be removed or commented out.
    -   **The Consequence (Race Condition):** This introduces a race condition. If the web app pod starts on the virtual node before the MySQL pod is ready on the standard node, the web app will fail to connect and crash.
    -   **The Workaround:** To handle this, you can either:
        -   Deploy the manifests in two steps: deploy all MySQL-related manifests first, wait for the database to be running, and *then* deploy the web app manifests.
        -   If the web app crashes, you can manually delete the pod (`kubectl delete pod <webapp-pod-name>`). The Deployment will automatically create a new one, which will then likely succeed in connecting to the now-running database.
3.  **Full DNS Name for Service (`DB_HOSTNAME`)**:
    -   When pods are running in different network environments (a standard node pool VNet vs. the ACI VNet for virtual nodes), it's a best practice to use the full **Fully Qualified Domain Name (FQDN)** of the service for connections.
    -   Instead of just `mysql`, we use `mysql.default.svc.cluster.local` (`<service-name>.<namespace>.svc.cluster.local`). This ensures robust DNS resolution across different parts of the cluster's network.

---

## 🛠️ Step 2: Deploying and Verifying the Mixed-Mode Application

1.  **Deploy All Manifests:**
    ```bash
    kubectl apply -f kube-manifests/
    ```
2.  **Verify Pod Scheduling:**
    -   This is the critical verification step. Use the `-o wide` flag to see which node each pod is scheduled on.
        ```bash
        kubectl get pods -o wide
        ```
    -   **Expected Output:**
        ```
        NAME                                READY   STATUS    NODE
        mysql-xxxx-yyyy                     1/1     Running   aks-agentpool-123...   # Standard VM-based node
        user-mgmt-webapp-zzzz-wwww          1/1     Running   virtual-node-aci-linux # The Virtual Node
        ```
    -   This output confirms that our `nodeSelector` worked correctly. The `mysql` pod is on the standard agent pool, while the `user-mgmt-webapp` pod is on the serverless virtual node.

3.  **Verify Nodes and Node Pools:**
    -   `kubectl get nodes -o wide`: You will see two nodes listed: your standard VM-based node and the `virtual-node-aci-linux`.
    -   `az aks nodepool list --cluster-name aksdemo1 --resource-group aks-rg1`: This Azure CLI command will list the node pools, showing your standard `agentpool`.

---

## ✅ Step 3: Accessing and Testing the Application

1.  **Get the Public IP:**
    ```bash
    kubectl get svc
    ```
    Copy the `EXTERNAL-IP` for the `user-mgmt-webapp-service`.

2.  **Log in and Test:**
    -   Access the IP in your browser.
    -   Log in with `admin101` / `password101`.
    -   Verify that you can list the default user and create a new user.

This confirms that the frontend application running on a serverless Virtual Node is successfully communicating with the backend database running on a persistent, VM-based standard node.

---

## 🧹 Step 4: Cleaning Up

1.  **Delete Kubernetes Resources:**
    ```bash
    kubectl delete -f kube-manifests/
    ```
2.  **Delete the AKS Cluster (Optional):**
    If you are finished with the cluster for the day and want to avoid ongoing costs for the node pool VM, you can delete the entire cluster.
    ```bash
    az aks delete --name aksdemo1 --resource-group aks-rg1
    ```

---

> [!summary] Conclusion
> This end-to-end demonstration successfully showcases a sophisticated **mixed-mode deployment strategy**. By using `nodeSelector` and `tolerations`, we can intelligently schedule different parts of our application to the most appropriate infrastructure:
> -   **Stateful, persistent workloads** (like databases) run on **standard node pools** backed by VMs and persistent disks.
> -   **Stateless, bursty workloads** (like web frontends) can run on serverless **Virtual Nodes**, optimizing for cost and eliminating infrastructure management.
> 
> This pattern allows you to get the best of both worlds, combining the reliability of traditional VMs with the cost-efficiency and scalability of serverless containers.