### Def
- Each Transaction 有唯一識別的 Time-stamp $TS(T_i)$ 紀錄交易開始時間
- 資料 $x$ 
	- $\text{read-TS(}x)$與$\text{write-TS}(x)$ 紀錄最後被read 或 write的timestamp

### Algo
- 交易 $T$ 提出 $read(x)$
	- 若 $\text{write-TS}(x)≤ \text{read-TS}(x)$
		- 執行$read$ 並 $$\text{read-TS(}x)=\max\{TS(T),\text{read-TS}(x)\}$$
	- 否則 ($\text{write-TS}(x)> \text{read-TS}(x)$)：$Abort$
	- 