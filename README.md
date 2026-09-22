# NetShield-ISP-Secure-Redundant-Network-Simulation

A secure, scalable, and redundant ISP network simulation developed in Cisco Packet Tracer. The project implements dynamic routing using OSPF and EIGRP, VLAN segmentation, inter-VLAN routing, route redistribution, network security, and a redundant backup path for reliable connectivity.

## Project Overview

This project simulates an ISP-style network environment using Cisco routers and switches. The network is designed to provide dynamic routing, VLAN-based segmentation, secure access, and redundancy.

The project demonstrates how different routing protocols can operate together through route redistribution while maintaining secure and reliable communication between different networks.

## Objectives

- Design and simulate an ISP network topology.
- Implement VLAN-based network segmentation.
- Configure inter-VLAN routing using Router-on-a-Stick.
- Implement OSPF and EIGRP dynamic routing.
- Configure OSPF and EIGRP route redistribution.
- Implement ARP and ICMP communication.
- Configure ACLs for traffic control.
- Implement Port Security on access ports.
- Configure SSH for secure remote management.
- Provide a redundant backup path using floating static routes.
- Test network connectivity and failover.

## Network Devices

- 3 × Cisco 2911 Routers
- 4 × Cisco 2960 Switches
- 8 × PCs
- Cisco Packet Tracer

## Network Topology

### Routers

- **R0** – ISP Core Router
- **R1** – OSPF Network
- **R2** – EIGRP Network

### Routing Structure

`R1 → OSPF → R0 → EIGRP → R2`

A direct redundant link is also configured between R1 and R2 for backup connectivity.

## IP Addressing Plan

| Device | Interface | IP Address | Network |
|--------|-----------|------------|---------|
| R0     | G0/0      | 10.0.0.1/30| R0–R1   |
| R1     | G0/0      | 10.0.0.2/30| R0–R1   |
| R0     | G0/1      | 10.0.0.5/30| R0–R2   |
| R2     | G0/0      | 10.0.0.6/30| R0–R2   |
| R1     | G0/2      | 10.0.0.9/30| Backup Link |
| R2     | G0/2      | 10.0.0.10/30| Backup Link |

## VLAN Configuration

| VLAN | Name  | Network         | Gateway      |
|------|-------|-----------------|--------------|
| 10   | USERS | 192.168.10.0/24 | 192.168.10.1 |
| 20   | ADMIN | 192.168.20.0/24 | 192.168.20.1 |
| 30   | USERS | 192.168.30.0/24 | 192.168.30.1 |
| 40   | ADMIN | 192.168.40.0/24 | 192.168.40.1 |

### VLAN Distribution

- **SW1** → VLAN 10
- **SW2** → VLAN 20
- **SW3** → VLAN 30
- **SW4** → VLAN 40

## Inter-VLAN Routing

Inter-VLAN routing is implemented using **Router-on-a-Stick**. Router subinterfaces are configured with IEEE 802.1Q encapsulation.

**Example:**

```
interface g0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface g0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

The same method is used for VLAN 30 and VLAN 40 on R2.

## Trunking
802.1Q trunking is configured between routers and switches and between interconnected switches. Trunk ports carry multiple VLANs over a single physical link.

Example:

```
interface fa0/24
 switchport mode trunk
```

## Dynamic Routing

### OSPF
OSPF is implemented between R1 and the ISP Core Router R0. OSPF is used to dynamically exchange routing information and calculate the best available path.

```
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 ```
### EIGRP
EIGRP is implemented between R0 and R2.

```
router eigrp 100
 network 10.0.0.4 0.0.0.3
 network 192.168.30.0 0.0.0.255
 network 192.168.40.0 0.0.0.255
 no auto-summary
```

## Route Redistribution
Since OSPF and EIGRP are used in different parts of the network, route redistribution is configured on the Core Router R0. This allows routes learned through one routing protocol to be advertised into the other routing domain.

### OSPF → EIGRP:

```
router eigrp 100
 redistribute ospf 1 metric 10000 100 255 1 1500
