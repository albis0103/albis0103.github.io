
#### STP (Spanning Tree Protocol 802.1D)
- 定義：透過建立MST (Minimum Spanning Tree)，**避免拓頗產生loop**

#### algo
1. Find root：找最低識別碼(Bridge ID, Priority + MAC 最小者)的 Node
2. 各自 boardcast交換組態
3. 建立 root 到各點的 MST
4. 其餘port blocking

#### RSTP (Root Spanning Tree Protocol, 802.1W)
- 定義：是 STP的演進，避免 Broadcast Storm與加快收斂時間