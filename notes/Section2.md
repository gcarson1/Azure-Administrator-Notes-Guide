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
* What it is – Edge caching for static or “mostly-static” objects (images, scripts, video segments, large downloads). Azure runs several flavors (Standard “Microsoft” POPs, plus Akamai & Verizon). The legacy Edgio tier is being shut down 15 Jan 2025—migrate now or your endpoints go dark. 
* When to use it – You need lower latency on global reads, thinner egress bills, or a shield in front of an origin that can’t take a Reddit hug-of-death.
* Gotchas –
- Purely HTTP(S) cache; no smart load-balancing logic.
- Refresh is TTL-driven—purge APIs exist, but batch purges can lag minutes.
- Still bills per-GB egress from POP to user and from origin to POP; big files can surprise you.
### Azure Front Door
* What it is – Microsoft’s next-gen global HTTP(S) edge: combines anycast routing, SSL offload, path-based routing, built-in WAF, and tiered caching (Standard vs Premium). Think “CDN + L7 load balancer + firewall, managed as code.” Upgrade/downgrade between tiers is now point-and-click (released H2 2024). 
* When to use it –
- Multi-region active-active apps that need instant failover.
- You want a single TLS cert + vanity domain at the edge.
- Central place for rate-limiting, geo-blocking, bot logic.
* Gotchas –
- Only speaks HTTP(S); for TCP/UDP you still need Load Balancer.
- Session affinity (cookie-based) lives at the edge—not in your VNet—test sticky workflows.
- WAF false positives are still a thing; stage in “Detection” before flipping to “Prevention.”
### Traffic Manager
* What it is – DNS-level load balancer. It hands out different IP/hostname answers based on health probes, geography, latency, or weighted splits. No inline data path—once DNS resolves, the client hits the endpoint directly. 
* When to use it –
- Cross-cloud or hybrid failover (it’ll route to any reachable public endpoint, not just Azure).
- Lightweight A/B or blue-green cuts without juggling certificates.
* Gotchas –
- Failover speed = DNS TTL (minimum 0-10 sec, but most apps cache 20-30 sec).
- Zero TLS offload, zero WAF; it’s steering only.
- Health probes run from inside Azure regions—if the public Internet between a region and users dies, TM won’t see it.
### Application Gateway
* What it is – Regional Layer-7 load balancer that lives inside your VNet. Gives you path-based routing, cookie/session affinity, URL & header rewrite, autoscaling, zone-redundancy, and an optional integrated WAF. 
* When to use it –
- You need policy decisions on private IP back-ends (AKS, VMs) that never see the public edge.
- Internal microservice meshes that want TLS end-to-end but still need header rewrites.
* Gotchas –
- Single-region scope—global failover demands Front Door or Traffic Manager on top.
- Cold-start spin-up is minutes, not seconds—plan min-instance autoscale for spikes.
- Throughput is fine for web traffic but not a drop-in for raw TCP.
### Load Balancer
* What it is – Layer-4 (TCP/UDP) distribution for any protocol: public IP (internet-facing) or internal. Supports cross-zone redundancy, outbound SNAT, and HA Ports mode (one rule for all ports). Latest builds add better diagnostics and per-rule metrics. 
* When to use it –
- Classic three-tier apps: VIP on port 443, spray to web VMs.
- Anything non-HTTP(S) (MQTT, gRPC over TCP, game servers).
- Outbound SNAT for VMs that don’t need inbound public IPs.
* Gotchas –
- No TLS offload, no rules above L4—pair with NGINX/Envoy/App GW if you need them.
- Standard SKU is zonal & secure by default but charges for inbound+outbound data; Basic is free data but lacks SLA and diagnostics.
- State-sync traffic isn’t masked; design for sticky flows if your protocol hates re-ordering.
## Networking - Monitoring
### Network Watcher
* What it is – The umbrella service that hosts every network-troubleshooting gadget Azure gives you: topology maps, connection monitor, next-hop, IP flow verify, packet capture, flow logs, and more. It’s enabled per region; if you spin up a VNet in “East US” without a watcher, some diagnostics simply won’t exist. 
* Why you use it
- One click (or ARM/Bicep) to turn on diagnostics for all VNets in the region.
- Central spot for Connection Monitor, which now supports hybrid endpoints and custom alert rules (Build 2025 update). 
- Houses the agent/extension needed for on-demand packet capture and advanced tests (latest agent version 1.4.3614.3; keep it current or captures fail). 
* Gotchas
- Regional scope—forget to enable it in one region and you’ll be blind there.
- Generates lots of logs; pipe them to a Log Analytics workspace with a retention policy or eat storage bills later.
### Metrics and Logs
* What they are –
- Metrics = real-time numeric counters (pps, Bps, dropped packets) stored in the high-performance Azure Monitor metrics database.
- Logs = time-series JSON groks stored in a Log Analytics workspace—flow logs, connection_monitor events, NSG/VNet flow logs, etc.
* Why you use them
- Alerting: fire on thresholds (e.g., >80 % LB SNAT ports used) or log queries (e.g., sudden spike of denied flows).
- Traffic Analytics: turnkey workbook that digests flow logs into top-talkers, geos, and abnormal patterns.
- VNet Flow Logs: the NSG flow-log replacement GA since 2024, broader visibility and will be required after 30 June 2025 when new NSG logs are blocked. 
* Gotchas
- Metrics keep only 93 days (unless you manually archive); Logs default to 30 days—set retention explicitly.
- Log ingestion is billed per-GB plus per-GB retention after the free first month—large flow-log volumes can dwarf VM costs.
- DNS & Firewall logs are still in separate schemas—queries get messy fast.
### Packet Capture
* What it is – Remote, “agent-assisted tcpdump.” You set filters (IP, port, bytes), start capture from the portal/CLI/REST, data is streamed to a storage account or temp disk. Works on Windows or Linux VMs that have the Network Watcher extension. 
* Why you use it
- Debug he-said/she-said latency issues without SSH/RDP onto the box.
- Verify TLS handshakes or weird MTU drops while traffic is live.
* Gotchas
- Not real-time—there’s a lag between “Start” click and actual capture start while the extension spins up.
- Captures stop at 5 GB or 180 minutes by default; long-running traces need explicit size/time bumps.
- Eats VM CPU; on busy boxes you will feel it—run during off-peak or capture on a clone.
- Still VM-only. Want switch-level sniffing? You’re out of luck.
