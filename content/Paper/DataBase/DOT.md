# DOT: Dynamic Knob Selection and Online Sampling for Automated Database Tuning

## 題名與來源

- 論文題名：DOT: Dynamic Knob Selection and Online Sampling for Automated Database Tuning
- 期刊 / 會議：PVLDB (Proceedings of the VLDB Endowment), Vol. 19, No. 4, pp. 589–602
- 年份：2025（arXiv 版 2026-03）
- 連結 / DOI：10.14778/3785297.3785302；arXiv:2603.15540；
- 程式碼 github.com/Orange-OpenSource/dot

## 研究問題

- 作者想解決什麼問題？
	- DBMS 調校是一個黑箱最佳化問題：$x^* = \arg\max_{x \in \mathcal{X}} f(x)$，評估 $f(x)$ 都要**每次實際跑 benchmark**，而且結果有雜訊（同組參數評估兩次結果不同）。
	- MySQL 有數百個 knob，但**只有少數真正影響效能**。因此現有 tuner 會先做 knob 選擇，把空間縮成 top 10–20 個（[49] 指出 knob 數超過某個門檻後，效能不再提升，只會增加開銷）。
	- **選擇階段本身成本**很高：

| 類別               | 方法                       | 階段 1：選 knob         | 階段 2：找參數               | 問題              |
| ---------------- | ------------------------ | ------------------- | ---------------------- | --------------- |
| Experience-based | CDBTune                  | 專家指定                | DDPG                   | 依賴DBA           |
|                  | UDO                      | 使用者指定               | MCTS（另外做 index tuning） |                 |
| Screening        | P&B、sensitivity analysis | 取樣後用統計排序            | 論文沒有說明                 | 分數粗糙無法補抓knob間關係 |
| ML-based         | OtterTune                | Lasso               | GP-BO                  | 需大量資料           |
|                  | Hunter                   | Random forest（CART） | 論文沒有說明                 |                 |
| LLM-based        | DB-BERT                  | BERT 讀文件            | DQN                    | 需解析文件，成本高       |
|                  | GPTuner                  | ChatGPT 讀文件         | SMAC                   |                 |

- 目標：能否不做獨立的 knob ranking 階段（warm-up），在 tuning 過程中同時完成 knob selection？
- 為什麼這個問題重要？(為什麼 knob selection 困難)
    - **knob 重要性依賴workload** ：三個 workload 重疊度低（ top-20 之間 Jaccard 只有 0.24–0.47），所以換一個 workload 就得重新選一次。![[Pasted image 20261014232211.png]]
	    - 前置實驗：Jaccard index 衡量 knob 集合穩定度，LHS明顯比隨機穩定![[Pasted image 20261014231053.png]]
	    - method: Online sampling
    - **knob 數量要取捨**：調太少會掉效能，調太多（全部 52 個）收斂時間增加。
	    - 最佳knob 數量：Sysbench 與 TPC-C 以 Top 20 最佳，TPC-H Top 5![[Pasted image 20261014232358.png]]
	    - REFCV + LRT
    - 預訓練與大量資料需求讓既有方法難以重現，也難以在實務上部署。

## 資料集與情境

- 資料從哪裡來？
    - 沒有使用靜態資料集。所有樣本 $(x, y)$ 都是在調校過程中實際跑 benchmark 產生的。
    - 三個 benchmark：Sysbench（OLTP）、TPC-C（OLTP，tpcc-mysql）、TPC-H（OLAP，tpch-dbgen）。
- 規模：
    - Sysbench：10 張表 × 每張 200 萬列，50 threads
    - TPC-C：scale 100，32 threads
    - TPC-H：SF 1
    - 搜尋空間：從 MySQL 全部可調參數中，排除與安全性、檔案路徑等非效能相關的變數，篩出 52 個效能相關 knob，以 InnoDB 參數為主（例如 `innodb_buffer_pool_size`、`innodb_read_io_threads`、`innodb_purge_threads`）
    - 每個設定重複 5 次（不同 seed，每次重開 DBMS），單次調校超過 10 小時、需要兩台機器
- 任務：
    - OLTP：最大化 TPS
	- OLAP：最小化查詢批次的總執行時間
- 限制：
    - 只測了單一 DBMS（MySQL 8）、單一硬體（4 vCPU / 16 GB VM，Orange FlexEngine）
    - 只用合成 benchmark，沒有真實 production workload
    - 量測雜訊：領域雜訊大，**系統雜訊變異 $<<$ 演算法隨機性**
