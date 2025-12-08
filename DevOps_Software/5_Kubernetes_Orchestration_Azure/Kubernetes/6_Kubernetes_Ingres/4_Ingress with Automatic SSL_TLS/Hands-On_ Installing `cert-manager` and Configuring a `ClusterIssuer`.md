#Cloud #Azure #Kubernetes #AKS #Networking #Ingress #Security #SSL #TLS #CertManager #LetsEncrypt #Helm #HandsOn #Tutorial

>  This is a hands-on guide to installing **`cert-manager`** into our [[Azure Kubernetes Service (AKS)|AKS]] cluster using its Helm chart. After installation, we will create a **`ClusterIssuer`** resource. This `ClusterIssuer` acts as a certificate authority within our cluster, configured to communicate with the **Let's Encrypt** service to automatically issue and renew SSL/TLS certificates.

---

This is the second hands-on step in our Ingress SSL implementation, setting up the core engine that will automate our certificate management.

## ⚙️ Step 1: Install `cert-manager` using Helm

### A. Prerequisite: Disable Namespace Resource Validation
`cert-manager` installs Custom Resource Definitions (CRDs) which can sometimes conflict with webhook validators. The official installation guide recommends disabling validation on the namespace where the controller will be installed.
```bash
# We will install cert-manager in the same namespace as our Ingress Controller
kubectl label namespace ingress-basic cert-manager.io/disable-validation=true
```

### B. Add the Helm Repository
We need to add the official Helm repository for `cert-manager`, which is maintained by Jetstack.
```bash
# Add the Jetstack repository
helm repo add jetstack https://charts.jetstack.io

# Update the local Helm chart repository cache
helm repo update
```

### C. Install the `cert-manager` Helm Chart
We will now install the `cert-manager` chart into our `ingress-basic` namespace.
```bash
helm install cert-manager jetstack/cert-manager \
  --namespace ingress-basic \
  --version v1.8.2 \
  --set installCRDs=true
```
**Command Breakdown:**
-   `helm install cert-manager jetstack/cert-manager`: Install a new release named `cert-manager` from the `jetstack` repository.
-   `--namespace ingress-basic`: Install all components into our existing Ingress namespace.
-   `--version v1.8.2`: **(Important)** Pinning to a specific, stable version of the chart is a best practice.
-   `--set installCRDs=true`: **(Critical for modern versions)** In newer versions of `cert-manager`, the Custom Resource Definitions (CRDs) are not installed by default. This flag explicitly tells Helm to install them. In older versions (0.x), this was a separate `kubectl apply` command.

### D. Verify the Installation
After the Helm installation completes, verify that the `cert-manager` pods are running in the `ingress-basic` namespace.
```bash
kubectl get pods -n ingress-basic
```
**Expected Output:** You should see three new pods running alongside your Ingress Controller pods:
-   `cert-manager-...`
-   `cert-manager-cainjector-...`
-   `cert-manager-webhook-...`

You can also verify the services with `kubectl get svc -n ingress-basic`.

---

## 🏛️ Understanding Core `cert-manager` Concepts

Before creating our `ClusterIssuer`, it's important to understand the key concepts that `cert-manager` introduces.
-   **`Issuer` and `ClusterIssuer`:** These are Kubernetes resources that represent Certificate Authorities (CAs).
    -   An **`Issuer`** is a **namespaced** resource. It can only be used to issue certificates for Ingresses or Certificate objects within the same namespace.
    -   A **`ClusterIssuer`** is a **cluster-wide** resource. It can be referenced by any Ingress or Certificate object in *any* namespace in the cluster. This is the one we will use.
-   **`Certificate`:** A Kubernetes resource that describes a desired X.509 certificate. You can create these manually, but for Ingresses, `cert-manager` can create them automatically for you.
-   **`CertificateRequest`:** A resource that represents a request for a signed certificate from an Issuer. `cert-manager` creates these internally.
-   **ACME Orders and Challenges:** `cert-manager` uses the ACME protocol to communicate with Let's Encrypt. To prove that you own the domain you are requesting a certificate for, it must solve a "challenge." For hostname-based routing, this is typically a **DNS-01 challenge**, where `cert-manager` temporarily creates a special TXT record in your DNS zone.

> [!info] **Let's Encrypt**
> Let's Encrypt is a free, automated, and open Certificate Authority. It provides trusted TLS certificates that are recognized by all major browsers.

---

## ✍️ Step 2: Creating the `ClusterIssuer` Manifest

Now we will create the manifest that tells `cert-manager` how to issue certificates.

### `cluster-issuer.yaml`
```yaml
# API Version for ClusterIssuer is from cert-manager's CRDs
apiVersion: cert-manager.io/v1
# The kind of object
kind: ClusterIssuer
metadata:
  # This name will be referenced in our Ingress annotations
  name: letsencrypt-staging
spec:
  # We are using the ACME protocol, which is what Let's Encrypt uses
  acme:
    # This is the URL for the Let's Encrypt STAGING server.
    # Use this for all testing to avoid rate limits.
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    
    # An email address for registration and recovery contact
    email: your-email@example.com
    
    # The name of a Kubernetes Secret to store the ACME account's private key
    privateKeySecretRef:
      name: letsencrypt-staging-private-key
      
    # 'solvers' define how cert-manager will prove ownership of a domain.
    solvers:
    - http01:
        ingress:
          class: nginx
```
**Key Fields Explained:**
-   **`apiVersion` and `kind`**: These are `cert-manager.io/v1` and `ClusterIssuer`, respectively. These are the new resource types (CRDs) that the Helm chart installed.
-   **`name: letsencrypt-staging`**: A descriptive name for our issuer. We will reference this exact name in our Ingress annotations.
-   **`spec.acme.server`**: This is the URL for the ACME server. **Crucially, we are using the `staging` URL.** The staging server issues certificates that are *not* trusted by browsers, but it does not have the strict rate limits of the production server. **Always use staging for development and testing.**
-   **`spec.acme.email`**: Your email address, used by Let's Encrypt for account registration and important notifications.
-   **`spec.acme.privateKeySecretRef`**: `cert-manager` needs to create an ACME account with Let's Encrypt. It will store the private key for this account in a Kubernetes Secret with this name.
-   **`spec.acme.solvers`**: This defines the mechanism for solving challenges. We are configuring an `http01` solver, which is the most common type. It works by temporarily modifying Ingress resources to serve a specific file at a specific path to prove to Let's Encrypt that we control the traffic for that domain.

### Deploying the `ClusterIssuer`
```bash
# Navigate to the 14-ingress-ssl-with-lets-encrypt directory
kubectl apply -f kube-manifests/01-cert-manager-cluster-issuer.yaml
```

---

> [!summary] Conclusion
> We have successfully installed `cert-manager` and configured a `ClusterIssuer` for the Let's Encrypt staging environment. Our cluster is now equipped with a powerful automation engine for certificate management.
>
> In the next lecture, we will:
> 1.  Deploy our sample applications.
> 2.  Create an `Ingress` manifest that includes a `tls` section and an annotation referencing our new `letsencrypt-staging` `ClusterIssuer`.
> 3.  Observe as `cert-manager` automatically provisions a valid SSL certificate for our application.