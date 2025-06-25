名稱: PBR (Policy-Based Routing) 實作與驗證



實驗手冊 (Lab Manual)
實驗編號 (Lab No): ENARSI-PBR-001

作者 (Author): CCIE企業訓練導師
日期 (Date): 2025年6月22日
## 摘要:
本實驗旨在引導學生配置和驗證策略路由 (PBR) 的核心概念與功能。PBR允許網路管理員基於比傳統路由表 (RIB) 更豐富的條件 (例如源IP地址、目的IP地址、協議類型、端口號等) 來轉發數據包，從而實現對特定流量路徑的精確控制。透過本實驗，您將學習如何定義流量匹配條件、設定策略路由動作、將策略應用於接口，並觀察PBR如何影響數據包的轉發路徑，使其跳脫傳統RIB的限制。

## 目的 (Purpose):
- 理解並配置基本的策略路由 (PBR) 功能。
- 掌握如何使用標準與擴展訪問控制列表 (ACL) 來定義PBR的流量匹配條件。
- 學習如何建立和應用路由圖 (Route Map) 來指定PBR的下一跳路由。
- 驗證PBR如何覆蓋路由表的預設行為，強制特定流量走指定路徑。
- 熟悉PBR的故障排除與驗證命令。

## 拓撲 (Topology):
本實驗將使用一個包含五台Cisco路由器 (R1, R2, R3, R4, R5) 的EVE-NG拓撲，模擬一個小型企業網路環境。

圖形化拓撲示意圖:
```
                                            +----------+
                                            |    R3    |
                                            |Loopback0 |
                                            |3.3.3.3/32|
                                            +----------+
                                             e1/1||e1/2
                                                 ||
                                                 ||
                                    _____________||___________
                       10.23.1.0/24|                          | 10.34.1.0/24
                                   |                          |
                                   |                          |
                                   |e1/3                      | e1/3
+-----------+                  +-----------+             +----------+            +----------+
|    R1     |------------------|    R2     |-------------|    R4    |------------|    R5    |
|           |10.12.1.0/24      |           |10.24.1.0/24 |          |10.45.1.0/24|          |
| Loopback0 |e1/0          e1/1|Loopback0  |e1/2     e1/2| Loopback0|e1/1    e1/0| Loopback0|
|1.1.1.1/32 |------------------|2.2.2.2/32 |-------------|4.4.4.4/32|------------|5.5.5.5/32|
+-----------+                  +-----------+             +----------+            +----------+
```
註: 在圖中，PBR的目標是將來自R1 Loopback 0的流量，原本應經由R2-R4路徑，強制重導經由R2-R3-R4路徑到達R5的Loopback 0。

## 軟體 (Software):
- Cisco IOS Software (Cisco 7200 series)
- Image: c7200-adventerprisek9-mz.152-4.S7.image 
  
## 配置流程 (Configuration Flow):

### IP 地址規劃:
**R1:**
Loopback0: 1.1.1.1/32
e1/0: 10.12.1.1/24
**R2:**
e1/1: 10.12.1.2/24
e1/2: 10.24.1.1/24
e1/3: 10.23.1.1/24
**R3:**
e1/1: 10.23.1.2/24
e1/2: 10.34.1.1/24
**R4:**
e1/1: 10.45.1.1/24
e1/2: 10.24.1.2/24
e1/3: 10.34.1.2/24
**R5:**
Loopback0: 5.5.5.5/32
e1/0: 10.45.1.2/24
## 配置步驟：

### 步驟 1: 配置所有路由器接口IP地址和Loopback接口。
R1:
```
configure terminal
hostname R1
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
interface Ethernet1/0
 ip address 10.12.1.1 255.255.255.0
 no shutdown
exit
```
R2:
```
configure terminal
hostname R2
interface Ethernet1/1
 ip address 10.12.1.2 255.255.255.0
 no shutdown
interface Ethernet1/2
 ip address 10.24.1.1 255.255.255.0
 no shutdown
interface Ethernet1/3
 ip address 10.23.1.1 255.255.255.0
 no shutdown
exit
```
R3:
```
configure terminal
hostname R3
interface Ethernet1/1
 ip address 10.23.1.2 255.255.255.0
 no shutdown
interface Ethernet1/2
 ip address 10.34.1.1 255.255.255.0
 no shutdown
exit
```
R4:
```
configure terminal
hostname R4
interface Ethernet1/1
 ip address 10.45.1.1 255.255.255.0
 no shutdown
interface Ethernet1/2
 ip address 10.24.1.2 255.255.255.0
 no shutdown
interface Ethernet1/3
 ip address 10.34.1.2 255.255.255.0
 no shutdown
exit
```
R5:
```
configure terminal
hostname R5
interface Loopback0
 ip address 5.5.5.5 255.255.255.255
interface Ethernet1/0
 ip address 10.45.1.2 255.255.255.0
 no shutdown
exit
```
### 步驟 2: 配置EIGRP路由協議以建立基礎連通性。
> 我們將在所有路由器上運行EIGRP AS 100，並宣告所有直連網絡，確保R1的Loopback0和R5的Loopback0可達。

