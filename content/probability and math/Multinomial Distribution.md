
Multinomial : is Binomial $2 \rightarrow k$ extension

- $n≥1$ and $n_1, \cdots , n_k≥0$ for which $n_1 + \cdots +n_k =n$ 
- the set of $n$ elements may be partitioned in $m$ subsets of size $n_1, \cdots, n_k$


$$\mathbf{X} \sim \text{Multinomial}(n, \mathbf{p}) = \text{Multinomial}(n,k,p_1,\cdots,p_k)$$

Condition：$\sum_{i=1}^k X_i = n$，$x_i \in \{0, 1, \dots, n\}$
 
### PMF
for each sequence ( ex. $x_1=1, x_2=2, ..$) Probability: $p_1^{x_1} p_2^{x_2} \cdots p_k^{x_k}$(Independent)


$$P(X_1=x_1, \dots, X_k=x_k) = \frac{n!}{x_1! \cdots x_k!} \prod_{i=1}^k p_i^{x_i}$$

when $k=2$: $\binom{n}{x_1} p_1^{x_1} p_2^{n-x_1}$。

### MGF

$$M_{\mathbf{X}}(\mathbf{t}) = E\left[e^{\sum_i t_i X_i}\right] = \left(\sum_{i=1}^k p_i e^{t_i}\right)^n$$

推導：直接對 PMF 求和，套用多項式定理（multinomial theorem）：

$$\sum_{x_1+\cdots+x_k=n} \frac{n!}{\prod x_i!} \prod_i (p_i e^{t_i})^{x_i} = \left(\sum_i p_i e^{t_i}\right)^n$$

## 期望值、變異數與共變異數

對邊際分布 $X_i \sim \text{Binomial}(n, p_i)$（將其餘類別合併視為「非 $i$」的兩類問題），可直接得：

$$E[X_i] = np_i, \qquad \text{Var}(X_i) = np_i(1-p_i)$$

共變異數（$i \neq j$）：

$$\text{Cov}(X_i, X_j) = -np_i p_j$$

推導共變異數的關鍵技巧：利用指示變數。設第 $t$ 次試驗中 $I_{t,i} = 1$ 若結果為 $i$，則 $X_i = \sum_{t=1}^n I_{t,i}$。單次試驗中 $E[I_{t,i} I_{t,j}] = 0$（$i\neq j$ 互斥），故

$$\text{Cov}(I_{t,i}, I_{t,j}) = 0 - p_i p_j = -p_i p_j$$

由獨立同分布試驗加總（共變異數具可加性）：

$$\text{Cov}(X_i, X_j) = n \cdot (-p_i p_j)$$

負相關的直覺：$n$ 固定，某類別次數增加必然壓縮其他類別的次數空間。

共變異數矩陣可寫成緊湊形式：

$$\Sigma = n\left(\text{diag}(\mathbf{p}) - \mathbf{p}\mathbf{p}^\top\right)$$

此矩陣是奇異的（rank $k-1$），因為 $\sum_i X_i = n$ 這個線性約束消去一個自由度。

## 最大概似估計（MLE）

給定單次觀測 $\mathbf{x} = (x_1, \dots, x_k)$，對數概似函數：

$$\ell(\mathbf{p}) = \log n! - \sum_i \log x_i! + \sum_i x_i \log p_i$$

在約束 $\sum_i p_i = 1$ 下用 Lagrange 乘子法：

$$\mathcal{L} = \sum_i x_i \log p_i + \lambda\left(1 - \sum_i p_i\right)$$

對 $p_i$ 求偏導並令為零：

$$\frac{x_i}{p_i} - \lambda = 0 ;\Rightarrow; p_i = \frac{x_i}{\lambda}$$

代入約束 $\sum p_i = 1$ 得 $\lambda = \sum_i x_i = n$，故

$$\hat{p}_i = \frac{x_i}{n}$$

即樣本比例，符合直覺。

## 與其他分布的關係

- **邊際分布**：$X_i$ 的邊際為二項分布 $\text{Binomial}(n, p_i)$。
- **條件分布**：給定 $X_i = x_i$，剩餘 $(X_j)_{j\neq i}$ 服從參數為 $(n-x_i, p_j/(1-p_i))$ 的多項分布（重新歸一化）。
- **Dirichlet 共軛**：在貝葉斯設定下，多項分布的共軛先驗是 Dirichlet 分布——這與你先前研究的共軛先驗主題直接相關，$\text{Dirichlet}(\boldsymbol\alpha)$ 先驗經多項概似更新後，後驗仍為 $\text{Dirichlet}(\boldsymbol\alpha + \mathbf{x})$。
- **Poisson 關係**：若 $Y_i \stackrel{iid}{\sim} \text{Poisson}(\lambda_i)$ 獨立，則在 $\sum Y_i = n$ 條件下，$(Y_1,\dots,Y_k) \mid \sum Y_i = n$ 服從 $\text{Multinomial}(n, p_i = \lambda_i/\sum\lambda_j)$。這是 Poisson 過程分拆的重要性質，也是 Poisson 迴歸與多項邏輯迴歸等價性的基礎。

## 應用場景

- Naive Bayes 文本分類（詞頻建模）
- Softmax 迴歸的概似基礎（多類別分類的生成模型解釋）
- 交叉熵損失的機率詮釋：單樣本 one-hot 標籤下的多項 MLE 即對應 softmax + cross-entropy 的訓練目標