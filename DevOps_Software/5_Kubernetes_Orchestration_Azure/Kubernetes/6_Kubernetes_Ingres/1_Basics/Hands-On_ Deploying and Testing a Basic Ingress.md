#Cloud #Azure #Kubernetes #AKS #Networking #Ingress #IngressController #Nginx #HandsOn #Tutorial

>  This is a hands-on guide that puts all the pieces together. We will deploy a sample Nginx application, its internal `ClusterIP` Service, and the basic [[Writing Kubernetes Manifests The Basic Ingress|`Ingress` resource]] we created. We will then verify the end-to-end traffic flow by checking the Ingress Controller's logs and accessing the application through the controller's public IP address.

---

This guide follows the final steps of the "Ingress Basic" section, assuming the [[Installing the Nginx Ingress Controller on AKS|Nginx Ingress Controller]] is already installed and running.

## 🛠️ Step 1: Deploy the Application and Ingress Manifests

We will apply all the manifests for our application and its routing rules at once.

1.  **Navigate to the Project Directory:**
    ```bash
    # Navigate to the 09-Ingress-Basic directory
    cd 09-Ingress-Basic/
    ```
2.  **Apply All Manifests:** Use `kubectl apply -f <directory>` to create all the resources defined in the `kube-manifests/` folder.
    ```bash
    kubectl apply -f kube-manifests/
    ```
    **Output:**
    ```
    deployment.apps/nginx-app1-deployment created
    service/app1-clusterip-service created
    ingress.networking.k8s.io/nginx-app1-ingress-service created
    ```
    This command creates the `Deployment` for our application, its internal `ClusterIP` Service, and the `Ingress` resource that defines the routing rule.

---

## 🔬 Step 2: Verify the Deployment and Observe the Ingress Controller

Now, we will verify that all our components are running and, most importantly, observe how the Ingress Controller reacts to the new `Ingress` resource.

### A. Verify Kubernetes Resources
-   **Check the Pods:**
    ```bash
    kubectl get pods
    ```
    You should see your `nginx-app1-deployment-...` pod in the `Running` state.
-   **Check the Services:**
    ```bash
    kubectl get svc
    ```
    You should see your `app1-clusterip-service` listed.
-   **Check the Ingress Resource:**
    ```bash
    kubectl get ingress
    ```
    **Initial Output:**
    ```
    NAME                         CLASS    HOSTS   ADDRESS   PORTS   AGE
    nginx-app1-ingress-service   <none>   *                 80      15s
    ```
    Notice that the `ADDRESS` field is initially **empty**. It takes the Ingress Controller a moment to process the new Ingress resource and for the cloud provider to associate the IP.

### B. Observe the Ingress Controller Logs (The Magic)
This is the most insightful step. We will watch the logs of the Ingress Controller pod to see it dynamically reconfigure itself in response to our new `Ingress` resource.

1.  **Find the Ingress Controller Pod:**
    ```bash
    kubectl get pods -n ingress-basic
    ```
    Copy the name of one of the `ingress-nginx-controller-...` pods.
2.  **Stream the Logs:**
    ```bash
    kubectl logs -f <ingress-controller-pod-name> -n ingress-basic
    ```
3.  **Analyze the Log Output:** You will see a sequence of events:
    -   `Event(v1.ObjectReference{...}): type: 'Normal' reason: 'Sync' Ingress default/nginx-app1-ingress-service`
    -   `Changes detected, backend reloaded.`
    -   `Successfully reloaded ...`
    -   `updating Ingress default/nginx-app1-ingress-service status from [] to [{IP: "52.123.45.67", Hostname:""}]`
    This log trail clearly shows the Ingress Controller detecting the new `Ingress` resource, reloading its Nginx configuration to apply the new routing rule, and then updating the `status` of the `Ingress` resource with its public IP address.

### C. Verify the Ingress Resource Again
After the controller has finished its work (usually within a minute), check the Ingress resource again.
```bash
kubectl get ingress
```
**Final Output:**
```
NAME                         CLASS    HOSTS   ADDRESS          PORTS   AGE
nginx-app1-ingress-service   <none>   *       52.123.45.67     80      1m
```
The `ADDRESS` field is now populated with the public IP of the Ingress Controller, confirming that the rule is active.

---

## ✅ Step 3: Access and Test the Application

1.  Copy the `ADDRESS` from the `kubectl get ingress` command. This is the static public IP of your Ingress Controller.
2.  Access this IP address in your browser.
    -   **Root Path (`/`):** Navigating to `http://<INGRESS_IP>/` will show the default "Welcome to nginx!" page from our `nginx-app1` application.
    -   **Sub-Path (`/app1/index.html`):** Even though our rule was for the root path (`/`), the `pathType: Prefix` means it also matches sub-paths. Accessing `http://<INGRESS_IP>/app1/index.html` will also be routed to our application and display its content.

This confirms that the entire routing flow is working correctly.

---

## 🧹 Step 4: Cleaning Up

It is a crucial best practice to delete all the resources you created to avoid ongoing cloud costs.

```bash
kubectl delete -f kube-manifests/
```
This single command will delete the `Deployment`, the `ClusterIP` Service, and the `Ingress` resource.

> [!info] The Ingress Controller itself and its static public IP remain. You would need to run `helm uninstall ingress-nginx -n ingress-basic` and delete the Azure Public IP resource to fully clean up the ingress infrastructure.

---

> [!summary] Additional References
> -   **Ingress Annotation Reference:** For advanced configurations (like cookie affinity, custom timeouts, etc.), you can add more annotations to your Ingress manifest. The official documentation for the Nginx Ingress Controller provides a [comprehensive list of available annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/).
> -   This basic example sets the stage for more advanced routing scenarios, such as context-path and hostname-based routing, which will be covered in upcoming lectures.