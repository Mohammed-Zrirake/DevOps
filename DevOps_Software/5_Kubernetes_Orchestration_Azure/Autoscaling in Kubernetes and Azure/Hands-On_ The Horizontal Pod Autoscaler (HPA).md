#Cloud #Azure #Kubernetes #AKS #Scaling #Autoscaling #HPA #HandsOn #Tutorial

>  This is a hands-on guide to testing the **Horizontal Pod Autoscaler (HPA)** feature in Kubernetes. The HPA automatically adjusts the number of **[[The Kubernetes Pod|Pods]]** in a [[The Kubernetes Deployment|Deployment]] or [[The Kubernetes ReplicaSet|ReplicaSet]] based on observed metrics like CPU utilization. We will create an HPA for a sample application, generate artificial load to increase CPU usage, and observe the HPA **scale out** (add more pods). When the load stops, we will observe the HPA **scale in** (remove the unneeded pods).

---

This guide follows the practical implementation of HPA concepts, building upon the [[Hands-On The AKS Cluster Autoscaler|Cluster Autoscaler]] demo.

## 🏛️ HPA vs. Cluster Autoscaler: The Key Difference
-   **[[Hands-On The AKS Cluster Autoscaler|Cluster Autoscaler (CA)]]:** Manages the number of **Nodes** (VMs) in your cluster. It adds nodes when there aren't enough resources to schedule new pods.
-   **Horizontal Pod Autoscaler (HPA):** Manages the number of **Pods** for a specific application. It adds pods when the existing pods are under heavy load (e.g., high CPU).

These two components often work together: high traffic causes the HPA to add more pods. If there isn't enough room on the existing nodes for these new pods, the Cluster Autoscaler will then add more nodes.

---

## 🛠️ Step 1: Deploy the Application and Create the HPA

For this demo, we will use an Nginx application and create an HPA for it using the imperative `kubectl autoscale` command.

1.  **Deploy the Nginx Application:**
    -   Apply the manifests to create the `hpa-demo-deployment-nginx` Deployment and its `hpa-demo-service-nginx` `LoadBalancer` Service.
2.  **Create the HPA Resource:**
    ```bash
    kubectl autoscale deployment hpa-demo-deployment-nginx --cpu-percent=20 --min=1 --max=10
    ```
    **Command Breakdown:**
    -   `autoscale deployment hpa-demo-deployment-nginx`: We are creating an HPA that targets this specific Deployment.
    -   `--cpu-percent=20`: The **target CPU utilization**. The HPA will try to maintain an average CPU usage of 20% across all pods. If the average goes above 20%, it will add more pods.
    -   `--min=1`: The minimum number of replicas. The HPA will never scale down below this number.
    -   `--max=10`: The maximum number of replicas. The HPA will never scale up beyond this number.

3.  **Verify the HPA:**
    ```bash
    kubectl get hpa
    ```
    **Output:**
    ```
    NAME                      REFERENCE                               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
    hpa-demo-deployment-nginx   Deployment/hpa-demo-deployment-nginx    1%/20%    1         10        1          30s
    ```
    -   **`TARGETS: 1%/20%`**: This shows the current average CPU utilization (`1%`) versus the target (`20%`).
    -   **`REPLICAS: 1`**: The HPA is currently maintaining 1 replica, which is the minimum.

---

## 🚀 Step 2: Generate Load and Trigger a Scale-Out Event

To test the HPA, we need to generate enough traffic to push the CPU utilization of our Nginx pod above the 20% target. We will do this by running a load-testing tool (`ApacheBench - ab`) from another pod inside the cluster.

1.  **Open Two Terminals:**
    -   **Terminal 1:** We will use this to monitor the HPA status.
    -   **Terminal 2:** We will use this to run the load generation command.

2.  **Monitor the HPA (Terminal 1):**
    Run the `get hpa` command in a watch loop to see live updates.
    ```bash
    # This command will re-run 'kubectl get hpa' every 2 seconds
    watch -n 2 kubectl get hpa
    ```

3.  **Generate the Load (Terminal 2):**
    Run a temporary `httpd` pod that executes the `ab` command to send a large number of requests to our Nginx service.
    ```bash
    kubectl run -it --rm apache-bench --image=httpd -- ab -n 500000 -c 1000 http://hpa-demo-service-nginx.default.svc.cluster.local/
    ```
    **Command Breakdown:**
    -   `kubectl run -it --rm apache-bench --image=httpd --`: Runs an interactive, temporary pod named `apache-bench` using the `httpd` image.
    -   `ab -n 500000 -c 1000`: `ApacheBench` command to send a total of 500,000 requests (`-n`), with a concurrency of 1,000 requests at a time (`-c`).
    -   `http://hpa-demo-service-nginx.default.svc.cluster.local/`: The internal DNS name of our Nginx service. We are generating the load from *inside* the cluster for maximum efficiency.

4.  **Observe the Scale-Out (Terminal 1 - The Magic Moment):**
    As the load generation starts, watch the output of your `watch` command.
    -   You will see the `TARGETS` value jump dramatically, for example, `200%/20%`. This indicates the single running pod is overwhelmed.
    -   The HPA controller will react to this. The `REPLICAS` count will start to increase: `1` -> `4` -> `8` -> `10`.
    -   You can verify this by running `kubectl get pods` in another terminal. You will see new pods being created until the maximum of 10 is reached.
    -   Once 10 replicas are running, the `TARGETS` value will start to decrease (e.g., `66%/20%`) because the load is now distributed across 10 pods. The HPA will not scale beyond the `--max=10` limit.

---

## ⏪ Step 3: Stop the Load and Observe the Scale-In Event

The load generation command in Terminal 2 will eventually complete.

1.  **Observe the HPA (Terminal 1):**
    -   Once the load stops, you will see the `TARGETS` CPU percentage drop back down to a low value (e.g., `12%/20%`, then `1%/20%`).
2.  **Wait for the Cool-Down Period:**
    -   The HPA has a default **cool-down period (or scale-down delay) of 5 minutes**. It waits this long after the metrics have dropped below the target to ensure the traffic spike is truly over before it starts removing pods.
    -   After approximately 5 minutes of low CPU usage, you will see the `REPLICAS` count start to decrease from `10` all the way back down to the minimum of `1`.

3.  **Final Verification:**
    ```bash
    kubectl get pods
    ```
    You will see that 9 of the pods have been terminated, leaving only the single, minimum replica running.

---

## 🧹 Step 4: Cleaning Up

1.  **Delete the HPA:**
    ```bash
    kubectl delete hpa hpa-demo-deployment-nginx
    ```
2.  **Delete the Application Manifests:**
    ```bash
    kubectl delete -f kube-manifests/
    ```
3.  **(If you created a new cluster for this) Delete the Cluster/Resource Group** in the Azure Portal to avoid costs.

---

> [!summary] Declarative Approach
> The instructor notes that while we used the imperative `kubectl autoscale` command, you can (and should, for production) define your HPA declaratively in a YAML manifest. This allows you to version-control your scaling policies alongside your application code.