```

### EIGRP → OSPF:

```
router ospf 1
 redistribute eigrp 100 subnets
 ```

## Redundancy & Failover
A direct backup connection is configured between R1 and R2:

R1 G0/2 → 10.0.0.9/30

R2 G0/2 → 10.0.0.10/30

Floating static routes are configured with an Administrative Distance of 200.

### R1:

```
ip route 192.168.30.0 255.255.255.0 10.0.0.10 200
ip route 192.168.40.0 255.255.255.0 10.0.0.10 200
```

### R2:

```
ip route 192.168.10.0 255.255.255.0 10.0.0.9 200
ip route 192.168.20.0 255.255.255.0 10.0.0.9 200
```
The higher Administrative Distance keeps these routes as backup routes. If the primary path becomes unavailable, the floating static route can be used as the alternate path.

## Access Control List (ACL)
An extended ACL is configured on the Core Router R0 to control ICMP traffic. The ACL blocks ICMP traffic from VLAN 10 to VLAN 30.

```
access-list 100 deny icmp 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 100 permit ip any any

interface g0/1
 ip access-group 100 out
```
This demonstrates traffic filtering between different network segments.

## Port Security
Port Security is configured on PC-facing access ports of all switches.
Configuration includes:

- Maximum 1 MAC address per port

- Sticky MAC address learning

- Shutdown on security violation

Example:

```
interface range fa0/2-3
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
```

## SSH Remote Management
SSH is configured for secure remote management of network devices.
Configuration includes:

- Local username authentication

- RSA key generation

- SSH Version 2

- VTY lines restricted to SSH

Example:

```
hostname R0
ip domain-name netshield.local
username admin privilege 15 secret <PASSWORD>

crypto key generate rsa
ip ssh version 2

line vty 0 4
 login local
 transport input ssh
```
The actual lab password should not be uploaded to GitHub. Use a placeholder such as <PASSWORD> in the public repository.

## ARP & ICMP

### ARP
ARP is used to resolve IPv4 addresses into MAC addresses within the local network.
Example verification:

```
arp -a
ICMP
ICMP is used for network connectivity and troubleshooting.
```

Example:

```
ping 192.168.20.10
```
ICMP traffic is also demonstrated through Cisco Packet Tracer Simulation Mode.

## Network Testing & Verification
The following commands were used to verify the network:

- VLAN Verification: show vlan brief

- Trunk Verification: show interfaces trunk

- Interface Status: show ip interface brief

- Routing Table: show ip route

- OSPF Neighbors: show ip ospf neighbor

- EIGRP Neighbors: show ip eigrp neighbors

- OSPF Routes: show ip route ospf

- EIGRP Routes: show ip route eigrp

- ACL Verification: show access-lists

- Port Security: show port-security, show port-security interface fa0/2

- SSH Verification: show ip ssh, show running-config | section line vty

## Connectivity Testing
The network was tested using:

- Same-VLAN connectivity

- Inter-VLAN connectivity

- Cross-routing-domain connectivity

- ARP table verification

- ICMP ping testing

- ACL traffic filtering

- Port Security verification

- SSH remote access

- Redundant path failover testing

- Cisco Packet Tracer Simulation Mode

## Cable Types

| Connection      |  Cable                  |
|-----------------|-------------------------|
| Router ↔ Router | Copper Cross-Over       |
| Router ↔ Switch | Copper Straight-Through |
| Switch ↔ Switch | Copper Cross-Over       |
| Switch ↔ PC     | Copper Straight-Through |

## Technologies Used

- Cisco Packet Tracer

- Cisco IOS

- IPv4 & Subnetting

- VLANs & 802.1Q Trunking

- Router-on-a-Stick (Inter-VLAN Routing)

- OSPF & EIGRP

- Route Redistribution

- Floating Static Routes

- Access Control Lists (ACL)

- Port Security

- SSH, ARP, ICMP


