# AWS VPC – Complete Interview Question Bank
**Target Audience:** 3–4+ years experience DevOps / Cloud / Network / SRE engineers  
**Scope:** All VPC modules + Real-time + Scenario-based + Troubleshooting  
**Source mix:** MNC interview patterns (TCS, Infosys, Wipro, Cognizant, Accenture, Deloitte, Capgemini, Amazon, Microsoft, IBM, HCL, LTIM, Mphasis, Salesforce, Oracle) + LinkedIn shared questions + real production network incidents.  
**Note:** No duplication with EC2 question bank.

---

## TABLE OF CONTENTS
1. VPC Fundamentals
2. CIDR & IP Addressing
3. Subnets (Public / Private / Isolated)
4. Route Tables
5. Internet Gateway (IGW)
6. NAT Gateway & NAT Instance
7. Egress-Only Internet Gateway (IPv6)
8. Security Groups vs NACL (VPC context)
9. VPC Peering
10. Transit Gateway (TGW)
11. Site-to-Site VPN
12. Client VPN
13. AWS Direct Connect
14. VPC Endpoints (Gateway / Interface)
15. AWS PrivateLink
16. VPC Flow Logs
17. DNS in VPC & Route 53 Resolver
18. DHCP Options Set
19. ENI Deep Dive (VPC context)
20. Secondary CIDR, IPv6 & IPAM
21. AWS Network Firewall
22. Traffic Mirroring
23. Reachability Analyzer & Network Access Analyzer
24. Shared VPC (AWS RAM)
25. VPC Lattice
26. Advanced: Carrier GW, Local Zones, Wavelength, Outposts
27. Scenario-Based Questions
28. Troubleshooting Questions
29. Real-time Production Issues (LinkedIn style)
30. DevOps / IaC Focused VPC Questions

---

## 1. VPC FUNDAMENTALS

**Q1. What is a VPC and what is it not?**  
Logically isolated virtual network in AWS where you launch resources. It is **not** a physical network — it's an SDN overlay on AWS's backbone. One VPC = one region, spans all AZs in that region.

**Q2. Default VPC vs Custom VPC — key differences:**  
| Aspect | Default | Custom |
|---|---|---|
| Created by | AWS per region | You |
| CIDR | `172.31.0.0/16` | Any RFC1918 (`/16`–`/28`) |
| Subnets | One public per AZ | You define |
| IGW | Attached | Must attach |
| Public IP auto-assign | Yes | Off by default |
| Default NACL/SG | Permissive | Same, but you should tighten |

**Q3. Can you have multiple VPCs in one region? Across regions?**  
Yes — default soft limit 5 per region (quota raise available). Each region is fully independent; inter-region requires peering, TGW, or VPN.

**Q4. What resources live at VPC level vs subnet level?**  
- **VPC**: CIDR, DHCP options, DNS resolution setting, IGW attachment, VPC endpoints (most), peering, VPN, flow logs.  
- **Subnet**: CIDR block, AZ, route table association, NACL, auto-assign public IP, available IP count.

**Q5. What is the VPC limit tree you should know?**  
- 5 VPCs per region (soft)  
- 200 subnets per VPC  
- 5 CIDR blocks per VPC (expandable to 50)  
- 5 IPv6 CIDRs per VPC  
- 200 route tables per VPC  
- 50 routes per route table (200 with BGP)  
- 500 SGs per VPC, 60 inbound + 60 outbound rules  
- 5 IGWs per region (1 per VPC)  
- 5 Elastic IPs per region

**Q6. What is "default tenancy" vs "dedicated tenancy" at VPC level?**  
Set on VPC creation. `dedicated` means all instances run on dedicated hardware regardless of per-instance tenancy. Rarely used (expensive, limits instance types). Cannot be changed easily — impacts every future instance.

**Q7. Walk me through packet flow inside a VPC.**  
Instance ENI → VPC router (implicit) → consults route table of subnet → forwarded to IGW / NAT GW / peer / VPN / endpoint / local. Stateful SG + stateless NACL evaluated. Every packet travels AWS's Hyperplane SDN — no customer-visible switches/routers.

**Q8. What is the VPC implicit router?**  
Invisible component at the heart of every VPC. All routing decisions happen here based on the subnet's associated route table. You cannot see it but you configure it via route tables.

---

## 2. CIDR & IP ADDRESSING

