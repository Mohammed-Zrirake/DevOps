#Cloud #Azure #Kubernetes #AKS #Networking #Ingress #Routing #HandsOn #Tutorial

>  This is a hands-on guide that puts the theory of [[Ingress Context Path-Based Routing|context path-based routing]] into practice. We will deploy three separate applications, each with its own `ClusterIP` Service. Then, we will deploy a single, unified [[Writing Kubernetes Manifests The Basic Ingress|`Ingress` resource]] with multiple path-based rules to route traffic to the correct application based on the URL path.

---

This guide follows the final steps of the "Ingress Context Path-Based Routing" section, assuming the [[Installing the Nginx Ingress Controller on AKS|Nginx Ingress Controller]] is already installed and running.

## 🛠️ Step 1: Deploy All Application and Ingress Manifests

We will deploy all the manifests for our three applications and the unified Ingress resource at once. The project is structured with sub-folders inside `kube-manifests/` to keep the application definitions organized.

### The `kubectl apply -r` Flag
Because our manifests are organized in subdirectories, we need to use the `-r` (or `--recursive`) flag with `kubectl apply`. This tells `kubectl` to process all `.yaml` files found in the specified directory *and all of its subdirectories*.

1.  **Navigate to the Project Directory:**
    ```bash
    # Navigate to the 10-ingress-context-path-based-routing directory
    cd 10-ingress-context-path-based-routing/
    ```
2.  **Apply All Manifests Recursively:**
    ```bash
    kubectl apply -r -f kube-manifests/
    ```
    **Output:**
    This command will create a large number of resources, including:
    -   `app1` Deployment and Service
    -   `app2` Deployment and Service
    -   All resources for the `user-management-webapp` (including the `PVC`, `ConfigMap`, `MySQL` Deployment and Service, `Secret`, etc.)
    -   And finally, our new `ingress-cpr` Ingress resource.

---

## 🔬 Step 2: Verify the Deployment

Now, we will verify that all our components are running correctly.

1.  **Check the Pods:**
    ```bash
    kubectl get pods
    ```
    You should see pods for `nginx-app1`, `nginx-app2`, `mysql`, and the `user-mgmt-webapp` all in the `Running` or `ContainerCreating` state.

2.  **Check the Services:**
    ```bash
    kubectl get svc
    ```
    You will see all the internal `ClusterIP` services for each of the three applications, plus the one for MySQL. There should be no new `LoadBalancer` services created.

3.  **Check the Ingress Resource:**
    ```bash
    kubectl get ingress
    ```
    You will see our new `ingress-cpr` resource listed, and after a minute, its `ADDRESS` field will be populated with the public IP of our Ingress Controller.

---

## ✅ Step 3: Access and Test the Routing

This is the final test to confirm our path-based routing rules are working as expected.

1.  Copy the `ADDRESS` from the `kubectl get ingress` command. This is the single, static public IP of your Ingress Controller.
2.  Test each path in your browser:
    -   **Test App1:** Navigate to `http://<INGRESS_IP>/app1/index.html`.
        -   You should see the "Welcome to NGINX - App1" page.
    -   **Test App2:** Navigate to `http://<INGRESS_IP>/app2/index.html`.
        -   You should see the "Welcome to NGINX - App2" page.
    -   **Test the Default Backend (User Management App):** Navigate to the root path, `http://<INGRESS_IP>/`.
        -   You will be redirected to the login page for the User Management Web Application.
        -   Log in with `admin101` / `password101`.
        -   The application should be fully functional.

This successfully demonstrates that the single Ingress Controller is intelligently routing requests to three completely different backend applications based on the URL path.

---

## 🧹 Step 4: Cleaning Up

It is a crucial best practice to delete all the resources you created to avoid ongoing cloud costs.

```bash
kubectl delete -r -f kube-manifests/
```
-   The `-r` flag works with `delete` as well, ensuring all resources defined in the directory and its subdirectories are deleted.
-   Because the `user-management-webapp` was configured to use one of the default `StorageClass` objects with `reclaimPolicy: Delete`, the underlying Azure Disk for the MySQL database will also be automatically deleted.

After the command completes, you can verify that accessing the public IP now results in a "404 Not Found" from the Nginx Ingress Controller, as there are no longer any routing rules for it to serve.

---

> [!summary]
> This completes the hands-on implementation of context path-based routing. We have successfully deployed a microservices-style architecture where multiple applications are exposed through a single, intelligent entry point. This is a foundational pattern for building and managing complex applications on Kubernetes.