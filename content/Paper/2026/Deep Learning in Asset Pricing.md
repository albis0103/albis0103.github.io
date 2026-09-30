# {{Deep Learning in Asset Pricing}}

## 題名與來源

- **論文題名**：Deep Learning in Asset Pricing
- **期刊 / 會議**：Management Science（INFORMS）
- **年份**：正式刊登 2024（Vol. 70, No. 2, pp. 714–750）；線上搶先發表 2023 年 2 月 20 日；投稿於 2020 年 9 月 15 日
- **連結 / DOI**：https://doi.org/10.1287/mnsc.2023.4695

## 研究問題

**作者想解決什麼問題？**

估計個股報酬的 stochastic discount factor（SDF），使其能同時滿足：

| problem                                                                                                  | solution                                                        |
| -------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| SDF依賴大量條件資訊(Firm Characteristics, Macro Variables)                                                       | **FFN** used$w(h_t,I_{t,i})\rightarrow SDF$                     |
| SDF函數形式不固定(CAPM, Fama-French)                                                                            |                                                                 |
| 怎麼描述、構造經濟狀態，SDF個股的$\beta_{t,i}, , w_{t,i}$隨他們變動($M_{t+1} = 1 - \sum_{i=1}^{N} \omega_{t,i} R^e_{t+1,i}$) | **LSTM**將178維micro序列${x_0,..,x_t}$ 壓成$h_t$                      |
| SNR低時會過擬合$\epsilon$ <br>($R^e_{t+1,i}=\mu(x_{t,i}+\epsilon_{t+1,i})$)                                    | 無套利條件loss + GAN $g$ 挑test asset（傳統做法人通挑25 Fama-French Profolio） |



**為什麼這個問題重要？**

- 線性因子模型（Fama-French）被大量文獻證明錯誤設定（factor zoo 問題）
- 現成的機器學習方法是為高 SNR 的預測任務設計，直接套用在低 SNR 的風險溢酬估計上表現不佳。

## 資料集與情境

**資料從哪裡來？**(46個公司特徵、178總體經濟變數)

CRSP（Center for Research in Security Prices）美股全體月報酬資料；46 個公司特徵取自 Kenneth French Data Library 或 Freyberger et al. (2020)；178 個總體經濟序列中 124 個取自 FRED-MD（McCracken & Ng 2016）、46 個為特徵橫斷面中位數、8 個取自 Welch & Goyal (2007)；無風險利率用 Kenneth French Data Library 的一個月期國庫券。

**規模**：

- 時間範圍：1967 年 1 月至 2016 年 12 月，共 50 年
- 切分：訓練 20 年（1967–1986）、驗證 5 年（1987–1991）、測試 25 年（1992–2016）
- 股票數：CRSP 全體約 31,000 檔，**篩選出當月 46 個特徵皆完整者**，約 10,000 檔
- 特徵維度：46 個公司特徵、178 個 macro 變數

**任務**：估計條件 SDF $M_{t+1}$（等價於估計 mean-variance efficient tangency portfolio 權重 $\omega_t$ 與風險暴露 $\beta_t$），用於解釋個股報酬橫斷面、建構效率投資組合、分解系統性與非系統性報酬成分。

**限制**：僅限有完整特徵資料的股票（排除多數極小型/新上市股票），僅美股，樣本止於 2016 年。

## 方法與模型

三個神經網路以無套利條件連結：
![[Pasted image 20260928205228.png]]

