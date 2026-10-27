
![[Pasted image 20261020230456.png]]

**1. Threat Environment（威脅環境）**  
根據威脅來源、威脅類型、攻擊面 (Attack Surface)、威脅情資(Threat Intelligent)，界定組織面對的威脅全貌
- 威脅來源：外部使用者、內部使用者
	- note：對應planning的risk analysis, least permission
- 威脅類型：惡意軟裡、社交工程、DDoS、應用層攻擊(SQL injection, XSS)
- 攻擊面(Attack Surface)：遠端存取、雲端設定錯誤、員工端點、第三方整合
	- note: 對應planning的 全面保護、縱深防禦、weakest link、single point
- 威脅情資(Threat Intelligent)：IoC (Indicate of Content)、TTP (Tactics, Techniques and Procedures)

**2. Planning（規劃）**  
根據威脅環境在有限資源下規劃最大效能安全計劃
- Risk analysis：評估風險決定對應風險處置（規避、降低、轉移、接受）
- Comprehensive Security：不能只保護單一方向，要保護整個 Attack Surface（**橫向範圍**）
- Defense in depth：多層互相獨立的防禦，單一層防禦失效不會整體失守（**縱向架構**）
	- 防火牆 $\rightarrow$ 網路分段 $\rightarrow$ EDR $\rightarrow$ 資料加密 $\rightarrow$ 身份
- Identifying weakest links：識別整體**最薄弱的元件**
- Identify Single point of takeover：找出能**讓攻擊者取得最大權限的節點** ex. Domain Controller / Active Derictory
- Assigning least permissions：只給予員工所需最小權限 ex. RBAC
- Policy - based Planning：將安全需求制度化

**3. Protect（防護）**  
- 實作預防型防禦 ex. 防火牆, EDR, 加密

**4. Respond（回應）**  
事件發生後的處置，資安團隊透過 SIEM 進行處理
- containment：隔離受影響範圍
- Eradication：移除惡意程式
- Recovery：恢復服務
- RCA根因分析並回饋到 plan


