#Cloud #Azure #Kubernetes #AKS #Deployment #kubectl #YAML #HandsOn #Tutorial

>  This is a hands-on guide to deploying a simple Nginx web application to an [[Azure Kubernetes Service (AKS)|AKS cluster]]. The process involves using `kubectl apply` to deploy resources defined in YAML manifest files, inspecting the running application with various `kubectl get`, `logs`, and `describe` commands, accessing it via a LoadBalancer service, and finally, cleaning up the resources with `kubectl delete`.

---

> [!info] A Note on Kubernetes Fundamentals
> This tutorial focuses on the *how* of deploying an application. A deep understanding of Kubernetes concepts like **Pods**, **Deployments**, and **Services** is not required to follow along. These topics will be covered in detail in a future "Kubernetes Fundamentals" section, where you will learn to write these YAML manifests from scratch. The goal here is to get a feel for the deployment workflow on a live cluster.

---

## 🏛️ Understanding the Kubernetes Manifests

In Kubernetes, we use a **declarative approach** to define our application's desired state using YAML files, which are called **manifests**. For this demo, we will use two manifest files located in the `kube-manifests/` directory.

### 1. `deployment.yaml` (The "What" to run)
This file defines a Kubernetes **Deployment**, which manages the lifecycle of our application's pods.

-   **`replicas: 2`**: We are telling Kubernetes that we want **two instances** (pods) of our application running at all times for high availability.
-   **`name: myapp1-deployment`**: The name of our Deployment object.
-   **`image: stacksimplify/kube-nginx:1.0.0`**: This is the [[Docker Image]] our application will run. It's a custom Nginx image.
-   **`containerPort: 80`**: The application inside the container is listening on port 80.

### 2. `service.yaml` (The "How" to access it)
This file defines a Kubernetes **Service**, which provides a stable network endpoint to access our application's pods.

-   **`type: LoadBalancer`**: We are asking the cloud provider (Azure) to provision an external, public-facing Load Balancer for our application.
-   **`port: 80`**: The Load Balancer will listen on port 80.
-   **`targetPort: 80`**: Traffic coming into the Load Balancer on port 80 will be forwarded to port 80 on the pods.

---

## 🛠️ Hands-On: Deploying and Testing the Application

This guide assumes you have already [[Azure Kubernetes Service (AKS)#🔌 Connecting to the AKS Cluster with kubectl|connected to your AKS cluster]] from your local desktop terminal.

### Step 1: Navigate to the Project Directory
In your terminal, navigate to the directory containing the Kubernetes manifests for this section.
```bash
cd 01-create-aks-cluster/
```

### Step 2: Apply the Manifests
Use the `kubectl apply -f <directory>` command to tell Kubernetes to create or update all the resources defined in the YAML files within the specified directory.
```bash
kubectl apply -f kube-manifests/
```
**Output:**
```
deployment.apps/myapp1-deployment created
service/myapp1-loadbalancer created
```
This confirms that both the Deployment and the Service have been created in the cluster.

### Step 3: Inspect the Running Workloads
Now, we'll use a series of `kubectl` commands to inspect what's happening in our cluster.

#### A. Check the Pods
Pods are the smallest deployable units in Kubernetes and they run our application containers.
```bash
kubectl get pods
```
**Output:**
```
NAME                                 READY   STATUS    RESTARTS   AGE
myapp1-deployment-5b5f7f9d5f-abcde   1/1     Running   0          30s
myapp1-deployment-5b5f7f9d5f-fghij   1/1     Running   0          30s
```
You can see that two pods have been created (as requested by `replicas: 2`) and are in the `Running` state.

#### B. Describe a Pod to See Events
To see the detailed lifecycle events of a specific pod, use `kubectl describe`.
```bash
# Copy one of the pod names from the previous command
kubectl describe pod myapp1-deployment-5b5f7f9d5f-abcde
```
In the `Events` section at the bottom of the output, you can see a step-by-step log of what Kubernetes did:
1.  **Scheduled:** Successfully assigned the pod to a worker node.
2.  **Pulling:** Started pulling the `stacksimplify/kube-nginx:1.0.0` image.
3.  **Pulled:** Successfully pulled the image.
4.  **Created:** Created the container.
5.  **Started:** Started the container.

#### C. View Pod Logs
To see the standard output from the application running inside the container, use `kubectl logs`.
```bash
kubectl logs -f myapp1-deployment-5b5f7f9d5f-abcde
```
-   `-f`: Follow the logs in real-time.
For an Nginx server, you won't see any logs until you access it.

#### D. Check the Deployment
A Deployment manages the pods. Check its status to ensure it has reached the desired state.
```bash
kubectl get deployments
# Alias: kubectl get deploy
```
**Output:**
```
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
myapp1-deployment   2/2     2            2           2m
```
The `READY` column `2/2` confirms that both of our desired replicas are up and running.

#### E. Check the Service and Get the External IP
This is the most important step for accessing our application from the internet.
```bash
kubectl get services
# Alias: kubectl get svc
```
**Output:**
```
NAME                   TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)        AGE
myapp1-loadbalancer    LoadBalancer   10.0.123.45   52.123.45.67     80:31234/TCP   3m
kubernetes             ClusterIP      10.0.0.1      <none>           443/TCP        1h
```
-   Kubernetes has created our `myapp1-loadbalancer` service.
-   Azure has provisioned a public IP address and assigned it to the `EXTERNAL-IP` field. It may take a minute or two for this IP to appear.

### Step 4: Access the Application
1.  Copy the `EXTERNAL-IP` address from the `kubectl get svc` command (e.g., `52.123.45.67`).
2.  Paste this IP address into your web browser.
3.  You should see the application's welcome page: **"Welcome to Stack Simplify Kubernetes fundamentals demo, Application V1."**

This confirms that your application has been successfully deployed to AKS and is accessible to the public via an Azure Load Balancer.

---

## 🧹 Cleaning Up Resources

It is a crucial best practice to delete the resources you create after you are done with a demo to avoid incurring unnecessary cloud costs.

### Use `kubectl delete`
The `kubectl delete -f <directory>` command is the inverse of `apply`. It will delete all the resources that were defined in the manifests in that directory.
```bash
kubectl delete -f kube-manifests/
```
**Output:**
```
deployment.apps "myapp1-deployment" deleted
service "myapp1-loadbalancer" deleted
```

### Verify Deletion
Run the `get` commands again to confirm that the resources are gone.
```bash
# Should show "No resources found in default namespace."
kubectl get pods
kubectl get deployments
kubectl get svc
```
This confirms that your application has been successfully removed from the cluster, and Azure will de-provision the associated Load Balancer.