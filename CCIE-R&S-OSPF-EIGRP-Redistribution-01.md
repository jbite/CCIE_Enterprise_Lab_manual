**Lab No**: CCIE-R&S-OSPF-EIGRP-Redistribution-01
**Author**: Jacky, copilot with AI
**Date**: 2024-05-02
## **Summary**: This lab simulates a common scenario in the financial industry where a company's network is undergoing a merger or acquisition. It requires the seamless integration of two disparate routing domains—one running OSPF and the other EIGRP—while maintaining granular control over route advertisements using route maps to meet specific security and operational policies.
## Purpose:

- Establish basic OSPF and EIGRP routing domains.
- Configure mutual redistribution between OSPF and EIGRP.
- Implement a route map to control which routes are redistributed from OSPF into EIGRP.
- Implement a separate route map to control which routes are redistributed from EIGRP into OSPF.
- Verify the successful and policy-compliant propagation of routes across the two routing protocols.

## Topology:
```
      +------------+                +------------+        +--------------+
      |    R1      |                |     R2     |        |     R3       |
      | OSPF Domain|----------------| OSPF/EIGRP |--------| EIGRP Domain |
      | Loopback0  | 10.12.1.0/24   |    ASBR    |        | 10.23.1.0/24 |
      | 1.1.1.1/32 |                |            |        | Loopback0    |
      +------------+                +------------+        +--------------+
            |                              |                     |
      Loopback1-4                          |               Loopback1-4
      172.16.1.1/24                        |               192.168.1.1/24
      172.16.2.1/24                        |               192.168.2.1/24
      172.16.3.1/24                        |               192.168.3.1/24
      172.16.4.1/24                        |               192.168.4.1/24
```
## Technologies:

- OSPFv2
- EIGRP (Named Mode)
- Route Redistribution
- Route Map
- Access Control Lists (ACLs)

## Software:
- Cisco IOSv (or any modern Cisco IOS image)

## Configuration Flow:
**R1** Configuration
```
!
hostname R1
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface Loopback1
 ip address 172.16.1.1 255.255.255.0
!
interface Loopback2
 ip address 172.16.2.1 255.255.255.0
!
interface Loopback3
 ip address 172.16.3.1 255.255.255.0
!
interface Loopback4
 ip address 172.16.4.1 255.255.255.0
!
interface GigabitEthernet0/0
 ip address 10.12.1.1 255.255.255.0
!
router ospf 1
 router-id 1.1.1.1
 network 1.1.1.1 0.0.0.0 area 0
 network 10.12.1.0 0.0.0.255 area 0
 network 172.16.1.0 0.0.0.255 area 0
 network 172.16.2.0 0.0.0.255 area 0
 network 172.16.3.0 0.0.0.255 area 0
 network 172.16.4.0 0.0.0.255 area 0
!
end
```
**R3 Configuration**
```
!
hostname R3
!
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
!
interface Loopback1
 ip address 192.168.1.1 255.255.255.0
!
interface Loopback2
 ip address 192.168.2.1 255.255.255.0
!
interface Loopback3
 ip address 192.168.3.1 255.255.255.0
!
interface Loopback4
 ip address 192.168.4.1 255.255.255.0
!
interface GigabitEthernet0/0
 ip address 10.23.1.3 255.255.255.0
!
router eigrp financial-services
 !
 address-family ipv4 unicast autonomous-system 100
  network 3.3.3.3 0.0.0.0
  network 10.23.1.0 0.0.0.255
  network 192.168.1.0
  network 192.168.2.0
  network 192.168.3.0
  network 192.168.4.0
 exit-address-family
!
end
```
**R2 Configuration**
```
!
hostname R2
!
interface GigabitEthernet0/0
 ip address 10.12.1.2 255.255.255.0
!
interface GigabitEthernet0/1
 ip address 10.23.1.2 255.255.255.0
!
router ospf 1
 router-id 2.2.2.2
 network 10.12.1.0 0.0.0.255 area 0
 redistribute eigrp 100 subnets route-map EIGRP-to-OSPF-MAP
!
router eigrp financial-services
 !
 address-family ipv4 unicast autonomous-system 100
  network 10.23.1.0 0.0.0.255
  redistribute ospf 1 metric 10000 100 255 1 1500 route-map OSPF-to-EIGRP-MAP
 exit-address-family
!
ip access-list standard OSPF-to-EIGRP-ACL
 permit 172.16.1.0 0.0.0.255
!
ip access-list standard EIGRP-to-OSPF-ACL
 permit 192.168.1.0 0.0.0.255
!
route-map OSPF-to-EIGRP-MAP deny 10
 match ip address OSPF-to-EIGRP-ACL
!
route-map OSPF-to-EIGRP-MAP permit 20
!
route-map EIGRP-to-OSPF-MAP deny 10
 match ip address EIGRP-to-OSPF-ACL
!
route-map EIGRP-to-OSPF-MAP permit 20
!
end
```

