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
 ip nhrp redirect
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel key 123
 tunnel protection ipsec profile DMVPN_PROFILE
 !Because horizon split rule, need to set this prevent interface keep flapping
 ip ospf network point-to-multipoint

crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key cisco address 0.0.0.0

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
 !  (Hub's Tunnel IP and Public IP)
 ip nhrp map 172.16.0.1 10.12.1.1
 ip nhrp map multicast 10.12.1.1
 ip nhrp network-id 100
 ip hnrp shortcut
 ip nhrp nhs 172.16.0.1
 ! (or appropriate physical interface)
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel key 123
 tunnel protection ipsec profile DMVPN_PROFILE

crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key cisco address 0.0.0.0

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
 ip hnrp shortcut
 ip nhrp nhs 172.16.0.1
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel key 123
 tunnel protection ipsec profile DMVPN_PROFILE

crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key cisco address 0.0.0.0

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
 ip hnrp shortcut
 ip nhrp nhs 172.16.0.1
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
 tunnel key 123
 tunnel protection ipsec profile DMVPN_PROFILE

crypto isakmp policy 10
 encr aes 256
 authentication pre-share
 group 5
crypto isakmp key cisco address 0.0.0.0

crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile DMVPN_PROFILE
 set transform-set TSET

router ospf 1
 router-id 4.4.4.4
 network 172.16.0.0 0.0.0.255 area 0
 network 10.0.0.4 0.0.0.0 area 0
```

## Verification:

Verify DMVPN Tunnel Status:
```
show dmvpn (on all routers)
R1#sh dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel0, IPv4 NHRP Details
Type:Hub, NHRP Peers:3,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 10.12.1.2            172.16.0.2    UP 00:01:02     D
     1 10.13.1.2            172.16.0.3    UP 00:01:15     D
     1 10.14.1.2            172.16.0.4    UP 00:01:46     D
```
```
show ip nhrp (on all routers)
R1#sh ip nhrp
172.16.0.2/32 via 172.16.0.2
   Tunnel0 created 00:01:43, expire 01:58:17
   Type: dynamic, Flags: unique registered used
   NBMA address: 10.12.1.2
172.16.0.3/32 via 172.16.0.3
   Tunnel0 created 00:01:55, expire 01:58:05
   Type: dynamic, Flags: unique registered used
   NBMA address: 10.13.1.2
172.16.0.4/32 via 172.16.0.4
   Tunnel0 created 00:02:27, expire 01:57:33
   Type: dynamic, Flags: unique registered used
   NBMA address: 10.14.1.2
```
```
show crypto isakmp sa (on all routers)
R1#sh crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
10.11.1.2       10.14.1.2       QM_IDLE           1002 ACTIVE
10.11.1.2       10.13.1.2       QM_IDLE           1001 ACTIVE
10.11.1.2       10.12.1.2       QM_IDLE           1003 ACTIVE

IPv6 Crypto ISAKMP SA
```
```
show crypto ipsec sa (on all routers)
R1#sh crypto ipsec sa

interface: Tunnel0
    Crypto map tag: Tunnel0-head-0, local addr 10.11.1.2

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (10.11.1.2/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (10.14.1.2/255.255.255.255/47/0)
   current_peer 10.14.1.2 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 42, #pkts encrypt: 42, #pkts digest: 42
    #pkts decaps: 62, #pkts decrypt: 62, #pkts verify: 62
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0
```

### Verify OSPF Neighbor Adjacencies:
```
show ip ospf neighbor (on all routers - look for full adjacencies on the tunnel interface)
R1#sh ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
5.5.5.5           1   FULL/DR         00:00:36    10.11.1.1       Ethernet1/0

```
Verify IP Routing Table:
```
show ip route ospf (on all routers - ensure all loopback networks are learned)
R1#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
O        10.12.1.0/30 [110/20] via 10.11.1.1, 00:15:21, Ethernet1/0
O        10.13.1.0/30 [110/20] via 10.11.1.1, 00:15:21, Ethernet1/0
O        10.14.1.0/30 [110/20] via 10.11.1.1, 00:15:21, Ethernet1/0
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