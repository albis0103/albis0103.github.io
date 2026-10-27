
**input**:$n$ items
- $\mathbf{X} : n \times p$ matrix
- $y$: label (length $n$) 

#### RFE algo
: given feature set $S = \{1,..,p\}$ ，目標特徵數 $k$（hyperparameter: `n_features_to_select`） , 每輪移除 `step` 個特徵
1. 在當前特徵集$S$ 訓練 estimator $f_{\theta}$ （$\text{rfe.fit}(X_S,y)$）
2. 計算每個特徵 Ranking criterion $c_j$ 
	- Linear Model: $c_j = |w_j|$
	- Tree - base Model: Impurity based ex. [[CART(Classification and Regression Tree)]]
3. Remove $c_j$分數最小的$s$ 個特徵
4. repeat 1.~ 3. until $|S| ≤ k$ 


### RFECV(RFE Cross Validation)
: 目標特徵數 $k$ 交由 Cross Validation $CV(m)$ 決定


#### RFECV algo
- CV :切割成 $F$ 個 fold , fold $i=1\cdots F$ ，對每個 fold
	- 訓練集：除了 $i^{th}$ fold, 驗證集：$i^{th}$ fold
	- 在訓練及跑 RFE，特徵數$|S_t|=m$從 $p,p-s,\cdots,k_{min}$ ，每輪紀錄特徵集 $S_t$ 
	- 每個特徵集 $S_t$ 的訓練集訓練 $f_{\theta}$ ，用驗證集算分數 ($CV(m)$)
- 對每個特徵數 $m$ 算 $F$個fold平均$$\text{CV}(m)=\frac{1}{F}\sum_{i=1}^F\text{score}_i(m)$$
- $k^* = \arg \min_m CV(m)$ ，分數最高的特徵數 $m$ 




ex. 
假設 $p=5$、$F=3$、scoring 為 f1：

|特徵數 $m$|fold 1|fold 2|fold 3|$\text{CV}(m)$|
|---|---|---|---|---|
|5|0.80|0.78|0.82|0.800|
|4|0.82|0.80|0.84|0.820|
|3|0.85|0.83|0.86|0.847|
|2|0.81|0.79|0.80|0.800|
|1|0.70|0.68|0.72|0.700|

$\text{CV}(3)$ 最高，所以 $k^*=3$。曲線形狀通常是：特徵太少欠擬合、分數低；移除噪音特徵後上升；之後移除有用特徵又下降。


```python
from sklearn.feature_selection import RFECV
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import StratifiedKFold

selector = RFECV(
    estimator=RandomForestClassifier(n_estimators=200, random_state=0),
    step=1,
    cv=StratifiedKFold(5),
    scoring="roc_auc",
    min_features_to_select=1,
    n_jobs=-1,
)
selector.fit(X, y)

selector.n_features_        # 最佳特徵數 k*
selector.support_           # boolean mask
selector.ranking_           # 1 = 被選中；數字越大越早被淘汰
selector.cv_results_        # mean_test_score, std_test_score
X_sel = selector.transform(X)
```

