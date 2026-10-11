# Secure-Enterprise-Network

Two-site enterprise network (Head Office and Branch Office) built in GNS3 with Cisco IOSv and IOSvL2.

## Objective
Design and build a small enterprise network that connects a Head Office to a Branch Office, with segmented VLANs, centralized DHCP and DNS, internet access through NAT, and basic security.

## Topology
![Network topology](topology/topology.png)

## Technologies
- **Done:** VLANs, 802.1Q trunking, inter-VLAN routing (router-on-a-stick)
- **In progress:** [DHCP relay], [NAT/PAT], [OSPF], [ACLs], [SSH], [port security]

## Tools
GNS3, Cisco IOSv 15.9(3)M6, Cisco IOSvL2 [version], VPCS, Docker (server)

## Project Status
- [x] Hostnames, IP addresses, DNS lookup disabled
- [x] VLANs and access ports
- [x] Trunk and inter-VLAN routing
- [ ] NAT/PAT on R1
- [ ] DHCP and DNS with relay
- [ ] OSPF
- [ ] ACLs and hardening

## VLAN Plan
| VLAN | Name | Subnet | Site |
|------|------|--------|------|
| 10 | Server | 10.1.10.0/24 | HQ |
| 20 | Staff | 10.1.20.0/24 | HQ |
| 99 | Management | 10.1.99.0/24 | HQ |
| 30 | Staff | 10.2.30.0/24 | Branch |
| 99 | Management | 10.2.99.0/24 | Branch |

## IP Address Table
| Device | Interface | IP Address | Prefix | Default Gateway | Notes |
|--------|-----------|------------|--------|-----------------|-------|
| R1 | Gi0/0 | DHCP | - | - | Outside (NAT1) |
| R1 | Gi0/1 | no IP | - | - | 802.1Q trunk to SW1 (VLAN 10, 20, 99) |
| R1 | Gi0/1.10 | 10.1.10.1 | /24 | - | VLAN 10 gateway |
| R1 | Gi0/1.20 | 10.1.20.1 | /24 | - | VLAN 20 gateway |
| R1 | Gi0/1.99 | 10.1.99.1 | /24 | - | VLAN 99 gateway |
| R1 | Gi0/2 | 10.0.0.1 | /30 | - | WAN to R2 |
| R2 | Gi0/2 | 10.0.0.2 | /30 | - | WAN to R1 |
| R2 | Gi0/0 | no IP | - | - | 802.1Q trunk to SW2 (VLAN 30, 99) |
| R2 | Gi0/0.30 | 10.2.30.1 | /24 | - | VLAN 30 gateway |
| R2 | Gi0/0.99 | 10.2.99.1 | /24 | - | VLAN 99 gateway |
| SW1 | Vlan99 | 10.1.99.2 | /24 | 10.1.99.1 | Management |
| SW2 | Vlan99 | 10.2.99.2 | /24 | 10.2.99.1 | Management |
| Server | eth0 | 10.1.10.10 | /24 | 10.1.10.1 | DHCP and DNS |
| Admin | e0 | 10.1.99.10 | /24 | 10.1.99.1 | Management PC |
| PC1-PC4 | e0 | DHCP | - | From DHCP | Addresses from the Server |

## How I Built It

### 1. Hostnames, IP addresses and DNS lookup

**Hostname** (repeat on every router and switch with its own name):

```
enable
configure terminal
hostname R1
end
```

- `enable` enters privileged EXEC mode (the prompt ends with `#`).
- `configure terminal` enters global configuration mode.
- `hostname R1` sets the device name.
- `end` returns to privileged EXEC mode.

![Hostname set on R1](screenshots/01-hostname.png)

**IP address** (example: the R1 WAN link; repeat for every routed interface in the table):

```
enable
configure terminal
interface gigabitethernet 0/2
ip address 10.0.0.1 255.255.255.252
no shutdown
exit
```

- `interface gigabitethernet 0/2` selects the interface to configure.
- `ip address` sets the IP address and subnet mask.
- `no shutdown` brings the interface up.
- `exit` returns to global configuration mode.

![R1 Gi0/2 address](screenshots/02-ip-address.png)

**Disable DNS lookup** (stops the CLI from trying to resolve a mistyped command as a hostname):

```
enable
configure terminal
no ip domain-lookup
end
write memory
```

`write memory` saves the running config (in RAM) to the startup config (in NVRAM), so it survives a reboot.

![DNS lookup disabled](screenshots/03-disable-dns.png)

### 2. VLANs and access ports
Created VLANs 10, 20 and 99 on SW1 and VLANs 30 and 99 on SW2, named them, and assigned each access port to its VLAN.

**Create a VLAN** (example: VLAN 30 on SW2; repeat for each VLAN on each switch):

```
enable
configure terminal
vlan 30
name Staff
end
```

- `vlan 30` creates the VLAN and enters VLAN configuration mode.
- `name Staff` gives the VLAN a name.
- `end` returns to privileged EXEC mode.

![VLANs created](screenshots/04-vlan-created.png)

**Assign an access port to a VLAN** (example: SW1 Gi0/1, the server port; repeat for each access port in the VLAN plan):

```
enable
configure terminal
interface gigabitethernet 0/1
switchport mode access
switchport access vlan 30
end
write memory
```

- `switchport mode access` makes the port an access port (one VLAN, for end devices).
- `switchport access vlan 30` puts the port in VLAN 10.
- `write memory` saves the configuration.

![Ports assigned to VLANs](screenshots/05-vlan-ports.png)

### 3. Trunk and inter-VLAN routing
Made the router-facing switch ports (SW1 Gi0/0 and SW2 Gi0/0) 802.1Q trunks. R1 Gi0/1 and R2 Gi0/0 have no IP address of their own and carry one sub-interface per VLAN (router-on-a-stick).

![show interfaces trunk](screenshots/06-trunk.png)

Full configs for each device are in `configs/`.

### 4. [NAT/PAT on R1]
### 5. [DHCP and DNS with relay]
### 6. [OSPF]
### 7. [ACLs and hardening]

## Verification
| Check | Command | Screenshot |
|-------|---------|------------|
| VLANs and trunks | `show vlan brief`, `show interfaces trunk` | [todo] |
| Interface status | `show ip interface brief` | [todo] |
| Routing table | `show ip route` | [todo] |
| NAT translations | `show ip nat translations` | [todo] |
| Connectivity | Ping between sites and to the internet | [todo] |

## Troubleshooting
| Symptom | Cause | Fix |
|---------|-------|-----|
| [what you saw] | [root cause] | [what fixed it] |

## What I Learned
- [Short, specific points]

## Repository Structure
- `topology/` network diagram
- `configs/` running configs for each device
- `screenshots/` verification output

Passwords and Cisco images are not included in this repository.
