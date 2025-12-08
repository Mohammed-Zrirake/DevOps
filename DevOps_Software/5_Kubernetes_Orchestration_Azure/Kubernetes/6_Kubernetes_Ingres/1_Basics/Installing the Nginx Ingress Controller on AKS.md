#Cloud #Azure #Kubernetes #AKS #Networking #Ingress #IngressController #Nginx #Helm #HandsOn #Tutorial

>  This is a hands-on guide to installing the **Nginx Ingress Controller** on an [[Azure Kubernetes Service (AKS)|AKS]] cluster using **Helm**. The process involves installing Helm, creating a dedicated namespace, adding the necessary Helm repositories, and then running a `helm install` command with custom parameters to associate our [[Creating a Static Public IP for an AKS Ingress Controller|static public IP]] and configure the controller for our environment.

---

This is the second hands-on step in our Ingress implementation. We will deploy the Ingress Controller itself, which is the engine that will read our `Ingress` resources and implement the routing rules.

## ✅ Prerequisites: Install Helm

-   **Helm:** Helm is the package manager for Kubernetes. It allows you to define, install, and upgrade even the most complex Kubernetes applications using "charts."
-   **Installation:** You need to have Helm v3 installed on your local machine. You can find installation instructions on the [official Helm website](https://helm.sh/docs/intro/install/).

## 🛠️ Step-by-Step Installation Process

### Step 1: Create a Dedicated Namespace
It is a best practice to install the Ingress Controller and its components in their own dedicated namespace for isolation and better management.
```bash
kubectl create namespace ingress-basic
```

### Step 2: Add the Required Helm Repositories
Helm charts are hosted in repositories. We need to add the official repository for the Nginx Ingress Controller so Helm knows where to find the chart.

```bash
# Add the official ingress-nginx repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# (Optional but shown by instructor) Add another common Google repo
helm repo add stable https://kubernetes-charts.storage.googleapis.com/

# Update the local Helm chart repository cache
helm repo update
```

### Step 3: Customize the Installation with Overrides
The Nginx Ingress Controller Helm chart has many configurable values. We can view all of them with:
```bash
helm show values ingress-nginx/ingress-nginx
```
We will override some of these default values during installation using the `--set` flag to customize it for our Azure environment.

**The `helm install` Command:**
```bash
# Define your static public IP in a variable (replace with your actual IP)
STATIC_IP="52.123.45.67"

# Run the helm install command
helm install ingress-nginx ingress-nginx/ingress-nginx \
    --namespace ingress-basic \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.service.externalTrafficPolicy=Local \
    --set controller.service.loadBalancerIP=$STATIC_IP
```

**Command Breakdown:**
-   `helm install ingress-nginx ingress-nginx/ingress-nginx`: Install a new release named `ingress-nginx` using the `ingress-nginx` chart from the `ingress-nginx` repository.
-   `--namespace ingress-basic`: Install all the components into our dedicated namespace.
-   `--set controller.replicaCount=2`: Override the default of 1 to run **two replicas** of the Ingress Controller pod for high availability.
-   `--set controller.nodeSelector."kubernetes\.io/os"=linux`: Ensure the controller pods are only scheduled on Linux worker nodes.
-   `--set controller.service.externalTrafficPolicy=Local`: A performance and source IP preservation setting. It ensures that traffic is only sent to controller pods running on the same node that received the traffic, preserving the original client's source IP address in the logs.
-   `--set controller.service.loadBalancerIP=$STATIC_IP`: **This is the most critical override.** This tells the Helm chart to configure the `LoadBalancer` service it creates to use the specific, static public IP we provisioned earlier, rather than requesting a new dynamic one.

> [!warning] Deprecated Chart Name
> The instructor notes that older Azure documentation might refer to a deprecated chart name. The correct, modern chart is `ingress-nginx/ingress-nginx`.

### Step 4: Verify the Installation
After the `helm install` command completes, we can verify that all the necessary Kubernetes resources were created.

1.  **Check the Pods:**
    ```bash
    kubectl get pods --namespace ingress-basic
    ```
    You will see two `ingress-nginx-controller-...` pods running, as we requested with `replicaCount=2`.

2.  **Check All Resources in the Namespace:**
    ```bash
    kubectl get all -n ingress-basic
    ```
    This will show all the objects created by the Helm chart, including the `Deployment` for the controller, its `ReplicaSet`, the `Service` of type `LoadBalancer`, and other configuration objects.

3.  **Check the Service's External IP:**
    ```bash
    kubectl get service -n ingress-basic
    ```
    You will see the `ingress-nginx-controller` service. Crucially, its `EXTERNAL-IP` will be the **same static public IP address** that we created and provided during the installation.

4.  **Test the Endpoint:**
    -   Copy the static public IP address.
    -   Access it in a browser.
    -   You will receive a **404 Not Found** error from Nginx. **This is expected and correct.** The Ingress Controller is running, but we haven't created any `Ingress` resources yet to tell it how to route traffic. It is correctly rejecting unknown requests.

### Step 5: Verify in the Azure Portal
1.  Navigate to your cluster's **Load Balancer** in the Azure Portal.
2.  Go to **Frontend IP configuration**. You will see that our static public IP (`myAKSPublicIPForIngress`) has been associated with the load balancer.
3.  Go to **Load balancing rules**. You will see that new rules have been created for HTTP (port 80) and HTTPS (port 443), forwarding traffic to the backend pool where the Ingress Controller pods are running.

---

> [!summary] Conclusion
> We have successfully installed and configured the Nginx Ingress Controller on our AKS cluster using Helm. The controller is now running and exposed to the internet via our dedicated static public IP. It is now "listening" for `Ingress` resources to be created.
>
> In the next lecture, we will deploy a sample application and create an `Ingress` manifest to define a routing rule, which will finally allow us to access our application through the Ingress Controller.