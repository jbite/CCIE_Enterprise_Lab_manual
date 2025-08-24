# Lab Manual: BGP AS_path control
Lab No: BGP_AS_PATH_CTRL_01
Author: Jacky, copilot with AI
Date: 2025年8月11日
## Summary:
此實驗旨在探討如何透過調整BGP的AS_path屬性來影響和控制路由的選路行為，特別是在有多個出口路徑時，如何透過AS_path的長度來達到流量工程的目的。我們將利用AS_path prepend來讓特定的路由路徑看起來較長，從而降低其被選為最佳路徑的可能性。

## Purpose:
掌握BGP的基本配置，並建立鄰居關係。

學習並應用AS_path prepend技術來控制BGP路由的選路。

理解AS_path屬性在BGP選路過程中的作用。

觀察AS_path prepend對路由表和網路流量的影響。

驗證BGP的路由選路機制，並確認AS_path prepend配置的有效性。

## Topology:
```
 +-----------------+            +-------------------+
 |      AS 65001   |            |      AS 65002     |
 |      +------+   |            |     +------+      |
 |      |  R1  |---|------------|-----|  R2  |      |
 |      +------+   |            |     +------+      |
 |         |       |10.12.1.0/24|         |         |
 |         |       |            |         |         |
 +-----------------+            +-------------------+
           |                              |           
      10.13.1.0/24                  10.24.1.0/24      
           |                              |           
 +------------------+             +------------------+
 |    +------+      |             |    +------+      |
 |    |  R3  |      |             |    |  R4  |      |
 |    +------+      |             |    +------+      |
 |    AS 65003      |             |    AS 65004      |
 +------------------+             +------------------+
 ```

## EVE-NG 圖像名稱:
- R1: iol-cisco-3725
- R2: iol-cisco-3725
- R3: iol-cisco-3725
- R4: iol-cisco-3725

## Technologies:
- BGP (Border Gateway Protocol)
- BGP AS_path prepend
- IP Routing
- Subnetting

## Software:
Cisco IOS on IOL (IOS on Linux)

## Configuration Flow:
R1 Configuration:
```
!router bgp 65001
 no synchronization
 bgp log-neighbor-changes
 neighbor 10.12.1.2 remote-as 65002
 neighbor 10.13.1.3 remote-as 65003
 network 10.1.1.0 mask 255.255.255.0
```
```
!R2 Configuration:

router bgp 65002
 no synchronization
 bgp log-neighbor-changes
 neighbor 10.12.1.1 remote-as 65001
 neighbor 10.23.1.3 remote-as 65003
 network 10.2.2.0 mask 255.255.255.0
```
!R3 Configuration:
```
router bgp 65003
 no synchronization
 bgp log-neighbor-changes
 neighbor 10.13.1.1 remote-as 65001
 neighbor 10.23.1.2 remote-as 65002
 network 10.3.3.0 mask 255.255.255.0
 neighbor 10.13.1.1 route-map PREPEND_R1 out
!
access-list 1 permit 10.255.10.3 0.0.0.0
!
route-map PREPEND_R1 permit 10
 match ip address 1
 set as-path prepend 65003 65003 65003
```
R4 Configuration (Optional - for additional verification):
```
router bgp 65004
 no synchronization
 bgp log-neighbor-changes
 neighbor 10.24.1.2 remote-as 65002
 network 10.4.4.0 mask 255.255.255.0
!(如果沒有R4設備，可忽略此部分)
```
## Verification:
在R1上檢查BGP路由表，觀察到達10.3.3.0/24的路徑：
```
show ip bgp
show ip route bgp
```
在R2上檢查BGP路由表，觀察到達10.3.3.0/24的路徑：
```
show ip bgp
show ip route bgp
```
在R3上移除route-map PREPEND_R1 out配置，然後再次在R1和R2上檢查BGP路由表，觀察路徑的變化：
```
config t
router bgp 65003
no neighbor 10.13.1.1 route-map PREPEND_R1 out
end
clear ip bgp * (清除BGP鄰居關係以便快速更新)
show ip bgp (在R1和R2上執行)
```
在R1上對10.3.3.0/24進行traceroute，觀察路由路徑是否經過預期的AS_path：
```
traceroute 10.3.3.1
```
在R3上檢視送給R1的BGP更新訊息，確認AS_pathprepend是否生效：
```
show ip bgp neighbors 10.13.1.1 advertised-routes
show ip bgp neighbors 10.13.1.1 routes
```