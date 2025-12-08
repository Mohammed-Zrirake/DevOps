#Cloud #Azure #Kubernetes #AKS #Storage #PersistentStorage #AzureFiles #StorageClass #PVC #RWX #HandsOn #Tutorial

>  This guide details how to use **Azure Files** to provide **`ReadWriteMany` (RWX)** persistent storage to workloads in [[Azure Kubernetes Service (AKS)|AKS]]. Unlike [[Persistent Storage in AKS with Azure Disks|Azure Disks]] (`ReadWriteOnce`), an Azure File share can be mounted as read-write by **multiple Pods simultaneously**. We will create a custom `StorageClass` for Azure Files, a `PersistentVolumeClaim` (PVC) to request a share, and then mount it into a multi-replica NGINX [[The Kubernetes Deployment|Deployment]].

---

This guide is a detailed review of the four YAML manifests required to provision and use an Azure File share as a persistent volume for a multi-replica application.

## 🏛️ Reviewing the YAML Manifests

We will be working with four manifest files in the `kube-manifests-v1/` directory.

### 1. The `StorageClass` Manifest (`storage-class.yaml`)
This manifest defines a *template* for creating our Azure File storage. It's a custom class, which allows us to fine-tune its behavior.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-azurefile-sc
# No 'spec' block, parameters are top-level
provisioner: kubernetes.io/azure-file
parameters:
  skuName: Standard_LRS
mountOptions:
- dir_mode=0777
- file_mode=0777
- uid=0
- gid=0
- mfsymlinks
- cache=strict
- actimeo=30
```

**Key Fields Explained:**
-   **`provisioner: kubernetes.io/azure-file`**: This is the critical field. It tells Kubernetes to use the built-in Azure Files volume plugin to provision the storage, as opposed to `kubernetes.io/azuredisk`.
-   **`parameters.skuName: Standard_LRS`**: Specifies the performance tier for the Azure File share. `Standard_LRS` (Locally-redundant storage) is a cost-effective option. `Premium_LRS` is also available for higher performance, but requires a minimum share size of 100GB.
-   **`mountOptions`**: A list of options that control how the file share is mounted onto the node's filesystem. This allows for fine-grained control over permissions. For example, you can specify a specific `uid` and `gid` to control ownership of the files, or change `dir_mode` and `file_mode` for more restrictive permissions than the default `0777`.

### 2. The `PersistentVolumeClaim` Manifest (`pvc.yaml`)
This manifest is a *request* for storage using the template we just defined.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-azurefile-pvc
spec:
  accessModes:
  - ReadWriteMany # The key feature of Azure Files
  storageClassName: my-azurefile-sc
  resources:
    requests:
      storage: 5Gi
```

**Key Fields Explained:**
-   **`accessModes: [ReadWriteMany]`**: This is the main reason to use Azure Files over Azure Disks. `ReadWriteMany` (RWX) allows the volume to be mounted as **read-write by many nodes (and thus, many Pods) simultaneously**. This is perfect for scenarios where multiple replicas of an application need to read and write to a shared filesystem.
-   **`storageClassName: my-azurefile-sc`**: This links the PVC to our custom `StorageClass`, ensuring an Azure File share is created with our specified `mountOptions`.
-   **`resources.requests.storage: 5Gi`**: We are requesting a 5 Gibibyte file share.

### 3. The NGINX `Deployment` Manifest (`deployment.yaml`)
This manifest defines our web server application, which will use the Azure File share.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 4 # We will run 4 replicas to demonstrate shared access
  selector:
    matchLabels:
      app: nginx-azurefile
  template:
    metadata:
      labels:
        app: nginx-azurefile
    spec:
      containers:
      - name: nginx-container
        image: stacksimplify/kube-nginx:1.0.0
        imagePullPolicy: Always
        # This section mounts the volume into the container
        volumeMounts:
        - name: my-azurefile-volume
          mountPath: /usr/share/nginx/html/app1
      # This section defines the volume at the Pod level
      volumes:
      - name: my-azurefile-volume
        persistentVolumeClaim:
          claimName: my-azurefile-pvc
```
**Key Fields Explained (The Storage Part):**
-   **`spec.template.spec.volumes`**: At the Pod template level, we define a volume named `my-azurefile-volume`. Its source is our `PersistentVolumeClaim` with the `claimName: my-azurefile-pvc`. This makes the Azure File share available to the Pod.
-   **`spec.template.spec.containers.volumeMounts`**: Inside the container definition, we create a `volumeMount`. We reference the volume by `name` (`my-azurefile-volume`) and specify a `mountPath`. This is the directory *inside the container* where the shared file system will be accessible (e.g., `/usr/share/nginx/html/app1`). All four NGINX pods will have this same file share mounted at this same location.

### 4. The `LoadBalancer` Service Manifest (`service.yaml`)
This is a standard `LoadBalancer` Service to expose our multi-replica NGINX deployment to the internet.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: nginx-azurefile # This matches the labels on our NGINX pods
  ports:
  - port: 80
    targetPort: 80
```

---

> [!summary]
> We have now reviewed the complete set of manifests required to deploy a multi-replica application that uses a shared, persistent `ReadWriteMany` volume backed by Azure Files.
>
> In the next lecture, we will:
> 1.  Deploy these manifests to our AKS cluster.
> 2.  Verify that the Azure File share and associated Kubernetes objects are created.
> 3.  Test the shared storage by creating a file in the volume from one Pod and verifying that it is immediately visible in the other Pods.