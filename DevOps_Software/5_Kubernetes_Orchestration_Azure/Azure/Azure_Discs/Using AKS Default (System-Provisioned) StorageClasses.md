#Cloud #Azure #Kubernetes #AKS #Storage #PersistentStorage #AzureDisk #StorageClass #PVC #HandsOn #Tutorial

>  [[Azure Kubernetes Service (AKS)|AKS]] clusters come with pre-configured, default **`StorageClass`** objects for Azure Disk and Azure Files. You can leverage these system-provisioned classes directly in your [[Writing Kubernetes Manifests The PersistentVolumeClaim (PVC)|PersistentVolumeClaim (PVC)]] without needing to create a custom `StorageClass` manifest. The key difference to be aware of is that these default classes typically use a **`reclaimPolicy: Delete`**, which means the underlying Azure Disk will be automatically deleted when the PVC is deleted.

---

## ❓ Why Use Default StorageClasses?

In the previous section, we created a [[Writing Kubernetes Manifests The StorageClass|custom `StorageClass`]] primarily to learn the concepts and to set a `reclaimPolicy: Retain` for our database. However, for many use cases, especially in development or for non-critical data, using the default classes is simpler and more convenient.

-   **Simplicity:** You don't need to write and maintain a `StorageClass` manifest.
-   **Convenience:** AKS has already defined classes for common storage tiers (e.g., standard HDD and premium SSD).
-   **Automatic Cleanup:** The `reclaimPolicy: Delete` is often desirable for temporary or development environments, as it automatically cleans up cloud resources and prevents orphaned disks and unwanted costs.

You can view the available default StorageClasses by running:
```bash
kubectl get storageclass
# Alias: kubectl get sc
```
**Example Output:**
```
NAME                PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
azurefile           kubernetes.io/azure-file   Delete          Immediate              true                   1h
azurefile-premium   kubernetes.io/azure-file   Delete          Immediate              true                   1h
default (default)   kubernetes.io/azuredisk    Delete          WaitForFirstConsumer   true                   1h
managed-premium     kubernetes.io/azuredisk    Delete          WaitForFirstConsumer   true                   1h
```

---

## 🛠️ Hands-On: Deploying MySQL with a Default `StorageClass`

This guide follows the instructor's process for adapting our previous MySQL deployment to use a default `StorageClass`.

### Step 1: Modify the `PersistentVolumeClaim` Manifest
The only change we need to make is in our `pvc-definition.yaml`. We will remove our custom `storage-class.yaml` file entirely.

1.  **Copy Manifests:** Copy the entire `kube-manifests` folder from the previous section (`05-01`) to the new section's folder (`05-02`).
2.  **Delete the StorageClass file:** Remove `01-storage-class.yaml`. We no longer need it.
3.  **Update the PVC:** Open the `pvc-definition.yaml` file and change the `storageClassName` to one of the default names. We will use `managed-premium`.

**Updated `pvc-definition.yaml`:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk-pvc
spec:
  accessModes:
  - ReadWriteOnce
  # We are now referencing the default StorageClass provided by AKS
  storageClassName: managed-premium
  resources:
    requests:
      storage: 5Gi
```
All other manifests (`ConfigMap`, `Deployment`, `Service`) remain exactly the same.

### Step 2: Deploy the Application
We can apply all the manifests in the directory at once.
```bash
kubectl apply -f kube-manifests/
```
This will create the `PVC`, `ConfigMap`, `Deployment`, and `Service`.

### Step 3: Verify the Provisioning
1.  **Check the `PVC` and `PV`:**
    ```bash
    kubectl get pvc
    kubectl get pv
    ```
    You will see that the `PVC` is created and quickly transitions to the `Bound` state after the `MySQL` pod starts creating (due to the `WaitForFirstConsumer` mode on the default class). A new `PersistentVolume` (PV) will be dynamically provisioned.

2.  **Inspect the `PV`:**
    ```bash
    kubectl describe pv <pv-name>
    ```
    Note that the `Reclaim Policy` for this PV is **`Delete`**, as inherited from the `managed-premium` StorageClass.

3.  **Check the Azure Disk:** Go to the **Disks** service in the Azure Portal. You will see a new managed disk has been created, corresponding to this PV.

4.  **Verify the MySQL Pod:**
    ```bash
    # Check that the pod is running
    kubectl get pods
    
    # Connect to the database and verify the schema was created
    kubectl run -it --rm --image=mysql:5.6 mysql-client -- mysql -h mysql -pdb_password
    mysql> SHOW DATABASES;
    ```
    You should see the `webappdb` database, confirming that the `ConfigMap` initialization worked correctly.

### Step 4: Test the `reclaimPolicy: Delete`
Now, let's observe the key difference in behavior when we clean up.

1.  **Delete All Kubernetes Resources:**
    ```bash
    kubectl delete -f kube-manifests/
    ```
    This deletes the `Deployment`, `Service`, `ConfigMap`, and importantly, the `PVC`.

2.  **Check the `PV` and Azure Disk:**
    -   `kubectl get pv`: The `PersistentVolume` will be gone almost immediately.
    -   **Azure Portal:** Go back to the **Disks** service and refresh. The Azure Managed Disk that was created for this build will also be **gone**.

---

> [!summary] Conclusion
> This demo clearly illustrates the behavior of the default `StorageClass` objects in AKS. By using the `reclaimPolicy: Delete`, they provide a convenient "self-cleaning" mechanism. When the `PVC` is deleted, the entire storage lifecycle—including the `PV` and the underlying cloud disk—is automatically terminated and cleaned up. This is ideal for development and testing environments.
>
> However, for production databases where data loss is unacceptable, it is critical to use a custom `StorageClass` with **`reclaimPolicy: Retain`** to protect against accidental deletion.