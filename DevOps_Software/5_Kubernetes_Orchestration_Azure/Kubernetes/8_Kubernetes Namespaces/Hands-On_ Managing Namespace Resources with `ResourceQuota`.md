#DevOps #Containerization #Kubernetes #CoreConcept #Namespaces #ResourceManagement #ResourceQuota #HandsOn #Tutorial

>  This is a hands-on guide to implementing a **`ResourceQuota`** in a [[Kubernetes Namespaces Virtual Clusters|Namespace]]. A `ResourceQuota` is a powerful administrative tool that provides constraints on the **total amount of resources** that can be consumed by all objects within a specific namespace. It allows cluster administrators to ensure fair resource sharing and prevent a single team or project from monopolizing cluster resources.

---

## ✍️ Reviewing the `ResourceQuota` Manifest

In our `kube-manifests/` directory, we have a unified file (`00-namespace-limitrange-resourcequota.yaml`) that defines our Namespace, a `LimitRange`, and our `ResourceQuota`. Let's focus on the `ResourceQuota` section.

```yaml
# ... (Namespace and LimitRange definitions above) ...
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ns-resource-quota
  namespace: dev3 # This quota applies ONLY to the 'dev3' namespace
spec:
  hard:
    # --- Compute Resource Quotas ---
    # Total sum of all container requests for CPU cannot exceed 1 core
    requests.cpu: "1" 
    # Total sum of all container limits for CPU cannot exceed 2 cores
    limits.cpu: "2"
    # Total sum of all container requests for memory cannot exceed 1 GiB
    requests.memory: 1Gi
    # Total sum of all container limits for memory cannot exceed 2 GiB
    limits.memory: 2Gi
    
    # --- Object Count Quotas ---
    # Maximum number of Pods that can exist in this namespace
    pods: "5"
    # Maximum number of Services
    services: "5"
    # Maximum number of ConfigMaps
    configmaps: "5"
    # Maximum number of PersistentVolumeClaims
    persistentvolumeclaims: "5"
```
-   **`namespace: dev3`**: It's critical to understand that a `ResourceQuota` is a **namespaced object**. This quota will only apply to resources created within the `dev3` namespace.
-   **`spec.hard`**: This defines the hard limits. Once a limit is reached, users will be forbidden from creating new resources of that type in the namespace.
-   **Compute Quotas:** These are aggregate limits. For example, `requests.cpu: "1"` means the sum of the CPU requests of *all pods* in the namespace cannot exceed 1 CPU core.
-   **Object Count Quotas:** These are simple counts on the number of objects of a certain type that can exist in the namespace.

---

## 🛠️ Deploying and Verifying the `ResourceQuota`

### Step 1: Deploy All Manifests
1.  Clean up any resources from the previous section (`16-02`).
2.  Navigate to the directory for this section (`16-03-namespaces-resource-quota/`).
3.  Apply all the manifests. This will create the `dev3` namespace, the `LimitRange`, the `ResourceQuota`, and then deploy our sample Nginx application into that namespace.
    ```bash
    kubectl apply -f kube-manifests/
    ```

### Step 2: Verify the `ResourceQuota`
1.  **Get the Quota:**
    ```bash
    kubectl get quota -n dev3
    ```
    This gives a quick summary. To see the details, we must `describe` it.

2.  **Describe the Quota (The Critical Verification Step):**
    ```bash
    kubectl describe quota ns-resource-quota -n dev3
    ```
    **Output Analysis:**
    This command provides a detailed breakdown of the quota, showing the `Hard` limit, and the current `Used` amount for each resource.
    ```
    Name:         ns-resource-quota
    Namespace:    dev3
    Resource      Used  Hard
    --------      ----  ----
    configmaps    0     5
    limits.cpu    500m  2
    limits.memory 512Mi 2Gi
    pods          1     5
    requests.cpu  300m  1
    ...
    ```
    This tells us that our single running pod is currently `Used` 300 millicores of its `requests.cpu` quota and 500 millicores of its `limits.cpu` quota.

---

## 💥 Step 3: Testing the Quota Limits in Action

Now, let's intentionally try to exceed the quota to see how Kubernetes enforces it.

1.  **Attempt to Scale the Deployment:** We will try to scale our `app1-nginx-deployment` to `6` replicas. Our `pods` quota has a hard limit of `5`.
    ```bash
    kubectl scale deployment/app1-nginx-deployment --replicas=6 -n dev3
    ```

2.  **Observe the Result:**
    -   **Check the Pods:**
        ```bash
        kubectl get pods -n dev3
        ```
        You will observe that only `5` pods are created and running. The 6th pod is not created.
    -   **Check the Deployment:**
        ```bash
        kubectl get deployments -n dev3
        ```
        The output will show that the deployment is not in a fully healthy state:
        ```
        NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
        app1-nginx-deployment   5/6     5            5           5m
        ```
        It shows that only 5 out of the 6 desired replicas are `READY`.
    -   **Describe the Deployment (Find the Reason):**
        ```bash
        kubectl describe deployment app1-nginx-deployment -n dev3
        ```
        In the `Events` section at the bottom, you will see an event with a reason like `FailedCreate` and a message similar to: **`Error creating: pods "app1-nginx-deployment-..." is forbidden: exceeded quota: ns-resource-quota, requested: pods=1, used: pods=5, limited: pods=5`**.

This event log is the proof that the `ResourceQuota` is working. Kubernetes' admission controller blocked the creation of the 6th pod because it would have violated the `pods: "5"` hard limit defined for the `dev3` namespace.

The instructor also points out that even if the pod quota was higher, we would have eventually been blocked by the aggregate CPU or memory quotas. For example, if we scaled to 4 pods, our `requests.cpu` usage would be `4 * 300m = 1200m`, which exceeds the hard limit of `1` (1000m), and the 4th pod would fail to be scheduled.

---

## 🧹 Step 4: Cleaning Up

1.  **(Optional) Scale Down First:** Before cleaning up, it's good practice to scale the deployment back down.
    ```bash
    kubectl scale deployment/app1-nginx-deployment --replicas=1 -n dev3
    ```
2.  **Delete the Namespace:** The easiest way to clean up is to delete the entire namespace, which will cascade-delete all the resources within it (Deployment, Service, LimitRange, ResourceQuota, etc.).
    ```bash
    kubectl delete namespace dev3
    ```

---

> [!summary] Conclusion
> This hands-on exercise demonstrates the power and necessity of `ResourceQuota` in a multi-tenant Kubernetes environment. By setting hard limits on resource consumption at the namespace level, cluster administrators can:
> -   Ensure fair resource sharing between teams and projects.
> -   Prevent a single namespace from destabilizing the entire cluster.
> -   Provide predictable resource availability for all tenants.
> 
> This is a key tool for effective cluster governance and management.