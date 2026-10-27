Target 在自身內部網路沒有做到最小權限管理等導致供應商被攻擊後橫向移動進 Target外洩用戶資料

![[Pasted image 20261023171044.png]]

1. 攻擊者透過 Spear Phishing 攻擊 Target 供應商 竊取 Target Vender Server 的憑證
2. 用憑證進入 Target 外部供應商路口，橫向移動到 AD(Active Directory)
3. 下放惡意程式抓取指令，POS主動向Malware Download Server請求下載 
4. 惡意程式掃描POS內記憶體明文資料，暫存到 Holding Server
5. 透過Extrusion Server，批次把資料往外送
6. SIEM產生 alert 但 SOC團隊沒處理

#### kill chain 角度 solution
[[Cyber Kill Chain]]
- 廠商最小權限管理：廠區的VLAN應該要被防火牆防止主動請求AD的規則擋住，從 Exploitation階段抵擋
- SoC團隊訓練：SIEM有 Alert但卻沒回應，若及時處理 Alert應該在 Installation時就阻擋