#DevOps #Containerization #Kubernetes #CoreConcept #Namespaces #Isolation #kubectl #Imperative #HandsOn #Tutorial

>  This is a hands-on guide to creating and managing [[Kubernetes Namespaces Virtual Clusters|Namespaces]] using imperative `kubectl` commands. We will create new namespaces, deploy the same application manifests into multiple, isolated namespaces, access each isolated instance via its own `LoadBalancer` Service, and finally, clean up the resources by deleting the namespaces themselves.

---

This guide follows the "imperative" approach, where we directly tell Kubernetes what to do via commands, before moving on to the declarative (YAML) approach in later sections.

## 🔬 Step 1: Exploring Existing Namespaces

Before creating our own, let's inspect the namespaces that exist in a default Kubernetes cluster.
```bash
kubectl get namespaces
# Alias: kubectl get ns
```
**Output:**
```
NAME              STATUS   AGE
default           Active   1h
kube-system       Active   1h
kube-public       Active   1h
kube-node-lease   Active   1h
```
-   **`kube-system`**: This is where the core Kubernetes control plane components run. You can inspect them with `kubectl get pods -n kube-system`. This will show pods for networking (like Azure CNI), DNS, the dashboard, metrics server, etc.
-   **`default`**: If you create a resource without specifying a namespace, it goes into the `default` namespace. So far, all our work has been done here. `kubectl get all` will show any resources in this namespace.
-   **`kube-public` & `kube-node-lease`**: These are used for internal cluster functions and can generally be ignored.

> [!danger] **Never Delete Default Namespaces**
> You should never delete the system-created namespaces (`default`, `kube-system`, etc.), as this can break your cluster.

---

## 🛠️ Step 2: Creating New Namespaces Imperatively

We will create two new namespaces, `dev1` and `dev2`, to represent two different environments.

```bash
# Create the first namespace
kubectl create namespace dev1

# Create the second namespace using the alias 'ns'
kubectl create ns dev2
```
Now, verify their creation:
```bash
kubectl get ns
```
You will see `dev1` and `dev2` in the list.

---

## 🚀 Step 3: Deploying Applications into Specific Namespaces

The key to deploying resources into a specific namespace is the `-n` (or `--namespace`) flag. We will deploy the same set of application manifests (`Deployment` and `LoadBalancer` Service) into three different namespaces: `default`, `dev1`, and `dev2`.

1.  **Navigate to the Project Directory:**
    ```bash
    # Navigate to the 16-01-namespaces-imperative directory
    cd 16-01-namespaces-imperative/
    ```
2.  **Deploy to `dev1`:**
    ```bash
    kubectl apply -f kube-manifests/ -n dev1
    ```
3.  **Deploy to `dev2`:**
    ```bash
    kubectl apply -f kube-manifests/ -n dev2
    ```
4.  **Deploy to `default`:**
    ```bash
    # The -n flag is not needed for the 'default' namespace
    kubectl apply -f kube-manifests/
    ```

---

## 🔍 Step 4: Verifying and Accessing the Isolated Applications

We now have three identical but completely isolated instances of our application running in the same cluster.

### A. Listing Resources in Each Namespace
To view the resources in a specific namespace, you must use the `-n` flag with your `get` commands.

-   **Check `default` namespace:**
    ```bash
    kubectl get pods,svc,deploy
    ```
-   **Check `dev1` namespace:**
    ```bash
    kubectl get pods,svc,deploy -n dev1
    ```
-   **Check `dev2` namespace:**
    ```bash
    kubectl get pods,svc,deploy -n dev2
    ```
Each command will show a separate `Deployment`, `Pod`, and `Service` for the Nginx application, each scoped to its own namespace.

### B. Accessing Each Application
Each `LoadBalancer` Service we created will get its own unique, public IP address.

1.  **Get the External IPs:**
    ```bash
    # Get IP for dev1
    kubectl get svc -n dev1
    
    # Get IP for dev2
    kubectl get svc -n dev2
    
    # Get IP for default
    kubectl get svc
    ```
2.  **Test in Browser:**
    -   Access `http://<dev1-external-ip>/app1/index.html`. You will see the App1 page.
    -   Access `http://<dev2-external-ip>/app1/index.html`. You will see the App1 page.
    -   Access `http://<default-external-ip>/app1/index.html`. You will see the App1 page.

This confirms that we have three fully independent, publicly accessible instances of the same application running within our single cluster, neatly isolated by namespaces.

---

## 🧹 Step 5: Cleaning Up

### A. Deleting Namespaces
The easiest way to clean up is to delete the entire namespace. This will **cascade-delete all resources** (Pods, Deployments, Services, etc.) contained within it.

```bash
# Delete the dev1 namespace and all its contents
kubectl delete namespace dev1

# Delete the dev2 namespace and all its contents
kubectl delete ns dev2
```
The deletion process can take a moment as the cloud provider de-provisions the load balancers.

### B. Deleting Resources in the `default` Namespace
> [!danger] As stated before, **DO NOT** delete the `default` namespace itself.

To clean up the resources in the `default` namespace, we must delete them using their manifest files.
```bash
kubectl delete -f kube-manifests/
```

---

> [!summary] Conclusion
> This hands-on exercise demonstrates the power and simplicity of using namespaces for multi-tenancy and environment isolation. By using the `-n` flag, we can deploy and manage identical application stacks in parallel without naming conflicts. Deleting a namespace provides a powerful and convenient way to tear down an entire environment and all of its associated resources.