
- 用於**監控與管理IP網路中的網路設備** (ex. Router, Switch, Server, Firewall, ..)
- 用 **UDP Port 161** (agent 接收 request / response), **162** (manager 接收 trap)

#### Architecture
![[Pasted image 20261019095613.png|420]]
- NMS (Network Management Station / Manager)：發出查詢、接收 alert
- Agent：執行在被管理的裝置上，維護資訊回應management
	- MIB (Management Information Base)
		- 用樹狀結構管理Object的DataBase
		- 每個Object用 `OID` 唯一識別![[Pasted image 20261019095809.png]]
		-  ex. `1.3.6.1.2.1.1.3.0 = sysUpTime`：設備開機時間
			- `1.3.6.1.2.1...`：標準OID，查設備開機時間、流量..
				- $\text{.iso}\rightarrow\text{.org}\rightarrow\text{.dod（美國國防部）}\rightarrow\text{.internet}\rightarrow\text{.mgmt}\rightarrow\text{.mib-2（標準管理資料庫）}\rightarrow\text{.system（系統資訊組）}\rightarrow\text{sysUpTime（設備持續運作時間）}\rightarrow\text{instance（特定object）}\rightarrow$  
			- 
	- SMI (Structure of Management Information)
		- MIB物件的描述語法

#### 版本差異

| 版本  | 認證                   | 加密               | 備註                                  |
| --- | -------------------- | ---------------- | ----------------------------------- |
| v1  | Community string（明文） | 無                | 32-bit counter，僅 Trap               |
| v2c | Community string（明文） | 無                | 新增 GetBulk、Inform、64-bit counter    |
| v3  | USM：HMAC-MD5/SHA 等   | DES/AES（privacy） | 基於 User-based Security Model 與 VACM |

#### 主要操作 (PDU)
$\text{Manager}\rightarrow \text{Agent}$：`Get`, `Set`，查詢 / 修改 MIB
- `GetRequest`：取的指定 OID設備 的值
- `GetNextRequest`：取下個 OID
- `GetBulkRequest`：一次取的大量資訊 (`v2v/v3`)
- `SetRequest`：修改指定 OID設備 的值
$\text{Agent}\rightarrow \text{Manager}$：`Trap`，agent 發起通報問題
- `Response`“回應request
- `Trap`：非同步事件，無確認
- `InformRequest`：需確認通知


## 實務用法

```bash
# v2c 走訪 system 群組
snmpwalk -v2c -c public 192.168.1.1 1.3.6.1.2.1.1

# v3 authPriv 查詢
snmpget -v3 -l authPriv -u monitor \
  -a SHA -A 'authpass' -x AES -X 'privpass' \
  192.168.1.1 1.3.6.1.2.1.1.3.0
```

常見工具：net-snmp、Zabbix、Prometheus `snmp_exporter`、LibreNMS、PRTG。

#### Case: 某 Switch port故障
##### Case 1: Agent Trap
- Agent 偵測到 Port link 狀態變化：`if Operstatus`: up(1) 變 down(2)
- agent發 Trap: `linkDown`
##### Case 2: manager polling
- Manager 會定期輪詢 Get Agent MIB 資訊 (`ifAdminStatus`, `ifOperStatus`)