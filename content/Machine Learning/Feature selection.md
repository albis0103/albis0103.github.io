

### 1. 過濾法（Filter）

與模型無關，先用統計量評分排序，再選 top-k。

**統計派**（線性假設）

- 相關係數（Pearson）：只能抓線性關係，對非線性依賴會低估
- t-test / ANOVA F-test：單變數檢定，忽略特徵間交互作用
- 卡方檢定（Chi-square）：類別型特徵對類別型標籤

**資訊理論派**（非參數，可抓非線性）

- Entropy / Information Gain（IG）：ID3 用的分裂準則，對高基數特徵有偏誤
- Gain Ratio：IG 除以 split info，修正高基數偏誤（C4.5）
- Gini Impurity：CART 用，計算比 entropy 便宜（不用 log）
- Mutual Information（MI）：$I(X;Y) = H(Y) − H(Y|X)$ ，衡量特徵與標籤的非線性依賴

**mRMR（Minimum Redundancy Maximum Relevance）** 不是單純排序法，而是在 MI 基礎上做**組合最佳化**： $$\max_{S} \left[ \frac{1}{|S|}\sum_{x_i \in S} I(x_i; y) - \frac{1}{|S|^2}\sum_{x_i, x_j \in S} I(x_i; x_j) \right]$$ 第一項最大化與標籤的相關性，第二項懲罰特徵間的冗餘（互資訊高代表資訊重疊）。解決了單純用 MI top-k 排序會選到高度相關特徵堆疊的問題。
[[mRMR(Minimum Redundancy Maximum Relevance)]]
### 2. 包裝法（Wrapper）

把特徵子集丟進實際模型跑，用驗證集表現當評分函數。

- Forward Selection / Backward Elimination
- RFE（Recursive Feature Elimination）
- 計算成本高（每個子集都要訓練），但考慮了特徵組合對模型的實際效果，比 filter 準確

### 3. 嵌入式法（Embedded）

特徵選擇跟模型訓練同時發生，選擇準則內建在目標函數裡。

- L1 正則化（Lasso）：稀疏解，係數為 0 的特徵直接被剔除
- 樹模型的 feature importance（RF、XGBoost 的 gain/split count/SHAP）
- Elastic Net：L1+L2 混合，處理高度相關特徵群時比純 L1 穩定

---

三者的權衡是計算成本 vs 準確度：Filter 最快但忽略特徵交互；Wrapper 最準但成本隨特徵數指數增長；Embedded 介於中間，是實務上最常用的。

你這份筆記是要接你 SVM/樹模型那段的延伸，還是獨立在準備一個特徵工程主題?