#DevOps #Containerization #Kubernetes #CoreConcept #Deployments #Rollouts #HandsOn #Tutorial

>  The `kubectl rollout pause` and `kubectl rollout resume` commands allow you to temporarily halt a [[The Kubernetes Deployment|Deployment's]] rollout process. This is a powerful technique for applying **multiple changes** to a Deployment at once, ensuring that only a single, unified rollout is triggered, rather than a new rollout for every individual change.

---

## 😫 The Problem: The "Chatter" of Multiple Imperative Changes

When you make a change to a Deployment's specification (e.g., updating the image, changing a resource limit), Kubernetes immediately initiates a [[Updating and Rolling Back Kubernetes Deployments|rolling update]] to apply that change.

> [!danger] The Unintended Consequence
> Imagine you have five separate changes to make to a live Deployment. If you apply them one after the other using imperative commands (`kubectl set image`, `kubectl set resources`, etc.), you will trigger **five separate rollouts**. Each rollout will terminate and recreate your application's pods, leading to unnecessary churn and instability.

## ✨ The Solution: `pause` and `resume`

The `kubectl rollout pause` and `resume` commands provide a mechanism to batch multiple changes into a single, controlled update.

> [!success] The Workflow
> 1.  **`kubectl rollout pause deployment/<deployment-name>`**: You "freeze" the Deployment. Any subsequent changes you make to its specification will be recorded but **will not trigger a new rollout**.
> 2.  **Apply All Your Changes:** You can now run multiple commands to update the image, set resource limits, change environment variables, etc.
> 3.  **`kubectl rollout resume deployment/<deployment-name>`**: You "unfreeze" the Deployment. At this point, Kubernetes detects all the accumulated changes and initiates a **single, unified rolling update** to bring the running pods to the new desired state.

---

## 🛠️ Hands-On: Pausing and Resuming a Deployment

This guide follows the instructor's step-by-step process for applying two changes (an image update and a resource limit update) to a Deployment while it is paused.

### Step 1: Check the Current State
First, verify the current state of the application and the deployment's history.
-   **Application Version:** Access the application in the browser. It should be showing **V3**.
-   **Rollout History:**
    ```bash
    kubectl rollout history deployment/my-first-deployment
    ```
    The history will show the previous revisions (e.g., 1, 4, 5, 6).

### Step 2: Pause the Deployment
Use the `rollout pause` command to freeze the Deployment.
```bash
kubectl rollout pause deployment/my-first-deployment
```
**Output:**
```
deployment.apps/my-first-deployment paused
```
> [!info] Pausing is an administrative action and **does not impact live application traffic**. Your V3 application is still running and accessible. Pausing only prevents new rollouts from being triggered.

### Step 3: Apply Multiple Changes While Paused
Now, we will apply two separate changes to the paused Deployment.

#### A. Change 1: Update the Container Image to V4
Use the `set image` command to change the application version from `3.0.0` to `4.0.0`.
```bash
kubectl set image deployment/my-first-deployment kube-nginx=stacksimplify/kube-nginx:4.0.0
```
**Output:**
```
deployment.apps/my-first-deployment image updated
```
**Verification:**
-   **Is the application updated?** NO. If you refresh the browser, it still shows **V3**.
-   **Is there a new rollout history?** NO. `kubectl rollout history ...` shows the same revisions.
-   **Is there a new ReplicaSet?** NO. `kubectl get rs` shows the same list of ReplicaSets.
The changes have been recorded in the Deployment's spec, but no action has been taken because it is paused.

#### B. Change 2: Set CPU and Memory Limits
Use the `set resources` command to apply resource limits to the container.
```bash
kubectl set resources deployment/my-first-deployment -c=kube-nginx --limits=cpu=20m,memory=30Mi
```
-   `-c=kube-nginx`: Specifies the container name to apply the limits to.
-   `--limits`: Sets the resource limits. `20m` is 20 millicores (2% of a CPU core), and `30Mi` is 30 Mebibytes of memory.

**Verification:** Again, no changes are visible in the running application or the number of ReplicaSets.

### Step 4: Resume the Deployment
Now that all our changes are applied, we can unfreeze the Deployment and trigger the rollout.
```bash
kubectl rollout resume deployment/my-first-deployment
```
**Output:**
```
deployment.apps/my-first-deployment resumed
```

### Step 5: Observe the Unified Rollout
Immediately after resuming, Kubernetes detects the changes and initiates a single rolling update.
-   **Check the rollout history:**
    ```bash
    kubectl rollout history deployment/my-first-deployment
    ```
    You will see a **new revision** has been created (e.g., `REVISION 7`).
-   **Check the ReplicaSets:**
    ```bash
    kubectl get rs
    ```
    You will see a **new ReplicaSet** has been created for the V4 application and is scaling up, while the old V3 ReplicaSet is scaling down.

### Step 6: Verify the Final State
1.  **Access the Application:** Refresh the browser. It will now show **V4**.
2.  **Inspect the Pods:** If you `describe` one of the new pods, you will see that it is running the `4.0.0` image and that the CPU and Memory limits have been applied.

This confirms that both changes were successfully applied in a single, controlled rollout.

### Step 7: Cleaning Up
Finally, delete the resources created during the demo.
```bash
# Delete the Deployment (which also deletes its ReplicaSets and Pods)
kubectl delete deployment my-first-deployment

# Delete the Service
kubectl delete service my-first-deployment-service
```

---

> [!summary] Conclusion
> The `pause` and `resume` commands are essential tools for managing complex updates to your Deployments. By pausing the rollout process, you can apply multiple configuration changes without causing a disruptive cascade of restarts, and then apply all changes at once with a single, unified, and controlled rolling update.