- 系統雜訊（Table 1，預設組態重跑 10 次）：：網路壅塞、OS/VM 背景活動、benchmark 本身的隨機性、CPU throttling

| Benchmark | Mean       | SD    | CV    |
| --------- | ---------- | ----- | ----- |
| TPC-H     | 80.13 s    | 0.42  | 0.52% |
| TPC-C     | 79.33 TPS  | 4.95  | 6.24% |
| Sysbench  | 608.63 TPS | 26.53 | 4.36% |
        
- 演算法本身的變異（Table 2，BO 收斂所需時間）：BO 本身有隨機性（初始點、acquisition 的選擇），不同 seed 跑同一個調參，收斂所需的迭代數與時間差很多。

| Benchmark | Iterations（mean ± SD） | 時間（小時）     | 時間 CV |
| --------- | --------------------- | ---------- | ----- |
| TPC-H     | 57.0 ± 41.3           | 1.21 ± 0.9 | 72.9% |
| TPC-C     | 215.6 ± 122.2         | 6.9 ± 4.4  | 64.3% |
| Sysbench  | 118.4 ± 40.3          | 3.4 ± 1.2  | 34.6% |

收斂定義：
- OLTP：令 $t^*$ 為第 100 步之後、過去 100 步內最佳 TPS 改善小於 5% 的第一個 iteration。  
$$P^* = \max_{t<t^*} \text{max\_tps}(t), \qquad Y_{\text{OLTP}} = \min\{t \mid \text{max\_tps}(t) \ge 0.95\,P^*\}$$
- OLAP：門檻改為 1%，
$$Y_{\text{OLAP}} = \min\{t \mid \text{exec}(t) \le 1.01\,E^*\}$$
- 評估協定：
    - 每個設定跑 5 個 seed，每次之間重開 DBMS。
    - 報告 mean ± SD，圖中標示 95% CI。
    - 兩兩比較用 Welch 雙尾 t-test，多方法比較用 Friedman test，顯著水準 $p<0.05$。

## 方法與模型

(使用哪些模型、流程或系統設計？)
```algo
K_curr ← top-k0(K);  D ← ∅;  b ← −∞;  E ← 5·k0
while elapsed < T_max:
	(D, y*) ← BO(K_curr, E, D_proj)        # 在子空間內跑 BO
	K*      ← RFECV(K_curr, D_proj)        # 剪除不重要的 knob
	a       ← LRT_Step(b, y*, E); b ← max(b, y*)      
		# note  b: 目前為止最佳, y*: 目前epoch 最佳
	a = 0 → K_curr ← K*,                 E ← 10       # stay / shrink
	a = 1 → K_curr ← K* ∪ 下 Δk 個 knob,  E ← 5·Δk     # expand
return D 中最佳組態
```
![[Pasted image 20261011155052.png]]

- **主迴圈**（Algorithm 1）：每個 epoch 依序執行 BO → RFECV → LRT。直到 epoch 數 ≥ 最大消耗資源($T_{max}$)
ex .$T_{\max}=200$：

| epoch | LRT 的動作 | $E$ | elapsed（epoch 結束後） | 繼續？          |
| ----- | ------- | --- | ------------------ | ------------ |
| 1     | 初始      | 100 | 100                | 是（100 < 200） |
| 2     | 擴張      | 25  | 125                | 是            |
| 3     | 維持      | 10  | 135                | 是            |
| …     | …       | …   | …                  | …            |
| 最後一個  | —       | —   | ≥ 200              | 否，結束         |

- **BO with GP**（scikit-optimize）：[[Bayesian Optimization]]
    - 若 $\mathcal{D}$ 為空，先隨機取 10 個組態。
    - 先以 $\mathcal{D}_{\text{proj}}$ 擬合 GP，跑 $E$ 次「acquisition function 選點 → adaptive benchmarking 評估 → 加入資料」。
- **RFECV**（scikit-learn）：[[RFE (Recursive Feature Elimination)]]
    - 以 random forest（CART，與 Hunter 相同）作為 estimator，遞迴剔除重要性最低的 knob，並以 CV 分數決定最佳子集大小，不需要人為設定 cutoff。
    - 至少保留 10 個 knob，以維持 BO 的搜尋多樣性。

