# Design and implement core networking infrastructure

## Virtual networks in Azure

- A VNet must exist within a single subscription and within a single region
- A resource can only be created in a virtual network that exists in the same region and subscription as the resource. You can, however, connect virtual networks that exist in different subscriptions and regions.
- You can deploy as many virtual networks as you need within each subscription, up to the subscription limit
- All resources in a VNet can communicate outbound to the internet by default
- **Layer 3**
  - It understands IP addresses
  - TCP
  - UDP
  - Cannot do broadcast or multicast
- When creating a VNet, it’s recommended that you use the address ranges enumerated in **RFC 1918** which have been set aside for private, non-routable address spaces
  - 10.0.0.0 - 10.255.255.255 (10/8 prefix)
  - 172.16.0.0 - 172.31.255.255 (172.16/12 prefix)
  - 192.168.0.0 - 192.168.255.255 (192.168/16 prefix)
- The following address ranges cannot be used:
  - 224.0.0.0/4 (Multicast)
  - 255.255.255.255/32 (Broadcast)
  - 127.0.0.0/8 (Loopback)
  - 169.254.0.0/16 (Link-local)
  - 168.63.129.16/32 (Internal DNS)
- **Azure reserves the first 4 and last IP addresses within each subnet.**
- For example, the IP address range of 192.168.1.0/24 has the following reserved addresses:
  - 192.168.1.0: Network address
  - 192.168.1.1: Reserved by Azure for the default gateway
  - 192.168.1.2, 192.168.1.3: Reserved by Azure to map the DNS IPs to the VNet space
  - 192.168.1.255: Network broadcast address
- When planning to implement virtual networks, you need to consider the following:
  - Ensure non-overlapping address spaces. Ensure your VNet address space (CIDR block) does not overlap with your organization's other network ranges.
  - Is any security isolation required?
  - Do you need to mitigate any IP addressing limitations?
  - Will there be connections between Azure VNets and on-premises networks?
  - Is there any isolation required for administrative purposes?
  - Are you using any Azure services that create their own VNets?
- **Service tags**
  - a group of IP address prefixes from a given Azure service
  - Microsoft manages these exclusively and publishes them weekly
  - Used for on-premises connections and NSGs
  - Read-only

## Virtual network peering

- Virtual network peering enables you to seamlessly connect two or more virtual networks in Azure
- The traffic between virtual machines in peered virtual networks uses the Microsoft backbone infrastructure
- Two types of peering
  - **Virtual network peering**: connects virtual networks within the same Azure region
  - **Global virtual network peering**: connects virtual networks across Azure regions
- Virtual network peering **is NOT transitive** by default. You have to build the routing paths yourself. In the image below, VNet A wouldn’t be able to communicate with VNet B by default.

![Untitled](Design%20and%20implement%20core%20networking%20infrastructur%204ccd5ceea8054b3eb96c5dfc0b6d4fe8/Untitled.png)

## Private endpoints

- A private endpoint is a network interface associated with a specific Azure data resource that uses a **private IP address** from your virtual network
- This network interface connects you privately and securely to a service that’s powered by Azure Private Link
- The resource can be accessed over peered networks and hybrid cloud connections
- It offers the same solution as the service endpoint, but whereas with service endpoints the resources are still accessible over the internet, with private endpoints the resource aren’t available over the internet

![Untitled](Design%20and%20implement%20core%20networking%20infrastructur%204ccd5ceea8054b3eb96c5dfc0b6d4fe8/Untitled%201.png)

## Subnets

- A subnet is a range of IP addresses in the VNet.
- You can segment VNets into different size subnets, creating as many subnets as you require for organization and security purposes within the subscription limit.
- Like in a traditional network, subnets allow you to segment your VNet address space into segments that are appropriate for the organization's internal network. This also improves address allocation efficiency.
- The smallest supported IPv4 subnet is /29, and the largest is /2 (using CIDR subnet definitions). IPv6 subnets must be exactly /64 in size.
- Each subnet must have a unique address range, specified in CIDR
- Certain Azure services require their own subnet
  - **Virtual network gateways** require a dedicated subnet specifically named *GatewaySubnet*. Its size must be at least a /29 in CIDR notation. It is recommended, however, that a larger address space be assigned, such as /27.
  - **Azure Firewall** requires a subnet specifically named *AzureFirewallSubnet*, with a recommended size of /26.
  - **Azure Application Gateway** requires a dedicated subnet within the virtual network, but it doesn’t need to have a specific name. The subnet must be empty, and not contain any Azure resources. The recommended size of the subnet will depend on the SKU chosen.
  - **Bastion** requires a dedicated subnet specifically named *AzureBastionSubnet*, with a prefix of at least /27.
  - **Azure Route Server** is a service that simplifies the routing for hybrid connectivity and requires a dedicated subnet with a specific name of *RouteServerSubnet* and a minimum size of /27.

## Name resolution

### Public DNS zones

- Azure DNS is a PaaS service that provides an authoritative DNS service for a domain name in your environment.
- The domain name should be a public domain that you can configure the name servers.
- DNS domains in Azure DNS are hosted on Azure’s global network of name servers.
- A DNS Zone hosts the DNS records for a domain.
- To delegate your domain to Azure DNS, you first need to know the name server names for your zone. Each time a DNS zone is created, Azure DNS allocates name servers from a pool. Once the Name Servers are assigned, Azure DNS automatically creates authoritative NS records in your zone.

<aside>
💡 Azure isn’t the domain registrar. You are just delegating your public zone for Azure to manage it.
</aside>

### Private DNS services

- Private DNS services resolve names and IP addresses for resources and services available from our own internal networks.
- The records contained in a private DNS zone aren’t resolvable from the internet. DNS resolution against a private DNS zone works only from virtual networks that are linked to it.
- You can link a private DNS zone to one or more virtual networks by creating virtual network links
- You can enable the **autoregistration** feature to automatically manage the life cycle of the DNS records for the virtual machines that get deployed in a virtual network.
