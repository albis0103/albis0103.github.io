DOT: Dynamic Knob Selection and Online Sampling for Automated Database Tuning（PVLDB 19(4), 2025）

---
[[Obsidian Vault/Paper/DataBase/DOT|DOT]]
## 1. 研究動機與問題定義

DBMS（如 MySQL）有數百個 knob，但只有少數對效能有顯著影響，且重要性取決於 workload 與硬體。現有 knob selection 方法分四類，各有成本問題：


| 類別               | 方法                       | 階段 1：選 knob         | 階段 2：找參數               | 問題              |
| ---------------- | ------------------------ | ------------------- | ---------------------- | --------------- |
| Experience-based | CDBTune                  | 專家指定                | DDPG                   | 依賴DBA           |
|                  | UDO                      | 使用者指定               | MCTS（另外做 index tuning） |                 |
| Screening        | P&B、sensitivity analysis | 取樣後用統計排序            | 論文沒有說明                 | 分數粗糙無法補抓knob間關係 |
| ML-based         | OtterTune                | Lasso               | GP-BO                  | 需大量資料           |
|                  | Hunter                   | Random forest（CART） | 論文沒有說明                 |                 |
| LLM-based        | DB-BERT                  | BERT 讀文件            | DQN                    | 需解析文件，成本高       |
|                  | GPTuner                  | ChatGPT 讀文件         | SMAC                   |                 |

![[Pasted image 20261015213339.png]]
- CART 效能最好，但需 220 個樣本；LLM 零樣本但 OLAP 較差。(method : RFECV 直接重用 BO 過程的樣本)
**核心問題**：能否不做獨立的 knob ranking 階段（warm-up），在 tuning 過程中同時完成 knob selection？

---

## 2. 前置實驗：為什麼 knob selection 困難


(a) 噪聲與可重現性（Sec. 2）：**雜訊很多，(method: 要重複5次，每次重啟DB)**


- 同一配置重複 10 次，CV：TPC-C 6.24%、Sysbench 4.36%、TPC-H 0.52%。
- BO 本身也有隨機性：收斂所需迭代數的 CV 為 34%–73%。
- 評估協議：每種方法 5 次不同 seed、每次重啟 DBMS；用 Welch t-test（兩兩比較）與 Friedman test（多重比較），顯著水準 p < 0.05。
![[Pasted image 20261015202932.png|633]]![[Pasted image 20261015203013.png|594]]

(b) 取樣成本 vs. 排名穩定性（Sec. 3.2）：**CART有隨機性、成本高、樣本多效益遞減**（method : E = 5 × 新加入的 knob 數）

- 以 Jaccard index 衡量 CART Top-20 knob 集合的穩定性：LHS 取樣明顯比純隨機取樣穩定；樣本數增加也更穩定。（OtterTune、Hunter 用 LHS，CDBTune 用純隨機。）
- 520 個樣本（每 knob 10 個）約需 14.4 小時。
- 效能遞減：100→200 樣本顯著（1,602.3→1,690.8 TPS），之後邊際遞減且不顯著。**取樣時間遠大於 tuning 時間**。
![[Pasted image 20261015201852.png]]
![[Pasted image 20261015203535.png]]
**(c) Workload 依賴性與可遷移性（Sec. 3.3）**(method : 每個 epoch 重跑 RFECV)

- 不同 workload 的 Top-20 knob 重疊度低（Jaccard 約 0.24–0.47），且 TPC-C 反而與 TPC-H 較像。
- 但跨 workload 重用仍可行：專家 knob 集 1,725 TPS，TPC-C 集 1,653，TPC-H 集 1,639，隨機 20 個僅 1,063（損失約 40%）。
- 只用「三者共有的 7 個 knob」效能接近，但收斂時間加倍（447 min vs. 約 217 min）。
![[Pasted image 20261014232211.png]]

**(d) 最佳 knob 數量（Sec. 3.4）**：依 workload 而異。Sysbench 與 TPC-C 以 Top 20 最佳或最佳折衷；TPC-H 對 knob 數不敏感，Top 5 即可。(method : LRT 決定擴張，RFECV 決定縮減)
![[Pasted image 20261015202819.png]]



---

## 3. DOT 方法

### 3.1 整體架構（Algorithm 1）

核心想法：**tuning 與 knob selection 合併**。以 BO 邊調邊收集樣本，用這些樣本在線上評估 knob 重要性。


![[Pasted image 20261015204026.png]]

參數：初始 knob 數 k₀ = 20、擴張步長 Δk = 5、初始 epoch 長度 E = 5·k₀。