**Q9. What CIDR ranges are allowed in VPC?**  
- RFC1918: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`  
- Also any publicly routable range you own (BYOIP)  
- Mask: `/16` (65,536 IPs) to `/28` (16 IPs)  
- Cannot be modified after creation — only add secondary CIDRs.

**Q10. How many usable IPs in a `/24` subnet in AWS?**  
256 − 5 reserved = **251**. AWS reserves:  
- `.0` – network  
- `.1` – VPC router  
- `.2` – DNS (VPC base + 2)  
- `.3` – reserved for future use  
- `.255` – broadcast (unused in AWS SDN)

**Q11. How do you plan CIDR for a multi-account / multi-region enterprise?**  
- Start with a large block (e.g., `10.0.0.0/8`) divided hierarchically.  
- Per region `/12` or `/14`, per account/VPC `/16`, per AZ `/20`, per subnet tier `/24`.  
- Avoid overlap across accounts — peering/TGW requires non-overlapping.  
- Use **AWS IPAM** for governance.  
- Reserve ranges for EKS pod CIDR (secondary), VPN clients, TGW CIDRs.

**Q12. Why should subnets NOT overlap between peered VPCs?**  
Peering/TGW has no NAT capability — overlapping IPs cause ambiguous routing. Solutions: re-IP (painful), secondary CIDR, PrivateLink (no route exchange), Transit Gateway Connect with middlebox NAT.

**Q13. Can you change VPC CIDR after creation?**  
Primary CIDR — no. But you can **add secondary CIDRs** (up to 4 more, expandable to 49) from any RFC1918 range (must not overlap).

**Q14. What is AWS IPAM?**  
IP Address Manager — centralized service to plan, track, audit, and monitor IP usage across accounts/regions. Features: pools hierarchy, auto-allocation to VPCs, compliance monitoring, BYOIP management, historical tracking.

**Q15. What's the difference between VPC CIDR and Subnet CIDR?**  
VPC = parent block. Subnet = non-overlapping chunk of VPC CIDR, bound to single AZ. Subnet CIDR must be within one of the VPC's CIDR blocks.

---

## 3. SUBNETS

**Q16. Public vs Private vs Isolated subnet — what defines each?**  
- **Public** — route table has `0.0.0.0/0 → IGW`. Resources can get public IP.  
- **Private** — route table has `0.0.0.0/0 → NAT GW`. No direct inbound from internet.  
- **Isolated** — no IGW / NAT route at all. Only VPC-local + endpoints. Used for DBs, sensitive workloads.

**Q17. Can a subnet span multiple AZs?**  
**No.** Always single AZ. Multi-AZ architecture = multiple subnets.

**Q18. What's the minimum and maximum subnet size?**  
`/28` (16 IPs, 11 usable) to `/16` (65,536 IPs). Most architectures use `/24` or `/26`.

**Q19. Why do EKS / EMR / ALB demand multiple subnets across AZs?**  
High availability requirement — AWS forces multi-AZ so failure of one AZ doesn't kill the service.

**Q20. How is subnet association with route table chosen if not explicit?**  
Every subnet implicitly associated with the **main route table** of the VPC until you explicitly associate a custom one. Change main via `ReplaceRouteTableAssociation`.

**Q21. What is "subnet sharing" via RAM — implications?**  
Account A owns VPC; shares specific subnets with Account B via RAM. Account B launches ENIs there but cannot modify subnet / route tables / SG. Good for hub-and-spoke with centralized networking team.

**Q22. Subnet-level settings you can control:**  
- Auto-assign public IPv4 / IPv6  
- DNS hostnames / DNS64  
- Enable/disable resource name DNS  
- `MapPublicIpOnLaunch`  
- Auto-assign customer-owned IP (Outposts)

**Q23. What is a "local zone" subnet?**  
Subnet in a Local Zone (extended AWS AZ close to metros). Parent region's VPC extends there. Limited services. Used for ultra-low-latency apps (gaming, media).

---

## 4. ROUTE TABLES

**Q24. Route evaluation order in a VPC:**  
1. Most specific (longest prefix match) wins.  
2. Static routes beat propagated routes of same specificity.  
3. Local (VPC CIDR) route cannot be deleted/overridden (until recently — now "local" can be replaced via gateway endpoint for specific use).

**Q25. What's the implicit "local" route?**  
Auto-created for each VPC CIDR pointing to `local` target. Ensures all subnets in VPC talk to each other. Now you **can** override local route in special cases (e.g., route through a firewall appliance), but this is a recent feature and risky.

**Q26. What's the default route table? Can you delete it?**  
Created with VPC; cannot be deleted. Can be demoted from "main" by setting another as main.

**Q27. What is a route propagation?**  
Virtual Private Gateway (VGW) or TGW can automatically inject learned BGP routes into route tables associated with propagation.

**Q28. Subnet route table vs Gateway route table — what's the difference?**  
- **Subnet RT** — standard, controls outbound from subnet.  
- **Gateway (Ingress) RT** — attached to IGW/VGW, controls inbound traffic from internet to subnets (for inserting a firewall appliance in path). Used with **VPC Ingress Routing**.

**Q29. How do you blackhole traffic intentionally?**  
Route with target = `blackhole` (TGW/peering) or route to a non-existent ENI. Used for isolating compromised subnets or testing failure scenarios.

**Q30. Can two route tables coexist with conflicting routes?**  
Yes — each subnet associates with exactly one RT. Different subnets = different routing decisions = expected design.

---

## 5. INTERNET GATEWAY (IGW)

**Q31. What exactly does IGW do?**  
- Provides a target in route tables for internet-bound traffic.  
- Performs 1:1 NAT for instances with public IP / EIP.  
- Horizontally scaled, redundant, HA by AWS — no bandwidth limit to care about.  
- One IGW per VPC.

**Q32. Does an IGW cost money?**  
IGW itself is free. You pay for data transfer out to internet and for public IPv4 (since Feb 2024 AWS charges $0.005/hr per public IPv4).

**Q33. What happens if you detach IGW from a VPC with running instances?**  
All outbound-to-internet traffic via those routes breaks instantly. Inbound internet traffic rejected. Useful in incident response — isolate VPC.

**Q34. Why is IGW not enough for instances in private subnet to reach internet?**  
Private instances have no public IP → return traffic has no way back. You need SNAT (NAT Gateway / NAT Instance) to masquerade.

---

## 6. NAT GATEWAY & NAT INSTANCE

**Q35. NAT Gateway vs NAT Instance — comprehensive comparison:**  
| Aspect | NAT Gateway | NAT Instance |
|---|---|---|
| Managed | Fully managed | You manage EC2 |
| HA | Within one AZ (create per AZ) | Manual (ASG + script) |
| Bandwidth | 5–100 Gbps auto-scale | Limited by instance type |
| Maintenance | AWS | You patch, kernel, OS |
| Port forwarding | No | Yes |
| Source/dest check | N/A | Must disable |
| Bastion use | No | Possible |
| Cost | Higher per hour + data | Cheaper for low traffic |
| Security groups | No (uses subnet NACL) | Yes |
| Flow logs | Yes (ENI) | Yes |

**Q36. Why place NAT Gateway in each AZ?**  
If NAT GW in AZ-a fails or AZ-a goes down, instances in AZ-b routed through AZ-a NAT have no internet + cross-AZ data charge. Per-AZ NAT = resilience + cost optimization.

**Q37. Common NAT GW cost trap:**  
Data transfer via NAT GW is **$0.045/GB** (us-east-1) **on top of** egress. Workloads pulling containers, logs, or cross-account S3 through NAT = huge surprise bills. Fix: VPC **Gateway endpoint for S3/DynamoDB** (free), Interface endpoints for others.

**Q38. Can NAT GW be used for inbound traffic?**  
No. NAT GW is egress-only (SNAT). For inbound, use IGW or LB.

**Q39. Can NAT GW route traffic to an on-prem network via VPN?**  
No — NAT GW only targets the IGW. For on-prem destinations, traffic uses the VGW/TGW route directly (no NAT needed typically unless overlapping CIDRs).

**Q40. Public NAT GW vs Private NAT GW:**  
- **Public** — has EIP, outbound to internet.  
- **Private** — no EIP, for VPC→TGW/peered VPC when you need to preserve a private source IP (e.g., overlapping CIDRs, communicating with on-prem that only accepts specific range).

**Q41. How do NAT GW ports/connections scale?**  
Each NAT GW supports up to 55,000 simultaneous connections to each unique destination IP/port combination. `ErrorPortAllocation` CloudWatch metric flags exhaustion. Fix: multiple NAT GWs, spread workloads.

**Q42. Does NAT GW preserve source IP?**  
No — it replaces source IP with its own (SNAT). Destination sees NAT GW's EIP.

---

## 7. EGRESS-ONLY INTERNET GATEWAY (IPv6)

**Q43. What is Egress-Only Internet Gateway (EIGW)?**  
IPv6 equivalent of NAT Gateway. Because IPv6 addresses are globally unique, there's no NAT — but you still want to prevent inbound. EIGW allows outbound IPv6 and blocks inbound, stateful.

**Q44. When do you need EIGW vs IGW for IPv6?**  
- **IGW** — inbound + outbound IPv6 (public-facing IPv6 endpoints).  
- **EIGW** — outbound only (private IPv6 workload that needs external APIs).

**Q45. IPv4 dual-stack vs IPv6-only subnet — design points:**  
- Dual-stack: no change to apps; still need NAT GW / EIPs for v4.  
- IPv6-only (2022+): no v4 NAT cost, but some AWS services still need v4; use **NAT64/DNS64** + **egress-only** pattern.

---

## 8. SECURITY GROUPS vs NACL (VPC context)

*(Basic differences covered in EC2 bank — here, VPC-specific deep dives.)*

**Q46. How many SGs can a VPC have? Why matter?**  
2,500 per VPC (default), 60 inbound + 60 outbound rules per SG. Drives architectural choices — e.g., avoid per-instance SG sprawl; use referenced SGs.

**Q47. NACL rule evaluation — explain with example:**  
Rules processed by ascending rule number. First match wins (allow or deny). Default custom NACL denies all; default NACL allows all. Ephemeral return port ranges (1024–65535 Linux, 49152–65535 Windows) must be explicitly allowed because stateless.

**Q48. When do you actually use NACL in production?**  
- Subnet-wide blanket deny (block a known bad IP range at subnet).  
- Compliance requirement (defense in depth).  
- PCI DSS / HIPAA segmentation.  
- Emergency incident containment (block all except forensics IP).

**Q49. Gotcha: You allowed TCP 443 inbound in NACL but users still can't connect.**  
Forgot ephemeral outbound return. Or the application uses a different port. Or NACL `*` deny rule has lower number than allow rule. Always check bidirectional and rule number ordering.

**Q50. Can SGs reference SGs across VPC peering?**  
**Yes, in the same region** (since 2018). Cross-region peering — no; must use CIDRs or prefix lists.

**Q51. Prefix Lists — what problem do they solve?**  
Named list of CIDRs maintained centrally. Reference it in SG/RT. Customer-managed (your IPs) and AWS-managed (S3, DynamoDB, CloudFront). Enables central edits without touching every SG.

---

## 9. VPC PEERING

**Q52. What is VPC peering?**  
1:1 connection between two VPCs (same/different account/region) using AWS backbone. Non-transitive. No SPOF, no bandwidth bottleneck.

**Q53. Limitations of VPC peering:**  
- No transitive routing (A↔B, B↔C doesn't give A↔C).  
- CIDR ranges cannot overlap.  
- No edge-to-edge routing (can't route peer→IGW/VPN/DX of the peer).  
- Limits: 125 active peerings per VPC (soft).  
- Inter-region peering supported but has data transfer cost.

**Q54. Steps to establish peering:**  
1. Requester VPC initiates.  
2. Accepter VPC accepts.  
3. Update route tables on **both** sides to the peer CIDR via `pcx-xxxx`.  
4. SGs/NACLs allow traffic.  
5. (Optional) Enable DNS resolution across peers.

**Q55. DNS resolution across peered VPCs:**  
By default, private DNS of peer VPC not resolvable. Enable on both sides: `modify-vpc-peering-connection-options --allow-dns-resolution-from-remote-vpc`.

**Q56. How to troubleshoot peering connectivity issue?**  
1. Peering status `active`?  
2. Routes on both sides?  
3. SG rules allow source CIDR / peer SG?  
4. NACL?  
5. Overlapping CIDR?  
6. Reachability Analyzer — run a check.  
7. VPC Flow Logs.

**Q57. Cost of VPC peering:**  
No hourly charge. Data transfer: same-region = free within AZ (peered), cross-AZ = $0.01/GB both ways, cross-region = inter-region data transfer rates.

---

## 10. TRANSIT GATEWAY (TGW)

**Q58. Why was TGW introduced over peering?**  
Peering is 1:1 and non-transitive → 100 VPCs = 4,950 peerings (mesh nightmare). TGW is a hub: attach VPCs, VPNs, DX, peered TGWs. Centralized routing, segmentation via route tables.

**Q59. TGW architecture components:**  
- **TGW** — regional hub.  
- **Attachments** — VPC, VPN, DX Gateway, peering with another TGW, Connect (SD-WAN/transit VPC).  
- **Route tables** — per attachment domain (e.g., prod, dev, shared services).  
- **Propagations** and **associations**.

**Q60. TGW route table: association vs propagation — what's the difference?**  
- **Association** — which RT an attachment *uses* (to make routing decisions from that attachment).  
- **Propagation** — which RT *learns* routes from an attachment.  
- Example: prod VPC associated with prod-RT, propagates to prod-RT (so prod VPCs can talk). Shared-services VPC propagates to both prod-RT and dev-RT (reachable from both), but associates only with its own RT (isolated return path).

**Q61. TGW bandwidth and limits:**  
- 50 Gbps per VPC attachment (max) / 5,000 attachments per TGW.  
- 10,000 routes per TGW RT.  
- BGP for DX/VPN.  
- Per GB data processing charge on top of hourly TGW attachment fee.

**Q62. TGW inter-region peering — points to know:**  
- Encrypted over AWS backbone.  
- Supports different route tables per region.  
- No overlapping CIDR.  
- Used for global segmentation / DR.

**Q63. Common TGW design patterns:**  
- **Flat** — one RT, everyone talks to everyone.  
- **Segmented** — prod/dev/shared RTs with controlled propagation.  
- **Hub-and-spoke with inspection** — all east-west routed through inspection VPC (Network Firewall / 3rd-party).  
- **Multi-region global** — peer TGWs across regions.

**Q64. TGW Connect — what is it?**  
Attachment type using GRE tunnels over a VPC attachment, used for SD-WAN/third-party virtual routers. Supports BGP for dynamic routing at higher throughput than VPN.

**Q65. TGW vs peering vs PrivateLink — when to use what?**  
- **Peering** — few VPCs, simple, cheap.  
- **TGW** — many VPCs, segmentation needed, centralized egress.  
- **PrivateLink** — expose a specific service, no route exchange, unidirectional, supports overlapping CIDRs.

---

## 11. SITE-TO-SITE VPN

**Q66. AWS S2S VPN components:**  
- **Customer Gateway (CGW)** — object representing your on-prem device (public IP, BGP ASN, optional cert).  
- **Virtual Private Gateway (VGW)** or **TGW** on AWS side.  
- **VPN Connection** — always 2 IPSec tunnels to different AWS endpoints for HA.

**Q67. Static vs Dynamic (BGP) VPN:**  
- **Static** — you define remote CIDRs manually.  
- **Dynamic (BGP)** — tunnels exchange routes, auto failover, supports AS path prepending for primary/secondary.

**Q68. VPN is using only one tunnel — why?**  
Most on-prem devices activate only primary by default; the second is passive unless routing policy (BGP weights / prepends) uses it. For active-active, configure ECMP (TGW supports equal-cost multipath over both tunnels).

**Q69. VPN throughput — what limits should I quote?**  
~1.25 Gbps per tunnel. Aggregate with ECMP on TGW to reach ~10 Gbps. For higher → Direct Connect or multiple VPNs.

**Q70. Accelerated Site-to-Site VPN — what changes?**  
Routes over AWS Global Accelerator to enter AWS backbone at nearest edge POP rather than nearest region. Lower jitter. Higher cost. Supported only with TGW attachment.

**Q71. What happens to VPN during AWS endpoint maintenance?**  
Only one tunnel is updated at a time — that's why 2 tunnels exist. If your on-prem uses only 1, expect periodic ~15-min outages.

**Q72. VPN troubleshooting — what logs are available?**  
- CloudWatch Logs for VPN tunnels (enable Tunnel Logging, 2021+).  
- Flow Logs at the VGW/TGW attachment ENI.  
- On-prem device logs.  
- `describe-vpn-connections` → tunnel status, last state change reason.

---

## 12. CLIENT VPN

**Q73. What is AWS Client VPN?**  
Managed OpenVPN-based endpoint that lets remote users connect into a VPC. Auth: AD, SAML (Okta/Azure AD), or mutual cert.

**Q74. Key components:**  
- Client VPN endpoint (TLS)  
- Associated target VPC subnets  
- Authorization rules (per CIDR)  
- Client CIDR (pool for clients)  
- Connection logs  
- Split-tunnel option

**Q75. Split-tunnel vs Full-tunnel:**  
- **Split** — only VPC-routed traffic goes through tunnel; rest via client's normal internet.  
- **Full** — all client traffic via AWS (mimics on-prem "VPN to office"). Higher data transfer cost.

**Q76. Client VPN pricing quirk:**  
You pay per **active association** (subnet hour) + per client connection hour. A single subnet association idle overnight = still billed. Tear down out of business hours if acceptable.

---

## 13. AWS DIRECT CONNECT (DX)

**Q77. Direct Connect vs VPN — high-level:**  
| | DX | VPN |
|---|---|---|
| Medium | Private line via partner | Internet |
| Bandwidth | 1/10/100 Gbps | up to ~1.25 Gbps/tunnel |
| Latency | Predictable, low | Variable |
| Encryption | None (add MACsec or VPN over DX) | IPsec |
| Setup | Weeks | Minutes |
| Cost | Port hrs + data | Tunnel hrs + data |

**Q78. Public VIF vs Private VIF vs Transit VIF:**  
- **Public VIF** — access public AWS services (S3, etc.) over DX.  
- **Private VIF** — connect to a single VGW / single VPC.  
- **Transit VIF** — connect to DX Gateway attached to a TGW (multi-VPC, multi-region).

**Q79. DX Gateway — what and why?**  
Global object decoupling DX physical connection from target VPCs. Lets one DX connect to multiple VGWs in different regions (same or different accounts via association).

**Q80. How do you achieve HA for DX?**  
- Two connections, different locations, different devices.  
- Two VIFs over LAGs.  
- VPN as failover (best practice).  
- BGP with AS path prepending / LOCAL_PREF.

**Q81. MACsec on DX — when needed?**  
For compliance requiring encryption at layer 2. Available on 10/100 Gbps dedicated. Otherwise layer-3 encryption via VPN over DX.

**Q82. BFD for BGP — why enable?**  
Bidirectional Forwarding Detection reduces failover time from ~90s (BGP hold) to ~1s. Critical for production DX.

---

## 14. VPC ENDPOINTS

**Q83. Gateway Endpoint vs Interface Endpoint — core diff:**  
| | Gateway | Interface |
|---|---|---|
| Services | S3, DynamoDB only | 100+ services |
| Implementation | Route table entry | ENI with private IP |
| Cost | Free | Hourly + data |
| DNS | Uses public DNS resolving to private | Private DNS enabled → transparent |
| Access | Only same region | Same region |

**Q84. Interface Endpoint (PrivateLink) — pros and gotchas:**  
Pros: no NAT/IGW needed, traffic stays on AWS backbone, enforce resource-based policies.  
Gotchas: hourly cost per endpoint per AZ, data processing charge, need to enable **Private DNS** so SDKs resolve to endpoint ENI, some services unsupported.

**Q85. How to restrict S3 access to only via VPC endpoint?**  
Two-layer policy:  
- **Endpoint policy** — restrict which buckets/accounts endpoint can talk to.  
- **S3 bucket policy** — `Deny` unless `aws:sourceVpce` matches. Prevents internet access to bucket even with valid IAM creds.

**Q86. What happens if you delete a gateway endpoint with live traffic?**  
Route to S3/DDB via endpoint removed → traffic falls back to IGW/NAT GW if route exists, else fails. Cost may spike.

**Q87. Cross-region endpoint — possible?**  
Gateway endpoints: no. Interface endpoints: only if PrivateLink service supports cross-region (recent feature for some); generally you use endpoints in each region.

**Q88. Can a VPC have endpoint for a service in another account?**  
Yes — PrivateLink consumer endpoint points to service ENI in provider account. Allowlist by principal.

---

## 15. AWS PRIVATELINK

**Q89. What is PrivateLink?**  
Technology enabling private connectivity from consumer VPC to a service without exposing it to internet or requiring peering. Service provider publishes NLB-backed endpoint service; consumers create interface endpoint.

**Q90. PrivateLink vs Peering — when to use which?**  
- PrivateLink — service-level, unidirectional, overlapping CIDR OK, scalable to many consumers.  
- Peering — full bidirectional network connectivity, overlapping CIDR disallowed, 1:1.

**Q91. PrivateLink components:**  
- **Service Consumer** — VPC endpoint (ENI)  
- **Service Provider** — NLB (or GWLB) + VPC Endpoint Service  
- **Allowlist** — principals allowed to connect  
- **Endpoint service DNS name** — `com.amazonaws.vpce.<region>.<id>`

**Q92. Can I front PrivateLink with an ALB?**  
Not directly. NLB is required. Workaround: NLB → ALB (NLB supports ALB targets since 2021). Or GWLB for inspection appliances.

**Q93. Resource endpoints vs service endpoints (2024+):**  
Newer PrivateLink model allows sharing specific resources (RDS, DynamoDB tables) via Resource endpoints without running an NLB — simpler for SaaS providers.

---

## 16. VPC FLOW LOGS

**Q94. What are VPC Flow Logs?**  
Metadata capture of IP traffic at VPC / subnet / ENI level. Not packet payload. Fields: srcaddr, dstaddr, srcport, dstport, protocol, packets, bytes, action (ACCEPT/REJECT), log-status, start, end, interface-id.

**Q95. Three delivery destinations:**  
- **CloudWatch Logs** — real-time query with Logs Insights.  
- **S3** — cheap long-term, query with Athena.  
- **Kinesis Data Firehose** — streaming to third party (Splunk, Datadog).

**Q96. What Flow Logs DO NOT capture:**  
- Instance metadata service calls  
- Amazon DNS server traffic (unless custom DNS)  
- DHCP  
- Windows license activation  
- Traffic to/from `169.254.169.254` and `169.254.169.253`  
- Mirrored traffic  
- Layer 2

**Q97. Custom log format — why use?**  
Default format is limited. Add fields: `tcp-flags`, `pkt-srcaddr` (original src before NAT), `pkt-dstaddr`, `region`, `az-id`, `sublocation-type`, `flow-direction`, `traffic-path`. Useful to see if traffic went via IGW, NAT, TGW, peering, etc.

**Q98. How to detect SSH brute force via Flow Logs?**  
Athena query: `WHERE dstport=22 AND action='REJECT' GROUP BY srcaddr HAVING count > 100 within 5min`. Send to Security Hub / GuardDuty (which already does this).

**Q99. Flow Logs are showing REJECT but SG/NACL allow — why?**  
- Maybe blocked by NACL on return (stateless).  
- Ephemeral port range not allowed.  
- Host firewall (iptables).  
- Service not listening on port.

---

## 17. DNS IN VPC & ROUTE 53 RESOLVER

**Q100. Two VPC DNS settings — what they mean:**  
- `enableDnsSupport` — DNS queries to Amazon provided DNS (VPC base +2) work.  
- `enableDnsHostnames` — instances get internal DNS name (`ip-10-0-1-5.ec2.internal`) and, if public IP, public DNS.  
Both needed for many integrations (private VPC endpoints, DB DNS).

**Q101. Amazon-provided DNS resolver IP?**  
`VPC CIDR base + 2` (e.g., `10.0.0.2` for `10.0.0.0/16`). Also `169.254.169.253`.

**Q102. Route 53 Resolver — what problem does it solve?**  
Hybrid DNS. Allows on-prem ↔ VPC private zone name resolution.  
- **Inbound endpoint** — on-prem resolvers can query VPC private zones.  
- **Outbound endpoint** — VPC resolvers forward queries for specific domains to on-prem DNS.  
- **Resolver rules** — which domains forward where.

**Q103. Private Hosted Zone (PHZ) vs Public Hosted Zone:**  
- **Public** — resolvable from internet.  
- **Private** — resolvable only from associated VPCs. Can associate with multiple VPCs, including cross-account via RAM-like pattern.

**Q104. Can two PHZs have overlapping domains? Conflict resolution?**  
Yes, but only one can be associated with a VPC per exact domain. Overlapping: more-specific wins (e.g., `staging.example.com` beats `example.com`).

**Q105. Route 53 Resolver DNS Firewall — what does it do?**  
Filter outbound DNS queries by domain (block known C2, DNS tunneling). Allowlist / denylist rule groups. Managed list from AWS also available.

**Q106. Conditional forwarding for hybrid DNS — design:**  
- VPC outbound resolver endpoint in 2 subnets (HA).  
- Rules: forward `corp.internal` → on-prem IPs.  
- Share rule with other VPCs via RAM → all spokes get same hybrid DNS.

**Q107. DNS64 and NAT64 — when needed?**  
IPv6-only subnet where target is IPv4-only. DNS64 synthesizes AAAA from A records to IPv6; NAT Gateway (configured for NAT64) translates v6→v4. Enables v6 migration without breaking v4 destinations.

---

## 18. DHCP OPTIONS SET

**Q108. What can you configure in DHCP options?**  
- `domain-name` (e.g., `ec2.internal`)  
- `domain-name-servers` (AmazonProvidedDNS or custom)  
- `ntp-servers`  
- `netbios-name-servers`  
- `netbios-node-type`  
- `ipv6-address-preferred-lease-time`

**Q109. When would you replace default DHCP options?**  
- Use corporate DNS (e.g., on-prem AD DNS for domain-joined instances).  
- Internal NTP servers for compliance.  
- Custom `domain-name` matching on-prem (AD FQDN).

**Q110. Gotcha: Replacing DHCP options mid-running:**  
Running instances keep old values until DHCP lease renews / reboot. Plan accordingly.

---

## 19. ENI DEEP DIVE (VPC context)

**Q111. ENI attachment types in VPC:**  
- Attached to EC2 (primary or secondary)  
- VPC endpoint (interface)  
- NAT Gateway (AWS-managed, read-only)  
- Transit Gateway attachment  
- Lambda VPC config  
- RDS / ElastiCache / ECS tasks (awsvpc mode)  
- Interface endpoints, Load Balancers  
- Fargate tasks  
- Workspaces, AppStream

**Q112. Why do Lambda + VPC cost more cold starts historically?**  
Pre-2019 Lambda created an ENI per invocation in VPC → cold start tens of seconds. Now Lambda Hyperplane ENIs are shared per security group + subnet, created once per unique combination.

**Q113. ENI trunking — what is it?**  
Nitro-based EC2 with ENA supports trunking for ECS `awsvpc` mode — lets each task get dedicated ENI without consuming EC2 ENI slot 1:1. Increases task density.

**Q114. Requester-managed ENIs — what are they?**  
ENIs created by AWS services on your behalf (ELB, RDS, Lambda, VPC endpoints). You can see them but not modify most properties. Account for them in IP planning.

---

## 20. SECONDARY CIDR, IPv6 & IPAM

**Q115. When do you add a secondary CIDR?**  
- Running out of IP space in primary  
- EKS pod CIDR exhaustion  
- Adding a new environment tier (e.g., DMZ)

**Q116. EKS IP exhaustion — a real-world pattern:**  
VPC CNI assigns one VPC IP per pod → `/24` subnet = ~250 pods. Solutions:  
- Use secondary CIDR (`100.64.0.0/10` CGNAT range is popular) attached to a CNI custom networking config.  
- Use `prefix delegation` (each ENI gets `/28` prefix) — 16× more pods per ENI.  
- Use VPC CNI with IPv6.

**Q117. IPv6 in VPC — basics:**  
- AWS-assigned `/56` block or BYOIP.  
- Subnets get `/64`.  
- Every IPv6 address is public-routable — use EIGW for private.  
- No NAT for v6.  
- Dual-stack or v6-only.

**Q118. IPAM tiers:**  
- Free tier — basic monitoring.  
- Advanced tier — cross-region, cross-account, auto-allocation, public IPv4 tracking, compliance. Per-IP monthly charge.

**Q119. BYOIP — what is it?**  
Bring Your Own IP range. Register ROA for prefix (≥ /24 IPv4 or /48 IPv6). Enables migrations without changing IPs. AWS advertises prefix from any region(s) you choose.

---

## 21. AWS NETWORK FIREWALL

**Q120. What is AWS Network Firewall?**  
Managed stateful/stateless firewall (Suricata-compatible) deployed in a VPC as a highly available gateway. Supports deep packet inspection, domain filtering, IPS rules.

**Q121. Where is Network Firewall deployed?**  
Usually in a central inspection VPC attached to TGW, or in dedicated firewall subnet with routes directed through it (via gateway route table / VPC ingress routing).

**Q122. Network Firewall vs SG vs NACL vs GWLB appliance:**  
- **SG/NACL** — L3/L4, identity-based, free.  
- **Network Firewall** — L3–L7, managed, Suricata rules.  
- **GWLB** — load balances traffic to 3rd-party firewalls (Palo Alto, Fortinet, Check Point).

**Q123. Common Network Firewall patterns:**  
- Central egress: all spoke VPCs → TGW → inspection VPC → Network Firewall → IGW.  
- East-west inspection between prod/dev via TGW.  
- Domain-based outbound filtering for PCI scope.

---

## 22. TRAFFIC MIRRORING

**Q124. What is VPC Traffic Mirroring?**  
Copy network packets from a source ENI to target (ENI or NLB) for analysis. Used with IDS (Suricata, Zeek), packet-capture, compliance.

**Q125. Filter & session concepts:**  
- **Mirror filter** — what traffic to capture (5-tuple).  
- **Mirror target** — where to send (ENI, NLB).  
- **Mirror session** — links filter + source ENI + target.  
- Limits: only Nitro instances as sources; throughput shared with instance bandwidth.

**Q126. Traffic Mirroring vs Flow Logs — difference:**  
- Flow Logs = metadata.  
- Traffic Mirroring = full packet payload (encapsulated in VXLAN).

---

## 23. REACHABILITY ANALYZER & NETWORK ACCESS ANALYZER

**Q127. Reachability Analyzer — what does it do?**  
Static analysis of your VPC config: "Can source X reach destination Y on port Z?" Returns path or blocking component (SG, NACL, route, etc.). Does not send real packets.

**Q128. Network Access Analyzer — different purpose?**  
Validates network against scopes (e.g., "No resource in VPC A should reach internet except via firewall"). Finds unintended paths. Compliance auditing tool.

**Q129. When do you prefer Reachability Analyzer over Flow Logs?**  
Before sending traffic (pre-deployment verification) or when traffic never leaves source (no flow log generated) and you don't know why.

---

## 24. SHARED VPC (AWS RAM)

**Q130. Shared VPC — key principle:**  
Owner account creates VPC + subnets. Uses AWS Resource Access Manager (RAM) to share subnets with participant accounts. Participants launch resources into those subnets but don't manage networking.

**Q131. Who owns what in shared VPC?**  
- Owner — VPC, subnets, route tables, IGW, NAT, TGW, SGs (can still create), NACLs, flow logs.  
- Participant — ENIs, instances, LBs, RDS they launch; their own SGs (scoped to shared subnet).  
- Neither can modify each other's resources.

**Q132. Benefits of shared VPC:**  
- Centralized networking team manages CIDR, NAT, TGW.  
- Fewer VPCs → fewer NAT/TGW attachments → lower cost.  
- Simpler DNS and routing.  
- Consistent security baselines.

**Q133. When NOT to use shared VPC:**  
- Strong account isolation requirement (compliance).  
- Participants need full networking control.  
- Heavy VPC endpoint usage (all participants share endpoints → large blast radius).

---

## 25. VPC LATTICE

**Q134. What is VPC Lattice (GA 2023)?**  
Application-layer service networking — connect services across VPCs/accounts without VPC peering/TGW. Handles service discovery, IAM auth, observability, path routing.

**Q135. VPC Lattice vs TGW:**  
- TGW — network-layer, general IP routing.  
- Lattice — service-layer (like service mesh at AWS level), per-service authorization. You don't care about IPs/routes.

**Q136. Lattice components:**  
- **Service network** — logical boundary.  
- **Service** — target (NLB, ALB, EC2, Lambda, IP targets).  
- **Listeners / routing rules** — HTTP/HTTPS/gRPC.  
- **Auth policies** — IAM-based.

---

## 26. ADVANCED: CARRIER GW, LOCAL ZONES, WAVELENGTH, OUTPOSTS

**Q137. Carrier Gateway — what is it?**  
Used in Wavelength Zones (inside telco 5G network). Routes traffic to mobile devices over carrier network instead of internet.

**Q138. Local Zones — networking model:**  
VPC extends to Local Zone subnet. Traffic to parent region uses AWS backbone. Low latency for metro users. Limited instance types and services.

**Q139. Outposts — VPC extension on-prem:**  
Customer-owned IP (CoIP) pool, local gateway (LGW) for on-prem connectivity, VPC subnets in Outposts rack. Traffic to AWS region over Outposts service link (VPN or DX).

**Q140. Wavelength vs Local Zones vs Outposts — quick pick:**  
- **Wavelength** — 5G edge, telco networks.  
- **Local Zones** — metro edge, public AWS hardware.  
- **Outposts** — your data center, AWS hardware.

---

## 27. SCENARIO-BASED QUESTIONS

**Q141. Design: multi-account, multi-region enterprise landing zone networking.**  
- Control Tower + AWS Organizations.  
- Shared networking account — hosts TGW per region, DX Gateway, Network Firewall, Route 53 Resolver endpoints, PHZ, IPAM.  
- Spoke VPCs attached to TGW.  
- TGW route tables: prod-RT, non-prod-RT, shared-services-RT, egress-RT.  
- Inter-region TGW peering.  
- Central egress inspection VPC.  
- Endpoints shared via RAM where practical.

**Q142. Team needs 10,000 EKS pods in one VPC — but only `/16` available (~65K IPs realistic 50K usable). Plan.**  
- Secondary CIDR in `100.64.0.0/10` range.  
- VPC CNI custom networking: pod ENIs in secondary CIDR, node ENIs in primary.  
- Prefix delegation: each ENI carries `/28` → massive pod density.  
- Alternative: IPv6 EKS cluster.

**Q143. Request: "Block instances in Subnet A from reaching internet, but still let them reach S3."**  
- Remove default route via NAT/IGW in A's RT.  
- Add Gateway endpoint for S3; route `pl-s3` via endpoint.  
- SG: outbound only to S3 prefix list.  
- Test with Reachability Analyzer.

**Q144. Two VPCs in different accounts must communicate but CIDR overlaps.**  
- Option A: Re-IP one (best but costly).  
- Option B: PrivateLink (exposes specific service only, unidirectional).  
- Option C: TGW Connect / third-party NAT in transit VPC performing 1:1 NAT.  
- Option D: Private NAT Gateway for SNAT on one side (limited to outbound).

**Q145. Prod VPC must reach on-prem `10.10.0.0/16`, but only for HTTPS to a specific host.**  
- DX or VPN to VGW/TGW.  
- Route `10.10.0.0/16` via tunnel.  
- SG allow 443 to specific IP.  
- Network Firewall domain allowlist (if DNS-based).  
- On-prem firewall mirrors allowlist.

**Q146. You need east-west traffic inspection across 20 VPCs.**  
- TGW with inspection VPC in-path.  
- Use appliance mode on TGW VPC attachment (ensures same-flow symmetric routing through appliance).  
- Network Firewall or GWLB with partner appliances.  
- Centralized logs → S3 → Athena.

**Q147. Migration from peering mesh (25 VPCs) to TGW. Without downtime.**  
- Create TGW, attach all VPCs.  
- Gradually add routes to TGW alongside peering (more-specific peering wins initially).  
- Shift one route at a time to TGW, monitor.  
- Remove peering one by one.  
- Validate via Reachability Analyzer and synthetic probes.

**Q148. CIO wants DR: if region A goes down, traffic moves to region B in < 60s.**  
- Active-active deployment; Route 53 health checks + failover / weighted.  
- Global Accelerator (anycast, 30s failover).  
- TGW inter-region peering for data replication.  
- Pre-warmed resources to avoid cold start.  
- Data layer: cross-region replication (Aurora Global, DynamoDB Global).

**Q149. Compliance: all internet-bound traffic must pass through a proxy with logging.**  
- Squid on ASG with NLB.  
- Private subnets route `0.0.0.0/0` to NLB.  
- NLB targets proxy fleet.  
- Proxy egresses via NAT GW → IGW.  
- Logs → S3 + Athena.  
- Alternative: Network Firewall with stateful domain inspection.

**Q150. Auditor asks for diagram + proof no prod workload is internet-accessible.**  
- Network Access Analyzer scope: "Prod VPC cannot reach IGW."  
- Automate via AWS Config rule + evidence in Audit Manager.  
- Produce Reachability Analyzer snapshots.  
- Flow Logs queried for any `dstaddr` public + action ACCEPT.

---

## 28. TROUBLESHOOTING QUESTIONS

**Q151. Instance in private subnet can't reach internet for `yum update` — checklist:**  
1. Route table has `0.0.0.0/0 → nat-xxx`?  
2. NAT GW status `available` and in *public* subnet?  
3. NAT GW's subnet RT has `0.0.0.0/0 → IGW`?  
4. NAT GW has EIP?  
5. NACLs on both private and public subnet allow inbound/outbound ephemeral?  
6. SG outbound allows 443/80?  
7. DNS resolving (`/etc/resolv.conf` → `x.x.x.2`)?  
8. Test: `curl -v https://s3.amazonaws.com`.

