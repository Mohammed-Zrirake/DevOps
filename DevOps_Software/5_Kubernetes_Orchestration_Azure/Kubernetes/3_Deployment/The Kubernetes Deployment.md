#DevOps #Containerization #Kubernetes #CoreConcept #Deployments #ReplicaSet #Scaling #Rollouts #HandsOn #Tutorial

>  A **Deployment** is the standard, modern, and recommended way to manage stateless applications in Kubernetes. It is a higher-level controller that manages [[The Kubernetes ReplicaSet|ReplicaSets]] for you. While a ReplicaSet provides self-healing, a Deployment builds on top of that to provide powerful declarative updates, including automated **rollouts** (updating to a new version) and **rollbacks** (reverting to a previous version).

---

## 🏛️ Deployment: A Superset of ReplicaSet

A Deployment can be thought of as a "superset" of a ReplicaSet. It provides all the features of a ReplicaSet (like ensuring a desired number of pods are running) and adds many more powerful capabilities on top.

> [!info] The Relationship
> When you create a Deployment, it automatically creates and manages a [[The Kubernetes ReplicaSet|ReplicaSet]] in the background. That ReplicaSet, in turn, manages the [[The Kubernetes Pod|Pods]]. You interact with the Deployment, and the Deployment takes care of the underlying details.

Accessing applications running under a Deployment is no different than with a ReplicaSet; you still use a [[Kubernetes Services The LoadBalancer Type in Azure (AKS)|Service]] to expose them.

### Key Features and Advantages of a Deployment
-   **Creates and Manages ReplicaSets:** Automatically rolls out a ReplicaSet when created.
-   **Declarative Updates (Rollouts):** This is the killer feature. When you update a Deployment (e.g., change the container image version), it performs a controlled, rolling update. It gradually terminates old Pods while simultaneously bringing up new Pods, ensuring zero downtime for your application.
-   **Version History and Rollbacks:** The Deployment records the history of each update (revision). If you discover a bug in a new version, you can easily and quickly roll back to a previous, stable version with a single command.
-   **Scaling:** Just like a ReplicaSet, you can easily scale the number of replicas up or down using `kubectl scale` or by updating the manifest.
-   **Pause and Resume:** If you need to make multiple changes to a Deployment, you can `pause` it, apply all your changes, and then `resume` it. This triggers a single rollout with all the changes at once, rather than a new rollout for every individual change.
-   **Canary Deployments:** Deployments provide the foundation for advanced deployment strategies like canary releases, where you can direct a small amount of traffic to a new version before rolling it out to everyone.

---

## 🛠️ Hands-On: Creating, Scaling, and Exposing a Deployment

This guide follows the instructor's step-by-step process for managing a Deployment using imperative `kubectl` commands.

### Step 1: Create a Deployment
We can create a simple Deployment directly from the command line.

```bash
kubectl create deployment my-first-deployment --image=stacksimplify/kube-nginx:1.0.0
```
-   `create deployment`: The imperative command.
-   `my-first-deployment`: The name of our new Deployment.
-   `--image`: The container image to use for the Pods. This is Version 1 of our application.

By default, this creates a Deployment with `replicas: 1`.

### Step 2: Inspect the Created Resources
1.  **Get the Deployment:**
    ```bash
    kubectl get deployments
    # Alias: kubectl get deploy
    ```
    This shows our `my-first-deployment` is running and `1/1` replicas are ready.

2.  **Describe the Deployment:**
    ```bash
    kubectl describe deployment my-first-deployment
    ```
    The `Events` section at the bottom will show that the Deployment "Scaled up replica set `my-first-deployment-<unique-id>` to 1".

3.  **Verify the ReplicaSet:** A Deployment creates a ReplicaSet. Let's see it.
    ```bash
    kubectl get replicaset
    # Alias: kubectl get rs
    ```
    You will see a ReplicaSet named `my-first-deployment-<unique-id>`, confirming it was created and is managed by the Deployment.

4.  **Verify the Pod:** The ReplicaSet creates the Pod.
    ```bash
    kubectl get pods
    ```
    You will see a single Pod running, with a name like `my-first-deployment-<replicaset-id>-<pod-id>`.

### Step 3: Scale the Deployment
Let's scale our application from 1 to 10 replicas.

#### A. Manual Scaling (Imperative)
We can use the `kubectl scale` command.
```bash
kubectl scale --replicas=10 deployment/my-first-deployment
```
-   `--replicas=10`: The new desired number of replicas.
-   `deployment/my-first-deployment`: The type and name of the resource to scale.

**Verify the Scaling:**
```bash
# Check the ReplicaSet to see the new desired count
kubectl get rs
# DESIRED will be 10, CURRENT will be 10

# Check the Pods to see the new instances being created
kubectl get pods
# You will see 10 pods in the 'Running' or 'ContainerCreating' state.
```
This is a manual scaling operation. In a real-world scenario, you would likely use a **HorizontalPodAutoscaler (HPA)** to automatically scale the number of pods based on CPU or memory load.

#### B. Scaling Down
The same command works for scaling down.
```bash
kubectl scale --replicas=2 deployment/my-first-deployment
```
If you run `kubectl get pods`, you will see 8 of the pods enter the `Terminating` state, leaving you with 2 running pods.

### Step 4: Expose the Deployment as a Service
To access our application from the internet, we need to create a `LoadBalancer` Service that targets the Pods managed by our Deployment.

```bash
kubectl expose deployment my-first-deployment --type=LoadBalancer --port=80 --target-port=80 --name=my-first-deployment-service
```
-   `expose deployment my-first-deployment`: We are targeting the Deployment. Kubernetes will use the Deployment's selector to find the correct Pods.
-   `--type=LoadBalancer`: We want an external, public IP address from our cloud provider (Azure).
-   `--port=80`: The Load Balancer will listen on port 80.
-   `--target-port=80`: Traffic will be forwarded to port 80 on the containers.
-   `--name=my-first-deployment-service`: The name of our new Service.

After a minute, get the service's external IP:
```bash
kubectl get svc
```
Copy the `EXTERNAL-IP` and paste it into your browser. You will see the "Welcome to Stack Simplify..." page for Version 1 of the application.

### Step 5: Cleaning Up
It's a best practice to delete the resources you created.
```bash
# Deleting the Deployment will cascade and delete the ReplicaSet and all its Pods
kubectl delete deployment my-first-deployment

# Delete the Service
kubectl delete service my-first-deployment-service
```

---

> [!summary] Conclusion
> The **Deployment** is the primary and most powerful workload controller for stateless applications in Kubernetes. It abstracts away the management of ReplicaSets and provides the critical, production-ready features of automated, zero-downtime rollouts and easy rollbacks. While it's important to understand the underlying ReplicaSet, you will almost always interact with Deployments in your day-to-day work.