1. **SDF network**：經濟狀態（LSTM將 178 維 macro 序列壓成 4 維狀態 $h_t$）+ 股票特質（FFN（$(h_t,I_{t,i})\to\omega$）），最小化 pricing error loss。
2. **Conditional network（對手）**：建構 8 種不同的檢驗方式
3. **GAN 式 minimax 訓練**：三步交替，每步優化至收斂（非傳統 GAN 的少量梯度交替）：(1) $g$ 設為常數估初始 SDF，(2) 固定 SDF 最大化找出最難定價的 $g$，(3) 固定 $g$ 重新最小化更新 SDF。[[GAN (Generative Adversarial Network)]]
$$
\min_{\omega}\max_{g}L(\omega,g)=\frac1N\sum_{i=1}^N\frac{T_i}{T}\left\|\frac1{T_i}\sum_{t\in\mathcal T_i}M_{t+1}R^e_{t+1,i}\,g(I_t,I_{t,i})\right\|^2
$$
$$
\min_{\omega}\max_{g}L(\omega,g)=\frac1N\sum_{i=1}^N\Big\|\,\mathbb{E}\big[M_{t+1}R^e_{t+1,i}\,g(I_t,I_{t,i})\big]\Big\|^2
$$
- note: $M_{t+1}=1-\sum_{j=1}^N\omega(I_t,I_{t,j})R^e_{t+1,j}$ 
$$
\min_{\omega}\max_{g}L(\omega,g)=\frac1N\sum_{i=1}^N\Big\|\,
\underbrace{\mathbb{E}\big[R^e_{t+1,i}\,g(I_t,I_{t,i})\big]}_{\text{只含資料與 }g}
-\underbrace{\mathbb{E}\Big[\Big(\sum_{j=1}^N\omega(I_t,I_{t,j})R^e_{t+1,j}\Big)R^e_{t+1,i}\,g(I_t,I_{t,i})\Big]}_{\text{含 }\omega\text{ 與 }g}
\Big\|^2
$$

## Baseline

（和哪些方法比較？比較是否合理？）

| Baseline              | 說明                                                                  |
| --------------------- | ------------------------------------------------------------------- |
| FFN（Gu et al. 2020）   | 純預測法，最小化 $(R^e_{t+1,i}-\mu(I_t,I_{t,i}))^2$，無 LSTM、無對抗網路，超參數沿用原作者設定 |
| LS                    | SDF 權重線性於特徵，$\omega_{t,i}=\theta^\top I_{t,i}$，closed-form 解        |
| EN                    | LS 加 elastic net 正則化，與 Kozak et al. (2020) 相近                       |
| Fama-French 3 因子、5 因子 | 傳統線性多因子基準，取其 tangency portfolio 最大 SR                               |

比較合理性：四個模型（GAN/FFN/EN/LS）都在完全相同的訓練/驗證/測試切分上重新估計，且各自的超參數都依驗證集獨立選取，這控制了「調參優勢」問題。FFN 超參數沿用原論文設定而非重新搜尋，這點是不對稱的（GAN 做了完整 384 組搜尋，FFN 沒有），可能略微低估 FFN 表現，但作者的理由是「為了與 Gu et al. (2020) 結果可比」。合理但非完全公平的比較。

## Metrics

（用哪些指標？是否足以回答研究問題？）

- **SR**（Sharpe ratio）：$\hat{\mathbb E}[F_t]/\sqrt{\widehat{\mathrm{Var}}(F_t)}$，衡量 SDF 組合的風險調整報酬
- **EV**（explained variation）：類似時間序列 $R^2$，衡量報酬變異被 $\beta$ 解釋的比例
- **XS-$R^2$**（cross-sectional $R^2$）：衡量平均報酬被解釋的比例，這是真正對應「risk premium」的指標

是否足以回答研究問題：作者用附錄模擬明確論證三個指標缺一不可——FFN 可靠極端組合拉高 SR 但不代表 loading 結構正確；線性因子也能有高 SR 但 EV/XS-$R^2$ 較低。三指標組合設計合理。**未提供的**：模型間差值的統計顯著性（無 standard error、無信賴區間），僅 GRS test 在 Table 2 對 $\beta$-sorted portfolios 給出正式假設檢定，且該檢定不是用來比較模型優劣，是用來檢定既有因子模型能否解釋 GAN 產生的組合。

## Ablation / Robustness

（是否有消融實驗？是否有穩健性測試或錯誤分析？）

**消融實驗**：

- Figure 6：拆解 macro 資訊處理方式——GAN(hidden states) vs. UNC（$g$ 設常數）vs. GAN(no macro) vs. FFN/EN/LS(no macro) vs. 四模型(all macro，178 維原樣輸入)。證明 LSTM 壓縮與對抗選 $g$ 兩者都有獨立貢獻。
- Figure 4：簡化例子（僅 size/value/investment 三特徵），拆解「SDF 權重納入哪些特徵」與「test assets 納入哪些特徵」兩件事的個別影響。