**Q152. App in VPC can't resolve `db.prod.internal` (PHZ).**  
- VPC associated with PHZ?  
- `enableDnsHostnames` and `enableDnsSupport` = true?  
- Record exists and correct type?  
- For cross-VPC: authorized association?  
- Using custom DHCP with non-Amazon DNS that doesn't forward PHZ.

**Q153. Peering shows `active` but `ping` across peers fails.**  
- Route tables updated on both sides?  
- SGs reference correct peer CIDR (security group references only same region)?  
- NACLs allow ICMP in both directions (echo + reply)?  
- Source/destination check (if custom routers)?  
- Overlapping CIDR silent fail?

**Q154. New VPC endpoint for S3 created, but `boto3` still goes via NAT GW.**  
- Gateway endpoint added to RT? Specifically the RTs of subnets that host these workloads?  
- Endpoint policy restrictive?  
- Region mismatch (endpoint region = bucket region)?  
- Client-side DNS cache using old IP (interface endpoint case).

**Q155. TGW attachment stuck in `pending`.**  
- Other account accepted the share?  
- Subnet in each AZ for TGW attachment (at least one per AZ you need)?  
- Availability of subnet IPs?  
- RAM invitation pending?

**Q156. VPN tunnel repeatedly flaps every 28,800 seconds.**  
Classic IPSec phase-2 rekey issue. Mismatched lifetime/PFS on on-prem side. Align parameters with AWS tunnel options or enable DPD (Dead Peer Detection).

