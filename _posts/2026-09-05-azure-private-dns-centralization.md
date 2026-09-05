---
title: "Azure Private DNS Architecture in a Hub-Spoke Network Architecture"
date: 2026-09-05
categories:
  - Azure
  - Networking

tags:
  - Azure DNS
  - Private Endpoint
  - Hub Spoke
  - Hybrid Networking
  - Azure Private Resolver

author: "Sibi Lawrence"
---
When organizations adopt Azure Private Endpoints for services such as Azure Storage Accounts, Azure SQL Databases, Cosmos DB, Key Vaults, and other PaaS services, networking becomes a critical aspect of the overall solution design.

A well-architected Azure landing zone typically consists of multiple subscriptions connected through a Hub-Spoke network topology. Each subscription may contain one or more Virtual Networks (VNets) hosting application workloads, while the central Hub subscription provides shared infrastructure services such as connectivity, security, DNS, and monitoring.

While much attention is often given to Private Endpoints themselves, DNS resolution is equally important. Without proper DNS design, applications may continue resolving Azure services through public endpoints even after Private Endpoints have been deployed.

This article explores how centralized Azure Private DNS and DNS forwarding can simplify enterprise-scale deployments while providing a consistent and secure name resolution experience.

**The Role of Private Endpoints**

A Private Endpoint assigns a private IP address from your VNet to an Azure PaaS service.

For example, a Storage Account that is normally accessed using:

"storageaccount1.blob.windows.net" can be connected to a VNet through a Private Endpoint and assigned an address such as:

10.10.10.1

Applications should continue using the same Azure service name, but the DNS resolution should return the private IP rather than the public endpoint address.

The success of this model depends entirely on proper DNS resolution.

**1. What Does Azure Private DNS Do?**

Azure automatically creates and manages DNS records within Azure Private DNS Zones that correspond to Private Endpoints.

For example:

storageaccount1.blob.windows.net may ultimately resolve through the Azure Private DNS Zone to: 10.10.10.1

The application remains unaware of the underlying networking changes.

This provides several benefits:

  - No application changes required
  - Consistent service names across environments
  - Transparent use of private connectivity
  - Reduced exposure to public endpoints

**2. Why Centralize Private DNS?**

In large enterprise environments, organizations commonly have multiple subscriptions:

**_Hub Subscription:_**

Used for shared infrastructure:

  - Central Networking
  - Azure Firewall
  - ExpressRoute / VPN
  - Azure DNS Private Resolver
  - Private DNS Zones

**_Spoke Subscriptions_**

Used for workloads:

  - Development
  - Test
  - UAT
  - Production

A common mistake is creating duplicate Private DNS Zones in every subscription.

For example: privatelink.blob.core.windows.net

may exist separately in:

  - Dev Subscription
  - Test Subscription
  - Prod Subscription

While functional, this approach increases operational overhead and can lead to inconsistent DNS configurations.

Instead, organizations can centralize Private DNS Zones within the Hub subscription and link all Spoke VNets to those zones.

Benefits include:

  - Centralized management
  - Consistent DNS records
  - Easier troubleshooting
  - Simplified governance
  - Reduced administrative overhead
  - Standardized architecture across environments

**_3. Understanding Virtual Network Links_**

Creating a Private DNS Zone alone is not sufficient.

The VNets that host applications must be linked to the Private DNS Zone.

Azure uses Virtual Network Links to establish this relationship.

For example: 

Private DNS Zone
privatelink.blob.core.windows.net

can be linked to:

  - Hub VNet
  - Dev VNet
  - Test VNet
  - Prod VNet

Once linked, resources within those VNets can resolve records hosted in the Private DNS Zone.

Without the VNet Link, DNS lookups will fail even though the DNS record exists.

This is one of the most common causes of Private Endpoint connectivity issues.

**4. What About On-Premises Environments?**

Things become slightly more complex when applications run on-premises.

Consider an on-premises application attempting to access: storageaccount1.blob.windows.net

The on-premises DNS server has no awareness of Azure Private DNS Zones.

Therefore, additional DNS forwarding is required.

This is where Azure DNS Private Resolver becomes an important component of the architecture.

**5. Azure DNS Private Resolver**

Azure DNS Private Resolver is a managed DNS forwarding service that allows DNS queries to be exchanged between Azure and external DNS systems without deploying custom DNS virtual machines.

The service consists of:

Inbound Endpoint

Receives DNS queries coming into Azure.

For example:

On-Prem DNS
     |
     v
Inbound Endpoint

**_Outbound Endpoint_**

Sends DNS queries from Azure to external DNS servers.

For example: 

                                                    Azure Workload
                                                         |
                                                         v
                                                    Outbound Endpoint
                                                         |
                                                         v
                                                    On-Prem DNS

This enables bidirectional name resolution across hybrid environments.

**6. DNS Forwarding Flow from On-Premises to Azure**

A common configuration is to create conditional forwarders on on-premises DNS servers.

For example: privatelink.blob.core.windows.net or blob.core.windows.net

can be forwarded to Azure.

The resolution path becomes:

                                                  On-Prem Client
                                                        |
                                                        v
                                                  On-Prem DNS Server
                                                        |
                                                  Conditional Forwarder
                                                        |
                                                        v
                                                  Azure DNS Private Resolver
                                                  (Inbound Endpoint)
                                                        |
                                                        v
                                                  Azure Private DNS Zone
                                                        |
                                                        v
                                                  Private Endpoint Record
                                                        |
                                                        v
                                                  10.10.10.1

The client receives the private IP address and traffic remains entirely on private connectivity.

**7. What Happens Without Conditional Forwarding?**

Without conditional forwarding, on-premises DNS servers do not know where Azure private namespaces exist.

The request follows the standard public DNS hierarchy.

The result may be: storageaccount1.blob.windows.net resolving to: Public Endpoint instead of: Private Endpoint

This can create:

Connectivity failures
Security concerns
Unexpected routing paths
Inconsistent application behavior

Therefore, DNS forwarding is a fundamental requirement of any Azure hybrid architecture utilizing Private Endpoints.

**8. Recommended Enterprise Architecture**

A scalable enterprise architecture typically includes:

**_Hub Subscription_**

Azure DNS Private Resolver
Central Private DNS Zones
Shared Networking Services

**_Spoke Subscriptions_**

Application VNets
Private Endpoints
Workload Resources

**_Connectivity Layer_**

ExpressRoute or VPN
DNS Forwarding Rules
Conditional Forwarders

**_DNS Design_**

Single set of centrally managed Private DNS Zones
VNet Links for all Spoke VNets
Centralized governance and monitoring

This approaches DNS as a shared platform service rather than an application-specific component.

**Key Takeaway**

In a multi-subscription Hub-Spoke environment, DNS should be treated as a foundational infrastructure service rather than an individual application responsibility. By centralizing Azure Private DNS Zones in the Hub, linking Spoke VNets through Virtual Network Links, and leveraging Azure DNS Private Resolver for hybrid name resolution, organizations can build a scalable, secure, and operationally efficient architecture. A well-designed DNS strategy not only simplifies management and troubleshooting, but also ensures that Private Endpoints consistently deliver the secure private connectivity they were intended to provide.





Author: Sibi Lawrence
Topics: Azure Networking, Private Endpoints, Azure DNS, Hub-Spoke Architecture, Hybrid Connectivity, Cloud Architecture, Azure Infrastructure