每個 epoch 迴圈三步：

1. **BO Exploration**：在目前 knob 子空間 K_curr 上跑 E 次 BO，更新全域觀測集 D（投影到 K_curr 得 D_proj）。
2. **Knob Selection**：用 RFECV 在 D_proj 上剪掉低重要性 knob，得 K*。
3. **Knob Set Expansion**：LRT 決定動作 a：
    - a = 0（stay/shrink）：K_curr ← K*，E ← 10
    - a = 1（expand）：K_curr ← K* ∪ 下一批 Δk 個 knob，E ← 5·Δk
	
重複直到預算 T_max 用完，回傳 D 中最佳配置。

**樣本重用**：新加入的 knob 初始為預設值，所以先前樣本投影到新空間後仍有效，可作為 BO 的初始化。

**有無先驗**：有排序清單就從 top-k₀ 開始（DOT+EXP）；沒有就隨機選 k₀ 個（DOT+RAN），靠 RFECV + LRT 自行發現重要 knob。

### 3.2 BO with Gaussian Process
[[Bayesian Optimization]]
GP 作為 surrogate model，以 acquisition function 選下一個配置；D 為空時先隨機取 10 個配置。每個 epoch 長度為新增 knob 數的 5 倍，使每個 knob 約有 5 個樣本供 RFECV 使用。

### 3.3 RFECV
[[RFE (Recursive Feature Elimination)]]
- 以 Random Forest（CART）為 estimator，遞迴移除最不重要的 knob，每輪用交叉驗證評分，取平均 CV 分數最高的子集。
- 優點：不像單純 CART ranking 需要人為設定截斷點。（-> 研究問題）
- **下限**：保留至少 10 個 knob，避免過度剪枝而失去 BO 的搜尋多樣性。
	- ex.  
	    - 子集有 20 個 knob → 跑一次 CV → 得到分數 A 
		- 子集剩 15 個 knob → 跑一次 CV → 得到分數 B
		- 子集剩 10 個 knob → 跑一次 CV → 得到分數 C
	- 選分數最高的那個大小

### 3.4 LRT 決定是否擴張

- 兩個動作 i ∈ {0,1}，各維護成功/失敗計數 (sᵢ, fᵢ)，初始皆為 1。

$$\hat p_a = \frac{s_a}{s_a+f_a}, \qquad \Lambda = \ln\frac{\hat p_1}{\hat p_0}, \qquad a_t = \mathbb{1}[\Lambda > 0]=\begin{cases}1, &\text{if }\Lambda > 0 \\0, &\text{if }\Lambda < 0 \end{cases}$$
- Reward：  算BO讓最佳效能提升多少比例（$\frac{\text{增加的量}}{原本的量}$），更新$s_a,f_a$
	- 依 $r_t$ 更新所選動作的 $s_{a_t}$ 或 $f_{a_t}$。
 
$$g_t = \frac{y^*-b}{b\cdot E}, \quad \delta = 0.001, \qquad r_t = \mathbb{1}[g_t > \delta]=\begin{cases}1, &\text{if }g_t>0.001 \\0, &\text{if }g_t≤0.001 \end{cases}$$

$$\begin{cases}s_{a_t}\leftarrow s_{a_t}+1, &
\text{if }r_t=1 \\ f_{a_t}\leftarrow f_{a_t}+1, &
\text{if }r_t=0\end{cases}$$

- 直覺：改進不足 → 探索（加 knob）；改進強 → 利用（保持精簡）。只需 4 個純量，開銷極小。

### 3.5 Adaptive Benchmarking

Benchmark 佔 tuning 時間很高（Sysbench 69.5%、TPC-H 57%），故提前終止不太可能超越當前最佳的測試：

- **OLTP**：完整跑 T_max = 90 s（取最後 30 s 算 TPS）。在 T_cut = 40 s 時若平均 TPS 已超過當前最佳才續跑，否則以部分結果作為 proxy。此時 95th percentile MAPE：TPC-C 13.7%、Sysbench 14.2%。
- **OLAP**：隨機抽 40% 的 query，若部分負載已比當前最佳總時間快才跑完整負載。

---

## 4. 實驗結果

**設定**：MySQL 8，VM 4 vCPU / 16 GB RAM；Sysbench、TPC-C（SF 100）、TPC-H（SF 1）；全域空間為 52 個專家挑選的效能相關 knob。