**Q157. Latency high between two EC2 in same VPC but different AZ — user blames AWS.**  
- Baseline cross-AZ ~1–2 ms.  
- Check: instance type (burstable network), ENA enabled, Placement Group cluster for <1ms.  
- TCP window scaling / MTU path issues.  
- Enable Jumbo frames (9001 MTU) within AZ (not cross-AZ reliably).  
- VPC Flow Logs show retransmits via custom format.

**Q158. Connection count to NAT GW exhausted — symptom?**  
`ErrorPortAllocation > 0` in CloudWatch. App sees intermittent connection timeouts to same destination. Fix: add NAT GW, or reduce concurrent connections to single destination, or move to VPC endpoint for that service.

**Q159. Customer sees `Destination Unreachable` from on-prem to AWS private IP even though DX is up.**  
- Missing VGW/TGW route for on-prem CIDR in VPC route table.  
- BGP advertising correct prefixes?  
- AS path filters on on-prem router?  
- DX Gateway associations / allowed prefixes list limits prefix count — maybe hitting limit.

**Q160. Instance in a new subnet can't reach other subnets in same VPC.**  
- Custom RT without local route? (Rare — AWS adds local automatically.)  
- NACL on either side denying.  
- Subnet's NACL default rule order altered.  
- ENI has multiple SGs with conflicting rules (most restrictive in outbound direction).

