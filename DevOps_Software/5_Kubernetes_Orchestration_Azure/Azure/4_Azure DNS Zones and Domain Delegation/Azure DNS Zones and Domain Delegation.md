#Cloud #Azure #Kubernetes #AKS #Networking #DNS #AzureDNS #DomainDelegation #CoreConcept

>  To use custom, user-friendly hostnames for applications running in [[Azure Kubernetes Service (AKS)|AKS]] (e.g., `app1.mydomain.com`), you need to manage your domain's DNS records in Azure. Since Azure is not a domain registrar, you must first register a domain with a third-party registrar (like GoDaddy or AWS Route 53) and then **delegate** that domain to **Azure DNS**. Delegation involves changing the domain's Name Server (NS) records at the registrar to point to Azure's name servers.

---

This is a foundational step required for implementing advanced [[An Introduction to Kubernetes Ingress|Ingress]] patterns like hostname-based routing and automatic SSL.

## 🏛️ Core DNS Concepts

### 1. Domain Registrar
-   **What it is:** A company that manages the reservation and registration of internet domain names (e.g., `mydomain.com`).
-   **Examples:** GoDaddy, Namecheap, Wix, and cloud providers like AWS Route 53.
-   **Function:** They verify if a domain name is available and allow you to purchase it, making you the legal owner.
-   **Azure's Role:** Azure DNS is **not** a domain registrar (with the limited exception of Azure App Services). You cannot buy a new domain directly from Azure.

### 2. DNS Zone
-   **What it is:** A DNS Zone is used to host the DNS records for a particular domain. It's the authoritative source of information for that domain.
-   **Example:** For the domain `stacksimplify.com`, the DNS Zone would contain various DNS records, such as:
    -   `mail.stacksimplify.com` (an `MX` record for a mail server).
    -   `www.stacksimplify.com` (an `A` record for a website).
    -   `courses.stacksimplify.com` (an `A` or `CNAME` record for a subdomain).

### 3. Azure DNS Zones
-   **What it is:** Azure DNS is a hosting service for DNS zones, providing name resolution using Microsoft Azure infrastructure.
-   **Function:** It allows you to host a DNS zone and manage the DNS records for your domain within Azure.

---

## 🔄 The Concept of Domain Delegation

Because Azure is not a registrar, you must own a domain from an external registrar first. **Domain Delegation** is the process of telling your registrar to hand over the authority for managing your domain's DNS records to Azure DNS.

> [!info] The Essence of Delegation
> Delegating a domain is simply the process of **changing the Name Server (NS) records** for your domain at your registrar. You will replace the registrar's default name servers (e.g., GoDaddy's name servers) with the name servers provided by your Azure DNS Zone.

**The Workflow:**
1.  **Own a Domain:** You have a domain registered, for example, `kubeoncloud.com` at AWS Route 53.
2.  **Create an Azure DNS Zone:** In Azure, you create a new DNS Zone resource for `kubeoncloud.com`.
3.  **Get Azure's Name Servers:** Azure will assign a set of unique name servers to your new DNS Zone (e.g., `ns1-01.azure-dns.com`, `ns2-01.azure-dns.net`, etc.).
4.  **Update NS Records at Registrar:** You log in to your registrar (AWS Route 53) and update the NS records for `kubeoncloud.com` to point to the Azure name servers you just received.
5.  **Delegation Complete:** After the DNS changes propagate (which can take some time), Azure DNS becomes the authoritative source for all DNS queries for `kubeoncloud.com`. You will now manage all your DNS records (A, CNAME, etc.) within the Azure DNS Zone.

### Verifying Delegation
You can use a command-line tool like `nslookup` to check which name servers are authoritative for your domain.
```bash
nslookup -type=NS kubeoncloud.com
```
Before delegation, this would show the registrar's name servers (e.g., AWS). After successful delegation, it will show the Azure DNS name servers.

---

> [!summary] Why are we doing this?
> This entire process is a prerequisite for our advanced Ingress sections:
> -   **Hostname-Based Routing:** We need a valid domain to create hostnames like `app1.kubeoncloud.com`.
> -   **External DNS:** The `external-dns` controller will need permissions to automatically create `A` records in our Azure DNS Zone.
> -   **Ingress with SSL (`cert-manager`):** The `cert-manager` tool will need to interact with our Azure DNS Zone to solve DNS-01 challenges to prove domain ownership and get a valid SSL certificate from Let's Encrypt.
>
> In the next lecture, we will perform the hands-on steps to create the Azure DNS Zone and delegate the domain.