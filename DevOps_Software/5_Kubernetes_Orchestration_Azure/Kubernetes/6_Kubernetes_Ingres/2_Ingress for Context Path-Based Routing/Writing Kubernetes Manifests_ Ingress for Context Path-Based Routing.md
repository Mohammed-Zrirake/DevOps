#Cloud #Azure #Kubernetes #AKS #Networking #CoreConcept #YAML #Ingress #Routing #Manifests #Declarative #HandsOn #Tutorial

>  This guide details how to write a declarative YAML manifest for an [[An Introduction to Kubernetes Ingress|`Ingress`]] resource that performs **context path-based routing**. We will define multiple rules within a single Ingress to route traffic to three different backend applications based on the URL path (`/app1`, `/app2`, `/`). This also introduces the `defaultBackend` concept as a catch-all for traffic that doesn't match any specific rule.

---

This is a hands-on guide to creating the `ingress-cpr.yaml` (Context Path Routing) file from scratch.

## 🏛️ The Application Architecture Review

Before writing the Ingress manifest, let's review the application components we will be deploying. We have three separate applications, each with its own `Deployment` and `ClusterIP` Service.

1.  **`01-nginx-app1/`**:
    -   A standard `Deployment` for a simple Nginx application (`nginx-app1-deployment`).
    -   A `ClusterIP` Service named `app1-nginx-clusterip-service`.
2.  **`02-nginx-app2/`**:
    -   A standard `Deployment` for a second, distinct Nginx application.
    -   A `ClusterIP` Service named `app2-nginx-clusterip-service`.
3.  **`03-user-management-webapp/`**:
    -   The Spring Boot web application from our previous storage section.
    -   **Important Change:** In this section, we are using a simplified version that does *not* require a persistent volume. The `LoadBalancer` service from the previous section has been changed to a `ClusterIP` Service named `usermgmt-webapp-clusterip-service`, as all external traffic will now be handled by the Ingress.

All three of these applications will be fronted by our single Ingress service.

---

## ✍️ Writing the `ingress-cpr.yaml` Manifest from Scratch

We will start with the basic Ingress manifest from the previous section and add multiple path rules.

```yaml
# API Version for Ingress
apiVersion: networking.k8s.io/v1
# The kind of object is 'Ingress'
kind: Ingress
# Metadata to identify the Ingress resource
metadata:
  name: ingress-cpr # CPR = Context Path Routing
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  # 1. 'defaultBackend' acts as a catch-all for any requests that don't match a rule.
  defaultBackend:
    service:
      name: usermgmt-webapp-clusterip-service
      port:
        number: 80

  # 2. 'rules' defines the specific routing logic.
  rules:
  - http:
      # 'paths' is a list of path-based rules, which are evaluated in order.
      paths:
      # Rule for App1
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-nginx-clusterip-service
            port:
              number: 80
      # Rule for App2
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-nginx-clusterip-service
            port:
              number: 80
```

### A Deep Dive into the `spec` Section

#### 1. `defaultBackend`
-   **What it is:** The `defaultBackend` is a fallback service. If an incoming request does not match *any* of the defined `rules` (e.g., no matching host or path), the Ingress Controller will forward the request to this service.
-   **Our Use Case:** We are using it to serve our `user-management-webapp` for any path that is not `/app1` or `/app2`. This is an alternative to defining a rule with `path: /`.

#### 2. `rules.http.paths`
-   This is now a list containing multiple rule objects. The Ingress Controller will evaluate these rules to find a match for the incoming request's URL path.
-   **Path `/app1`**: Any request whose path starts with `/app1` (e.g., `/app1`, `/app1/page.html`) will be routed to the `app1-nginx-clusterip-service`.
-   **Path `/app2`**: Any request whose path starts with `/app2` will be routed to the `app2-nginx-clusterip-service`.

### A Deep Dive into `pathType`
The `pathType` field is crucial for defining how the `path` is matched.

| `pathType` | Description | Example Match (`path: /foo`) |
| :--- | :--- | :--- |
| **`Prefix`** | **(Recommended)** Matches based on a URL path prefix. A path `/foo` will match `/foo`, `/foo/`, and `/foo/bar`. It does not support wildcards. This is the most common and flexible type, used in 99.9% of cases. | ✅ `/foo`<br/>✅ `/foo/`<br/>✅ `/foo/bar` |
| **`Exact`** | Matches the URL path case-sensitively and exactly. A path `/foo` will **only** match `/foo`. It will **not** match `/foo/` (with a trailing slash) or `/foo/bar`. | ✅ `/foo`<br/>❌ `/foo/`<br/>❌ `/foo/bar` |
| **`ImplementationSpecific`** | The matching behavior is left up to the specific Ingress Controller (`ingress.class`). This is rarely used and can lead to non-portable behavior. | Varies by controller. |

> [!tip]
> For context path-based routing, **`Prefix`** is almost always the correct choice as it correctly handles both the base path and any sub-paths.

---

> [!summary]
> In the next lecture, we will:
> 1.  Deploy all the application manifests (`app1`, `app2`, `user-management-webapp`).
> 2.  Deploy our new `ingress-cpr.yaml` manifest.
> 3.  Test the routing by accessing the Ingress Controller's public IP with the different paths (`/app1`, `/app2`, `/`) to verify that we are correctly routed to each of the three distinct applications.