**穩健性測試（Section 4.8，多數細節在 online appendix，此 PDF 未附）**：

- 滾動窗口估計 vs. 固定函數形式：兩者高度相關，僅小幅改善（證明函數形式時間穩定）
- 排除小型股、非流動股後模型仍表現良好
- 超參數選擇的穩健性：驗證集上表現最好的幾組超參數，在測試集上本質相同
- 交易摩擦分析（Figure 17）：依市值/價差/週轉率設 cutoff，觀察 SR 隨門檻變化的 trade-off

**未做的穩健性測試**：無 bootstrap 或標準誤估計；無非美股市場的外部驗證；無 2016 年後資料的樣本外測試（受限於發表時間）；未討論 CRSP/Compustat 資料回溯修訂（point-in-time 問題）對「樣本外」定義的影響。

## Limitation

**作者承認哪些限制？**

- 訓練/驗證/測試樣本內外表現落差大（Table 1：訓練 SR 2.68 vs. 測試 0.75），作者承認這暗示一定程度的 overfitting，並主張應看「相對」樣本外表現而非絕對數字
- Section 4.9 明講 trading friction 分析是下界，因為模型未重新估計，只是事後將部分股票權重設零
- 對抗網路（conditional network）的最佳結構其實是 0 層（廣義線性），作者承認模型複雜度主要來自 LSTM 的 macro states 而非深層網路本身
- 估計出的 SDF 的正值性未被明確施加約束，只是觀察樣本內恰好為正（腳註 11）

**還有哪些未說明的限制？**

- $\hat\beta$ 只與真值成比例（需標準化），並非直接可解釋的絕對數值
- 論文對「獨立觀測單位」沒有討論：報酬橫斷面相關（同月股票共同暴露）與序列相關（同股票跨月），論文的評估指標未做 cluster-robust 校正，有效樣本數可能遠小於 $N\times T$
- 完全依賴 CRSP 篩選出「特徵完整」的約 10,000 檔股票，存在存活偏誤／流動性偏誤的可能，雖然穩健性分析部分處理了這點
- 計算成本高（完整估計含超參數搜尋約三天，兩組各 8 張 Titan V 的 GPU 叢集），限制了可複製性與可延伸性

## 可改進處

（若由自己延伸，應補哪些資料、實驗或討論？）

- 補上模型間差值（如 GAN vs. FFN 的 SR 差）的統計檢定，例如用 block bootstrap（保留時間序列相關結構）估計信賴區間
- 補上 2017 年後的樣本外測試，特別涵蓋 COVID-19 期間與後續升息週期，檢驗函數形式是否在極端 regime 下仍穩定
- 補上非美股市場（如台股、其他已開發市場）的外部驗證，檢驗方法的泛化性
- 針對 SDF 正值性給出明確約束或事後檢驗報告（目前僅腳註帶過）
- 補上完整交易成本下的重新估計（而非僅設權重為零的下界分析），評估真實可執行策略的報酬
- 討論資料回溯修訂（point-in-time data）對樣本外宣稱的影響

## 個人結論

**對我的研究 / 專案有何啟發**：

- 你之前討論的房價預測專案與雷達 nowcasting 專案都屬於預測任務；這篇論文的核心訊息「目標函數（loss）的選擇比模型架構的彈性更重要」值得參考——如果你的任務也存在「預測準確 vs. 真正關心的量」之間的落差（例如預測房價的 MSE vs. 決策者真正在意的分位數誤差），思考是否該把領域結構直接寫進 loss，而非只換更深的模型。
- LSTM 處理高維時間序列、先做動態壓縮再輸入下游模型的設計（而非直接餵原始序列或只取差分最後一期），這個思路可以遷移到你的雷達回波時間序列處理上。

**值得引用的地方**：

- Eq. (3)/(4) 的 adversarial GMM 設計，若你之後接觸「如何用神經網路解 GMM/moment condition 問題」相關主題，這是一篇機制清楚的範例
- Table 1、Figure 6 的消融實驗設計方式（拆解每個元件的獨立貢獻），是撰寫自己論文 ablation 章節時可參考的呈現格式