- **LRT 擴張決策（Algorithm 2）**：
    - stay（$a=0$）與 expand（$a=1$）兩個動作各自維護成功/失敗次數 $(s_a, f_a)$初始值都是 1。
    - 第一次呼叫時，以 Bernoulli(0.5) 隨機決定動作。
    - 之後的決策：  
$$\hat p_a = \frac{s_a}{s_a+f_a}, \qquad \Lambda = \ln\frac{\hat p_1}{\hat p_0}, \qquad a_t = \mathbb{1}[\Lambda > 0]=\begin{cases}1, &\text{if }\Lambda > 0 \\0, &\text{if }\Lambda < 0 \end{cases}$$
    - Reward：  算BO讓最佳效能提升多少比例（$\frac{\text{增加的量}}{原本的量}$），更新$s_a,f_a$ 
$$g_t = \frac{y^*-b}{b\cdot E}, \quad \delta = 0.001, \qquad r_t = \mathbb{1}[g_t > \delta]=\begin{cases}1, &\text{if }g_t>0.001 \\0, &\text{if }g_t≤0.001 \end{cases}$$

	- 依 $r_t$ 更新所選動作的 $s_{a_t}$ 或 $f_{a_t}$。
$$\begin{cases}s_{a_t}\leftarrow s_{a_t}+1, &
\text{if }r_t=1 \\ f_{a_t}\leftarrow f_{a_t}+1, &
\text{if }r_t=0\end{cases}$$
    - 設計動機：
        - $\delta = 0.001$ 在 100 次 iteration 累積下約等於 10% 的改善，高於量測雜訊。
        - 整個規則只需要四個純量，不需要探索排程。
        - 改善小時傾向探索（加入 knob），改善大時傾向利用（留在目前集合）。
- **Adaptive benchmarking**：大部分組態 $x$ 比目前最好的差，看起來贏不了目前最佳的組態不用跑完 $T_{max}$
    - OLTP(Sysbench, TPC-C)：在 $T_{\text{cut}} = 40$ 秒時檢查，若平均 TPS 沒有超過目前最佳值就中止，並以部分結果作為代理值（$y=\text{TPS\_40}$）。
    - OLAP：隨機抽 40% 查詢，先跑 40% 的查詢，若沒有比目前最佳值快，就不跑完整 workload。

| 組態  | 40 秒時的 TPS | 比 1700 | 動作  | 花的時間 | $y$                     |
| --- | ---------- | ------ | --- | ---- | ----------------------- |
| A   | 1500       | 低      | 停   | 40 秒 | 1500（粗略值）               |
| B   | 1800       | 高      | 繼續  | 90 秒 | 最後 30 秒的平均，例如 1760（精確值） |
| C   | 1650       | 低      | 停   | 40 秒 | 1650（粗略值）               |

- **超參數**：$k_0 = 20$、$\Delta k = 5$；有先驗時 knob 清單依專家排序，沒有先驗時隨機排列。

## 實驗結果

**主要結果（Table 8）**

| Workload | 方法      | 效能                 | Tuning 時間         |
| -------- | ------- | ------------------ | ----------------- |
| Sysbench | DOT+EXP | 1,728.3 ± 28.3 TPS | 57.7 ± 28.2 min   |
| Sysbench | BO+EXP  | 1,725.2 ± 31.1 TPS | 217.8 ± 42.7 min  |
| Sysbench | DOT+RAN | 1,724.7 ± 50.6 TPS | 171.1 ± 121.2 min |
| TPC-C    | DOT+EXP | 377.7 ± 16.9 TPS   | 121.6 ± 72.1 min  |
| TPC-C    | BO+EXP  | 354.2 ± 14.8 TPS   | 411.9 ± 264.9 min |
|          |         |                    |                   |
- EXP expert ranking：專家挑 knob
- RAN random ranking：隨機 knob


## Baseline

(和哪些方法比較？, 比較是否合理？)

- 比較對象：
    - BO+**EXP**（專家 top-20）、BO+**CART**（每 knob 10 筆樣本，取樣時間計入）、BO **All knobs**（52 個）
    - Incremental（OtterTune 式：從 top-4 開始，每個 epoch 加 2 個）
    - Statistical Elimination（先跑 $2N$ 次 BO，再以 Welch t-test 剔除 knob）
    - CDBTune cold start（只支援 OLTP）、DB-BERT（只支援 OLAP）、GPTuner 式的 LLM 選 knob
    - 每個方法分兩種情境測試：有專家排名（+EXP）與排名隨機或缺失（+RAN）
