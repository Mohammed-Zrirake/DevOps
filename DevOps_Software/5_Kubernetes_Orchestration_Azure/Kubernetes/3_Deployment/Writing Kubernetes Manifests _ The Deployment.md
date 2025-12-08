#DevOps #Containerization #Kubernetes #CoreConcept #YAML #Deployments #Manifests #Declarative #HandsOn #Tutorial

>  This guide details how to write a declarative YAML manifest for a [[The Kubernetes Deployment|Deployment]]. A Deployment is the standard way to manage stateless applications in Kubernetes, providing self-healing via [[The Kubernetes ReplicaSet|ReplicaSets]] and powerful update strategies like rollouts and rollbacks. The YAML structure for a Deployment is nearly identical to that of a ReplicaSet, making the transition seamless.

---

## 🏛️ From ReplicaSet to Deployment: The YAML Definition

As discussed, a Deployment is a "superset" of a ReplicaSet. This is clearly reflected in their YAML definitions. The `spec` for a Deployment contains the exact same core fields as a ReplicaSet (`replicas`, `selector`, `template`), making it incredibly easy to upgrade a ReplicaSet manifest to a Deployment.

> [!tip] The Only Change Needed
> To convert a `ReplicaSet` manifest to a `Deployment` manifest, you simply need to change the `kind` from `ReplicaSet` to `Deployment`.

### The `deployment-definition.yaml` Manifest
We will start by copying the `replicaset-definition.yaml` from the previous lecture and making the necessary modifications.

```yaml
# 1. API Version for Deployment is 'apps/v1'
apiVersion: apps/v1
# 2. The kind of object is 'Deployment' (changed from ReplicaSet)
kind: Deployment
# 3. Metadata to identify the Deployment
metadata:
  # Renaming for clarity
  name: myapp3-deployment
spec:
  # 'replicas' defines how many pods should be running.
  replicas: 3
  
  # 'selector' tells the Deployment which pods its ReplicaSet should manage.
  selector:
    matchLabels:
      # Renaming label for clarity
      app: myapp3
      
  # 'template' is the blueprint for the pods that will be created.
  template:
    metadata:
      # Pods created from this template will get these labels.
      # This MUST match the 'selector.matchLabels' above.
      labels:
        app: myapp3
    spec:
      containers:
      - name: myapp3-container
        # Using a new version of the application image
        image: stacksimplify/kube-nginx:3.0.0
        ports:
        - containerPort: 80
```
**Changes Made:**
1.  **`kind`:** Changed from `ReplicaSet` to `Deployment`.
2.  **`apiVersion`:** Verified from the official Kubernetes documentation that `Deployment` also uses `apps/v1`, so no change is needed here.
3.  **Names and Labels:** Updated all names and labels from `myapp2` to `myapp3` for clarity and to avoid conflicts with our previous resources.
4.  **`image`:** Updated the container image to `3.0.0` to represent a new version of our application.

The core structure (`replicas`, `selector`, `template`) remains identical to the ReplicaSet definition.

---

## 🛠️ Hands-On: Creating and Exposing the Deployment

### Step 1: Create the Deployment
1.  Navigate to the directory containing your `deployment-definition.yaml` file.
2.  Apply the manifest to the cluster:
    ```bash
    kubectl apply -f deployment-definition.yaml
    ```
    **Output:** `deployment.apps/myapp3-deployment created`

### Step 2: Create the `LoadBalancer` Service
We will create a corresponding `LoadBalancer` Service to expose our new Deployment. We can copy the service manifest from the previous lecture and update the `name` and `selector` to match our new `myapp3` labels.

#### The `deployment-loadbalancer-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp3-deployment-loadbalancer-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    # This selector MUST match the labels on the Pods created by the Deployment
    app: myapp3
```

#### Create the Service
```bash
kubectl apply -f deployment-loadbalancer-service.yaml
```
**Output:** `service/myapp3-deployment-loadbalancer-service created`

### Step 3: Inspect and Test the Application
1.  **Check the Resources:**
    ```bash
    # Verify the Deployment, ReplicaSet, and Pods are all running
    kubectl get deployment
    kubectl get rs
    kubectl get pods
    ```
    You will see a new Deployment, a new ReplicaSet managed by that Deployment, and three new Pods running the `3.0.0` image.

2.  **Get the Service's Public IP:**
    ```bash
    kubectl get svc
    ```
    Find the `EXTERNAL-IP` for `myapp3-deployment-loadbalancer-service`.

3.  **Access the Application:**
    -   Copy the external IP address and paste it into your browser.
    -   It may take a moment on the first access for the load balancer to become fully operational and route the traffic.
    -   You will see the **V3** version of the application, confirming that your Deployment was successful.

---

> [!summary] Conclusion
> You have successfully created a `Deployment` and a corresponding `Service` using declarative YAML manifests. This is the **standard, best-practice pattern** for deploying and managing stateless applications in Kubernetes. By defining your application's desired state in code, you gain the benefits of version control, repeatability, and the powerful rollout/rollback features that Deployments provide.