**Q161. Client VPN user connected but can't reach internal resource.**  
- Authorization rule missing for destination CIDR?  
- Routes on Client VPN endpoint map CIDRs to associated subnets?  
- Split-tunnel enabled — must include target CIDR in routes pushed to client.  
- SG on target allows source CIDR (Client VPN CIDR).

---

## 29. REAL-TIME PRODUCTION ISSUES (LinkedIn style)

**Q162. We suddenly had 40% increase in NAT Gateway data processing charges. How to investigate?**  
- CloudWatch NAT GW metrics: `BytesOutToDestination` per NAT.  
- VPC Flow Logs (Athena): top `dstaddr` and `srcaddr` consuming through NAT ENI.  
- Typical culprits: ECR image pulls, package mirrors, S3 (solved by Gateway endpoint), CloudWatch Logs, STS.  
- Fix: add Gateway/Interface endpoints, mirror packages internally.

**Q163. After CIDR expansion, some ALB health checks fail.**  
ALB SG auto-generated rules may not include new CIDR. Or ALB scales into AZ where subnet is secondary CIDR and target SG doesn't allow that range. Fix: reference ALB SG (not CIDR) in target SG.

**Q164. TGW route table size approaching the 10K limit.**  
- Aggregate CIDRs (summarize subnets into supernet).  
- Use multiple TGW RTs to segment domains so each carries subset.  
- Use prefix lists to reference.

