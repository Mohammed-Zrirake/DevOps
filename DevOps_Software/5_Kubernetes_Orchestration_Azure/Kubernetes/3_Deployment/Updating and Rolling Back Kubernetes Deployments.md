#DevOps #Containerization #Kubernetes #CoreConcept #Deployments #Rollouts #Rollbacks #HandsOn #Tutorial

>  The key feature of a [[The Kubernetes Deployment|Deployment]] is its ability to manage **declarative updates** in a safe, controlled manner, ensuring zero downtime. This is achieved through a **rolling update** strategy. The Deployment also maintains a history of these updates (revisions), allowing you to easily **roll back** to a previous, stable version if a new release has issues.

---

This guide follows a hands-on demo of updating an application from V1 to V2 to V3, and then rolling it back.

## 🚀 Updating a Deployment

There are two primary ways to update a Deployment imperatively.

### Option 1: Using `kubectl set image` (Recommended for Simple Image Changes)
This command is a direct and explicit way to change the container image for a specific deployment.

#### Step 1: Inspect the Current State
First, let's verify the current image version of our `my-first-deployment`.
```bash
kubectl get deployment my-first-deployment -o yaml
```
By inspecting the YAML output under `spec.template.spec.containers`, you can find the current `image` tag (e.g., `stacksimplify/kube-nginx:1.0.0`).

#### Step 2: Perform the Update
We will update the application from version `1.0.0` to `2.0.0`.
```bash
kubectl set image deployment/my-first-deployment kube-nginx=stacksimplify/kube-nginx:2.0.0 --record=true
```
-   `set image deployment/my-first-deployment`: The resource type and name to update.
-   `kube-nginx=...`: The name of the container within the Pod to update. You can find this name by inspecting the deployment's YAML (`spec.containers.name`).
-   `stacksimplify/kube-nginx:2.0.0`: The new image tag.
-   `--record=true`: **(Important)** This flag records the command in the deployment's revision history, making it easier to see what changed later.

#### Step 3: Observe the Rolling Update
Kubernetes does not just kill all the old pods and start new ones. It performs a **rolling update** to ensure zero downtime.
-   **Check the rollout status:**
    ```bash
    kubectl rollout status deployment/my-first-deployment
    ```
    For a small deployment, this will likely finish instantly. For larger ones, you would see the progress in real-time.
-   **Describe the deployment to see the events:**
    ```bash
    kubectl describe deployment my-first-deployment
    ```
    In the `Events` section, you will see a detailed, step-by-step log of the rolling update process:
    1.  `Scaling up replica set <new-replicaset-id> to 1`: The Deployment creates a **new** ReplicaSet for the new version and starts one new Pod.
    2.  `Scaling down replica set <old-replicaset-id> to 1`: It then terminates one Pod from the **old** ReplicaSet.
    3.  `Scaling up replica set <new-replicaset-id> to 2`: It starts a second new Pod.
    4.  `Scaling down replica set <old-replicaset-id> to 0`: It terminates the last old Pod.
    This careful, sequential process ensures that at no point is your application completely unavailable.

-   **Verify the ReplicaSets:**
    ```bash
    kubectl get rs
    ```
    You will now see **two** ReplicaSets: the new one with a `DESIRED` count of `2`, and the old one scaled down to `0`. The old ReplicaSet is kept around to enable quick rollbacks.

### Option 2: Using `kubectl edit` (For Multiple or Complex Changes)
This command opens the live YAML manifest of the deployment in your default terminal editor (like `vi` or `nano`), allowing you to make changes directly.

1.  **Open the editor:**
    ```bash
    kubectl edit deployment/my-first-deployment --record=true
    ```
2.  **Make the Change:** Navigate to the `spec.template.spec.containers.image` field and change the tag from `2.0.0` to `3.0.0`.
3.  **Save and Exit:** Save the file and exit the editor (e.g., `:wq` in `vi`).
4.  **Observe:** Kubernetes detects the change and automatically initiates another rolling update, creating a third ReplicaSet for version `3.0.0`.

### Step 4: Access and Verify the Application
After each update, you can get the `EXTERNAL-IP` of your service and access it in a browser. You will see the application version change from "V1" to "V2" and then to "V3".

---

## ⏪ Rolling Back a Deployment

If a new deployment introduces a bug, the most valuable feature of a Deployment is the ability to quickly and safely roll back to a previous, stable version.

### Step 1: Check the Rollout History
The `--record=true` flag we used earlier makes the history human-readable.
```bash
kubectl rollout history deployment/my-first-deployment
```
**Output:**
```
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/my-first-deployment ...
3         kubectl edit deployment/my-first-deployment ...
```
This shows us a list of all the revisions (updates) our deployment has gone through.

### Step 2: Inspect a Specific Revision
You can get more details about what changed in a specific revision.
```bash
kubectl rollout history deployment/my-first-deployment --revision=2
```
This will show you the annotations and, most importantly, the container `image` used in that revision (`2.0.0`).

### Step 3: Roll Back to the Previous Version
The `undo` command is the simplest way to roll back. It reverts the deployment to its *immediately previous* state.
```bash
kubectl rollout undo deployment/my-first-deployment
```
-   **What happens:** Kubernetes will perform another rolling update, but this time it will scale *down* the current (V3) ReplicaSet and scale *up* the previous (V2) ReplicaSet.
-   **Verify:** Check the `rollout history` again. You will see a new revision (e.g., `REVISION 4`) has been created. If you inspect `REVISION 4`, you'll see it's using the `2.0.0` image. Accessing the application in the browser will now show "V2".

### Step 4: Roll Back to a Specific Version
Instead of just going back one step, you can roll back to *any* specific revision in the history.
1.  **Check the history again:**
    ```bash
    kubectl rollout history deployment/my-first-deployment
    ```
    Let's say we want to go all the way back to `REVISION 1` (which was our `1.0.0` image).
2.  **Roll back to the specific revision:**
    ```bash
    kubectl rollout undo deployment/my-first-deployment --to-revision=1
    ```
-   **What happens:** Kubernetes will perform a rolling update to bring the cluster's state back to what was defined in `REVISION 1`.
-   **Verify:** Accessing the application will now show "V1".

---

## 🔄 Other Useful `rollout` Commands

### Rolling Restart
If you need to force a restart of all the pods in a deployment (e.g., to pick up a change from a ConfigMap that doesn't trigger a rollout), you can use `rollout restart`.
```bash
kubectl rollout restart deployment/my-first-deployment
```
This will perform a safe, rolling restart of all the pods, one by one, ensuring zero downtime.

---

> [!summary] Conclusion
> The **Deployment** object is the cornerstone of managing application lifecycles in Kubernetes. Its automated rolling update strategy provides zero-downtime deployments, while the built-in revision history and `rollout undo` command offer a powerful and safe mechanism for instantly reverting to a known good state, making it an essential tool for robust CI/CD pipelines.