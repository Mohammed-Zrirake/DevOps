#Cloud #Azure #Kubernetes #AKS #Serverless #VirtualNodes #ACI #NodeSelector #Scaling #HandsOn #Tutorial

>  This is a hands-on guide to deploying an application to **Azure Virtual Nodes** in an [[Azure Kubernetes Service (AKS)|AKS]] cluster. Virtual Nodes provide a **serverless container infrastructure**, allowing you to run pods without managing underlying Virtual Machines. We will use a **`nodeSelector`** in our [[The Kubernetes Deployment|Deployment]] manifest to explicitly schedule our pods onto this virtual infrastructure and then test its rapid scaling capabilities.

---

This guide follows the practical implementation of scheduling workloads onto the Virtual Nodes we enabled in a previous setup.

## ✍️ Step 1: Reviewing the Deployment Manifest

The key to scheduling a pod on a Virtual Node is to use a `nodeSelector`. This tells the Kubernetes scheduler that the pod has a specific requirement for the node it can run on.

### The `nginx-deployment.yaml` with `nodeSelector`
We will use a standard Nginx Deployment manifest, with one critical addition to the pod template's `spec`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1-nginx-deployment
spec:
  replicas: 2
  template:
    # ... (pod metadata and labels) ...
    spec:
      # --- CRITICAL ADDITION IS HERE ---
      # This section tells the scheduler where this pod can run.
      nodeSelector:
        # This key-value pair MUST match the labels on the Virtual Node.
        kubernetes.io/role: agent
        beta.kubernetes.io/os: linux
        type: virtual-kubelet
      
      containers:
      - name: app1-nginx
        image: stacksimplify/kube-nginx:1.0.0
        ports:
        - containerPort: 80
```
-   **`spec.template.spec.nodeSelector`**: This is a dictionary of key-value pairs. The Kubernetes scheduler will only place this pod on a node that has **all** of these labels.
-   **`type: virtual-kubelet`**: This is the specific label that identifies the Azure Virtual Node. By including this, we are explicitly telling Kubernetes to schedule our pods on the serverless Azure Container Instances (ACI) infrastructure, rather than on our regular VM-based node pool.

The `loadbalancer-service.yaml` manifest remains a standard `LoadBalancer` Service to expose this deployment.

---

## 🛠️ Step 2: Deploying the Application

1.  **Navigate to the Project Directory:**
    ```bash
    # Navigate to the 17-01-azure-virtual-node-basics directory
    cd 17-01-azure-virtual-node-basics/
    ```
2.  **Apply All Manifests:**
    ```bash
    kubectl apply -f kube-manifests/
    ```
    This creates the `Deployment` and the `Service`.

---

## 🔬 Step 3: Verifying the Deployment on Virtual Nodes

This is the most important verification step. We need to confirm that our pods were scheduled on the Virtual Node and not our standard node pool.

1.  **Check the Pods with `-o wide`:**
    The `-o wide` flag provides additional information, including the `NODE` where each pod is running.
    ```bash
    kubectl get pods -o wide
    ```
    **Output:**
    ```
    NAME                                     READY   STATUS              RESTARTS   AGE     IP             NODE                     NOMINATED NODE   READINESS GATES
    app1-nginx-deployment-5b5f7f9d5f-abcde   1/1     Running             0          2m      10.240.0.4     virtual-node-aci-linux   <none>           <none>
    app1-nginx-deployment-5b5f7f9d5f-fghij   1/1     Running             0          90s     10.240.0.5     virtual-node-aci-linux   <none>           <none>
    ```
    The `NODE` column clearly shows **`virtual-node-aci-linux`**, confirming that our `nodeSelector` worked correctly and the pods are running on the serverless ACI infrastructure.

2.  **Observe Provisioning Time:** The instructor notes that the very first pod scheduled on a Virtual Node can take a little longer to start (1-2 minutes) as the underlying infrastructure is provisioned. Subsequent pods, and especially scaling operations, are much faster.

3.  **Access the Application:**
    ```bash
    kubectl get svc
    ```
    Copy the `EXTERNAL-IP` of the `app1-nginx-loadbalancer-service` and access `http://<EXTERNAL-IP>/app1/index.html` in your browser. The application should be accessible as usual.

---

## 🚀 Step 4: Testing the Scalability of Virtual Nodes

One of the primary benefits of Virtual Nodes is the ability to rapidly scale to handle bursty workloads without being limited by the capacity of your VM-based node pool.

1.  **Scale the Deployment:** Let's scale our deployment from 2 to 10 replicas.
    ```bash
    kubectl scale deployment/app1-nginx-deployment --replicas=10
    ```
2.  **Observe the Scaling Speed:** Immediately check the pods.
    ```bash
    kubectl get pods -o wide
    ```
    You will see 8 new pods being created. They will quickly transition from `Pending` to `ContainerCreating` to `Running`, often in less than a minute.

**The Key Advantage:**
-   If you were using a regular node pool with limited capacity, scaling this aggressively would fail. Kubernetes would see there aren't enough resources (CPU/memory) on the existing VMs. It would then trigger the **cluster autoscaler** to provision *new VMs* for the node pool. This process is slow, often taking several minutes.
-   With Virtual Nodes, the scalability is effectively "unlimited" and much faster, as you are leveraging the massive, on-demand capacity of the underlying Azure Container Instances platform. You can scale to hundreds of pods almost instantly.

---

## 🧹 Step 5: Cleaning Up

```bash
kubectl delete -f kube-manifests/
```
This command will delete the `Deployment` and the `Service`. The pods running on the Virtual Nodes will be terminated, and the underlying ACI resources will be de-provisioned.

> [!warning] Do not delete the cluster itself. The Virtual Node configuration will be used in subsequent lectures.

---

> [!summary] Conclusion
> Azure Virtual Nodes provide a powerful, serverless extension to your AKS cluster. By using a simple `nodeSelector`, you can offload bursty or unpredictable workloads to ACI, gaining near-infinite, on-demand scalability without the overhead of managing and paying for idle VMs in your node pool. This is an ideal solution for handling spiky traffic or running short-lived, resource-intensive jobs.