- 合理性：
    - 合理之處：
        - 兩種情境分開比較，公平地處理了「有沒有先驗」這個變因。
        - BO+CART 把取樣時間算進總時間，反映真實成本。
        - 所有方法用相同的 52-knob 空間與相同的評估協定。
    - 不合理之處：
        - CDBTune 與 DB-BERT 被強制在 cold start 下執行，偏離原作的設計條件，結果偏弱是預期之內。
        - Hunter、DDPG++、OtterTune 的 workload mapping 都沒有比較。
        - LLM baseline 是作者改過 prompt 的版本，不是 GPTuner 原系統。
        - 「無先驗」情境的搜尋空間仍是專家篩過的 52 個 knob，並非真正沒有先驗。

## Metrics

(用哪些指標？, 是否足以回答研究問題？)

- 使用的指標：
    - 最終效能：Max TPS（OLTP）、執行時間（OLAP）
    - 調校時間：以收斂定義 $Y = \min{t \mid \text{max_tps}(t) \ge 0.95 P^*}$ 計算，OLAP 門檻為 1%
    - 統計檢定：Friedman rank 用於多方法比較，Welch t-test 用於兩兩比較，顯著水準 $p < 0.05$
    - 輔助指標：
        - Jaccard index：knob 集合的穩定度
        - MAPE：部分 benchmark 與完整 benchmark 的誤差
        - knob 數、累積重要性、coverage rate：DOT 的行為分析
        - 各元件的 wall-clock 佔比：系統開銷
- 是否足夠：
    - 效能加上時間兩個維度，可以回答「能否在不降效能的前提下省時間」。
    - 不足之處：
        - 收斂時間的定義依賴事後才知道的 $P^*$，對各方法都偏樂觀。
        - 只看效能峰值，沒有看延遲（p95/p99 latency）或資源使用量。
        - 沒有報告調校過程中套用差組態造成的累積損失（regret），而這對線上調校是重要成本。
        - 「累積重要性」本身來自 CART，屬於循環論證，不能當作 ground truth。

## Ablation / Robustness

(是否有消融實驗？, 是否有穩健性測試或錯誤分析？)

- 消融實驗（Sysbench，無先驗）：
    - 移除 adaptive benchmarking：收斂變慢，最終效能不變。
    - LRT 換成 Thompson Sampling，或完全不做擴張：結果統計相當。這代表 LRT 這個元件本身的貢獻沒有被證明。
    - 每個 knob 的 BO 次數從 5 增加到 10：沒有提升，只是收斂延後。
	    - 初始 $E = 5 \cdot k_0$
	    - expand: $E \cdot \Delta k$ 
- 敏感度分析：
    - $k_0 \in {5, 10, 20, 30}$：收斂速度相近，較大的 $k_0$ 起點較好。
    - $T_{\text{cut}} \in {10, 20, 40, 60}$（TPC-C，雜訊最大的 benchmark）：最終品質差異不大。
- 穩健性：
    - 週期性 workload（TPC-C 與 Sysbench 交替）：DOT 能隨 workload 切換重新收斂，混合訓練得到的組態與專門調 Sysbench 的組態統計上無差異。
    - 量測雜訊與演算法隨機性都在 Section 2 量化過。
- 錯誤分析：
    - 有報告部分 benchmark 的 MAPE 分布（OLTP 的 95th percentile 約 14%）。
    - 但沒有分析 DOT 失敗的案例，例如重要 knob 被 RFECV 誤剪、或 LRT 卡在 stay 的情況。

## Limitation

(作者承認哪些限制？, 還有哪些未說明的限制？)

- 作者承認的限制：
    - 只重複 5 次，原因是成本，作者也承認無法完全消除 bias。
    - Section 3.1 的 grid search 每個組態只跑一次。
    - CART 算出的重要性不等於真實重要性。
    - 混合 workload 時沒有機制區分 workload，所以效能低於專用組態。
    - 近似 benchmark 會讓 surrogate 變弱，需要更多 iteration。
    - 最佳樣本數與 knob 數會隨 workload 改變。
    - Future work：其他 DBMS、真實 workload、workload characterization、多目標最佳化。
