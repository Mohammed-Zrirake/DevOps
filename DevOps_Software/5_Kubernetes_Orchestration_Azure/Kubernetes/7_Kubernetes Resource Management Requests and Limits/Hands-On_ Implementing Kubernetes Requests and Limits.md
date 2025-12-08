#DevOps #Containerization #Kubernetes #CoreConcept #ResourceManagement #Requests #Limits #HandsOn #Tutorial

>  This is a hands-on guide to applying **[[Kubernetes Resource Management Requests and Limits|resource requests and limits]]** to a [[The Kubernetes Deployment|Deployment]] manifest. We will add a `resources` block to our container definition, deploy it to the cluster, and then use `kubectl describe pod` to verify that the specified CPU and memory constraints have been successfully applied to the running Pod.

---

This guide follows the practical implementation of the concepts discussed in the previous lecture.

## ✅ Step 1: Prerequisite Checks

Before deploying, it's good practice to ensure your environment is ready.
-   **Confirm Cluster Access:** Make sure your `kubectl` is configured to communicate with your [[Azure Kubernetes Service (AKS)|AKS cluster]]. If you have a new terminal or a new cluster, you may need to run `az aks get-credentials` again.
-   **Check Worker Nodes:** Ensure you have at least one healthy worker node in your cluster.
    ```bash
    kubectl get nodes
    ```

## ✍️ Step 2: Reviewing the Manifests

For this demo, we will use a simple Nginx deployment defined in the `kube-manifests-v1/` folder.

### The `nginx-deployment.yaml` with Resources
This is a standard Deployment manifest, but with the addition of the crucial `resources` block at the container level.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1-nginx-deployment
spec:
  replicas: 1
  template:
    # ... (pod metadata and labels) ...
    spec:
      containers:
      - name: app1-nginx
        image: stacksimplify/kube-nginx:1.0.0
        ports:
        - containerPort: 80
        
        # This is the key section for resource management
        resources:
          # 'requests' defines the guaranteed amount of resources.
          # Used by the scheduler for placing the pod.
          requests:
            cpu: "100m"      # 100 millicores (0.1 CPU cores)
            memory: "128Mi"  # 128 Mebibytes
            
          # 'limits' defines the maximum amount of resources the container can use.
          # Enforced by the Kubelet.
          limits:
            cpu: "200m"      # 200 millicores (0.2 CPU cores)
            memory: "256Mi"  # 256 Mebibytes
```
-   **Container-Level Resource:** It is critical to understand that `requests` and `limits` are defined per-container, not per-pod. If you have a multi-container pod, each container would have its own `resources` block.

The `loadbalancer-service.yaml` manifest remains a standard `LoadBalancer` Service to expose this deployment.

---

## 🛠️ Step 3: Deploying and Verifying the Resources

1.  **Deploy the Manifests:**
    ```bash
    # Navigate to the section 15 directory
    kubectl apply -f kube-manifests-v1/
    ```
    This will create the `Deployment` and the `Service`.

2.  **Verify Pod Creation:**
    ```bash
    kubectl get pods
    ```
    Wait for the `app1-nginx-deployment-...` pod to enter the `Running` state.

3.  **Inspect the Pod with `kubectl describe` (The Critical Verification Step):**
    This is how you confirm that your resource settings have been applied correctly.
    ```bash
    kubectl describe pod <pod-name>
    ```
    Scroll down to the `Containers` section in the output. You will see a dedicated section for the applied resources:
    ```
    Containers:
      app1-nginx:
        ...
        Limits:
          cpu:     200m
          memory:  256Mi
        Requests:
          cpu:        100m
          memory:     128Mi
        ...
    ```
    Seeing this output confirms that the Kubernetes `kubelet` is aware of and will enforce these requests and limits for the running container.

---

## ✅ Step 4: Accessing the Application

1.  **Get the Service's Public IP:**
    ```bash
    kubectl get svc
    ```
    Copy the `EXTERNAL-IP` for the `app1-nginx-loadbalancer` service.

2.  **Test in Browser:**
    -   Access the IP in your browser. You will see the default Nginx welcome page.
    -   Access `http://<EXTERNAL-IP>/app1/index.html`. You will see the "App1" welcome page.

This confirms that the application is running correctly with the resource constraints applied.

---

## 🧹 Step 5: Cleaning Up

```bash
kubectl delete -f kube-manifests-v1/
```
This command will delete the `Deployment` and the `Service`, and the cloud provider will automatically de-provision the public IP address.

---

> [!tip] **Assignment: Apply to a Real Application**
> The instructor provides a `kube-manifests-v2/` folder as an assignment. This contains the manifests for the more complex User Management Spring Boot application. The task is to:
> 1.  Analyze the application and decide on appropriate `requests` and `limits` for CPU and memory.
> 2.  Add the `resources` block to the `user-management-webapp-deployment.yaml`.
> 3.  Deploy all the manifests and verify that the resource constraints have been correctly applied using `kubectl describe pod`.