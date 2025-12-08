#Cloud #Azure #Kubernetes #AKS #Scaling #Autoscaling #ClusterAutoscaler #HandsOn #Tutorial

>  This is a hands-on guide to testing the **Cluster Autoscaler** feature in [[Azure Kubernetes Service (AKS)|AKS]]. The Cluster Autoscaler automatically adjusts the number of **worker nodes** in a node pool. We will simulate a resource shortage by scaling up the number of [[The Kubernetes Pod|Pods]] in a [[The Kubernetes Deployment|Deployment]] beyond the capacity of the current nodes. This will trigger the Cluster Autoscaler to **scale out** (add new nodes). We will then scale the pods back down to observe the Cluster Autoscaler **scale in** (remove the underutilized nodes).

---

This guide follows the practical implementation of the concepts discussed in the previous lecture. It assumes you have an AKS cluster created with the autoscaler enabled for its default node pool.

## ✍️ Step 1: Reviewing the Application Manifests

For this demo, we will deploy a simple Spring Boot "Hello World" application. The critical part of this application's `Deployment` manifest is the `resources` block.

### The `deployment.yaml` with Resource Requests
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ca-java-app-deployment # ca = Cluster Autoscaler
  labels:
    app: ca-java-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ca-java-app
  template:
    metadata:
      labels:
        app: ca-java-app
    spec:
      containers:
      - name: ca-java-app-container
        image: stacksimplify/kube-helloworld:1.0.0
        ports:
        - containerPort: 8080
        # This resources block is the key to our simulation
        resources:
          requests:
            cpu: "200m"
            memory: "512Mi"
          limits:
            cpu: "400m"
            memory: "1Gi"
```
-   **The Simulation Strategy:** By defining significant [[Kubernetes Resource Management Requests and Limits|resource requests]] (`200m` CPU and `512Mi` memory), we ensure that each pod reserves a substantial chunk of a node's resources. When we scale the number of replicas to a high number (e.g., 20), the sum of all the requested resources will exceed the capacity of the single running node, forcing pods into a `Pending` state and triggering a scale-out event.

---

## 🛠️ Step 2: Deploy the Application and Verify Initial State

1.  **Prerequisite Check:** Ensure your `kubectl` is configured to your autoscaling-enabled cluster and that it is running.
    ```bash
    kubectl get nodes
    ```
    You should see one node in the `Ready` state.

2.  **Deploy the Application:**
    ```bash
    # Navigate to the 22-01 directory
    kubectl apply -f kube-manifests/
    ```
    This creates the `Deployment` and its `LoadBalancer` Service.

3.  **Verify the Initial State:**
    -   `kubectl get pods`: You will see one pod running.
    -   `kubectl get svc`: Get the `EXTERNAL-IP` of the service.
    -   **Test the App:** Access `http://<EXTERNAL-IP>/hello` in your browser or with `curl`. You should see the "Hello World V1" response, which includes the unique ID of the pod that served the request. This confirms the application is working correctly.

---

## 🚀 Step 3: Trigger a Scale-Out Event

Now, let's force the Cluster Autoscaler to add more nodes.

1.  **Scale the Deployment:** Use `kubectl scale` to dramatically increase the number of replicas from 1 to 20.
    ```bash
    kubectl scale deployment/ca-java-app-deployment --replicas=20
    ```

2.  **Observe the Pods:** Immediately check the status of your pods.
    ```bash
    kubectl get pods
    ```
    **Output:**
    You will see one or two pods in the `Running` state, and the remaining 18-19 pods will be in the **`Pending`** state.
    
    If you `describe` one of the pending pods (`kubectl describe pod <pod-name>`), the `Events` section will show a message like: `FailedScheduling... 0/1 nodes are available: 1 Insufficient cpu, 1 Insufficient memory.` This is exactly what we want. The scheduler has determined there are not enough resources on the existing node to place these new pods.

3.  **Observe the Nodes (The Magic Moment):**
    Now, watch the nodes in your cluster. This process can take **2-3 minutes**.
    ```bash
    # Run this command every 30 seconds or so
    kubectl get nodes
    ```
    -   Initially, you'll see one node.
    -   After a minute or two, you will see new nodes appear in the `NotReady` state.
    -   Shortly after, they will transition to the `Ready` state. The instructor's demo scaled out to **4 nodes** to accommodate the 20 pods.
    
4.  **Final Verification:**
    -   `kubectl get pods`: All 20 pods will now be in the `Running` state, as they have been scheduled onto the newly provisioned nodes.
    -   **Test the App Again:** If you access the application with `curl` multiple times, you will see responses from many different pod IDs, confirming that the load balancer is distributing traffic across all 20 running instances.

---

## ⏪ Step 4: Trigger a Scale-In Event

The Cluster Autoscaler also removes nodes that are underutilized.

1.  **Scale Down the Deployment:** Scale the application all the way back down to a single replica.
    ```bash
    kubectl scale deployment/ca-java-app-deployment --replicas=1
    ```

2.  **Observe the Pods:**
    ```bash
    kubectl get pods
    ```
    You will see 19 of the pods enter the `Terminating` state, leaving only one `Running`.

3.  **Observe the Nodes (Patience is Key):**
    The scale-in process is intentionally slower than scaling out to avoid disrupting workloads. It can take **5 to 20 minutes** for the Cluster Autoscaler to identify and remove the now-empty, underutilized nodes.
    ```bash
    # Run this command every few minutes
    kubectl get nodes
    ```
    -   You will see the node count decrease from 4, perhaps to 2, and then finally back down to the minimum of **1 node**.

---

## 🧹 Step 5: Cleaning Up

```bash
kubectl delete -f kube-manifests/
```
This will delete the `Deployment` and the `LoadBalancer` Service.

---

> [!summary] Conclusion
> This hands-on exercise demonstrates the power and behavior of the **AKS Cluster Autoscaler**. We have shown that by simply increasing the resource demands of our workloads (by scaling the number of pods), the cluster can automatically provision new worker nodes to meet that demand (scale-out). Conversely, when the demand decreases, the autoscaler automatically removes the unneeded nodes to save costs (scale-in). This is a foundational feature for building elastic, cost-efficient applications on Kubernetes.