**Q165. You need to roll out Network Firewall across 50 accounts with zero missed paths.**  
- Central inspection VPC + TGW.  
- Account Factory / CloudFormation StackSet to enforce: default route → TGW.  
- SCP blocks IGW creation in spoke accounts.  
- Network Access Analyzer scope verifies.  
- Flow logs centralized to security account.

**Q166. Bug: instances in new subnet randomly get IPs outside the subnet CIDR.**  
Not possible under AWS constraints. Red flag: client-side misconfigured VPN client, bridge interface, Docker overlay. Check instance OS networking.

**Q167. Customer reports "asymmetric routing dropping packets" through inspection appliance.**  
- TGW attachment "appliance mode" not enabled → return path different appliance instance → stateful appliance drops.  
- Enable appliance mode on inspection VPC attachment.

**Q168. Accidentally deleted the main route table.**  
Cannot delete main — AWS prevents. But you can delete non-main default RT or accidentally replace main. Recovery: create new RT with correct routes, set as main; reassociate subnets as needed.

**Q169. PCI scope reduction — must block non-HTTPS from one prod VPC to another.**  
- TGW appliance mode through Network Firewall.  
- Rule: drop all except TCP/443 between those CIDRs.  
- Logs for audit.  
- Evidence via Network Access Analyzer.

