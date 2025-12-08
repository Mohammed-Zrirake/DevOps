#Cloud #Azure #Kubernetes #AKS #Networking #DNS #AzureDNS #DomainDelegation #HandsOn #Tutorial

>  This is a hands-on guide to creating an **Azure DNS Zone** and then **delegating** a domain you own from an external registrar (in this case, AWS Route 53) to Azure. The process involves creating the DNS Zone in Azure to get its assigned Name Servers, and then updating the Name Server (NS) records at your domain registrar to point to the Azure Name Servers.

---

This is the practical implementation of the [[Azure DNS Zones and Domain Delegation|domain delegation concepts]] discussed previously, and it's a critical prerequisite for advanced Ingress patterns like hostname-based routing.

## 🛠️ Step 1: Create the DNS Zone in Azure

First, we need to create a DNS Zone in Azure to host the DNS records for our domain.

1.  **Navigate to DNS Zones:**
    -   In the Azure Portal, search for and select **DNS zones**.
2.  **Start Creation:**
    -   Click the **Create** or **Add** button.
3.  **Configure the DNS Zone:**
    -   **`Subscription`:** Select your subscription.
    -   **`Resource group`:** You can use an existing one or create a new one. The instructor creates a dedicated resource group named `dns-zones` to keep DNS resources separate.
    -   **`Name`:** Enter the **exact domain name** that you own at your external registrar (e.g., `kubeoncloud.com`).
    -   **`This zone is a child of an existing zone...`**: Leave this unchecked.
4.  **Review and Create:**
    -   Click **Review + create**.
    -   Click **Create**. The DNS Zone will be provisioned very quickly.

## 📝 Step 2: Make a Note of the Azure Name Servers

Once the DNS Zone is created, Azure will assign a set of authoritative name servers to it. We need these to update our registrar.

1.  **Open the DNS Zone:** Navigate to the DNS Zone you just created (e.g., `kubeoncloud.com`).
2.  **Find the Name Servers:** On the **Overview** page, you will see a list of four **Name server** hostnames.
    -   **Example:**
        -   `ns1-01.azure-dns.com.`
        -   `ns2-01.azure-dns.net.`
        -   `ns3-01.azure-dns.org.`
        -   `ns4-01.azure-dns.info.`
3.  **Copy these four values.** You will need them for the next step.

---

## 🔄 Step 3: Update the Name Servers at Your Domain Registrar

Now, you need to log in to your domain registrar's management console and replace the existing Name Server (NS) records with the ones you copied from Azure. The instructor uses AWS Route 53, but the process is conceptually the same for any registrar (GoDaddy, Namecheap, etc.).

1.  **Log in to your Registrar:** Go to your domain registrar's website (e.g., AWS Route 53).
2.  **Find Your Domain:** Navigate to the management page for your registered domain (e.g., `kubeoncloud.com`).
3.  **Edit the Name Servers:** Find the section for managing the domain's name servers.
4.  **Replace the Values:** Delete the existing registrar-provided name servers and paste the four Azure name servers you copied in the previous step.
5.  **Save the Changes.**

> [!warning] DNS Propagation Time
> After you save the changes, it can take some time for the DNS update to propagate across the internet. This can range from **15-30 minutes to as long as 24-48 hours**, though it's typically on the shorter end. During this time, DNS queries for your domain might resolve to either the old or new name servers.

---

## ✅ Step 4: Verify the Delegation

Once you've waited for the DNS changes to propagate, you can verify that the delegation was successful using a command-line tool like `nslookup`.

1.  **Open a terminal.**
2.  **Run the `nslookup` command** to check the NS records for your domain.
    ```bash
    nslookup -type=NS kubeoncloud.com
    ```

**Expected Output:**
The output should now list the **Azure DNS name servers** as the authoritative name servers for your domain, confirming that the delegation is complete.
```
Non-authoritative answer:
kubeoncloud.com	nameserver = ns1-01.azure-dns.com.
kubeoncloud.com	nameserver = ns2-01.azure-dns.net.
...
```

---

> [!summary] Conclusion
> You have successfully created an Azure DNS Zone and delegated your domain from an external registrar to be managed by Azure. Azure DNS is now the authoritative source for your domain. This completes the necessary prerequisite setup.
>
> In the next section, we will leverage this by implementing Ingress hostname-based routing, where we will create `A` records in our new Azure DNS Zone to point subdomains (like `app1.kubeoncloud.com`) to our Ingress Controller's static public IP.