- 未說明的限制：
    - **LRT 名不副實**：沒有虛無假設與檢定統計量分布，本質上是對兩臂取經驗成功率較高者的 greedy bandit，而且可能永久鎖死在某一臂。
    - **Reward 公式只適用最大化**：OLAP 是最小化目標，該怎麼處理沒有交代；$b$ 初始為 $-\infty$ 時的計算方式也沒有說明。
    - **被剪除 knob 的值沒有交代**：剪除後 knob 回到預設值還是固定在最佳值，文中沒寫。投影 $\mathcal{D} \to \mathcal{D}_{\text{proj}}$ 會丟掉被剪維度上的資訊，相當於假設該維度對 $y$ 沒有影響。
    - **代理值與真值混用**：早停的代理值（帶條件偏差、且雜訊不同）和完整量測一起餵給假設 homoscedastic noise 的 GP，沒有做雜訊建模。
    - **統計論證不嚴謹**：Table 4 的 Friedman $p = 0.062$ 並不顯著，卻被用來推論約 200 筆樣本是最佳點。
    - **宣稱誇大**：結論寫調校時間縮減「orders of magnitude」，但實測是 21%–74%。
    - **表格疑點**：Table 3 中 LLM 與 Sensitivity 在 Sysbench 的數值完全相同（1411.1），疑似抄寫錯誤；TPC-H 的標紅值與內文敘述不一致。
    - **沒有考慮線上安全性**：調校時直接套用到 DBMS 上的組態可能讓服務中斷或嚴重降級，文中沒有處理。

## 可改進處

(若由自己延伸，應補哪些資料、實驗或討論？)

- 資料與情境：
    - 加入 PostgreSQL，並測試不同硬體規格（記憶體大小直接影響 `innodb_buffer_pool_size` 這類 knob 的最佳值）。
    - 使用真實 trace 或 YCSB 等不同讀寫比例的 workload。
    - 從 MySQL 全部 knob 出發，檢驗「52 個預篩 knob」這個先驗的影響。
- 方法：
    - 把 LRT 換成有理論保證的 bandit（UCB、Thompson），或真正的 sequential test（如 SPRT），並分析 regret。
    - GP 改用 heteroscedastic noise 模型，或把部分量測與完整量測當作 multi-fidelity BO（如 BOHB、MF-GP-UCB），讓早停在理論上站得住。
    - 被剪除的 knob 改為固定在目前最佳值，並和固定在預設值的版本做 ablation。
    - 用 SHAP 或 GP 的 ARD length-scale 取代 CART importance，和 RFECV 比較。
    - 加入 workload characterization，讓週期性 workload 能切換對應的組態。
- 實驗：
    - 增加重複次數，或報告 effect size 與 CI，而不只是 Friedman rank。
    - 加入 regret、tail latency 與安全組態違規次數等指標。
    - 補做 OtterTune（含 workload mapping）與 Hunter 的比較，以及 CDBTune 在 warm start 下的比較。
    - 在 TPC-C 與 TPC-H 也做完整 ablation（目前只做了 Sysbench）。

## 個人結論

- 對我的研究 / 專案有何啟發：
    - **線上特徵選擇與最佳化合併**：用最佳化過程中產生的樣本同時做特徵選擇，可以省掉獨立的取樣階段。這個思路適用於任何「評估很貴、維度很高」的黑箱問題，例如模型超參數搜尋或系統組態調校。
    - **RFECV 的用法**：以 CV 分數決定保留幾個特徵，不必人為設定 top-k cutoff。這可以直接用在傳統 ML 專案的特徵選擇上，例如 House Prices 的特徵篩選。
    - **雜訊環境下的評估協定**：多 seed、每次重置環境、Friedman 加 Welch t-test、報告 CI。這套做法可以直接套用到深度學習實驗，因為模型訓練的隨機性也很大。
    - **早停代理評估**：先用便宜的部分評估過濾，只對有希望的候選做完整評估。概念上等同 successive halving / Hyperband，可用在訓練成本高的模型搜尋。
- 值得引用的地方：
    - Table 1–2：DBMS benchmark 的量測雜訊與 BO 收斂時間的 CV，可作為「調校實驗必須重複」的佐證。
    - Table 5–6：knob 重要性隨 workload 改變，但 knob 集合可以有限度地跨 workload 轉移。
    - Table 7：knob 數量與收斂時間之間的取捨，可佐證 curse of dimensionality 在 BO 中的實際影響。
    - Fig. 13 與 Section 5.8：GP 的計算開銷在 knob 數超過約 20 後快速上升，RFECV 的開銷則幾乎可以忽略（約 0.4%）。
    - Section 2 的收斂定義與評估協定。