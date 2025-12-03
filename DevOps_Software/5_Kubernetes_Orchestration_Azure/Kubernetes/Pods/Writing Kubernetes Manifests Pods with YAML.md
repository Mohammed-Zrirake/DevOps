#DevOps #Containerization #Kubernetes #CoreConcept #YAML #Pods #Manifests #Declarative #HandsOn #Tutorial

>  The declarative approach, using **YAML manifests**, is the standard and recommended way to define and manage resources in Kubernetes. Every manifest is structured with four top-level objects: `apiVersion`, `kind`, `metadata`, and `spec`. This guide will break down these objects and then use them to build a [[The Kubernetes Pod|Pod]] definition from scratch.

---

## 🏛️ The Four Top-Level Objects of a Kubernetes Manifest

Every Kubernetes YAML manifest is built around four essential, top-level keys. Understanding these is fundamental to writing any Kubernetes definition.

### 1. `kind`
-   **What it is:** A string that specifies the **type of Kubernetes resource** you are defining.
-   **Values:** This can be `Pod`, `ReplicaSet`, `Deployment`, `Service`, `StatefulSet`, `Job`, `ConfigMap`, etc. The value is case-sensitive (e.g., `Pod`, not `pod`).
-   **Purpose:** It tells Kubernetes what kind of object to create.

### 2. `apiVersion`
-   **What it is:** A string that specifies the version of the Kubernetes API you are using to create this object.
-   **Why it's needed:** Kubernetes has a versioned API that evolves over time. Different `kind`s of objects belong to different API groups and versions. For example:
    -   Core objects like `Pod` and `Service` often use `v1`.
    -   Workload objects like `Deployment` and `ReplicaSet` belong to the `apps` group and use `apps/v1`.
-   **How to find it:** The official Kubernetes API documentation is the definitive source for finding the correct `apiVersion` for any given `kind`.

### 3. `metadata`
-   **What it is:** A dictionary (or map) that contains data to help uniquely identify the object.
-   **Core Fields:**
    -   **`name`:** A string that provides a **unique name** for the object within its namespace. This is a **mandatory** field for almost every Kubernetes object.
    -   **`labels`:** A dictionary of key-value pairs that are attached to the object. Labels are the primary way to organize and select subsets of objects. While optional for some objects, they are **effectively mandatory for Pods** that need to be managed by a controller like a ReplicaSet or targeted by a Service.
    -   **`namespace`:** A string that specifies the namespace the object belongs to. If omitted, the object is created in the `default` namespace.

### 4. `spec` (Specification)
-   **What it is:** This is the most important section. It is a dictionary where you define the **desired state** for the object.
-   **Content:** The structure of the `spec` is different for every `kind` of Kubernetes object. For a Pod, the `spec` describes the containers that should run inside it. For a Service, it describes which pods to target and which ports to expose.

---

## 🛠️ Hands-On: Writing a Pod Definition from Scratch

Let's apply these concepts to write a YAML manifest for a simple Pod. This will be saved in a file named `pod-definition.yaml`.

### The `pod-definition.yaml` Manifest
```yaml
# 1. API Version for a Pod is 'v1'
apiVersion: v1
# 2. The kind of object we are creating is a 'Pod' (case-sensitive)
kind: Pod
# 3. Metadata to identify the Pod
metadata:
  # The unique name for this Pod
  name: myapp-pod
  # Labels to categorize this Pod. This is a dictionary.
  labels:
    app: myapp
spec:
  # 4. Specification of the desired state for this Pod
  # 'containers' is a list, as a Pod can have multiple containers.
  containers:
  # The hyphen indicates the start of a list item (the first container).
  - name: myapp-container
    image: stacksimplify/kube-nginx:1.0.0
    # 'ports' is also a list of ports this container exposes.
    ports:
    - containerPort: 80
```
**YAML Breakdown:**
-   `apiVersion` and `kind` are simple key-value pairs.
-   `metadata` is a dictionary containing `name` and another nested dictionary, `labels`.
-   `spec` is a dictionary that contains a list named `containers`.
-   Each item in the `containers` list is a dictionary describing a container, with keys like `name`, `image`, and a list of `ports`.

> [!info] **Multi-Container Pods**
> The reason `containers` is a list is to support the advanced (and rare) use case of multi-container pods. In 99% of cases, this list will only contain a single container definition.

### Creating the Pod in the Cluster
1.  Navigate to the directory containing your `pod-definition.yaml` file.
2.  Use the `kubectl apply` (or `create`) command to create the resource in your cluster.
    ```bash
    kubectl apply -f pod-definition.yaml
    ```
    **Output:** `pod/myapp-pod created`

3.  Verify that the pod is running:
    ```bash
    kubectl get pods
    ```
    You will see your `myapp-pod` in the `Running` state.

---

## 🔜 Next Steps: Exposing the Pod

While we have successfully created a Pod using a declarative YAML manifest, this Pod is currently only accessible from within the cluster's internal network. The next logical step would be to create a [[Kubernetes Services The LoadBalancer Type in Azure (AKS)|LoadBalancer Service]] (also defined in YAML) to expose this Pod to the public internet.

This completes the foundational process of creating a Pod declaratively. This same pattern (`apiVersion`, `kind`, `metadata`, `spec`) will be used for every other Kubernetes object you create.