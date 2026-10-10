# Windows Server Active Directory & DNS Resolution, Backup & Troubleshooting Lab

This repository documents the implementation, configuration, architectural flow, and troubleshooting of **Active Directory Domain Services (AD DS)**, **DNS Resolution Mechanisms**, advanced DNS features (Forward, Reverse, CNAME, and Backup DNS), and client domain-joining procedures within a virtualized enterprise environment.
<img width="1914" height="828" alt="Untitled" src="https://github.com/user-attachments/assets/74cacfa7-f480-4cba-886c-0da099de279d" />

---

## 🌐 Understanding DNS (Domain Name System)
The **Domain Name System (DNS)** functions as the "phonebook" of computer networks. Its primary responsibility is to translate human-readable domain names (e.g., `server1.xyz.domain`) into numerical IP addresses (e.g., `192.168.1.200`) required for network routing, and vice-versa.

---

## 🔄 The DNS Name Resolution Process (Step-by-Step Flow)
When a client or server attempts to resolve a domain name, the system queries various layers sequentially until it finds the matching record:

1. **Local DNS Client Cache:** The operating system first checks its local memory cache to see if the domain name has been recently resolved.
2. **Local Hosts File (`C:\Windows\System32\drivers\etc\hosts`):** If not found in the cache, the system checks the static local hosts file for manual overrides.
3. **DNS Query (Preferred DNS Server):** If local lookups fail, the query is dispatched to the configured Preferred DNS Server (e.g., the Domain Controller).
4. **Root Hints / Root Name Servers:** If the local DNS server is authoritative for the zone, it answers directly. Otherwise, it queries global Root Name Servers for top-level guidance.
5. **Top-Level Domain (TLD) Servers:** The root server directs the query to the specific TLD server (e.g., `.com`, `.net`, or local domain structure).
6. **Authoritative DNS Server:** Finally, the query reaches the designated Authoritative DNS Server holding the exact, verified record for the requested domain name, returning the corresponding IP address back to the client.

---

## ⚙️ Core DNS Query Types & Architectural Distinctions

### 1. Recursive Query 
* **Definition:** A query where the DNS client demands a definitive answer from the DNS server.
* **Mechanism:** The DNS server takes full responsibility for retrieving the final answer on behalf of the client—either from its cache/zones or by querying other servers—and returning the final IP or an error message.

### 2. Iterative / Non-Recursive Query 
* **Definition:** A query typically executed between DNS servers themselves.
* **Mechanism:** The queried server does not hunt down the final answer completely. Instead, it returns the *best possible referral* or pointer for the querying server to follow up on independently.

### 3. DNS Forwarders
* **Definition:** A configuration rule applied to a DNS server to route unresolved external queries to external trusted upstream DNS servers rather than processing them via root hints.

---

## 🛠️ Tools & Technologies Used 

To build, configure, and troubleshoot this enterprise lab environment, the following toolset and platforms were utilized:

* **Virtualization Platform:** VMware Workstation / ESXi for hosting virtual machine nodes.
* **Operating Systems:** 
  * Microsoft Windows Server 2019 / 2022 (For Primary and Additional Domain Controllers).
  * Windows 10 / 11 (For domain-joined client workstations).
* **Core Management Consoles:** 
  * **Server Manager:** For deploying roles and managing server services.
  * **DNS Manager:** For configuring zones, records, and zone transfers.
* **Diagnostic Utilities:** Command Prompt (`cmd`) utilizing network commands such as `ping` and configuration diagnostics to verify connectivity and replication status.

---

## 🛠️ Practical DNS Implementation in the Lab

Within the VMware virtual lab environment, the following DNS zones and record types were successfully deployed and verified:

* **Forward Lookup Zone:** Maps human-readable Fully Qualified Domain Names (FQDNs) to IPv4/IPv6 addresses[cite: 12].
* **Reverse Lookup Zone:** Performs PTR record mappings, translating an IP address back to its corresponding domain name for security and auditing[cite: 8].
* **CNAME (Canonical Name / Alias):** Creates an alternative alias name pointing to a canonical base domain name[cite: 4, 12].
* **Backup DNS (Secondary Zone on ADC):** Configured on the Additional Domain Controller to provide zone transfers, fault tolerance, and high availability across the directory infrastructure[cite: 4, 12].

---

## ⚠️ Troubleshooting & Challenges Faced 

During the deployment of this lab, specific technical hurdles were encountered and successfully resolved:

### 1. Client Domain-Join Issues 
* **The Problem:** When attempting to join client workstations (Windows 10/11) to the domain (`xyz.domain`), errors occurred regarding network path not found or inability to communicate with the Domain Controller.
* **Root Cause:** The client machine's preferred DNS server was not pointing to the Domain Controller's IP address, resulting in failed SRV record lookups required for domain discovery.
* **The Solution:** 
  1. Updated the client's network adapter IPv4 properties to point the **Preferred DNS Server** directly to the DC's IP (`192.168.1.200`)[cite: 10].
  2. Verified bidirectional connectivity using `ping`[cite: 6].
  3. Re-attempted the domain join using valid administrative credentials, which successfully authenticated and bound the machine to the domain[cite: 7].

### 2. DNS Backup / Secondary Zone Transfer Errors
* **The Problem:** While setting up the Secondary DNS Zone on the Additional Domain Controller (ADC), an error stating *"Zone Not Loaded by DNS Server"* or zone transfer failure appeared[cite: 12].
* **Root Cause:** Zone transfer security permissions on the Primary DNS server restricted unauthorized transfers, or the ADC IP was not explicitly listed in the allowed master servers list, blocking synchronization.
* **The Solution:**
  1. Configured the Zone Transfer settings on the Primary DNS server under the zone properties to allow transfers **"To the following servers"** or **"Only to server listed on the Name Servers tab"** and explicitly added the ADC's IP address[cite: 13].
  2. Verified that the Name Servers (NS) tab on both servers correctly listed mutual entries[cite: 13].
  3. Refreshed and reloaded the secondary zone from the master server via the DNS Manager console, successfully resolving the synchronization error[cite: 12, 13].

---

## 🎯 Conclusion & Summary

This lab successfully demonstrates the robust deployment of an enterprise-grade Active Directory and DNS infrastructure. By implementing primary and backup (secondary) DNS servers, setting up precise forward and reverse lookup zones, leveraging alias records (CNAME), and systematically resolving domain-join and zone-transfer challenges, the architecture ensures high availability, fault tolerance, and seamless name resolution across all domain resources. 

---
*Author: Abdalla Mohamed Nieam (Network & Security Engineer)*
