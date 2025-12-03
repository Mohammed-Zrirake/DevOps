#DevOps #Containerization #Kubernetes #CoreConcept #YAML #Services #LoadBalancer #Networking #Manifests #HandsOn #Tutorial

>  This guide details how to write a declarative YAML manifest for a `LoadBalancer` [[Kubernetes Services A Deep Dive|Service]]. This service type exposes an application to the public internet by automatically provisioning a cloud load balancer. The most critical part of the service definition is the **`selector`**, which uses [[#The Critical Link The selector|labels]] to identify which [[The Kubernetes Pod|Pods]] should receive traffic.

---

This is a hands-on guide to creating a `LoadBalancer` service for the [[Writing Kubernetes Manifests Pods with YAML|Pod we created previously]].

## 🏛️ The `LoadBalancer` Service Manifest (`pod-loadbalancer-service.yaml`)

We will build the manifest using the four [[Writing Kubernetes Manifests Pods with YAML#🏛️ The Four Top-Level Objects of a Kubernetes Manifest|top-level objects]].

```yaml
# 1. API Version for a Service is 'v1'
apiVersion: v1
# 2. The kind of object we are creating is a 'Service'
kind: Service
# 3. Metadata to identify the Service
metadata:
  name: myapp-pod-loadbalancer-service
spec:
  # 4. Specification of the desired state for this Service
  
  # 'type' defines how the service is exposed.
  # 'LoadBalancer' makes it accessible via a public IP from a cloud provider.
  type: LoadBalancer
  
  # 'ports' is a list defining how traffic is routed.
  ports:
  - name: http
    # 'port' is the port the Service will listen on.
    port: 80
    # 'targetPort' is the port on the container that the traffic should be sent to.
    targetPort: 80
    
  # 'selector' is the crucial link between the Service and the Pods.
  selector:
    app: myapp
```

### A Deep Dive into the `spec` Section

The `spec` for a Service has three core fields for this use case:

#### 1. `type`
-   **What it is:** Defines how the Service is exposed.
-   **Value:** `LoadBalancer`. This tells our cloud provider (e.g., [[Azure Kubernetes Service (AKS)|AKS]]) to provision an external load balancer and assign it a public IP address. Other common values are `ClusterIP` and `NodePort`.

#### 2. `ports`
-   **What it is:** A list of port definitions. Each item in the list specifies a port mapping.
-   **`port`:** The port that the Service itself exposes and listens on. In this case, the external load balancer will accept traffic on port `80`.
-   **`targetPort`:** The port on the **container** inside the Pod to which the traffic should be forwarded. Our Nginx container is listening on port `80`, so we set this to `80`.

#### 3. `selector` (The Critical Link)
-   **What it is:** This is the most important part of the Service definition. It is a dictionary of key-value pairs that tells the Service **which Pods to send traffic to**.
-   **How it works:** The Service continuously scans the cluster for Pods that have **labels matching this selector**. Any Pod with the label `app: myapp` will become a backend for this Service.
-   **The Connection:** If you look at our `pod-definition.yaml`, we defined our Pod with this exact label:
    ```yaml
    # From pod-definition.yaml
    metadata:
      labels:
        app: myapp
    ```
    This is how the `myapp-pod-loadbalancer-service` knows to send traffic to the `myapp-pod`. If you create more pods with the same label, the Service will automatically discover them and start load balancing traffic across all of them.

> [!tip] **Labels in Services vs. Pods**
> - In a `Pod` manifest, defining `metadata.labels` is effectively **mandatory** if you want it to be managed by another object.
> - In a `Service` manifest, defining `metadata.labels` is **optional**. The `spec.selector` is what's critical.

---

## 🛠️ Hands-On: Creating and Testing the `LoadBalancer` Service

This guide assumes you have already created the `myapp-pod` from the previous lecture.

### Step 1: Apply the Service Manifest
1.  Navigate to the directory containing your `pod-loadbalancer-service.yaml` file.
2.  Use the `kubectl apply` command to create the Service in your cluster.
    ```bash
    kubectl apply -f pod-loadbalancer-service.yaml
    ```
    **Output:** `service/myapp-pod-loadbalancer-service created`

### Step 2: Inspect the Service and Get the Public IP
1.  Check the status of your services. It may take a minute or two for the cloud provider to provision and assign a public IP address.
    ```bash
    kubectl get service
    # Alias: kubectl get svc
    ```
2.  The output will show the service. Initially, the `EXTERNAL-IP` might show as `<pending>`. Wait and run the command again until an IP address appears.
    ```
    NAME                            TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)        AGE
    myapp-pod-loadbalancer-service  LoadBalancer   10.0.123.45   52.123.45.67     80:31234/TCP   60s
    ```

### Step 3: Access the Application
1.  Copy the `EXTERNAL-IP` address.
2.  Paste it into your web browser.
3.  You should see the welcome page for the Nginx application running in your Pod: **"Welcome to stack simplify Kubernetes fundamentals demo application service version V1."**

This confirms that the entire flow is working:
**External Request → Azure Load Balancer (Public IP) → Kubernetes Service → Pod (with matching label) → Nginx Container**

---

> [!summary] Conclusion
> You have successfully created a `LoadBalancer` Service using a declarative YAML manifest. This is the standard, production-ready way to expose your applications to the internet when running on a cloud provider. The key takeaway is the power of the `selector`, which decouples the Service from specific Pods and allows it to dynamically discover and load balance across any Pods that match the specified labels.