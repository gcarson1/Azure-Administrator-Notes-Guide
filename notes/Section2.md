# Virtual Machine
## Can be placed on a virtaul network, arranged in "availabilty sets", placed behind "load balancers'


# Learn
## Microservices
### Service Fabric
* Azure Service Fabric is a Distributed Systems platform designed to simplify the packaging, deployment, and management of scalable and reliable microservices and containers. It focuses on building stateful services, which maintain their state across sessions, unlike stateless services that do not retain information between requests.
### Azure Functions
* Serverless option code execution unit service. Azure functions allow you to use less code and less infrastructure at lower costs. Uses even-driven triggers connected to other services. C#, Java, JavaScript, PowerShell, Python.
### Azure Logic Apps
* No/Low code cloud platform for workflows dealing with other infrastructure.
### API Management
* Frontend platform for managing security, running analysis, and monitoring your APIs. Also allows the creation, change, and removal of APIs.
### Azure Container Services
* Also known as Azure Kubernetes Service, orchestrates and manages everything for your hosted containers. Scalability, Load balancing, server placement, etc.

## Networking - Connectivity
### Virtual Network (VNet)
* What it is: Your slice of Azure’s fabric—an isolated Layer-3 network where you drop subnets, NICs, and security controls. Think of it as the cloud equivalent of a data-center LAN.
* Why you’d use it: Every IaaS or PaaS resource that needs a private IP lives here. You can bolt on Network Security Groups (firewalls), route tables, load balancers, Private Link endpoints—the works.
Watch-outs: Address-range mistakes are forever (no shrinking a CIDR block). Over-subnetting explodes route tables; under-subnetting forces painful re-addressing later. VNet alone doesn’t give you inter-region transit—see Peering or Virtual WAN for that.
### Virtual WAN
* What it is: Microsoft’s managed SD-WAN backbone: a global “hub-and-spoke-as-a-service.” You create one or more vWAN hubs; Azure stuffs high-throughput VPN, ExpressRoute, and third-party SD-WAN appliances inside, plus “any-to-any” routing between hubs, VNets, and branches.
* Why you’d use it: You want a single pane to onboard dozens/hundreds of sites, remote users (P2S), and VNets—without DIY route tables. New in April 2025: Route-maps (now GA) let you finally filter/modify route adverts instead of accepting Azure’s one-size-fits-all propagation. 
* Watch-outs: Priced per hub + data transfer. Force-multiplying transits can burn cash fast. Still not full-mesh magic—latency follows the hub layout you pick.
### ExpressRoute
* What it is: A private, dedicated circuit (layer-3 BGP) from your colo or on-premises edge into Microsoft’s backbone. Bypasses the public Internet entirely.
* Why you’d use it: Predictable latency for large data transfers, regulatory pressure against Internet tunnels, or you simply hate IPSec overhead. Metro peering and Global Reach just expanded again in 2025, widening the list of cities and letting two on-prem sites talk through Microsoft’s core. 
* Watch-outs: Carrier contracts, cross-connect fees, and NNI upgrades add hidden cost. Disaster-proof setups need two circuits in separate facilities; skimp and you’ll learn this lesson the hard way.
### VPN Gateway
* What it is: An Azure-managed IPSec/IKE VPN head-end that sits in a dedicated subnet of a VNet. Handles Site-to-Site, Point-to-Site, and VNet-to-VNet tunnels.
* Why you’d use it: Cheap, quick connectivity when “good-enough over the Internet” beats ExpressRoute economics. Also works as crypto-failover if your ExpressRoute circuit drops.
* Watch-outs: Throughput is bound to SKU (Basic to Ultra) and single-core encryption limits; don’t assume wire-speed. Every Gateway adds ~30 minutes of deploy time—plan ahead.
### Azure DNS
* What it is: Authoritative DNS hosting in Azure, covering public zones, Private DNS zones for internal names, and a managed DNS Firewall layer. March 2025 brought Private DNS Internet Fallback—automatic failover to public resolvers if the private resolver goes down. 
* Why you’d use it: Low-latency, SLA-backed name service that plugs straight into Resource Manager APIs; no bind servers to babysit.
* Watch-outs: It’s DNS-only—no DNS-based Global-Traffic-Manager tricks (that’s Traffic Manager/Front Door). Charges per zone + millions of queries; “set-and-forget” can still surprise you on a high-traffic domain.
### Peering
Two flavors, two purposes:
* VNet Peering – Private, high-bandwidth link between VNets (same or different regions). Acts like a giant switch port: no gateways, negligible latency, separate billing meter.
* Azure Peering Service – A public peering program with ISPs/IXPs so your Office 365 or public-IP traffic hops onto Microsoft’s backbone at the closest edge PoP, improving jitter and route predictability. 

Why you’d use them:
* VNet Peering: micro-segmentation without routing headaches.
* Peering Service: better SaaS performance from on-prem without buying ExpressRoute.
Watch-outs:
* VNet Peering is not transitive—A-B and B-C doesn’t mean A-C (use hub-and-spoke or vWAN).
* Peering Service still rides the ISP’s last-mile; congestion there is your problem, not Microsoft’s.

## Networking - Security
### Network Security Groups (NSG)
### Azure Private Link
### DDoS Protection
### Azure Firewall
### Web Application Firewall (WAF)
### Virtual Network Endpoints
### Bastion

## Networking - Delivery
### CDN
### Azure Front Door
### Traffic Manager
### Application Gateway
### Load Balancer

## Networking - Monitoring
### Network Watcher
### Metrics and Logs
### Packet Capture