## Verification:
R2
```
R2#sh ip route ospf | begin Gateway
Gateway of last resort is not set

      1.0.0.0/32 is subnetted, 1 subnets
O        1.1.1.1 [110/11] via 10.12.1.1, 00:55:20, Ethernet0/1
      172.16.0.0/32 is subnetted, 4 subnets
O        172.16.1.1 [110/11] via 10.12.1.1, 00:55:20, Ethernet0/1
O        172.16.2.1 [110/11] via 10.12.1.1, 00:55:20, Ethernet0/1
O        172.16.3.1 [110/11] via 10.12.1.1, 00:55:20, Ethernet0/1
O        172.16.4.1 [110/11] via 10.12.1.1, 00:55:20, Ethernet0/1

R2#sh ip route eigrp | begin Gateway
Gateway of last resort is not set

      3.0.0.0/32 is subnetted, 1 subnets
D        3.3.3.3 [90/409600] via 10.23.1.3, 00:43:51, Ethernet0/0
D     192.168.1.0/24 [90/409600] via 10.23.1.3, 00:20:58, Ethernet0/0
D     192.168.2.0/24 [90/409600] via 10.23.1.3, 00:20:55, Ethernet0/0
D     192.168.3.0/24 [90/409600] via 10.23.1.3, 00:20:53, Ethernet0/0
D     192.168.4.0/24 [90/409600] via 10.23.1.3, 00:20:51, Ethernet0/0

R2#sh ip ospf database

            OSPF Router with ID (10.12.1.2) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.12.1.2       10.12.1.2       328         0x80000005 0x00EEB8 2
172.16.4.1      172.16.4.1      1406        0x80000006 0x00DA3C 6

                Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.12.1.1       172.16.4.1      1406        0x80000002 0x005034

                Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
3.3.3.3         10.12.1.2       989         0x80000001 0x0008FA 120
10.23.1.0       10.12.1.2       989         0x80000001 0x00EFFC 120
192.168.2.0     10.12.1.2       989         0x80000001 0x00CAD8 120
192.168.3.0     10.12.1.2       989         0x80000001 0x00BFE2 120
192.168.4.0     10.12.1.2       989         0x80000001 0x00B4EC 120

R2#sh ip eigrp topology
EIGRP-IPv4 Topology Table for AS(1)/ID(2.2.2.2)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status

P 192.168.3.0/24, 1 successors, FD is 409600
        via 10.23.1.3 (409600/128256), Ethernet0/0
P 192.168.2.0/24, 1 successors, FD is 409600
        via 10.23.1.3 (409600/128256), Ethernet0/0
P 10.12.1.0/24, 1 successors, FD is 281600
        via Connected, Ethernet0/1
P 172.16.4.1/32, 1 successors, FD is 256256
        via Redistributed (256256/0)
P 192.168.1.0/24, 1 successors, FD is 409600
        via 10.23.1.3 (409600/128256), Ethernet0/0
P 2.2.2.2/32, 1 successors, FD is 256256
        via Redistributed (256256/0)
P 192.168.4.0/24, 1 successors, FD is 409600
        via 10.23.1.3 (409600/128256), Ethernet0/0
P 172.16.2.1/32, 1 successors, FD is 256256
        via Redistributed (256256/0)
P 172.16.3.1/32, 1 successors, FD is 256256
        via Redistributed (256256/0)
P 3.3.3.3/32, 1 successors, FD is 409600
        via 10.23.1.3 (409600/128256), Ethernet0/0
P 1.1.1.1/32, 1 successors, FD is 256256
        via Redistributed (256256/0)
P 10.23.1.0/24, 1 successors, FD is 281600
        via Connected, Ethernet0/0

R2#ping 1.1.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R2#ping 3.3.3.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
R1
```
R1#sh ip route 192.168.1.0
% Network not in table
R1#sh ip route 192.168.2.0
Routing entry for 192.168.2.0/24
  Known via "ospf 1", distance 110, metric 20
  Tag 120, type extern 2, forward metric 10
  Last update from 10.12.1.2 on Ethernet0/0, 00:23:41 ago
  Routing Descriptor Blocks:
  * 10.12.1.2, from 10.12.1.2, 00:23:41 ago, via Ethernet0/0
      Route metric is 20, traffic share count is 1
      Route tag 120
R1#sh ip route 192.168.3.0
Routing entry for 192.168.3.0/24
  Known via "ospf 1", distance 110, metric 20
  Tag 120, type extern 2, forward metric 10
  Last update from 10.12.1.2 on Ethernet0/0, 00:23:45 ago
  Routing Descriptor Blocks:
  * 10.12.1.2, from 10.12.1.2, 00:23:45 ago, via Ethernet0/0
      Route metric is 20, traffic share count is 1
      Route tag 120
```

R3
```
R3#sh ip route eigrp 1
Gateway of last resort is not set

      1.0.0.0/32 is subnetted, 1 subnets
D EX     1.1.1.1 [170/281856] via 10.23.1.2, 00:50:05, Ethernet0/1
      2.0.0.0/32 is subnetted, 1 subnets
D EX     2.2.2.2 [170/281856] via 10.23.1.2, 00:48:53, Ethernet0/1
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
D        10.12.1.0/24 [90/307200] via 10.23.1.2, 00:53:36, Ethernet0/1
      172.16.0.0/32 is subnetted, 3 subnets
D EX     172.16.2.1 [170/281856] via 10.23.1.2, 00:50:05, Ethernet0/1
D EX     172.16.3.1 [170/281856] via 10.23.1.2, 00:50:05, Ethernet0/1
D EX     172.16.4.1 [170/281856] via 10.23.1.2, 00:50:05, Ethernet0/1
```

## Analysis:

- On R3, verify that routes for 172.16.1.0/24 and 172.16.2.0/24 are present as D EX routes.
- On R3, verify that routes for 172.16.3.0/24 and 172.16.4.0/24 are not present in the routing table.
- On R1, verify that routes for 192.168.1.0/24 and 192.168.3.0/24 are present as O E2 routes.
- On R1, verify that routes for 192.168.2.0/24 and 192.168.4.0/24 are not present in the routing table.
- Ping from R1's loopbacks to R3's loopbacks to confirm end-to-end connectivity for the allowed routes. For example, ping from 172.16.1.1 to 192.168.1.1.