#DevOps #Containerization #Kubernetes #CoreConcept #YAML #ReplicaSet #Manifests #Declarative #HandsOn #Tutorial

>  This guide details how to write a declarative YAML manifest for a [[The Kubernetes ReplicaSet|ReplicaSet]]. A ReplicaSet ensures that a specified number of [[The Kubernetes Pod|Pod]] replicas are running at all times. The manifest is built around three key `spec` fields: **`replicas`**, **`selector`**, and **`template`**. The `selector` is the critical link that connects the ReplicaSet to the Pods it manages.

> [!warning] A Note on Deployments
> In modern Kubernetes, it is highly recommended to use a [[The Kubernetes Deployment|Deployment]] instead of creating a ReplicaSet directly. A Deployment is a "superset" of a ReplicaSet, providing all its self-healing capabilities plus powerful features like rollouts and rollbacks. The YAML structure we learn here for a ReplicaSet's `spec` is almost identical to the `spec` of a Deployment, making this a crucial foundational lesson.

---

## 🏛️ The `ReplicaSet` Manifest (`replicaset-definition.yaml`)

We will build the manifest from scratch using the four [[Writing Kubernetes Manifests Pods with YAML#🏛️ The Four Top-Level Objects of a Kubernetes Manifest|top-level objects]].

```yaml
# 1. API Version for ReplicaSet is 'apps/v1'
apiVersion: apps/v1
# 2. The kind of object is 'ReplicaSet'
kind: ReplicaSet
# 3. Metadata to identify the ReplicaSet
metadata:
  name: myapp2-rs
spec:
  # 4. Specification of the desired state for this ReplicaSet
  
  # 'replicas' defines how many pods should be running.
  replicas: 3
  
  # 'selector' tells the ReplicaSet which pods it is responsible for managing.
  selector:
    matchLabels:
      app: myapp2
      
  # 'template' is a blueprint for the pods that this ReplicaSet will create.
  # This is essentially a complete Pod definition, nested inside.
  template:
    metadata:
      # Pods created from this template will get these labels.
      # This MUST match the 'selector.matchLabels' above.
      labels:
        app: myapp2
    spec:
      containers:
      - name: myapp2-container
        image: stacksimplify/kube-nginx:2.0.0
        ports:
        - containerPort: 80
```

### A Deep Dive into the `spec` Section

The `spec` for a ReplicaSet has three essential fields:

#### 1. `replicas`
-   **What it is:** A number that specifies how many Pods you want to be running for this application.
-   **Value:** `3`. This tells the ReplicaSet controller to always ensure that three pods matching its selector are running.

#### 2. `selector`
-   **What it is:** This tells the ReplicaSet **which Pods it owns and is responsible for managing**.
-   **`matchLabels`:** The controller will look for all Pods that have labels matching what is defined here.
-   **The Crucial Link:** The labels in `selector.matchLabels` **must exactly match** the labels defined in the `template.metadata.labels` section below. This is how the ReplicaSet identifies its children.

#### 3. `template`
-   **What it is:** This is a **Pod template**. It's a complete blueprint for the Pods that the ReplicaSet will create.
-   **Structure:** The content inside the `template` key is essentially a `Pod` manifest, but without the `apiVersion` and `kind`. It must have its own `metadata` (with labels) and a `spec` (defining the containers).
-   **`template.metadata.labels`:** This is where you define the labels that will be applied to every Pod created by this ReplicaSet. **This is what the `selector` matches against.**
-   **`template.spec`:** This is the standard Pod specification, defining the containers to run, their images, ports, etc.

---

## 🛠️ Hands-On: Creating and Managing the ReplicaSet

### Step 1: Create the ReplicaSet
1.  Navigate to the directory containing your `replicaset-definition.yaml` file.
2.  Apply the manifest to the cluster:
    ```bash
    kubectl apply -f replicaset-definition.yaml
    ```
    **Output:** `replicaset.apps/myapp2-rs created`

### Step 2: Inspect the Resources
1.  **Get the ReplicaSet:**
    ```bash
    kubectl get replicaset
    # Alias: kubectl get rs
    ```
    This will show that `myapp2-rs` desires `3` replicas and has `3` ready.

2.  **Get the Pods:**
    ```bash
    kubectl get pods
    ```
    You will see three new pods running, with names like `myapp2-rs-abcde`, confirming they were created by the ReplicaSet.

### Step 3: Test High Availability (Self-Healing)
Let's simulate a Pod crash by manually deleting one.
1.  Pick one of the pod names and delete it:
    ```bash
    kubectl delete pod <pod-name>
    ```
2.  Immediately get the pods again:
    ```bash
    kubectl get pods
    ```
    You will see that a **brand new Pod was automatically created** to replace the one that was deleted, bringing the total back to 3. This demonstrates the self-healing power of the ReplicaSet.

### Step 4: Expose the ReplicaSet with a `LoadBalancer` Service
Now, we'll create a `LoadBalancer` Service to expose our application to the internet. We can do this with another YAML manifest.

#### The `replicaset-loadbalancer-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp2-rs-loadbalancer-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    # This selector MUST match the labels on the Pods created by the ReplicaSet
    app: myapp2
```
-   The `selector` `app: myapp2` is the critical piece that tells this Service to send traffic to the Pods managed by our `myapp2-rs` ReplicaSet.

#### Create the Service
```bash
kubectl apply -f replicaset-loadbalancer-service.yaml
```
After a minute, get the external IP:
```bash
kubectl get svc
```
Access `http://<EXTERNAL-IP>` in your browser. You should see the "V2" version of the application.

---

> [!summary] Conclusion
> You have successfully created a ReplicaSet and a corresponding Service using declarative YAML manifests. This is the foundational pattern for building reliable, self-healing applications on Kubernetes. The key takeaway is the critical role of **labels and selectors** in connecting these different objects, allowing the ReplicaSet to manage its Pods and the Service to discover and load balance traffic to them.