#Cloud #Azure #Kubernetes #AKS #Serverless #VirtualNodes #ACI #Networking #HandsOn #Tutorial

>  This is a hands-on guide to creating a new **[[Azure Kubernetes Service (AKS)|AKS]] cluster** with the **Virtual Nodes** add-on enabled. Virtual Nodes, powered by **Azure Container Instances (ACI)**, allow you to run Kubernetes [[The Kubernetes Pod|Pods]] in a serverless container environment. This provides a fast way to "burst" workloads without needing to manage and pay for underlying Virtual Machine (VM) nodes.

---

This guide follows the instructor's step-by-step process for provisioning a new AKS cluster designed to support Virtual Nodes.

## 🏛️ Introductory Concepts Review
-   **Virtual Kubelet:** The open-source technology that makes Virtual Nodes possible. It registers itself with the Kubernetes API server as a node with seemingly infinite capacity.
-   **Azure Container Instances (ACI):** The underlying serverless container runtime. When a pod is scheduled to a Virtual Node, the Virtual Kubelet actually provisions an ACI instance to run that pod's container.

## 🛠️ Step 1: Create a New AKS Cluster with Virtual Nodes Enabled

This process is done through the Azure Portal (`portal.azure.com`).

1.  **Navigate and Start Creation:**
    -   In the Azure Portal, search for and select **Kubernetes services**.
    -   Click **Create > Create a Kubernetes cluster**.

2.  **Configure the Basics Tab:**
    -   **Project Details:**
        -   **Resource Group:** Create a new one for this demo (e.g., `AKS-RG2`).
    -   **Cluster Details:**
        -   **Kubernetes cluster name:** `aksdemo2`.
        -   **Region:** `Central US`.
        -   **Availability zones:** Leave as default.
        -   **Node size:** Leave as default.
        -   **Node count:** Set to **`1`**. We only need one physical node to run the core system pods (like CoreDNS, ACI connector, etc.). Our application pods will run on the serverless Virtual Node.
    -   **Virtual nodes:**
        -   **Crucially, check the `Enable virtual nodes` box.** This is the add-on that enables the feature.

3.  **Configure Authentication:**
    -   The instructor recommends moving away from the default `Service principal` to the more modern and secure **`System-assigned managed identity`**. Select this option.
    -   Leave `Role-based access control (RBAC)` enabled.

4.  **Configure Networking:**
    -   **Network configuration:** When you enable Virtual Nodes, the portal **automatically selects `Azure CNI`**.
    -   **Why?** Virtual Nodes **require** the `Azure CNI` networking plugin. They cannot work with the more basic `kubenet`. `Azure CNI` provides native VNet integration, which is necessary for the ACI instances to communicate with the rest of the cluster.
    -   The portal will also automatically propose the creation of a new subnet specifically for the Virtual Nodes (e.g., `virtual-node-aci`). Leave these auto-populated settings as default.

5.  **Configure Integrations and Advanced:**
    -   **Integrations:** Leave as default. We will not attach an Azure Container Registry at this stage.
    -   **Advanced:** Leave as default.
    -   **Tags:** Leave as default.

6.  **Review and Create:**
    -   Click **Review + create**.
    -   Once validation passes, click **Create**. The cluster creation will take a few minutes.

> [!info] Virtual Node Provisioning Time
> After the cluster itself is created, it can take an additional 5-10 minutes for the Virtual Node add-on to be fully provisioned and for the `virtual-node-aci-linux` node to appear in `kubectl get nodes`. Be patient.

---

## 🔬 Step 2: Verify the Cluster and Virtual Node Components

Once the deployment is complete, we will use `kubectl` to inspect the cluster and confirm that the Virtual Node components are present and ready.

1.  **Get Cluster Credentials:** Configure your local `kubectl` to connect to the new cluster.
    ```bash
    az aks get-credentials --resource-group AKS-RG2 --name aksdemo2
    ```

2.  **Get the Nodes:** This is the most important verification step.
    ```bash

    kubectl get nodes -o wide
    ```
    **Expected Output:** You will see **two** nodes listed:
    1.  **The Physical Node:** `aks-agentpool-...` - This is the single VM from our default node pool. It's a standard Kubernetes node.
    2.  **The Virtual Node:** `virtual-node-aci-linux` - This is **not a real VM**. This is the Virtual Kubelet registering itself as a node with the API server. Its role is to accept pods and run them on ACI.

3.  **Inspect the ACI Connector Pod:**
    The Virtual Node functionality is powered by a special pod that runs on your physical node pool. This pod is the "ACI Connector."
    ```bash
    kubectl get pods -n kube-system
    ```
    In the list of pods in the `kube-system` namespace, you will find a pod named `aci-connector-linux-...`. This is the pod that contains the Virtual Kubelet implementation. If you have issues scheduling pods to the Virtual Node, checking the logs of this connector pod is the first step in troubleshooting.
    ```bash
    # To check the logs for errors
    kubectl logs <aci-connector-linux-pod-name> -n kube-system
    ```

---

> [!summary] Conclusion
> We have successfully provisioned a new AKS cluster with the **Virtual Nodes add-on enabled**. We have verified through `kubectl` that we now have both a standard, physical node pool and a serverless `virtual-node-aci-linux` node ready to accept workloads.
>
> In the next lecture, we will:
> 1.  Deploy a sample application to this new cluster.
> 2.  Use Kubernetes scheduling concepts (`nodeSelector` and `tolerations`) to explicitly schedule the application's pod to run on the **Virtual Node**.
> 3.  Verify that the pod is running on ACI and test its functionality.