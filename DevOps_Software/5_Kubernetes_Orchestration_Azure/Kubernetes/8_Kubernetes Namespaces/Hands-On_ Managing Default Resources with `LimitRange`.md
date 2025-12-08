#DevOps #Containerization #Kubernetes #CoreConcept #Namespaces #ResourceManagement #LimitRange #Declarative #HandsOn #Tutorial

>  This is a hands-on guide to implementing a **`LimitRange`** in Kubernetes. A `LimitRange` is a policy object for a [[Kubernetes Namespaces Virtual Clusters|Namespace]] that provides constraints on resource consumption. Its most common use case is to automatically assign **default [[Kubernetes Resource Management Requests and Limits|resource requests and limits]]** to any container that is created within the namespace *without* its own explicit resource definitions.

---

This guide follows the declarative approach, where we define our `Namespace` and `LimitRange` using YAML manifests.

## ✍️ Step 1: Reviewing the Manifests

The project is structured with multiple YAML files, ordered with a numeric prefix to guide the deployment sequence.

### A. `00-namespace.yaml`
Unlike the imperative `kubectl create namespace` command, we will define our namespace declaratively.
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev3
```

### B. `01-limit-range-default.yaml`
This is the core manifest for this lecture. It defines the default resource constraints for the `dev3` namespace.
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-cpu-mem-limit-range
  # This LimitRange will apply to the 'dev3' namespace
  namespace: dev3
spec:
  limits:
  - type: Container # This policy applies to individual containers
    # 'defaultRequest' sets the resource requests if a container does not specify its own.
    defaultRequest:
      memory: "256Mi"
      cpu: "300m"
    # 'default' sets the resource limits if a container does not specify its own.
    default:
      memory: "512Mi"
      cpu: "500m"
```
-   **`namespace: dev3`**: This is critical. A `LimitRange` is a namespaced object; this line ensures the policy is applied only to the `dev3` namespace.
-   **`spec.limits`**: A list of constraints.
-   **`type: Container`**: This constraint applies to all containers within the namespace.
-   **`defaultRequest`**: If a developer deploys a pod to this namespace and doesn't specify a `resources.requests` block for a container, this `LimitRange` will **automatically inject** a request of `256Mi` of memory and `300m` of CPU.
-   **`default`**: Similarly, if a container doesn't specify a `resources.limits` block, this `LimitRange` will **automatically inject** a limit of `512Mi` of memory and `500m` of CPU.

### C. `02-deployment.yaml` and `03-loadbalancer-service.yaml`
These are standard manifests for an Nginx deployment and a service, with two key modifications:

1.  **No `resources` Block:** The `Deployment` manifest **intentionally omits** the `resources` section for the container. We are doing this to test whether our `LimitRange` correctly injects the default values.
2.  **`namespace: dev3`:** Both the `Deployment` and the `Service` manifests include `metadata.namespace: dev3` to ensure they are created in our new, policy-controlled namespace.

---

## 🛠️ Step 2: Deploying and Verifying the `LimitRange`

1.  **Prerequisite Check:** Ensure your `kubectl` is configured and can connect to your cluster (`kubectl get nodes`).
2.  **Deploy All Manifests:**
    ```bash
    # Navigate to the 16-02 directory
    kubectl apply -f kube-manifests/
    ```
    This will create the `dev3` namespace, the `LimitRange` object within it, and then the `Deployment` and `Service`.

3.  **Verify the `LimitRange` Object:**
    ```bash
    # Get the LimitRange in the dev3 namespace
    kubectl get limits -n dev3
    
    # Describe it to see the details
    kubectl describe limits default-cpu-mem-limit-range -n dev3
    ```
    The describe output will clearly show the default request and limit values that you defined in your manifest.

4.  **Verify the Pod's Resources (The Magic Moment):**
    This is the most important verification step. We need to check if our pod, which had no resource definitions in its manifest, received the defaults from the `LimitRange`.
    ```bash
    # First, get the pod name in the dev3 namespace
    kubectl get pods -n dev3
    
    # Then, describe that pod
    kubectl describe pod <pod-name> -n dev3
    ```
    In the output, under the `Containers` section, you will see that the `resources` have been **automatically applied**:
    ```
    Containers:
      app1-nginx:
        ...
        Limits:
          cpu:     500m
          memory:  512Mi
        Requests:
          cpu:        300m
          memory:     256Mi
        ...
    ```
    This confirms that the `LimitRange` policy is working correctly. It intercepted the pod creation request and injected the default resource requests and limits because none were explicitly provided in the deployment's pod template.

5.  **Access the Application:**
    ```bash
    kubectl get svc -n dev3
    ```
    Get the `EXTERNAL-IP` of the load balancer and access the application in your browser to confirm it's running as expected.

---

> [!summary] Conclusion
> The `LimitRange` object is a powerful administrative tool for enforcing resource management policies across a namespace. By setting default requests and limits, you ensure that even applications deployed without explicit resource definitions are still scheduled intelligently and are prevented from consuming excessive resources. This improves overall cluster stability and resource fairness in a multi-tenant environment. In the next lecture, we will explore the `ResourceQuota` object, which sets aggregate resource consumption limits for an entire namespace.