**Q170. SaaS vendor wants to access only one DynamoDB table — how to expose privately?**  
- They can create their own Interface endpoint to DynamoDB — uses their creds.  
- Or provide PrivateLink to your service fronting DynamoDB API.  
- Not peering (exposes too much).

**Q171. Post-migration: customer reports slow S3 uploads from EC2 — "was fast before."**  
- Previously direct via IGW with S3 transfer acceleration?  
- Now via NAT GW to S3 → throughput bottleneck.  
- Fix: S3 Gateway endpoint → zero cost, backbone path.

**Q172. Flow logs show `NODATA` for an ENI intermittently.**  
`log-status=NODATA` means no traffic during window. `SKIPDATA` means capacity/space issue at AWS. Investigate ENI recent attachment or security/instance state.

**Q173. CIS benchmark requires no unused SGs. How to automate cleanup?**  
- Lambda scan: list SGs and ENIs, find SGs not attached.  
- AWS Config `ec2-security-group-attached-to-eni`.  
- Automate deletion with approval workflow (SSM Change Manager).

**Q174. Spike of REJECTs in Flow Logs on port 3389 from same ASN.**  
Scan/attack. Options: drop via NACL, WAF if exposed via ALB, block ASN prefix list (dynamic block). Long-term: remove RDP exposure, use SSM Session Manager / Bastionless.

