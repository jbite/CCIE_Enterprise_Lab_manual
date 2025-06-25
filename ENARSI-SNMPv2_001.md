## Lab Manual: Troubleshooting SNMPv2
## Lab No: SNMPv2-TS-001
Author: Jacky, copilot with AI
Date: 2025年6月25日
## Summary:
本實驗旨在引導您建立一個包含Cisco路由器和一個網路管理站（NMS）的小型網路環境，以實踐SNMPv2c的配置與故障排除。您將學習如何設定SNMP community strings、ACLs，並從NMS端發起SNMP查詢以驗證配置。實驗將模擬常見的配置錯誤，並指導您透過Cisco IOS的show和debug命令來診斷和解決問題，從而加深您對SNMPv2c工作原理和故障排除技巧的理解。

## Purpose:
- 配置SNMPv2c Community Strings與ACLs： 掌握在Cisco路由器上設定SNMPv2c read-only和read-write community strings，並應用標準ACLs來限制SNMP存取。

- 驗證SNMP代理功能與可達性： 從NMS發起SNMP GET和SET操作，確認SNMP代理（Agent）能夠正確響應查詢，並確保NMS與代理之間的路徑可達。

- 診斷常見SNMPv2c故障： 識別並解決因community string不匹配、ACL配置錯誤或路由問題導致的SNMP通訊失敗。

- 利用IOS調試工具進行故障排除： 熟練運用show snmp、debug snmp packet等命令來分析SNMP流量和代理狀態，精準定位故障點。

- 理解SNMP陷阱與資訊通知： 選擇性地配置並驗證SNMP陷阱或informs的發送，了解如何監控特定網路事件。

## Topology:
```
+----------------+        10.12.1.0/24         +----------------+
|                |-----------------------------|                |
|       R1       |Gi0/0                   Gi0/0|       R2       |
|   (SNMP Agent) |                             |   (Transit)    |
|                |                             |                |
+----------------+                             +----------------+
Loopback0: 1.1.1.1/32                                  Gi0/1
                                                         |
                                                         | 10.12.2.0/24
                                                         |
                                                 +----------------+
                                                 |      NMS       |
                                                 | (SNMP Manager) |
                                                 |                |
                                                 +----------------+
```
## Eve-NG Image Names Reference:

- R1, R2: Cisco IOSv (e.g., i86bi-linux-l3-adventerprisek9-ms.vmdk.SPA.156-2.T)

- NMS: Linux (e.g., Ubuntu Server or Tiny Core Linux with snmpd and snmp-utils installed, or a dedicated NMS appliance image)

## Technologies:
- SNMPv2c (Simple Network Management Protocol v2c)

- IP Addressing (IPv4)

- Static Routing (或OSPF/EIGRP for reachability)

- Access Control Lists (ACLs)

## Software:
- Cisco IOSv (用於R1和R2)

- Linux OS (用於NMS，需安裝snmpd和snmp-utils套件，例如snmpget, snmpwalk, snmpset)

## Configuration flow:
IP Addressing Scheme:
```
R1 Loopback0: 1.1.1.1/32

R1 GigabitEthernet0/0: 10.12.1.1/24

R2 GigabitEthernet0/0: 10.12.1.2/24

R2 GigabitEthernet0/1: 10.12.2.1/24

NMS Ethernet0: 10.12.2.10/24
```

R1 (SNMP Agent) Configuration:
!
hostname R1
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface GigabitEthernet0/0
 ip address 10.12.1.1 255.255.255.0
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 10.12.1.2 ! Default route to R2
!
! SNMPv2c Configuration
! Define an ACL to permit NMS access
ip access-list standard SNMP_NMS_ACCESS
 permit host 10.12.2.10
!
! Configure read-only community "public_ro" with ACL restriction
snmp-server community public_ro RO SNMP_NMS_ACCESS
!
! Configure read-write community "private_rw" with ACL restriction
snmp-server community private_rw RW SNMP_NMS_ACCESS
!
! Configure SNMP trap receiver (optional, for purpose 5)
snmp-server host 10.12.2.10 version 2c public_ro
!
! Enable SNMP traps for specific events (optional, for purpose 5)
snmp-server enable traps snmp authentication link-status coldstart warmstart
!
end
R2 (Transit) Configuration:
!
hostname R2
!
interface GigabitEthernet0/0
 ip address 10.12.1.2 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 ip address 10.12.2.1 255.255.255.0
 no shutdown
