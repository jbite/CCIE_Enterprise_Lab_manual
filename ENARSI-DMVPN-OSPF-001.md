## Lab Manual: DMVPN with OSPF
Name: DMVPN with OSPF

Lab No: ENARSI-DMVPN-OSPF-001

Author: Jacky, copilot with AI

Date: June 28, 2025

## Summary: 
This lab provides a hands-on experience in configuring and verifying a Dynamic Multipoint VPN (DMVPN) network integrated with OSPF as the routing protocol. It focuses on establishing secure, scalable connectivity between a central hub and multiple spokes, using Phase 2 DMVPN for direct spoke-to-spoke communication.

## Purpose:

- To successfully configure a DMVPN Phase 2 network, including NHRP and GRE tunnels, between a hub and multiple spoke routers.

- To integrate OSPF as the routing protocol over the DMVPN tunnel, ensuring proper neighbor adjacencies and route propagation.

- To verify spoke-to-spoke direct tunnel establishment and traffic flow over the DMVPN network.

- To demonstrate the scalability and dynamic nature of DMVPN in an OSPF environment.

- To troubleshoot common DMVPN and OSPF configuration issues.

## Topology:
```
      +---------------------+
      |       Internet      |
      +---------------------+
        |    |   |       |
        |    |   |       |
+---------+  |   |  +------------+
| R1 (Hub)|  |   |  | R2 (Spoke) |
+---------+  |   |  +------------+
             |   |     
             |   |     
             |   +---------------+
             |                   |
             |                   |
      +-----------+           +------------+
      | R3 (Spoke)|           | R4 (Spoke) |
      +-----------+           +------------+
```
## EVE-NG Image Names:

- R1, R2, R3, R4: iol/unl/Cisco-IOU L3 - i86_64

## Technologies:

- DMVPN (Dynamic Multipoint VPN)
- GRE (Generic Routing Encapsulation)
- NHRP (Next Hop Resolution Protocol)
- OSPF (Open Shortest Path First)
- IPSec (Internet Protocol Security) - implicit with DMVPN for encryption (Phase 2)

## Software:

- EVE-NG Community Edition (or Professional)
- Cisco IOSv (or IOL) Images for Routers
- SecureCRT, PuTTY, or other SSH/Telnet client
- Configuration Flow:

## Initial Setup (All Routers):

- Configure basic hostname and enable secret.
- Configure loopback interfaces for router IDs and simulated LANs (e.g., Loopback0: 10.0.0.X/32 where X is router number).
- Configure external facing interfaces with IP addresses in the 10.12.1.0/24 subnet (between R1-R2), 10.12.2.0/24 (between R1-R3), and 10.12.3.0/24 (between R1-R4) as per the topology.
- Configure default routes pointing to a simulated internet or a designated next hop for reachability.

## R1 (Hub) Configuration:
```
interface Tunnel0
 ip address 172.16.0.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 100
 tunnel source GigabitEthernet0/0  (or appropriate physical interface)
 tunnel mode gre multipoint
 tunnel key 123
 tunnel protection ipsec profile DMVPN_PROFILE

crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key cisco address 0.0.0.0 0.0.0.0

crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile DMVPN_PROFILE
 set transform-set TSET

router ospf 1
 router-id 1.1.1.1
 network 172.16.0.0 0.0.0.255 area 0
 network 10.0.0.1 0.0.0.0 area 0
```

## R2 (Spoke) Configuration:
```
interface Tunnel0
 ip address 172.16.0.2 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map 172.16.0.1 10.12.1.1  (Hub's Tunnel IP and Public IP)
 ip nhrp map multicast 10.12.1.1
 ip nhrp network-id 100
 ip nhrp nhs 172.16.0.1
 tunnel source GigabitEthernet0/0 (or appropriate physical interface)
 tunnel mode gre multipoint
 tunnel key 123
 tunnel protection ipsec profile DMVPN_PROFILE

crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key cisco address 10.12.1.1

crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile DMVPN_PROFILE
 set transform-set TSET

router ospf 1
 router-id 2.2.2.2
 network 172.16.0.0 0.0.0.255 area 0
 network 10.0.0.2 0.0.0.0 area 0
```

## R3 (Spoke) Configuration:

```
interface Tunnel0
 ip address 172.16.0.3 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map 172.16.0.1 10.12.2.1
 ip nhrp map multicast 10.12.2.1
 ip nhrp network-id 100
 ip nhrp nhs 172.16.0.1
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel key 123
 tunnel protection ipsec profile DMVPN_PROFILE

crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key cisco address 10.12.2.1

crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile DMVPN_PROFILE
 set transform-set TSET

router ospf 1
 router-id 3.3.3.3
 network 172.16.0.0 0.0.0.255 area 0
 network 10.0.0.3 0.0.0.0 area 0
```
### R4 (Spoke) Configuration:
```
interface Tunnel0
 ip address 172.16.0.4 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map 172.16.0.1 10.12.3.1
 ip nhrp map multicast 10.12.3.1
 ip nhrp network-id 100
 ip nhrp nhs 172.16.0.1
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel key 123
 tunnel protection ipsec profile DMVPN_PROFILE

crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key cisco address 10.12.3.1

crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile DMVPN_PROFILE
 set transform-set TSET

router ospf 1
 router-id 4.4.4.4
 network 172.16.0.0 0.0.0.255 area 0
 network 10.0.0.4 0.0.0.0 area 0
```

Verification:

Verify DMVPN Tunnel Status:
```
show dmvpn (on all routers)

show ip nhrp (on all routers)

show crypto isakmp sa (on all routers)

show crypto ipsec sa (on all routers)

Verify OSPF Neighbor Adjacencies:

show ip ospf neighbor (on all routers - look for full adjacencies on the tunnel interface)

Verify IP Routing Table:

show ip route ospf (on all routers - ensure all loopback networks are learned)

ping 10.0.0.X (from R2 to R3, R2 to R4, R3 to R4, and all spokes to R1's loopback, etc. to test full reachability)
```
## Verify Spoke-to-Spoke Tunnels (Triggering):
From R2, ping 10.0.0.3 (R3's loopback). Then, from R2, run show dmvpn and observe a direct tunnel entry to R3's public IP. Repeat for other spoke-to-spoke pings.

Troubleshooting (if needed):
```
debug ip nhrp

debug crypto isakmp

debug crypto ipsec

debug ip ospf adj

show interface Tunnel0 (check for errors, MTU issues)
```