R1:
```
configure terminal
router eigrp 100
 network 1.1.1.1 0.0.0.0
 network 10.12.1.0 0.0.0.255
 no auto-summary
exit
```
R2:
```
configure terminal
router eigrp 100
 network 10.12.1.0 0.0.0.255
 network 10.24.1.0 0.0.0.255
 network 10.23.1.0 0.0.0.255
 no auto-summary
exit
```
R3:
```
configure terminal
router eigrp 100
 network 10.23.1.0 0.0.0.255
 network 10.34.1.0 0.0.0.255
 no auto-summary
exit
```
R4:
```
configure terminal
router eigrp 100
 network 10.45.1.0 0.0.0.255
 network 10.24.1.0 0.0.0.255
 network 10.34.1.0 0.0.0.255
 no auto-summary
exit
```
R5:
```
configure terminal
router eigrp 100
 network 5.5.5.5 0.0.0.0
 network 10.45.1.0 0.0.0.255
 no auto-summary
exit
```
### 步驟 3: 在R2上配置PBR。
我們的目標是將從R1 Loopback 0 (1.1.1.1) 發送，目的為R5 Loopback 0 (5.5.5.5) 的流量，強制從R2的e1/3接口 (指向R3) 轉發，而不是通過EIGRP計算的最佳路徑 (通常是R2-R4)。

R2:
```
configure terminal
! 步驟 3.1: 定義流量匹配條件 (匹配源IP 1.1.1.1)
ip access-list extended PBR_SOURCE_MATCH
 permit ip host 1.1.1.1 any
! 步驟 3.2: 建立路由圖並定義匹配與設置動作
route-map PBR_REDIRECT permit 10
 match ip address PBR_SOURCE_MATCH
 set ip next-hop 10.23.1.3  ! 將下一跳設置為R3的e1/1接口IP
! 步驟 3.3: 將路由圖應用於接收流量的接口 (R2的e1/1)
interface Ethernet1/1
 ip policy route-map PBR_REDIRECT
end
```
## 驗證 (Verification):

1. 驗證基礎連通性:
在R1上，ping R5的Loopback0，確保在PBR配置之前，流量可以正常到達。
```
R1#ping 5.5.5.5 source Loopback0
Sending 5, 100-byte ICMP Echos to 5.5.5.5, timeout is 2 seconds:
Packet sent with a source address of 1.1.1.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 80/84/88 ms

R1#traceroute 5.5.5.5 source lo0
Type escape sequence to abort.
Tracing the route to 5.5.5.5
VRF info: (vrf in name/id, vrf out name/id)
  1 10.12.1.2 56 msec 56 msec 56 msec
  2 10.24.1.4 56 msec 56 msec 52 msec
  3 10.45.1.5 80 msec *  72 msec

```
在PBR配置完成後，再次從R1 ping R5的Loopback0，它仍應可達。

1. 驗證路由表 (在R2上):
查看R2到5.5.5.5的路由條目，它應該顯示EIGRP學習到的最佳路徑 (通常是經由R2-R4)。
```
R2#show ip route 5.5.5.5
```
預期輸出：下一跳為10.24.1.2 (R4的e1/2)

1. 驗證路由圖和ACL配置 (在R2上):
檢查ACL和路由圖是否已正確配置。
```
R2#show access-lists PBR_SOURCE_MATCH
R2#show route-map PBR_REDIRECT
```
2. 驗證PBR在接口上的應用 (在R2上):
確認PBR策略已應用於R2的e1/1接口。
```
R2#show ip policy
Interface      Route map
Ethernet1/1    PBR_REDIRECt

R2#show running-config interface Ethernet1/1
```
3. 追蹤路徑以驗證PBR效果 (在R1上):
使用 traceroute 命令從R1的Loopback0發起，目的為R5的Loopback0。
```
R1#traceroute 5.5.5.5 source Loopback0
Type escape sequence to abort.
Tracing the route to 5.5.5.5
VRF info: (vrf in name/id, vrf out name/id)
  1 10.12.1.2 56 msec 56 msec 60 msec
  2 10.23.1.3 56 msec 52 msec 52 msec
  3 10.23.1.2 68 msec 60 msec 52 msec
  4 10.24.1.4 84 msec 84 msec 84 msec
  5 10.45.1.5 88 msec *  60 msec

```
預期輸出：正常情況下，路徑應為 10.12.1.2 (R2) -> 10.24.1.2 (R4) -> 10.45.1.2 (R5)。
在PBR生效後，路徑應變為 10.12.1.2 (R2) -> 10.23.1.2 (R3) -> 10.34.1.2 (R4) -> 10.45.1.2 (R5)。這將證明PBR成功重導了流量。

4. 檢查PBR統計信息 (在R2上):
查看PBR的匹配次數，確認流量是否通過PBR策略轉發。
```
R2#show route-map PBR_REDIRECT
route-map PBR_REDIRECt, permit, sequence 10
  Match clauses:
    ip address (access-lists): PBR_SOURCE_MATCH
  Set clauses:
    ip next-hop 10.23.1.3
  Policy routing matches: 30 packets, 1800 bytes
```
關注匹配條件的 "match" 次數是否增加。

5. 一個可能是bug的現象
當我在`ip access-list extend `中，`permit ip any any log`加上log時，會出現`FIB policy rejected - normal forwarding`
使得流量無法使用PBR。



