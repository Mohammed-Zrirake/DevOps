#Cloud #Azure #Kubernetes #AKS #Networking #CoreConcept #YAML #Ingress #Manifests #Declarative #HandsOn #Tutorial

>  This guide details how to write a declarative YAML manifest for a basic **`Ingress`** resource. An `Ingress` defines a set of HTTP/HTTPS routing rules. This manifest will create a simple rule to route all incoming traffic from the root path (`/`) to our sample Nginx application's `ClusterIP` [[Kubernetes Services A Deep Dive|Service]].

---

This is a hands-on guide to creating the `ingress-basic.yaml` file from scratch, which will be the first routing rule for our [[Installing the Nginx Ingress Controller on AKS|Nginx Ingress Controller]].

## 🏛️ The `Ingress` Manifest (`ingress-basic.yaml`)

We will build the manifest from scratch using the four [[Writing Kubernetes Manifests Pods with YAML#🏛️ The Four Top-Level Objects of a Kubernetes Manifest|top-level objects]] and by referencing the official Kubernetes API documentation.

> [!warning] API Version Deprecation
> The instructor notes that older Ingress manifests used a deprecated `apiVersion` (like `extensions/v1beta1`). The modern, stable, and correct version is `networking.k8s.io/v1`.

```yaml
# 1. API Version for Ingress is from the networking.k8s.io group
apiVersion: networking.k8s.io/v1
# 2. The kind of object is 'Ingress'
kind: Ingress
# 3. Metadata to identify the Ingress resource
metadata:
  name: nginx-app1-ingress-service
  annotations:
    # This annotation is CRITICAL. It tells the Nginx Ingress Controller
    # that it is responsible for handling this Ingress resource.
    kubernetes.io/ingress.class: "nginx"
spec:
  # 4. 'rules' defines the routing logic for incoming requests. It's a list.
  rules:
  - http:
      # 'paths' is a list of path-based rules.
      paths:
      - path: /
        pathType: Prefix
        # 'backend' specifies where to send the traffic that matches this path.
        backend:
          service:
            # The name of the backend ClusterIP Service.
            name: app1-clusterip-service
            # The port on that service to send traffic to.
            port:
              number: 80
```

---

### A Deep Dive into the `Ingress` Structure

#### 1. `apiVersion` and `kind`
-   **`kind`**: `Ingress`
-   **`apiVersion`**: `networking.k8s.io/v1`. This is found by consulting the official Kubernetes API Reference documentation for the `Ingress` resource.

#### 2. `metadata`
-   **`name`**: A unique name for our Ingress object (e.g., `nginx-app1-ingress-service`).
-   **`annotations`**: This is a critical section. Annotations are key-value pairs used to attach arbitrary, non-identifying metadata to objects. Ingress controllers use specific annotations to configure their behavior.
    -   **`kubernetes.io/ingress.class: "nginx"`**: This annotation is a special instruction. It acts as a "selector" for Ingress controllers. When you have multiple Ingress controllers in a cluster (e.g., Nginx, Traefik, AGIC), this annotation tells the Nginx controller specifically that it is responsible for processing and implementing the rules defined in this manifest.

#### 3. `spec`
This is where the routing logic is defined. For a basic Ingress, it contains a list of `rules`.
-   **`rules`**: A list of routing rules. Each rule is typically associated with a host (for hostname-based routing, which we'll cover later) or just HTTP paths.
-   **`http.paths`**: A list of path-based rules.
    -   **`path`**: The URL path to match. Here we use `/` to match all incoming traffic.
    -   **`pathType`**: Specifies how the `path` should be matched.
        -   **`Prefix` (our choice):** Matches any path that starts with the specified value (e.g., `/` matches `/`, `/foo`, `/foo/bar`).
        -   `Exact`: Matches the URL path exactly.
        -   `ImplementationSpecific`: The matching behavior depends on the `ingress.class`.
    -   **`backend`**: Defines the destination for the traffic that matches the rule.
        -   **`service.name`**: The name of the backend [[Kubernetes Services A Deep Dive|Service]] to which the traffic should be routed. This must be the name of the `ClusterIP` service for our Nginx App1 (`app1-clusterip-service`).
        -   **`service.port.number`**: The specific port on that service to send the traffic to (port `80`).

---

## 🛠️ Hands-On: Deploying the Application and Ingress

This guide assumes the [[Installing the Nginx Ingress Controller on AKS|Nginx Ingress Controller]] is already installed.

### Step 1: Review the Application Manifests
In the `kube-manifests/` folder, we have two standard files:
-   `01-nginx-app1-deployment.yaml`: A simple [[The Kubernetes Deployment|Deployment]] to run one replica of a basic Nginx application.
-   `02-app1-clusterip-service.yaml`: A `ClusterIP` Service to provide an internal endpoint for the Nginx application pods. This is the service that our Ingress rule will target.

### Step 2: Deploy the Application and Ingress
1.  Navigate to the directory for this section (`09-Ingress-Basic/`).
2.  Apply all the manifests in the `kube-manifests/` folder. This will create the Deployment, the ClusterIP Service, and our new Ingress resource.
    ```bash
    kubectl apply -f kube-manifests/
    ```

### Step 3: Test the Application
1.  Get the **static public IP address** of your Nginx Ingress Controller's service (the one we created in the previous lecture).
    ```bash
    kubectl get service -n ingress-basic
    ```
2.  Access this IP address in your browser.
3.  You should now see the "Welcome" page from your **Nginx App1** application, not the "404 Not Found" from the Ingress Controller itself.

This confirms that the entire routing flow is working:
**User → Static Public IP → Azure Load Balancer → Nginx Ingress Controller → Ingress Rule → App1 `ClusterIP` Service → App1 Pod**

---

> [!summary] Conclusion
> You have successfully created a basic `Ingress` resource using a declarative YAML manifest. This is the foundational pattern for managing external HTTP/HTTPS traffic in Kubernetes. We have defined a rule that routes all traffic to a backend service, effectively exposing our application through the centralized Ingress Controller. In the upcoming lectures, we will build upon this foundation to implement more advanced routing scenarios like context-path and hostname-based routing.