!
ip route 1.1.1.1 255.255.255.255 10.12.1.1 ! Route to R1's Loopback
ip route 0.0.0.0 0.0.0.0 10.12.2.10 ! Default route for NMS (or a more specific route)
!
end
NMS (SNMP Manager) Configuration (Generic Linux Host):
安裝SNMP工具：

Bash

sudo apt update
sudo apt install snmp snmp-mibs-downloader
sudo download-mibs
配置IP地址：

Bash

sudo ip address add 10.12.2.10/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 10.12.2.1
(注意：根據您的Linux發行版和網路設定工具，實際命令可能有所不同，例如使用netplan或NetworkManager)

Verification:
在NMS上執行以下命令來驗證SNMPv2c功能。

基本連通性測試：
在NMS上Ping R1的Loopback介面，確保路徑可達。

Bash

ping 1.1.1.1
在R1上Ping NMS的IP地址，確保雙向連通。

R1#ping 10.12.2.10
從NMS使用Read-Only Community進行SNMP GET操作：
獲取R1的系統描述 (OID: .1.3.6.1.2.1.1.1.0)。

Bash

snmpget -v 2c -c public_ro 1.1.1.1 .1.3.6.1.2.1.1.1.0
期望結果： 返回R1的IOS版本資訊。

從NMS使用Read-Only Community進行SNMPWALK操作：
遍歷R1的介面資訊 (OID: .1.3.6.1.2.1.2.2.1.2)。

Bash

snmpwalk -v 2c -c public_ro 1.1.1.1 .1.3.6.1.2.1.2.2.1.2
期望結果： 列出R1的所有介面名稱。

從NMS嘗試使用Read-Write Community進行SNMP SET操作：
嘗試修改R1的系統名稱 (OID: .1.3.6.1.2.1.1.5.0)。

Bash

snmpset -v 2c -c private_rw 1.1.1.1 .1.3.6.1.2.1.1.5.0 s "R1-NewHostname"
期望結果： 如果成功，R1的hostname將被修改。在R1上執行show running-config | include hostname驗證。

模擬故障排除場景：

場景一：Community String不匹配
在R1上修改public_ro community string為錯誤的值 (例如snmp-server community public_test RO SNMP_NMS_ACCESS)。然後在NMS上再次嘗試步驟2的snmpget命令。
期望結果： Timeout: No Response from 1.1.1.1 或 No Such Instance currently exists at this OID.
故障排除：
* 檢查R1的show running-config | include snmp-server community確認community string是否與NMS的設定一致。
* 在R1上使用debug snmp packet，然後從NMS發起查詢。觀察debug輸出是否顯示"bad community name"等錯誤資訊。
R1#debug snmp packet R1#terminal monitor

場景二：ACL錯誤
在R1的SNMP_NMS_ACCESS ACL中，將permit host 10.12.2.10修改為deny host 10.12.2.10。然後在NMS上再次嘗試步驟2的snmpget命令。
期望結果： Timeout: No Response from 1.1.1.1。
故障排除：
* 檢查R1的show access-lists SNMP_NMS_ACCESS確認ACL配置是否正確。
* 檢查show snmp community確認ACL是否正確應用。
* 使用debug ip packet detail在R1上查看是否有SNMP（UDP 161）流量被ACL阻擋。
R1#debug ip packet detail 161 R1#terminal monitor

場景三：路由不可達
在R2上移除到R1 Loopback0的靜態路由 (no ip route 1.1.1.1 255.255.255.255 10.12.1.1)。然後在NMS上再次嘗試步驟2的snmpget命令。
期望結果： Timeout: No Response from 1.1.1.1。
故障排除：
* 在NMS上traceroute 1.1.1.1，確認路由路徑。
* 在R1上show ip route，確認到NMS的回程路由。
* 在R2上show ip route，確認到R1 Loopback0和NMS的路由。
* 使用debug ip icmp在R1或R2上查看ICMP流量，判斷是否有不可達訊息。

清除Debug：
在完成故障排除後，務必使用以下命令清除所有調試：

R1#undebug all
R1#terminal no monitor