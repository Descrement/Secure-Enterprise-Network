# Secure-Enterprise-Network

Two-site enterprise network (Head Office and Branch Office) built in GNS3 with Cisco IOSv and IOSvL2.

## Objective
Design and build a small enterprise network that connects a Head Office to a Branch Office, with segmented VLANs, centralized DHCP and DNS, internet access through NAT, and basic security.

## Topology
![Network topology](topology/topology.png)

## Technologies
VLANs, 802.1Q trunking, inter-VLAN routing (router-on-a-stick), DHCP relay, NAT/PAT, [OSPF], [ACLs], [SSH], [port security]

## Tools
GNS3, Cisco IOSv 15.9(3)M6, IOSvL2 [version], VPCS, Docker (server)

## VLAN Plan
| VLAN | Name | Subnet | Site |
|------|------|--------|------|
| 10 | Server | 10.1.10.0/24 | HQ |
| 20 | Staff | 10.1.20.0/24 | HQ |
| 99 | Management | 10.1.99.0/24 | HQ |
| 30 | Staff | 10.2.30.0/24 | Branch |
| 99 | Management | 10.2.99.0/24 | Branch |

## IP Address Table
| Device | Interface | IP Address | Prefix | Notes |
|--------|-----------|------------|--------|-------|
| R1 | Gi0/0 | DHCP | - | Outside (NAT1) |
| R1 | Gi0/1.10 | 10.1.10.1 | /24 | VLAN 10 gateway |
| R1 | Gi0/1.20 | 10.1.20.1 | /24 | VLAN 20 gateway |
| R1 | Gi0/1.99 | 10.1.99.1 | /24 | VLAN 99 gateway |
| R1 | Gi0/2 | 10.0.0.1 | /30 | WAN to R2 |
| R2 | Gi0/2 | 10.0.0.2 | /30 | WAN to R1 |
| R2 | Gi0/0 | 10.2.30.1 | /24 | VLAN 30 gateway |
| R2 | Gi0/2.99 | 10.2.99.1 | /24 | VLAN 99 gateway |
| SW1 | Vlan99 | 10.1.99.2 | /24 | Management |
| SW2 | Vlan99 | 10.2.99.2 | /24 | Management |
| Server | eth0 | 10.1.10.10 | /24 | DHCP and DNS |
| Admin | e0 | 10.1.99.10 | /24 | Management PC |
| PC1-PC4 | e0 | DHCP | - | From Server |

## How I Built It
1. Set hostnames and IP addresses
2. Created VLANs and access ports
3. Configured the trunk and inter-VLAN routing
4. [Configured NAT/PAT on R1]
5. [Configured DHCP and DNS with relay]
6. [Configured OSPF]
7. [Applied ACLs and hardening]

## Verification
Add a short caption and a screenshot for each:
- `show vlan brief` and `show interfaces trunk`
- `show ip interface brief`
- `show ip route`
- `show ip nat translations`
- Ping tests between sites and to the internet

## Troubleshooting
| Symptom | Cause | Fix |
|---------|-------|-----|
| [what you saw] | [root cause] | [what fixed it] |

## What I Learned
- [Short, specific points]

## Repository Structure
- `topology/` diagram
- `configs/` running configs for each device
- `screenshots/` verification output
