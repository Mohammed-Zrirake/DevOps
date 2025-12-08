#Cloud #Azure #Kubernetes #AKS #Storage #PersistentStorage #AzureFiles #StorageClass #PVC #RWX #HandsOn #Tutorial

>  This is a comprehensive, hands-on guide to using **[[Persistent Storage in AKS with Azure Files|Azure Files]]** as a persistent, **shared** storage solution in [[Azure Kubernetes Service (AKS)|AKS]]. We will define a custom `StorageClass` and `PersistentVolumeClaim` (PVC) to dynamically provision an Azure File share, deploy a multi-replica Nginx [[The Kubernetes Deployment|Deployment]] that mounts this shared volume, and then dynamically upload content to the file share to test live access from the web server pods.

---

This tutorial demonstrates the "V1" approach, where we create a custom `StorageClass` to have fine-grained control over our storage.

## ✍️ Step 1: Writing the Manifests (`kube-manifests-v1/`)

### The `storage-class.yaml`
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-azurefile-sc
# The provisioner for Azure Files is different from Azure Disks
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict
parameters:
  # Defines the redundancy level of the underlying Azure Storage Account
  skuName: Standard_LRS
```
-   **`provisioner: kubernetes.io/azure-file`**: This is the key that tells Kubernetes to use the Azure Files provisioner.
-   **`mountOptions`**: These are Linux mount options that control permissions and behavior of the mounted file share. `0777` grants read/write/execute permissions to all users.
-   **`parameters.skuName`**: We are requesting `Standard_LRS` (Locally-Redundant Storage), which is a cost-effective HDD-based option.
-   **Implicit Defaults:** The instructor notes that we have omitted fields like `reclaimPolicy` and `volumeBindingMode`. Therefore, they will fall back to their defaults:
    -   `reclaimPolicy`: `Delete`
    -   `volumeBindingMode`: `Immediate`

### The `persistent-volume-claim.yaml`
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-azurefile-pvc
spec:
  accessModes:
    # This is the key feature of Azure Files: one volume can be mounted by many pods
    - ReadWriteMany
  storageClassName: my-azurefile-sc
  resources:
    requests:
      storage: 5Gi
```
-   **`accessModes: [ReadWriteMany]`**: This is the critical difference from Azure Disks. `RWX` allows the volume to be mounted as read-write by **multiple nodes simultaneously**.

### The `deployment.yaml` and `service.yaml`
These are standard manifests for an Nginx deployment (with 4 replicas) and a `LoadBalancer` Service to expose it. The key is how the volume is mounted.

```yaml
# In deployment.yaml
# ...
spec:
  template:
    # ...
    spec:
      containers:
      - name: my-nginx-container
        image: nginx
        volumeMounts:
        # Mounts the volume named 'my-azurefile-volume' into the container
        - name: my-azurefile-volume
          # This is the path where Nginx serves static content
          mountPath: /usr/share/nginx/html/app1
      volumes:
      # Defines a volume available to the Pod, sourced from our PVC
      - name: my-azurefile-volume
        persistentVolumeClaim:
          claimName: my-azurefile-pvc
```

---

## 🛠️ Step 2: Deploying and Verifying the Initial State

1.  **Deploy All Manifests:**
    ```bash
    # Navigate to the 08-azure-files directory
    kubectl apply -f kube-manifests-v1/
    ```
    This creates the `StorageClass`, `PVC`, `Deployment`, and `Service`.

2.  **Verify Storage Objects:**
    -   `kubectl get sc`: Shows our `my-azurefile-sc` has been created.
    -   `kubectl get pvc`: Shows `my-azurefile-pvc` is in the `Bound` state. Because `volumeBindingMode` was `Immediate`, the volume was provisioned right away.
    -   `kubectl get pv`: Shows that a `PersistentVolume` has been created and is bound to the PVC.

3.  **Verify Workloads:**
    -   `kubectl get pods`: Shows that all four Nginx pods are running.
    -   `kubectl get svc`: Get the `EXTERNAL-IP` of the Nginx `LoadBalancer` service.

4.  **Test the Application (Expect 404):**
    -   Access the `EXTERNAL-IP` in your browser. You will see the default Nginx "Welcome" page. This is served from the container's image, not our file share.
    -   Now, navigate to the path where we mounted our volume: `http://<EXTERNAL-IP>/app1/index.html`.
    -   You will receive a **404 Not Found** error. **This is expected.** We have mounted an empty Azure File share over the `app1` directory inside the container, effectively hiding any files that might have been there in the image. The file share currently contains no files.

---

## 📥 Step 3: Uploading Static Content to the Azure File Share

Now we will upload our static HTML files directly to the Azure File share, and they will become immediately available to all running Nginx pods.

1.  **Navigate to the Storage Account:**
    -   In the Azure Portal, go to **Storage accounts**.
    -   Find the storage account associated with your AKS cluster's infrastructure resource group (the one starting with `MC_...`).
2.  **Find the File Share:**
    -   Inside the storage account, go to **File shares**.
    -   You will see a file share with a name that matches the `PersistentVolume` created by Kubernetes (e.g., `kubernetes-dynamic-pvc-<some-id>`). This is our volume.
3.  **Upload Files:**
    -   Click on the file share to open it.
    -   Click the **Upload** button.
    -   Select the static HTML files to upload (e.g., `file1.html` and `file2.html`).
    -   Confirm the upload.

---

## ✅ Step 4: Accessing the Application and Testing

1.  **Access the Files:** Go back to your browser and refresh the pages for the files you just uploaded.
    -   `http://<EXTERNAL-IP>/app1/file1.html` -> Now displays "Welcome to Stack Simplify - File 1".
    -   `http://<EXTERNAL-IP>/app1/file2.html` -> Now displays "Welcome to Stack Simplify - File 2".

2.  **Test Dynamic Updates:** Go back to the Azure Portal, delete `file2.html` from the file share, and then refresh `http://<EXTERNAL-IP>/app1/file2.html`. It will now show a 404 error again.

This demonstrates that all four Nginx pods are reading from the **same, single, shared** Azure File share. Any content you add, remove, or update in that central location is instantly reflected across all instances of your application.

---

## 🧹 Step 5: Cleaning Up

Because our custom `StorageClass` defaulted to `reclaimPolicy: Delete`, the cleanup is fully automated.

```bash
kubectl delete -f kube-manifests-v1/
```
-   This command deletes the `Deployment`, `Service`, `PVC`, and `StorageClass`.
-   When the `PVC` is deleted, the Kubernetes controller automatically deletes the `PV` and signals to Azure to **delete the underlying Azure File share** as well.
-   If you check the **File shares** section in the Azure Portal after a minute, the dynamically created share will be gone.

---

> [!summary] V2 Approach: Using the Default `StorageClass`
> The instructor also provides a `kube-manifests-v2` folder. This approach is identical to V1, except:
> -   The `storage-class.yaml` is removed.
> -   The `pvc.yaml` is updated to use one of the default `storageClassName`s provided by AKS, like `azurefile`.
> 
> This is a simpler approach if you don't need the granular control (like geo-redundancy) offered by a custom `StorageClass`.