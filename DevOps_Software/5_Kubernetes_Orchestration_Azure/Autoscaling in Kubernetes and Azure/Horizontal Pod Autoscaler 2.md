#Cloud #Azure #Kubernetes #AKS #Scaling #Autoscaling #HPA #MetricsServer #HandsOn #Tutorial

>  The **Horizontal Pod Autoscaler (HPA)** is a Kubernetes controller that automatically scales the number of [[The Kubernetes Pod|Pods]] in a [[The Kubernetes Deployment|Deployment]] or [[The Kubernetes ReplicaSet|ReplicaSet]] based on observed metrics, most commonly **CPU utilization**. If the average CPU usage across all pods exceeds a target threshold, the HPA will **increase the number of replicas** (scale-out). If the usage drops below the target, it will **decrease the number of replicas** (scale-in).

---

## 🛠️ Step 1: Deploying the Sample Application

Before we can autoscale an application, we need an application to scale. We will deploy a simple Nginx application with resource requests and limits defined, which is a prerequisite for CPU-based autoscaling.

### The Application Manifests (`kube-manifests/apps/`)
-   **`deployment.yaml`:** A standard `Deployment` for a simple Nginx application (`hpa-demo-deployment`).
    -   **Crucially**, it includes a `resources` block with `requests` and `limits` for CPU and memory. The HPA uses the `requests` value as the baseline for calculating utilization percentage.
        ```yaml
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
        ```
-   **`service.yaml`:** A standard `LoadBalancer` Service to expose the application to the internet.

### Deployment Commands
1.  **Navigate to the correct directory:**
    ```bash
    # Go to the section 22-02 directory
    cd 22-02-Azure-AKS-HPA-Horizontal-Pod-Autoscaling/
    ```
2.  **Apply the manifests:**
    ```bash
    kubectl apply -f kube-manifests/apps/
    ```
3.  **Verify the deployment:**
    ```bash
    # Check that the pod is running
    kubectl get pods
    
    # Get the external IP of the service
    kubectl get svc
    ```
4.  **Test in browser:** Access the external IP to confirm the Nginx welcome page is displayed.

---

## 🚀 Step 2: Creating the Horizontal Pod Autoscaler (HPA)

We can create an HPA resource in two ways: imperatively (with a command) or declaratively (with a YAML manifest).

### A. The Imperative Approach (`kubectl autoscale`)
This is a quick way to create an HPA for an existing deployment.

```bash
kubectl autoscale deployment hpa-demo-deployment --cpu-percent=20 --min=1 --max=10
```
**Command Breakdown:**
-   `autoscale deployment hpa-demo-deployment`: We are creating an HPA that targets our `hpa-demo-deployment`.
-   `--cpu-percent=20`: The target CPU utilization. The HPA will try to maintain an average CPU usage of **20% of the pod's requested CPU** across all replicas. (Note: 20% is a very low value chosen specifically for this demo to make it easy to trigger scaling. A real-world value is often 70-80%).
-   `--min=1`: The minimum number of replicas the HPA will maintain.
-   `--max=10`: The maximum number of replicas the HPA can scale out to.

### B. The Declarative Approach (YAML Manifest)
This is the recommended approach for production, as it allows you to version-control your scaling policies.

**The `hpa-declarative.yaml` Manifest:**
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-demo-declarative
spec:
  maxReplicas: 10
  minReplicas: 1
  # 'scaleTargetRef' points to the resource this HPA should manage
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-demo-deployment
  # 'targetCPUUtilizationPercentage' is the scaling threshold
  targetCPUUtilizationPercentage: 20
```
-   **`scaleTargetRef`**: This is the crucial block that links the HPA to the `Deployment` it's supposed to scale.

> [!info] You can use either the imperative command or apply the declarative manifest to create the HPA. The result is the same.

### The Prerequisite: Metrics Server
-   For the HPA to function, it needs a source of resource metrics (like CPU and memory usage) for the pods. This is provided by the **Metrics Server**.
-   **Azure AKS vs. AWS EKS:** The instructor highlights a key difference. In AWS EKS, the Metrics Server is **not** installed by default and you must install it manually. In **Azure AKS**, the Metrics Server **is installed by default** as part of the cluster setup, so HPA works out of the box. You can verify its pods are running in the `kube-system` namespace.

---

## 🔬 Step 3: Verifying the HPA

After creating the HPA, you can inspect its status.

```bash
# Get the HPA object
kubectl get hpa

# Describe the HPA for detailed information
kubectl describe hpa hpa-demo-deployment
```
**`describe` Output:**
```
Name:                                                  hpa-demo-deployment
Namespace:                                             default
...
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  1% / 20%
MinReplicas:                                           1
MaxReplicas:                                           10
Deployment Pods:                                       1 current / 1 desired
Events:                                                <none>
```
-   **`TARGETS: 1% / 20%`**: This is the key status field. It shows that the current average CPU utilization is **1%** of the requested amount, and the target is **20%**. Since the current usage is well below the target, the HPA is not taking any action, and the desired number of pods remains at the minimum of 1.

---

> [!summary] Conclusion
> We have successfully deployed a sample application and configured a **Horizontal Pod Autoscaler (HPA)** to manage its replicas based on CPU utilization. We've also understood the critical role of the **Metrics Server**, which is conveniently provided by default in AKS. The HPA is now active and monitoring our deployment's CPU usage.
>
> In the next lecture, we will:
> 1.  Generate a high CPU load on our application.
> 2.  Watch the `kubectl get hpa` and `kubectl get pods` commands in real-time.
> 3.  Observe as the HPA detects the increased CPU usage, surpasses the 20% target, and automatically scales up the number of pods to a maximum of 10.
> 4.  Stop the load and watch as the HPA scales the pods back down to the minimum of 1.