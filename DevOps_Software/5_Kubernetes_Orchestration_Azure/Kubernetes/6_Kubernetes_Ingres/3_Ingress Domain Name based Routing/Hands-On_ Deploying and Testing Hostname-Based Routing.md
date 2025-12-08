#Cloud #Azure #Kubernetes #AKS #Networking #Ingress #Routing #DNS #ExternalDNS #HandsOn #Tutorial

>  This is a hands-on guide that puts the theory of [[Ingress Hostname-Based Routing|hostname-based routing]] into practice. We will deploy three separate applications, each with its own `ClusterIP` Service. Then, we will deploy a single, unified `Ingress` resource with multiple **host-based rules**. We will verify that the **`external-dns`** controller automatically creates the necessary DNS records, and finally test the routing by accessing each application via its unique hostname.

---

This guide follows the final steps of the "Domain Name Based Routing" section. It assumes the [[Installing the Nginx Ingress Controller on AKS|Nginx Ingress Controller]], [[Azure DNS Zones and Domain Delegation|delegated Azure DNS Zone]], and `external-dns` controller are already set up.

## ✍️ Step 1: Reviewing the Manifests

The project structure for this section is organized into sub-folders, one for each application and one for the Ingress resource itself.

-   **Application Manifests:** The folders for `nginx-app1`, `nginx-app2`, and `usermgmt-webapp` contain the standard `Deployment` and `ClusterIP` Service manifests for each application. These are similar to what we've used in previous sections.
-   **Ingress Manifest:** The core of this lecture is the `ingress-domain-name-based-routing.yaml`.

### The `ingress-domain-name-based-routing.yaml` Manifest
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: domain-name-based-routing-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  # Rule 1: For eapp1.kubeoncloud.com
  - host: eapp1.kubeoncloud.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-nginx-clusterip-service
            port:
              number: 80
  # Rule 2: For eapp2.kubeoncloud.com
  - host: eapp2.kubeoncloud.com
    http:
      paths:
      - path: /app2 # Note: This app is served at a sub-path
        pathType: Prefix
        backend:
          service:
            name: app2-nginx-clusterip-service
            port:
              number: 80
  # Rule 3: For eapp3.kubeoncloud.com
  - host: eapp3.kubeoncloud.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: usermgmt-webapp-clusterip-service
            port:
              number: 80
```
-   **`host` key:** Each rule is now explicitly tied to a hostname. The rule will only apply to HTTP requests that have a matching `Host` header.

> [!info] **Legacy API Version (`v1beta1`)**
> The instructor notes that the repository contains a `kube-manifests-old` folder. This is for reference and contains an older version of the Ingress manifest using the deprecated `apiVersion: networking.k8s.io/v1beta1`. We will be using the modern, stable `v1` version.

---

## 🛠️ Step 2: Deploying All Manifests

We will apply all manifests for the applications and the Ingress resource at once using the recursive (`-R` or `-r`) flag.

1.  **Navigate to the Project Directory:**
    ```bash
    cd 13-Ingress-ExternalDNS-Domain-Name-Based-Routing/
    ```
2.  **Apply All Manifests Recursively:**
    ```bash
    kubectl apply -R -f kube-manifests/
    ```
    This command will create all the Deployments, Services, and the Ingress resource.

---

## 🔬 Step 3: Verifying the Deployment and DNS

Now, we will verify that all our components are running and, critically, that `external-dns` has done its job.

1.  **Check the Pods and Services:**
    ```bash
    kubectl get pods
    kubectl get svc
    ```
    Verify that pods and `ClusterIP` services for all three applications are running.

2.  **Check the Ingress Resource:**
    ```bash
    kubectl get ingress
    ```
    You will see our new `domain-name-based-routing-ingress` resource. It will list all three hosts (`eapp1...`, `eapp2...`, `eapp3...`) and will be assigned the public IP address of our Ingress Controller.

3.  **Observe the `external-dns` Logs (The Magic):**
    This is a crucial verification step. Check the logs of the `external-dns` pod.
    ```bash
    # Find the external-dns pod name (it will be in its own namespace if you created one)
    kubectl get pods -n <external-dns-namespace>
    
    # Stream the logs
    kubectl logs -f <external-dns-pod-name> -n <external-dns-namespace>
    ```
    You will see log entries indicating that it has detected the new hosts in the Ingress resource and is creating/updating `A` records in your Azure DNS Zone for `eapp1`, `eapp2`, and `eapp3`.

4.  **Verify DNS Records in Azure Portal and with `nslookup`:**
    -   **Azure Portal:** Navigate to your `kubeoncloud.com` **DNS Zone** in Azure. You will see three new `A` records have been automatically created, all pointing to the static public IP of your Ingress Controller.
    -   **`nslookup`:** Use the command line to confirm that the DNS names resolve correctly.
        ```bash
        nslookup eapp1.kubeoncloud.com
        nslookup eapp2.kubeoncloud.com
        nslookup eapp3.kubeoncloud.com
        ```
        Each command should return the static public IP of your Ingress Controller.

---

## ✅ Step 4: Accessing and Testing the Applications

Now that DNS is resolving correctly, we can test the routing by accessing each hostname in the browser.

1.  **Test App1:** Navigate to `http://eapp1.kubeoncloud.com/app1/index.html`.
    -   You should see the "Welcome to NGINX - App1" page.

2.  **Test App2:** Navigate to `http://eapp2.kubeoncloud.com/app2/index.html`.
    -   You should see the "Welcome to NGINX - App2" page.

3.  **Test the User Management App:** Navigate to `http://eapp3.kubeoncloud.com/`.
    -   You will be redirected to the login page for the User Management Web Application. Log in and test its functionality.

4.  **Test the Restriction:** Try to access content from one app using another app's hostname. For example, navigate to `http://eapp3.kubeoncloud.com/app1/index.html`.
    -   The request will still be routed to the User Management app (and likely result in a 404 or redirect to the login page), **not** to the App1 content. This proves that the Ingress rule for `eapp3.kubeoncloud.com` correctly restricts access to only the `usermgmt-webapp-clusterip-service` backend.

---

## 🧹 Step 5: Cleaning Up

```bash
kubectl delete -R -f kube-manifests/
```
-   The `-R` flag works with `delete`, deleting all resources defined in the directory and its subdirectories.
-   The `external-dns` controller will detect that the Ingress resource has been deleted and will **automatically delete the `A` records** from your Azure DNS Zone, cleaning everything up.
-   The underlying Azure Disk for the MySQL database will also be automatically deleted if you used a `StorageClass` with `reclaimPolicy: Delete`. If not, it will need to be cleaned up manually as shown in previous lectures.

---

> [!summary] Conclusion
> This end-to-end demonstration successfully showcases a sophisticated, production-ready routing pattern. By combining an **Ingress resource with host-based rules** and the **`external-dns` controller**, we have created a fully automated system that can host multiple applications on a single IP address, with DNS records managed automatically. This is a foundational pattern for building and managing microservices on Kubernetes.