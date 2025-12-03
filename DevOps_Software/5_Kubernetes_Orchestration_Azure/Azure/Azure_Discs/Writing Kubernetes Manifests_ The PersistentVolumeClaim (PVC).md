#Cloud #Azure #Kubernetes #AKS #Storage #PersistentStorage #AzureDisk #StorageClass #PVC #HandsOn #Tutorial

>  A **`PersistentVolumeClaim`** (PVC) is a **request for storage** by a user or application. It's how a [[The Kubernetes Pod|Pod]] asks for a piece of persistent storage without needing to know the underlying infrastructure details. The PVC specifies the *size* and *access mode* required and references a [[Writing Kubernetes Manifests The StorageClass|StorageClass]] that will dynamically provision the actual storage (the `PersistentVolume` and the underlying Azure Disk).

---

This guide is the second part of our stateful application setup. We will create a PVC manifest that uses the custom `StorageClass` we defined previously.

## 🏛️ The `PersistentVolumeClaim` Manifest (`pvc-definition.yaml`)

We will build the manifest from scratch using the four [[Writing Kubernetes Manifests Pods with YAML#🏛️ The Four Top-Level Objects of a Kubernetes Manifest|top-level objects]].

```yaml
# 1. API Version for PVC is 'v1' (it's a core object)
apiVersion: v1
# 2. The kind of object is 'PersistentVolumeClaim'
kind: PersistentVolumeClaim
# 3. Metadata to identify the PVC
metadata:
  name: azure-managed-disk-pvc
spec:
  # 4. Specification of the storage request
  
  # 'accessModes' defines how the volume can be mounted.
  # 'ReadWriteOnce' means the volume can be mounted as read-write by a single node.
  accessModes:
  - ReadWriteOnce
  
  # 'storageClassName' links this PVC to a specific StorageClass.
  # This is how we use our custom, 'Retain' policy class.
  storageClassName: managed-premium-retain
  
  # 'resources' defines the amount of storage being requested.
  resources:
    requests:
      storage: 5Gi
```

### A Deep Dive into the `spec` Section

The `spec` for a PVC has three essential fields:

#### 1. `accessModes`
-   **What it is:** A list that defines the read/write permissions for the volume and how it can be mounted. The access mode is a constraint on the underlying `PersistentVolume`.
-   **`ReadWriteOnce` (RWO):** This is the most common mode for block storage like Azure Disk. It means the volume can be mounted as **read-write by a single node at a time**.
-   **Other Modes:** `ReadOnlyMany` (ROX) - read-only by many nodes; `ReadWriteMany` (RWX) - read-write by many nodes (supported by file storage like Azure Files, not Azure Disk); `ReadWriteOncePod` (RWOP) - read-write by a single Pod.

#### 2. `storageClassName`
-   **What it is:** The name of the `StorageClass` that this PVC should use for dynamic provisioning.
-   **Our Value:** `managed-premium-retain`. This is the crucial link to the custom `StorageClass` we created in the previous lecture. By specifying this, our PVC inherits all its properties, including the `reclaimPolicy: Retain` and `volumeBindingMode: WaitForFirstConsumer`.

#### 3. `resources`
-   **What it is:** A dictionary where you define the storage requirements.
-   **`requests.storage`:** This is where you specify how much storage you need. We are requesting `5Gi` (5 Gibibytes).

---

## 🛠️ Hands-On: Deploying the `StorageClass` and `PVC`

### Step 1: Deploy the Manifests
1.  Navigate to the directory containing your manifest files (`kube-manifests/`).
2.  Apply the `StorageClass` manifest first, followed by the `PVC` manifest.


   ```bash
    # Create the StorageClass
    kubectl apply -f 01-storage-class.yaml
    
    # Create the PersistentVolumeClaim
    kubectl apply -f 02-persistent-volume-claim.yaml
    ```

### Step 2: Verify the Created Resources
Now, let's inspect the state of our storage objects in the cluster.

#### A. Verify the `StorageClass`
```bash
kubectl get storageclass
# Alias: kubectl get sc
```
**Output:** You will see our custom `managed-premium-retain` class listed, alongside the default classes provided by AKS. Note its `RECLAIMPOLICY` is `Retain` and its `VOLUMEBINDINGMODE` is `WaitForFirstConsumer`.

#### B. Verify the `PersistentVolumeClaim`
```bash
kubectl get persistentvolumeclaim
# Alias: kubectl get pvc
```
**Output:**
```
NAME                   STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS             AGE
azure-managed-disk-pvc   Pending                                    managed-premium-retain   15s
```
-   **`STATUS: Pending`**: This is the expected and correct state. The PVC is "pending" because our `StorageClass` is configured with `volumeBindingMode: WaitForFirstConsumer`. The PVC will remain in this state, and no actual Azure Disk will be created, until a Pod that claims this PVC is scheduled.

#### C. Verify the `PersistentVolume`
```bash
kubectl get persistentvolume
# Alias: kubectl get pv
```
**Output:** `No resources found.` This is also expected. No `PersistentVolume` (and no underlying Azure Disk) has been provisioned yet, because the PVC is still pending.

### The Benefit of `WaitForFirstConsumer`
The instructor highlights a key benefit of this volume binding mode, especially in multi-zone cloud environments.
-   If `volumeBindingMode` were `Immediate` (the default for older versions), the Azure Disk would be created instantly in a specific availability zone (e.g., `us-east-1a`).
-   Later, when you create your MySQL Pod, the Kubernetes scheduler might decide to place it in a *different* availability zone (e.g., `us-east-1b`).
-   Because most cloud block storage (like Azure Disk) is zone-specific, the Pod in `1b` would be **unable to mount the disk** from `1a`, causing the Pod to fail to start.
-   By using `WaitForFirstConsumer`, the disk provisioning is delayed until the Pod is scheduled. The storage provisioner can then ensure the disk is created in the **same availability zone** as the Pod, avoiding this cross-zone latency and failure issue.

---

> [!summary] Conclusion
> You have successfully created a `PersistentVolumeClaim` using a declarative YAML manifest. We've seen how it requests a specific amount of storage and links to a `StorageClass` to define *how* that storage should be provisioned. We also understand that our PVC is correctly in a `Pending` state due to the `WaitForFirstConsumer` binding mode, and it is now waiting for a Pod (our upcoming MySQL deployment) to claim it before the actual Azure Disk is created.