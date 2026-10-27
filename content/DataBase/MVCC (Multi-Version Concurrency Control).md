### Def
- 基於[[Time-stamp Concurrency Control]]對同一資料維護多個版本(Version)
- Read 操作 依交易 snapshot 選可見版本 $d_i$ 
	- 讀 $$\max \{\text{write-TS}(d_i)\}\quad ,\text{if write-TS}(d_i)≤\text{read-TS}(d_i)$$
	- 並設定$$\text{read-TS}(d_i)=\max\{TS(T),\;\text{read-TS}(d_i)\}$$
- Write 操作 建立新版本，不會in-place update 
	- find 版本 $d_i$ such that$$\max_{d_i}\{\text{write-TS}(d_i)\} \ \quad \text{if write-TS}(d_i)≤\text{read-TS}(d_i)$$
	- 若 $\text{read-TS}(d_i)≤TS(T)$ 則產生並寫入新版本 $d_j$ 
		- then $\text{write-TS}(d_j)=\text{read-TS}(d_j)=TS(T)$
	- 若 $\text{read-TS}(d_i)>TS(T)$ ：$abort$

#### Version Chain
每個版本高煩
- 建立者 Transaction ID (begin timestamp / `xmin`)
- 刪除 / 取代者 Transaction ID (end timestamp / `xmax`) 
	- ex. InnoDB: `DB_TRX_ID`
- 資料內容
- 前後指標 ex.InnoDB : `DB_ROW_ID` 

