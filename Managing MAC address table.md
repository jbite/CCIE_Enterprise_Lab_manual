# Managing MAC address table

[Troubleshoot Mac Address Table Manager on Catalyst 9000 Series Switches]( https://www.cisco.com/c/en/us/support/docs/switches/catalyst-9300-series-switches/222060-troubleshoot-mac-address-table-manager-o.html#toc-hId--436766453)

主題：Cisco Catalyst 9000 (IOS-XE) MAC 位址表管理與故障排除
1. MAC Address Table (MAC 位址表)
Naming: 儲存 MAC 位址與實體/邏輯出口連接埠映射關係的資料庫，又稱 CAM Table。
Attending: 在 Cat 9000 中，由 IOS (軟體) 與 FED (硬體 ASIC) 共同維護。
Memorizing: MAC 對連接埠，二層轉發之本。
Expressing: 使用 show mac address-table 查詢。
Storytelling: 就像大樓的管理員清單，記住每個住戶住在幾號房。
2. MATM (MAC Address Table Manager)
Naming: Cat 9000 專有的進程，負責同步軟體與硬體間的 MAC 資訊。
Attending: 它是 IOS-XE 軟體與底層 ASIC 晶片之間的橋樑。
Memorizing: MATM，軟硬同步的管家。
Expressing: 當 MAC 學習異常時，需檢查 MATM 是否運作正常。
Storytelling: 像是大樓經理，確保管理室的名單與保全手上的名冊一致。
3. FED (Forwarding Engine Driver)
Naming: 負責管理 ASIC 轉發引擎的驅動程式，處理硬體層級的位址寫入。
Attending: 若 FED 寫入失敗，即會出現軟體看得到 MAC 但通訊失敗的現象。
Memorizing: FED，硬體轉發的掌舵者。
Expressing: 使用 show platform software fed switch active matm macTable 檢查。
Storytelling: 像是建築的地基與管線，雖然看不見但決定了通行的效率。
4. TCAM (Ternary Content-Addressable Memory)
Naming: 用於極速查詢 MAC、ACL 與路由表項的專用硬體記憶體。
Attending: 空間有限，一旦寫滿，交換器將無法學習新 MAC 或轉向泛洪。
Memorizing: TCAM，硬體查表加速器。
Expressing: show platform hardware fed switch active fwd-asic resource tcam utilization。
Storytelling: 就像管理員腦袋裡的快速索引，空間滿了就記不住新面孔。
5. Dynamic Learning (動態學習)
Naming: 交換器根據進入封包的來源 MAC 與進入連接埠，自動建立表項的過程。
Attending: 預設啟用，表項會隨老化時間（Aging Time）到期而消失。
Memorizing: 封包進，位址留，動態記。
Expressing: 連接新電腦後，交換器自動將其 MAC 存入表內。
Storytelling: 像新住戶搬入，管理員看到他在 A 門進出，就記下他在 A 門。
6. Aging Time (老化時間)
Naming: 動態學到的 MAC 在表中保留的時間長度，預設 300 秒。
Attending: 若 300 秒內該裝置未發送任何流量，表項即被清除。
Memorizing: 過期不候，定時清理。
Expressing: mac address-table aging-time 600。
Storytelling: 住戶太久沒出現，管理員會認定他已搬走，並將其名子劃掉。
7. Static MAC (靜態 MAC)
Naming: 手動配置且永久保留在表中的 MAC 位址，不會被老化清除。
Attending: 用於伺服器或重要設備，確保其轉發路徑不變且更具安全性。
Memorizing: 手動釘死，永不消失。
Expressing: mac address-table static 0011.2233.4455 vlan 10 interface Gi1/0/1。
Storytelling: 像大樓的永久住戶（VIP），名牌會焊在牆上不更動。
8. MAC Flapping (MAC 漂移/跳變)
Naming: 同一個 MAC 地址短時間內在不同連接埠間頻繁切換的現象。
Attending: 這是檢測網路環路（Layer 2 Loop）最重要的指標。
Memorizing: 左右橫跳，必有環路。
Expressing: 查看 Syslog 訊息 %L2-5-MAC_MOVE。
Storytelling: 住戶一秒前在東門，下一秒在西門出現，這顯然是撞鬼或空間混亂。
9. Unknown Unicast Flooding (未知單播泛洪)
Naming: 當交換器找不到目的地 MAC 的連接埠時，將封包發送到該 VLAN 所有連接埠。
Attending: 過度泛洪會導致網路擁塞。
Memorizing: 找不到，全場喊。
Expressing: 抓包分析發現流量被發送到無關的連接埠。
Storytelling: 管理員找不到人，只好對著整棟大樓廣播：「某某某請回話！」
10. Sticky MAC (黏性 MAC)
Naming: Port Security 的一種模式，動態學到的 MAC 會轉變為不消失的配置。
Attending: 用於安全防禦，防止其他設備接入該連接埠。
Memorizing: 一旦黏上，別想換人。
Expressing: switchport port-security mac-address sticky。
Storytelling: 住戶第一次刷卡後，指紋就跟這個門鎖死，別人進不去。
11. Port Security (埠安全)
Naming: 限制連接埠上可連接的 MAC 數量及特定位址的機制。
Attending: 超過限制會觸發 Violation，如關閉連接埠（Shutdown）。
Memorizing: 控制存取，守護接口。
Expressing: switchport port-security maximum 2。
Storytelling: 守門人規定一個房間只能住兩個人，多進去一個就報警封鎖。
12. MAC Violation (位址違規)
Naming: 當 Port Security 檢測到非授權 MAC 或超過數量限制時觸發的動作。
Attending: 模式包含 Protect、Restrict 與 Shutdown。
Memorizing: 違規即罰，安全至上。
Expressing: switchport port-security violation restrict。
Storytelling: 抓到外來者，保全選擇直接趕走（Protect）或關閉整區通訊（Shutdown）。
13. Bridge Domain (橋接域)
Naming: IOS-XE 處理二層轉發的進階邏輯容器，常用於 BGP EVPN。
Attending: 取代傳統 VLAN 的邏輯，提供更靈活的擴充性。
Memorizing: 邏輯分區，比 VLAN 更強。
Expressing: 在 EVPN 設定中定義 bridge-domain 10。
Storytelling: 像是一個跨區域的大包廂，裡面的人即便距離遠也能像在同個房間對話。
14. EPC (Embedded Packet Capture)
Naming: 內建於 IOS-XE 的抓包工具，可直接在交換器記憶體中攔截流量。
Attending: 排除 MAC 學習問題時，確認封包是否真的抵達 CPU 或入口。
Memorizing: 內建側錄，流量現形。
Expressing: monitor capture MYCAP interface Gi1/0/1 both match any。
Storytelling: 像是隱藏錄影機，把進出大門的每個人都拍下來供事後比對。
15. Mac Address Notification (位址變動通知)
Naming: 當 MAC 地址增加或刪除時，主動發送 SNMP Trap 給網管伺服器。
Attending: 有助於監控重要資產是否離線或非法接入。
Memorizing: 變動即報，掌握動態。
Expressing: mac address-table notification change。
Storytelling: 管理員只要看到名單有改動，就立刻發簡訊給老闆。
16. SVI MAC (三層接口 MAC)
Naming: 交換器虛擬介面（VLAN Interface）的 MAC 地址，用於網關轉發。
Attending: 通常整台交換器所有 SVI 共享同一個 Base MAC。
Memorizing: 網關專用，路由核心。
Expressing: show interface vlan 10 查看 Hardware Address。
Storytelling: 就像大樓服務台的專屬窗口地址，所有人要寄信出去都得找它。
17. Multicast MAC (組播 MAC)
Naming: 用於傳輸組播流量的特定 MAC（01:00:5E 開頭）。
Attending: 映射自組播 IP 地址，涉及 IGMP Snooping 學習。
Memorizing: 標記群組，一對多發送。
Expressing: show mac address-table multicast。
Storytelling: 像是興趣小組的名單，只要是這個組的成員，廣播內容都會送達。
18. IGMP Snooping (IGMP 偵聽)
Naming: 交換器監聽 L3 終端的 IGMP 訊息，以精確維護組播 MAC 表。
Attending: 防止組播流量泛洪到沒有訂閱的連接埠。
Memorizing: 偷聽請求，精確發送。
Expressing: show ip igmp snooping groups。
Storytelling: 管理員在門口聽住戶說想看哪台電視，之後只有那個住戶會收到訊號。
19. MAC Flushing (位址清除)
Naming: 手動或由協定（如 STP）觸發，清空部分或全部 MAC 表項的動作。
Attending: 用於快速恢復拓撲變化後受損的轉發路徑。
Memorizing: 快速清零，重新學習。
Expressing: clear mac address-table dynamic。
Storytelling: 為了防止走錯路，管理員把整本名錄撕掉，讓大家重新登記。
20. Hardware Consistency Check (硬體一致性檢查)
Naming: 檢查硬體 ASIC 中的 MAC 表與軟體 IOS-XE 表是否一致的進階排錯流程。
Attending: 當通訊不正常且 show mac 看起來沒問題時使用。
Memorizing: 軟硬對帳，找出幽靈。
Expressing: 比較 show mac... 與 show platform software fed...。
Storytelling: 財務稽核，核對帳本上的數字與金庫裡的現鈔是否吻合。

<table>
<thead>
<tr>
<th style="background-color: #f2f2f2;">類別</th>
<th style="background-color: #f2f2f2;">操作目標</th>
<th style="background-color: #f2f2f2;">指令 (CLI Command)</th>
<th style="background-color: #f2f2f2;">說明</th>
</tr>
</thead>
<tbody>
<tr>
<td rowspan="3"><b>管理配置</b></td>
<td>靜態位址</td>
<td><code>mac address-table static &lt;MAC&gt; vlan &lt;ID&gt; interface &lt;INT&gt;</code></td>
<td>手動綁定 MAC 與連接埠。</td>
</tr>
<tr>
<td>老化時間</td>
<td><code>mac address-table aging-time &lt;seconds&gt;</code></td>
<td>調整動態表項保留時間。</td>
</tr>
<tr>
<td>學習控制</td>
<td><code>no mac address-table learning vlan &lt;ID&gt;</code></td>
<td>停用特定 VLAN 的 MAC 學習。</td>
</tr>
<tr>
<td rowspan="2"><b>安全維護</b></td>
<td>埠安全啟用</td>
<td><code>switchport port-security</code></td>
<td>在介面層級啟用安全限制。</td>
</tr>
<tr>
<td>黏性 MAC</td>
<td><code>switchport port-security mac-address sticky</code></td>
<td>將動態學到的 MAC 永久鎖定。</td>
</tr>
<tr>
<td rowspan="4" style="color: #d9534f;"><b>故障排查</b></td>
<td>軟體表驗證</td>
<td><code>show mac address-table</code></td>
<td>查看 IOS-XE 邏輯層的 MAC 表。</td>
</tr>
<tr>
<td>硬體表驗證</td>
<td><code>show platform software fed switch active matm macTable</code></td>
<td><b>最重要！</b>檢查 ASIC 硬體是否同步。</td>
</tr>
<tr>
<td>資源檢查</td>
<td><code>show platform hardware fed switch active fwd-asic resource tcam utilization</code></td>
<td>檢查 TCAM 空間是否已滿。</td>
</tr>
<tr>
<td>清除表項</td>
<td><code>clear mac address-table dynamic</code></td>
<td>強制清空表項以排除錯誤學習。</td>
</tr>
</tbody>
</table>