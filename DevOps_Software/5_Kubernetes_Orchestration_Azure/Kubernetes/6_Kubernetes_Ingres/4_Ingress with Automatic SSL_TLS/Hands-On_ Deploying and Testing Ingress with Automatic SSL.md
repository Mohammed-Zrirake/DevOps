#Cloud #Azure #Kubernetes #AKS #Networking #Ingress #Security #SSL #TLS #CertManager #LetsEncrypt #HandsOn #Tutorial

>  This is the final hands-on guide for our Ingress setup. We will deploy our sample applications and a new **`Ingress` manifest that includes a `tls` section**. We will then observe as **`cert-manager`** automatically communicates with **Let's Encrypt** to provision valid, trusted SSL certificates, stores them in Kubernetes [[Managing Kubernetes Secrets|Secrets]], and configures our [[Installing the Nginx Ingress Controller on AKS|Nginx Ingress Controller]] to use them for HTTPS traffic.

---

This guide follows the final steps of the "Ingress SSL with Let's Encrypt" section. It assumes the Nginx Ingress Controller, `external-dns`, and `cert-manager` with a `ClusterIssuer` are already installed and configured.

## ✍️ Step 1: Reviewing the Manifests

### Application Manifests (`02-demo-applications/`)
-   The manifests for `app1` and `app2` are standard. They create a `Deployment` and an internal `ClusterIP` Service for each Nginx application.

### The Ingress Manifest with TLS (`03-ingress-ssl.yaml`)
This is the core of this lecture. It is an `Ingress` resource that specifies not only the routing rules but also the TLS/SSL configuration.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-ssl-demo
  annotations:
    kubernetes.io/ingress.class: "nginx"
    # This annotation tells cert-manager which issuer to use.
    # The name MUST match the ClusterIssuer we created.
    cert-manager.io/cluster-issuer: "letsencrypt-staging"
spec:
  # 1. The 'tls' block is where we define our SSL requirements.
  tls:
  - hosts:
    - sapp1.kubeoncloud.com
    # 'secretName' is where cert-manager will store the generated certificate.
    secretName: sapp1-tls-secret
  - hosts:
    - sapp2.kubeoncloud.com
    secretName: sapp2-tls-secret

  # 2. The 'rules' block defines the standard hostname-based routing.
  rules:
  - host: sapp1.kubeoncloud.com
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-nginx-clusterip-service
            port:
              number: 80
  - host: sapp2.kubeoncloud.com
    http:
      paths:
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-nginx-clusterip-service
            port:
              number: 80
```
-   **`cert-manager.io/cluster-issuer: "letsencrypt-staging"`**: This annotation is a special instruction for `cert-manager`, telling it to use our `letsencrypt-staging` `ClusterIssuer` to obtain certificates for this Ingress.
-   **`spec.tls`**: This block is what triggers the entire automatic SSL process. For each entry, we specify:
    -   `hosts`: The domain name(s) the certificate should be valid for.
    -   `secretName`: The name of the Kubernetes `Secret` that `cert-manager` will create to store the signed certificate and private key. The Nginx Ingress Controller will automatically detect and use this secret to terminate TLS for the specified host.

---

## 🛠️ Step 2: Deploying All Manifests

We can apply all manifests at once. `cert-manager` will be watching for the new `Ingress` resource.

```bash
# Navigate to the 14-ingress-ssl-with-lets-encrypt directory
kubectl apply -R -f kube-manifests/
```
This will create the `ClusterIssuer` (if not already present), the `app1` and `app2` Deployments and Services, and our new `Ingress` resource.

---

## 🔬 Step 3: Observing the Certificate Provisioning Process

This is the most insightful step. We will watch the logs and resources to see `cert-manager` work its magic.

### A. Check the `Certificate` Resources
`cert-manager` automatically creates `Certificate` custom resources when it sees an Ingress with a `tls` block.
```bash
kubectl get certificate
```
**Initial Output:**
```
NAME                 READY   SECRET               AGE
sapp1-tls-secret     False   sapp1-tls-secret     15s
sapp2-tls-secret     False   sapp2-tls-secret     15s
```
-   `READY: False`: This is the initial state. It means `cert-manager` has started the process but has not yet received the signed certificate from Let's Encrypt.

### B. Observe the `cert-manager` Logs (The Magic)
This is the best place to see what's happening.
```bash
# Find the cert-manager pod name
kubectl get pods -n ingress-basic

# Stream the logs
kubectl logs -f <cert-manager-pod-name> -n ingress-basic
```
You will see a detailed log trail showing the ACME challenge process:
-   `cert-manager` will state it is "presenting challenge" for your domains.
-   It will perform a "self check" by trying to access a temporary endpoint it creates to solve the challenge.
-   You may see messages like `Waiting for DNS record propagation...`. This is normal. It's waiting for the `A` records created by `external-dns` to be resolvable on the internet.
-   Eventually, you will see messages about "validating" and "finalizing" the order.
-   Finally, you'll see a success message indicating the certificate was issued.

> [!warning] This process is not instant. It can take **5 minutes to an hour**, depending on DNS propagation. If your domain is not correctly delegated, it will fail.

### C. Verify the Final State
After the process completes (it took the instructor about 5-6 minutes):
-   **Check the Certificates Again:**
    ```bash
    kubectl get certificate
    ```
    The `READY` status should now be `True` for both certificates.

-   **Check the Secrets:** The certificates and private keys are now stored in Kubernetes Secrets.
    ```bash
    kubectl get secret sapp1-tls-secret
    kubectl get secret sapp2-tls-secret
    ```
    You will see that the secrets have been created and are of type `kubernetes.io/tls`.

---

## ✅ Step 4: Accessing the Secure Applications

1.  **Use HTTP:** Open your browser and navigate to the **HTTP** version of your URL, for example: `http://sapp1.kubeoncloud.com/app1/index.html`.
2.  **Automatic Redirect:** The Nginx Ingress Controller, by default, will automatically **redirect you to the HTTPS version** of the site.
3.  **Verify the Certificate:**
    -   You will now be on `https://sapp1.kubeoncloud.com/app1/index.html`.
    -   Click the padlock icon in your browser's address bar to inspect the certificate.
    -   You will see that it was issued to `sapp1.kubeoncloud.com` by a **Let's Encrypt staging issuer**.
        > [!note] Because we used the staging issuer, your browser will show a warning that the certificate is not trusted. This is expected. In a production scenario, you would use the production issuer, and the certificate would be fully trusted.
    -   Repeat the process for `sapp2.kubeoncloud.com` to verify its certificate as well.

---

## 🧹 Step 5: Cleaning Up

1.  **Delete Application and Ingress Resources:**
    ```bash
    kubectl delete -R -f kube-manifests/
    ```
    -   This will delete the Deployments, Services, and the Ingress resource.
    -   `external-dns` will automatically delete the DNS records.
    -   `cert-manager` will automatically delete the `Certificate` resources and the `Secrets` containing the certs.

2.  **(Optional) Uninstall Ingress and Cert-Manager Infrastructure:**
    To fully clean up the cluster, you can uninstall the Helm charts and delete the namespace.
    ```bash
    helm uninstall cert-manager -n ingress-basic
    helm uninstall ingress-nginx -n ingress-basic
    kubectl delete namespace ingress-basic
    ```

---

> [!summary] Conclusion
> This end-to-end demonstration showcases a sophisticated, production-grade, and fully automated SSL setup. By combining an **Ingress resource with a `tls` block**, the **Nginx Ingress Controller**, and **`cert-manager`**, we have created a system that automatically provisions, configures, and renews valid SSL/TLS certificates for our applications without any manual intervention.