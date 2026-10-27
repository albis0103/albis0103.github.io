#### 定義
- 以邏輯方式（**設定組態表**）分配 packet 到正確 port 來**分群**，實現**Layer 2 切分成多個獨立 Broadcast domain**
- 依 Node 是否支援 VLAN 決定 packet 是否貼上 VLAN Tag

### 802.1Q Tag 格式
在 Ethernet frame 的 Source MAC 之後插入 4 bytes：

|欄位|長度|說明|
|---|---|---|
|TPID|16 bits|固定 0x8100|
|PCP|3 bits|Priority Code Point（802.1p，QoS 優先權）|
|DEI|1 bit|Drop Eligible Indicator|
|VID|12 bits|VLAN ID，範圍 0–4095|

### 埠類型

#### **Access port**

- 屬於單一 VLAN，連接終端裝置。
- 進入時由交換器依 PVID 加上 tag，離開時移除 tag；終端不感知 VLAN。

#### **Trunk port**

- 同時承載多個 VLAN，用於交換器之間、交換器與 router 之間。
- 以 tag 區分 VLAN。
- **Native VLAN**（802.1Q 的 untagged VLAN）：該 VLAN 的 frame 在 trunk 上不帶 tag；兩端 native VLAN 設定不一致會造成 VLAN 洩漏與 STP 異常（Cisco 以 CDP 報 native VLAN mismatch）。
- 可用 allowed VLAN list 限制 trunk 通過的 VLAN。


### 轉送行為

- MAC address table 以 (VID, MAC) 為索引（Independent VLAN Learning），同一 MAC 可存在於不同 VLAN。
- Unknown unicast、broadcast、multicast 只在同一 VLAN 內 flood。

### Cisco 設定範例

```
vlan 10
 name ENG
!
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
!
interface GigabitEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 switchport trunk native vlan 999
!
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
```