**Baseline**：評估兩種情境：有專家排序 / 無（隨機排序）。
- BO（搭配 EXP / CART / 全部 knob）
- Incremental Knob Tuning（OtterTune 式）
- Statistical Elimination
- CDBTune
- DB-BERT
- LLM-based（改自 GPTuner）

**主要結果（Table 8）**

| Workload | 方法      | 效能                 | Tuning 時間         |
| -------- | ------- | ------------------ | ----------------- |
| Sysbench | DOT+EXP | 1,728.3 ± 28.3 TPS | 57.7 ± 28.2 min   |
| Sysbench | BO+EXP  | 1,725.2 ± 31.1 TPS | 217.8 ± 42.7 min  |
| Sysbench | DOT+RAN | 1,724.7 ± 50.6 TPS | 171.1 ± 121.2 min |
| TPC-C    | DOT+EXP | 377.7 ± 16.9 TPS   | 121.6 ± 72.1 min  |
| TPC-C    | BO+EXP  | 354.2 ± 14.8 TPS   | 411.9 ± 264.9 min |
- 核心結論：
	- DOT效能不輸BO 但 Tuniing快很多
		- Sysbench：DOT+EXP 與 BO+EXP 效能相當，tuning 時間縮短 **73.6%**。
		- TPC-C：DOT+EXP 效能排名第一且優於 BO+EXP（統計顯著），時間縮短 **70.5%**。
	- 可以沒有先驗知識
		- **無先驗時** DOT+RAN 仍能匹配 BO+EXP；BO+CART 效能好但需額外 14.4 小時預取樣；BO All knobs 最終相近但收斂很慢。
	- TPC-H個方法差異小，workload 對 tuning不敏感，不具參考價值
		- DOT+RAN 以 45.5 ± 1.3 s 取得最佳平均，時間比 BO+EXP 少 53.6%。



**消融（Sec. 5.7）**

- 移除 adaptive benchmarking：最終效能不變，但收斂較慢。
- LRT 換成 Thompson Sampling 或移除擴張策略：t-test 下無顯著差異，框架對選擇策略穩健。[[Two-armed Bernoulli bandit]]
- 每 knob 的 BO epoch 從 5 加到 10：無效能增益且延遲收斂。
- 對 k₀（5/10/20/30）與 T_cut（10–60）不敏感。

**系統開銷（Sec. 5.8）**：benchmarking 佔 56%、BO bookkeeping 約 27%、RFECV 約 0.4%、其餘約 16%、LRT < 0.01%。

---

## 5. 結論、限制與討論

**貢獻**

1. 提出可重現的實驗框架與公平比較 Tuner方法（多 seed、重啟 DBMS、統計檢定判斷）。
2. 實證 knob selection 的成本與 workload 依賴性。
3. DOT：免 warm-up、可選擇性使用先驗的輕量框架。
4. 與 SOTA 持平或更佳，tuning 開銷大幅降低。

**限制**

- 實驗僅在單一 DBMS（MySQL 8）、單一 VM 規格；論文自己列出「跨多個 DBMS 與真實 workload 驗證」為未來工作。
- 全域空間已預先縮減到 52 個專家挑選的 knob，不是完整的數百個 knob，實際大空間下的表現未驗證。
- TPC-H 為 SF 1，規模較小，DOT不一定適合OLAP
- 每組僅 5 次重複，作者自述受成本限制。
- CDBTune/DB-BERT 在 cold-start 設定下受限，比較對這些方法偏不利
	- CDBTune是 DDPG，本來就需要大量樣本讓actor-critic收斂
	- DB-BERT依賴完整文件
- LRT 的獎勵閾值 δ = 0.001 與 RFECV 下限 10 為經驗設定。
- 論文未包含 Hunter、DDPG++（無公開實作）與 UDO（偏 index tuning）的比較。


**未來方向**
- Workload characterization : ：把 workload 表示成可比較的特徵（例如 query 類型比例、讀寫比、DBMS 內部 metrics、query plan 的編碼），讓系統知道「現在跑的是什麼樣的負載」
- Other Optimization object : DOT只最佳化OLAP或OLTP的執行時間，可以同時考去更多元：
	- Latency
	- Utilization (CPU, memory)
	- 能耗
- 其他 Hyperparameter Optimization Engine，作者承認 BO 在 knob數 超過 20 成本快速上升
	- 可能替換方向：SMAC（以 random forest 作 surrogate，較適合混合空間）、TPE、或 DDPG 等強化學習方法。
- 真實場景與多 DBMS 驗證
	- 生產環境負載會漂移、有尖峰、混合多種查詢型態，噪聲特性也不同。