#Cloud #Azure #Kubernetes #AKS #Storage #PersistentStorage #AzureDisk #StorageClass #PVC #HandsOn #Tutorial

>  A **`StorageClass`** is a Kubernetes object that allows administrators to define different "classes" of storage. It acts as a **template for dynamically provisioning storage**. When a user requests storage via a [[Persistent Storage in AKS with Azure Disks#🧠 Kubernetes Concepts to be Mastered|PersistentVolumeClaim (PVC)]], the `StorageClass` tells the cluster *what kind* of underlying physical storage to create (e.g., a standard HDD vs. a premium SSD Azure Disk) and *how* to behave (e.g., what to do with the storage when the claim is deleted).

---

## 🏛️ The Need for a Custom `StorageClass`

When you create an [[Azure Kubernetes Service (AKS)|AKS cluster]], it comes with several **default `StorageClass` objects** already provisioned. You can see them by running:
```bash
kubectl get sc # 'sc' is the alias for storageclasses
```
This will list classes for `azure-disk` and `azure-file`, with different performance tiers (e.g., `default` and `managed-premium`).

So, why would we create a *custom* `StorageClass`? The primary reason is to **override the default behavior** for critical, stateful applications like a MySQL database.

The default `StorageClass` provided by Azure uses a `reclaimPolicy` of `Delete`. This means when you delete the `PersistentVolumeClaim` (PVC) that requested the storage, the underlying Azure Disk is **automatically deleted**, and your data is lost forever. For a production database, this is extremely dangerous.

> [!danger] The Goal of Our Custom StorageClass
> We will create a custom `StorageClass` with a `reclaimPolicy: Retain`. This ensures that even if the Kubernetes workload (Pod, Deployment) and its PVC are deleted, the underlying Azure Disk on which the data is stored **will be preserved**. This provides a critical safety net for our database.

---

## ✍️ Writing the `storage-class.yaml` Manifest from Scratch

We will build the manifest using the four [[Writing Kubernetes Manifests Pods with YAML#🏛️ The Four Top-Level Objects of a Kubernetes Manifest|top-level objects]].

```yaml
# 1. API Version for StorageClass
apiVersion: storage.k8s.io/v1
# 2. The kind of object is 'StorageClass'
kind: StorageClass
# 3. Metadata to identify the StorageClass
metadata:
  name: managed-premium-retain
  
# 4. No 'spec' for StorageClass, it uses top-level parameters

# Defines the underlying storage provider. For Azure Disks, this is the value.
provisioner: kubernetes.io/azuredisk

# Defines what happens to the underlying storage when the PVC is deleted.
# 'Retain' keeps the Azure Disk. 'Delete' is the default.
reclaimPolicy: Retain

# Controls when the volume binding and dynamic provisioning should occur.
# 'WaitForFirstConsumer' delays binding until a Pod that uses the PVC is scheduled.
volumeBindingMode: WaitForFirstConsumer

# Allows the volume to be resized after creation.
allowVolumeExpansion: true

# Cloud-provider specific parameters
parameters:
  # Defines the performance tier of the Azure Disk.
  storageaccounttype: Premium_LRS
  # Specifies that a managed disk should be created, not an unmanaged one in a storage account.
  kind: Managed
```

### A Deep Dive into the `StorageClass` Fields

Unlike other Kubernetes objects, `StorageClass` does not have a `spec` block. Its configuration parameters are at the top level.

-   **`apiVersion`:** For `StorageClass`, this is `storage.k8s.io/v1`. This indicates it's part of the core Kubernetes storage API group.
-   **`kind`:** `StorageClass`.
-   **`metadata.name`:** We'll name our custom class `managed-premium-retain` to be descriptive.
-   **`provisioner`:** This is the most important field. It specifies which volume plugin is used for provisioning the physical storage. For Azure Disks, the value must be `kubernetes.io/azuredisk`.
-   **`reclaimPolicy`:**
    -   **`Retain` (Our choice):** When the PVC is deleted, the corresponding PersistentVolume (PV) and the underlying Azure Disk are *not* deleted. They remain available for manual recovery or re-attachment. This is the safest option for production databases.
    -   **`Delete` (The default):** When the PVC is deleted, the PV and the Azure Disk are also deleted. This is convenient for temporary or development storage but dangerous for critical data.
-   **`volumeBindingMode`:**
    -   **`WaitForFirstConsumer` (Our choice):** This is the recommended mode. It delays the dynamic provisioning of the Azure Disk until a Pod that actually uses the PVC is created and scheduled to a node. This allows the scheduler to make intelligent decisions about where to place the Pod, considering storage topology and availability zones.
    -   **`Immediate` (The default):** As soon as you create the PVC, Kubernetes immediately provisions a PV and the underlying Azure Disk, without waiting for a Pod to consume it.
-   **`allowVolumeExpansion`:** Setting this to `true` allows you to resize the PVC at a later time, which will trigger the resizing of the underlying Azure Disk.
-   **`parameters`:** This is a dictionary of key-value pairs that are specific to the `provisioner`.
    -   **`storageaccounttype`:** Defines the performance tier of the Azure Disk. `Premium_LRS` corresponds to a high-performance SSD. Other options include `Standard_LRS` (HDD).
    -   **`kind`:** `Managed`. This tells the provisioner to create a modern Azure Managed Disk, which is the standard. The alternative, `shared` or `dedicated`, is for older, unmanaged disks.

---

## 🔜 Next Steps: The `PersistentVolumeClaim` (PVC)

Now that we have created the **template** for our storage (the `StorageClass`), the next step is for our application to **request** storage using that template. This is done by creating a **`PersistentVolumeClaim`** (PVC).

In the next lecture, we will write the YAML manifest for a PVC that references our new `managed-premium-retain` `StorageClass` and requests a specific amount of storage (e.g., 1 GiB) for our MySQL database.