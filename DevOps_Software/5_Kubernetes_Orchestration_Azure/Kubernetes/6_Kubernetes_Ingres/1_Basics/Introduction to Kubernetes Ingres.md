#Cloud #Azure #Kubernetes #AKS #Networking #CoreConcept #Ingress #Routing #SSL #TLS

>  **Ingress** is a Kubernetes API object that manages external access to the services in a cluster, typically HTTP and HTTPS. It acts as an **L7 (HTTP/HTTPS) load balancer** or "reverse proxy," providing advanced routing capabilities like context-path and hostname-based routing, as well as SSL/TLS termination. It is the standard way to expose multiple services under a single public IP address.

---

## 😫 The Problem: The Limitations of a `LoadBalancer` Service

While a [[Kubernetes Services The LoadBalancer Type in Azure (AKS)|`LoadBalancer` service]] is great for exposing a single application, it has significant limitations when managing a complex, multi-service application.

> [!danger] The `LoadBalancer` Proliferation Problem
> Imagine you have three different microservices (App1, App2, App3) running in your AKS cluster. If you expose each one with a `type: LoadBalancer` service, you will get:
> -   **Three separate public IP addresses**, one for each application.
> -   **Increased Cost:** Each public IP and load balancer configuration can incur costs.
> -   **No L7 Features:** A `LoadBalancer` service is an L4 (TCP) load balancer. It cannot perform intelligent HTTP-level routing. It has no built-in capabilities for:
>     -   SSL/TLS termination.
>     -   Routing based on the URL path (e.g., `/app1` goes to one service, `/app2` goes to another).
>     -   Routing based on the hostname (e.g., `app1.mydomain.com` goes to one service, `app2.mydomain.com` goes to another).

In short, native Kubernetes Services cannot perform the "reverse proxy magic" required by modern web applications.

## ✨ The Solution: Kubernetes Ingress

Ingress was designed to solve these problems. It provides a single entry point for all HTTP/HTTPS traffic into your cluster and intelligently routes it to the appropriate backend services.

### Core Ingress Features
-   **Single Public IP:** Expose multiple services under one IP address.
-   **Context Path-Based Routing:** Route traffic based on the URL path (e.g., `mydomain.com/app1` → App1 Service).
-   **Hostname-Based Routing (Virtual Hosting):** Route traffic based on the hostname in the request (e.g., `api.mydomain.com` → API Service).
-   **TLS/SSL Termination:** Handle HTTPS traffic, decrypt it, and forward unencrypted traffic to your internal services, simplifying certificate management.

---

## 🏛️ Ingress Terminology: Controller vs. Resource

It is critical to understand that "Ingress" is made up of two distinct components that work together.

### 1. The Ingress Controller
-   **What it is:** An **application** (a pod or set of pods) that runs inside your cluster. It's the actual reverse proxy engine (e.g., Nginx, Traefik, HAProxy) that watches for Ingress resources and configures itself accordingly.
-   **How it works:** The Ingress Controller is typically exposed to the internet via a single `LoadBalancer` service. It listens for all incoming traffic and then routes it based on the rules defined in your Ingress Resources.
-   **Cloud-Specific Controllers:** Cloud providers often offer their own managed Ingress Controllers that integrate with their native load balancers (e.g., AWS ALB Ingress Controller, Azure Application Gateway Ingress Controller - AGIC). The instructor notes that while powerful, AGIC can sometimes be less mature than community-driven solutions like the Nginx Ingress Controller.

### 2. The Ingress Resource
-   **What it is:** A **Kubernetes object** (defined in a YAML manifest) that you create.
-   **Purpose:** The Ingress resource contains a set of **routing rules**. You define rules like "requests for the host `api.mydomain.com` with the path `/users` should be sent to the `user-service` on port `8080`."
-   **The Relationship:** You define the *desired state* in the Ingress resource, and the Ingress *Controller* is the engine that makes that state a reality by configuring the underlying proxy.

---

## 🚀 A Roadmap of Ingress Concepts to be Implemented

This section outlines a comprehensive, multi-part, hands-on tutorial that will be covered in the upcoming lectures.

### 1. Context Path-Based Routing
-   **Goal:** Use a single public IP to route traffic to different applications based on the URL path.
-   **Example:**
    -   `http://<PUBLIC_IP>/app1` → `app1-service`
    -   `http://<PUBLIC_IP>/app2` → `app2-service`
    -   `http://<PUBLIC_IP>/` → `user-management-webapp-service`

### 2. Hostname-Based Routing
-   **Goal:** Use a single public IP to route traffic to different applications based on the hostname. This is also known as "name-based virtual hosting."
-   **Example:**
    -   `http://app1.kubeoncloud.com` → `app1-service`
    -   `http://app2.kubeoncloud.com` → `app2-service`

### 3. Integrating with External DNS (`external-dns`)
-   **The Problem:** When you define hostnames in your Ingress resource, you still need to manually create the corresponding DNS records (e.g., A records in Azure DNS Zones) to point those hostnames to the Ingress Controller's public IP.
-   **The Solution:** The `external-dns` controller is a Kubernetes tool that automates this. It watches for Ingress resources and **automatically creates and manages DNS records** in your cloud provider's DNS service. This requires setting up a DNS Zone (e.g., `kubeoncloud.com`) in Azure and granting the AKS cluster permissions to modify it.

### 4. Automatic SSL/TLS with `cert-manager`
-   **The Problem:** Manually creating, managing, and renewing SSL certificates for all your hostnames is a tedious and error-prone process.
-   **The Solution:** `cert-manager` is a powerful Kubernetes tool that completely automates certificate management.
    -   It integrates with public Certificate Authorities like **Let's Encrypt**.
    -   When you create an Ingress resource with TLS configuration, `cert-manager` automatically:
        1.  Generates a Certificate Signing Request (CSR).
        2.  Communicates with Let's Encrypt to get a valid, trusted SSL certificate.
        3.  Stores the certificate in a Kubernetes Secret.
        4.  Configures the Ingress Controller to use that secret for TLS termination.
        5.  **Automatically renews the certificate** before it expires.

By the end of this section, you will have built a sophisticated, production-grade ingress setup that can intelligently route traffic, automatically manage DNS, and automatically secure your applications with valid, auto-renewing SSL certificates.