---

## 30. DEVOPS / IaC FOCUSED VPC QUESTIONS

**Q175. Terraform: best practice to structure a VPC module.**  
- Reusable `vpc` module: inputs `cidr`, `az_count`, `public/private/isolated subnet bits`, `enable_nat_gateway`, `single_nat`.  
- Outputs subnet IDs per tier.  
- Prefer open-source `terraform-aws-modules/vpc/aws` for baseline.  
- State: remote (S3 + DynamoDB lock) split per env.

**Q176. Terraform pitfall: tainting NAT Gateway recreates it and breaks routes — how to avoid?**  
Use `create_before_destroy = true` in lifecycle; update route to new NAT, then destroy old. Or stage via separate `terraform apply -target` and manual RT update.

**Q177. Terraform drift on security group rules — common cause?**  
Rules added manually in console, not in code. Detect via `terraform plan` / AWS Config. Prevent via SCP restricting `AuthorizeSecurityGroupIngress` to CI role.

**Q178. How to represent hub-and-spoke TGW in Terraform across multiple accounts?**  
- Central account: TGW + resource share via `aws_ram_resource_share`.  
- Spoke accounts: `aws_ec2_transit_gateway_vpc_attachment` referencing shared TGW ID via `var`.  
- Pipeline orchestrates: central applied first, spoke next.  
- Consider Terragrunt for DRY.

**Q179. CloudFormation StackSet scenario: deploying base VPC config across 100 accounts.**  
- Service-managed StackSet via AWS Organizations.  
- OU-targeted deployment.  
- Drift detection enabled.  
- Parameters per OU (e.g., CIDR block ranges from IPAM).

**Q180. How do Kubernetes (EKS) and VPC interact w.r.t. load balancer controller?**  
- AWS Load Balancer Controller creates ALB/NLB from `Service` / `Ingress` resources.  
- Requires subnet tags: `kubernetes.io/role/elb=1` (public), `kubernetes.io/role/internal-elb=1` (private).  
- IRSA role with `ec2:*` limited + `elasticloadbalancing:*`.  
- Target-type `ip` (pods via VPC CNI direct) preferred over `instance` (uses NodePort).

**Q181. How do you inject VPC config into Jenkins pipelines that spin up ephemeral EC2 for builds?**  
- Jenkins nodes via EC2 Fleet plugin — ASG in dedicated build subnet.  
- SG: allow outbound, no inbound.  
- Subnet with NAT / endpoints for ECR, S3, Systems Manager.  
- Build artifacts to S3 via Gateway endpoint (no NAT cost).

**Q182. How to avoid VPC config as a bottleneck for platform teams serving 100 dev teams?**  
- Shared VPC pattern via RAM.  
- Account Factory for Terraform (AFT) hands out accounts pre-wired to TGW.  
- Service Catalog products for VPC endpoints, NACL updates (self-service with guardrails).

**Q183. GitOps for networking — how to handle sensitive changes (e.g., opening a new CIDR to internet)?**  
- PR review with CODEOWNERS (security/networking must approve).  
- Terraform plan output as PR comment.  
- Atlantis / Terraform Cloud for apply.  
- Tag-based drift monitoring.  
- AWS Config rules + Security Hub to catch post-merge deviations.

**Q184. Observability for VPC in production:**  
- VPC Flow Logs → OpenSearch/Datadog.  
- NAT GW, TGW, VPN CloudWatch metrics → dashboards.  
- Synthetic canaries hitting via ALB, cross-region.  
- Reachability Analyzer scheduled for critical paths.  
- AWS Network Manager (global dashboard).

---

## BONUS: 20 RAPID-FIRE ONE-LINERS

1. Smallest VPC CIDR? → `/28` (16 IPs).  
2. Smallest subnet? → `/28`.  
3. Can IGW be used with private subnet? → No (instances need public IP).  
4. Max EIPs per region? → 5 (default).  
5. Can NAT Gateway be in a private subnet? → No, it must be in a public subnet (has EIP).  
6. Default VPC CIDR? → `172.31.0.0/16`.  
7. Can you disable default VPC? → You can delete (uncommon); recreate via support.  
8. SG default inbound rule? → None (deny all).  
9. SG default outbound rule? → Allow all.  
10. Default NACL default rules? → Allow all (rule 100) in/out.  
11. Transit Gateway limit of VPC attachments per TGW? → 5,000 (default).  
12. Does TGW support multicast? → Yes.  
13. How many VIFs per DX port? → 50 per physical/LAG.  
14. VPC peering max? → 125 active (default).  
15. Can you peer the same two VPCs twice? → No.  
16. VPN tunnels per connection? → 2.  
17. Can you attach multiple IGWs to one VPC? → No, only 1.  
18. Gateway endpoint supported services? → S3, DynamoDB only.  
19. Can Flow Logs be enabled on a VPC with existing resources? → Yes.  
20. Latency guarantee within AZ placement group cluster? → Lowest, often <1 ms.  

---

## COMMON FOLLOW-UP "WHY / HOW" QUESTIONS TO PREP

- Why does AWS require 2 tunnels for VPN?  
- Why is peering non-transitive?  
- Why did AWS launch VPC Lattice when TGW exists?  
- Why can't you modify the local route (traditionally)?  
- Why does NAT GW need a public subnet?  
- Why is PrivateLink unidirectional?  
- Why does TGW cost more than peering?  
- Why is Gateway Endpoint free but Interface Endpoint paid?  
- Why does Lambda in VPC need at least 2 subnets for HA?  
- How would you design a multi-region globally-resilient VPC architecture for a bank?

---

## SUGGESTED 4-WEEK PREP (matches your 3–4 yr profile)

1. **Week 1** — Fundamentals, CIDR, subnets, RT, IGW, NAT (Q1–Q45)  
2. **Week 2** — SG/NACL, peering, TGW, VPN, DX (Q46–Q82)  
3. **Week 3** — Endpoints, PrivateLink, Flow Logs, DNS, IPAM, Firewall (Q83–Q140)  
4. **Week 4** — Scenario + troubleshooting + real-time + DevOps + draw architectures from memory (Q141–Q184)  
5. Daily: 1 whiteboard architecture (e.g., "design hub-and-spoke for 200 accounts").  
6. Hands-on: build a full lab — 2 VPCs peered + TGW + VPN simulation with strongSwan + endpoints + flow logs.  
7. Always close answers with: **cost, security, HA, observability, IaC**.

---

*End of Document – 184+ unique questions across all VPC modules. No duplication with EC2 bank. Good luck!*
