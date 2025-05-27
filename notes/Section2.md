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
* What it is: Stateful ACLs for VNets and subnets/VM NICs. You write L4 rules (source/dest IP/CIDR, port, proto) to allow or deny traffic. Azure handles the connection tracking. 
* Sweet spot: Basic micro-segmentation inside a VNet—east-west and north-south—without paying for an appliance.
* Gotchas:
- 1,000 rules max per NSG; every new rule burns CPU cycles on all hosts in that subnet.
- Rule changes only hit new flows; lingering sessions survive until they time out—easy to misread during testing.
- No TLS inspection, no FQDN filtering (that’s Firewall territory).
### Azure Private Link
* What it is: A private endpoint (NIC) for an Azure PaaS service, a partner SaaS, or your own service behind a Standard LB. Traffic stays on Microsoft’s backbone—never touches a public IP. 
* Sweet spot: Kill the “public-DNS + IP allow list” pattern; expose Storage, SQL, etc., only to selected VNets.
* Gotchas:
- One Private Endpoint ≠ one subnet gateway; each consumes an IP. Poor planning = subnet exhaustion.
- DNS is on you—if you don’t flip the zone to the PE’s private IP, clients still leak to the public address.
- No transitive routing: on-prem must hairpin through a VNet (or use Private Link Service + NVA).
### DDoS Protection
* What it is: Always-on volumetric attack scrubbing bolted to a VNet. “IP Protection” is the cheap per-public-IP tier; “Network Protection” covers the whole VNet and offers cost-credit SLA. 
* Sweet spot: When you expose public IPs (App GW, Firewall, AKS, etc.) and want Microsoft’s 100-Tbps pipes to soak the floods before they hit you.
* Gotchas:
- Only stops volumetric and protocol attacks at L3/L4; L7 stuff (slow-loris, bot swarms) still needs WAF.
- Enable on all inbound IPs or the attacker will just pivot to the unprotected address.
### Azure Firewall
* What it is: A cloud-native, fully stateful L3–L7 firewall with SNAT/DNAT, TLS decryption (Premium), threat-intel filtering, and now Autoscale + Flow Trace Logs GA as of early 2025. 
* Sweet spot: Central egress control, FQDN/URL filtering, forced-tunneling hub-and-spoke designs without DIY NVAs.
* Gotchas:
- Price = base instance + per-GB + TLS inspect tax. Surprise bills for chatty microservices.
- Latency is fine inside the same region but non-trivial if you hairpin traffic across regions.
- Limited feature parity with big-iron firewalls (no IDS/IPS signatures, limited app-ID catalog).
### Web Application Firewall (WAF)
* What it is: OWASP-rules engine bolted onto Application Gateway, Front Door, or API Mgmt. Central GUI for policies; v2 supports per-site rules and custom signatures. 
* Sweet spot: Block SQLi/XSS, geo-filter, or bot mitigation for HTTP(S) workloads without touching code.
* Gotchas:
- High false-positive rate if you just slam it to “Prevention” mode—tune or you’ll DOS yourself.
- It’s pattern-matching; no behavior/ML magic. Still need secure coding.
- Only sees HTTP(S). Anything else rides right past it.
### Virtual Network Endpoints
* What it is: Adds the VNet/subnet tag to the system route table of a public PaaS service, so traffic stays on the backbone and the service can trust that subnet’s identity. 
* Sweet spot: Quick “good enough” alternative to Private Link when you just want to lock Storage/SQL to certain VNets and aren’t worried about using public IPs.
* Gotchas:
- Still uses the service’s public IP—traffic just takes a private path. If your auditors hate “public IP” anything, use Private Link.
- Not available for every service, and not cross-region.
- No inbound capability; it only secures calls to PaaS, not from PaaS to you.
### Bastion
* What it is: Managed jump-box: HTML5 RDP/SSH over TLS directly through the portal/CLI—no public IPs on your VMs. New builds now default to SKU “Standard” with IP-based session recording and Kerberos support (rolled out March 2025). 
* Sweet spot: Ops teams that need break-glass access without scattering VPN creds or opening port 3389/22 to the world.
* Gotchas:
- Charged by hour and outbound data; leave it running in non-prod and the meter keeps spinning.
- No clipboard file transfer on Basic SKU; and GUI performance can lag over high-latency links.
- Doesn’t solve east-west movement; it’s just a safer door.

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

