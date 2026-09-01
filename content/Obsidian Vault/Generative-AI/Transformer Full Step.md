![[Pasted image 20260715214742.png]]


- $X \in \mathbb{R}^{seq \times d_{model}}$：輸入序列的 embedding 矩陣
- $d_{model}$：模型隱層維度；$d_k, d_v$：單一 head 的 key/value 維度（通常 $d_k = d_v = d_{model}/h$）
- $h$：head 數量
- $|V|$：字彙表大小



## Input Embedding

$$ \text{Input} = \text{TokenEmb}(x) + \text{PE}(pos) $$

- **TokenEmb**：查表，embedding matrix $W_E \in \mathbb{R}^{|V| \times d_{model}}$
- **PE**(Position Encoding)：固定的 sin/cos 位置編碼，注入位置資訊（因為 self-attention 沒有位置資訊（permutation equivariant））
- 兩者逐元素相加，shape：$[seq, d_{model}]$

---

## Encoder Block

### Self-Attention（single head）

$$ W^Q, W^K, W^V \in \mathbb{R}^{d_{model} \times d_k}, \quad Q = XW^Q,\ K = XW^K,\ V = XW^V $$

$$ \text{Attention}(Q, K, V) = \text{Softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V $$

step：

1. $QK^\top$：每個 query 對所有 key 做點積，shape $[seq, seq]$，數值代表語義相關性
2. $/\sqrt{d_k}$：假設 $q_i, k_i \sim N(0,1)$ i.i.d.，則 $\text{Var}(q^\top k) = d_k$，會隨維度增大而發散，導致 softmax 梯度消失，除以 $\sqrt{d_k}$ 把方差校正回 1。**$d_k$ 是單一 head 的維度，不是 $d_{model}$**
		$Var(x * q^Tk)=x^2Var(q^Tk)$ 
3. Softmax：沿 key 維度歸一化，每列（row）加總為 1
4. Self-Attention：建立各 token 與上下文的關聯
5. output attention score matrix : $\alpha=\sum_j\alpha_{ij}v_j$ 

### Multi-Head Attention

$$ W^Q_i, W^K_i, W^V_i \in \mathbb{R}^{d_{model} \times d_k}, \quad i = 1,\dots,h $$

$$ \text{head}_i = \text{Attention}(XW^Q_i, XW^K_i, XW^V_i) $$

$$ Z_1 = \text{Concat}(\text{head}_1, \dots, \text{head}_h),W^O, \quad W^O \in \mathbb{R}^{h d_k \times d_{model}} $$

$h$ 個 head 平行運算後拼接，再投影回 $d_{model}$，得到 $Z_1$。

###  Add & Norm

$$ Z_1' = \text{LayerNorm}(X + Z_1) $$

- 殘差連接讓梯度可直接求導，緩解梯度消失：

$$ \frac{\partial L}{\partial X} = \frac{\partial L}{\partial (X + F(X))} \cdot \left(1 + \frac{\partial F(X)}{\partial X}\right) $$
- 無 Residual Connection
	$\frac{\partial L}{\partial X}=\frac{\partial L}{\partial y_n}\frac{\partial y_n}{\partial y_{n-1}}\dots\frac{\partial y_1}{\partial x}$ 

- LayerNorm 對**單一 token 的特徵向量**（沿 $d_{model}$ 維度）做正規化，不跨 batch、不跨 token

###  Feed-Forward Network

$$ \text{FFN}(x) = \text{ReLU}(xW_1 + b_1)W_2 + b_2 $$

- 先升維再降維，逐 token 獨立運算（position-wise），token 之間不互相影響
- 再做一次 Add & Norm：

$$ H_{enc} = \text{LayerNorm}(Z_1' + \text{FFN}(Z_1')) \in \mathbb{R}^{seq \times d_{model}} $$

Encoder block 共 **2 個 Add & Norm**（self-attn 後、FFN 後），整個 block 重複堆疊 $N$ 層。

---

## Decoder Block

Decoder 輸入序列的來源，training 與 inference **不同**（見第 5 節），此處先給單一時間步的計算流程。

### Masked Self-Attention

對 decoder 輸入做 self-attention，但加上 causal mask：將注意力矩陣中 $j > i$ 的位置填為 $-\infty$，softmax 後變 0，確保位置 $i$ 只能看到 $\le i$ 的 token。

$$ Z_1 = \text{LayerNorm}\big(X_{dec} + \text{MaskedMultiHeadAttn}(X_{dec})\big) $$

### Cross-Attention

$$ Q = Z_1 W^Q \ (\text{來自 decoder}),\quad K = H_{enc}W^K,\quad V = H_{enc}W^V \ (\text{來自 encoder 輸出}) $$

$$ \text{CrossAttn}(Q,K,V) = \text{Softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V $$

$$ Z_2 = \text{LayerNorm}(Z_1 + \text{CrossAttn}(Q,K,V)) $$

Query 來自 decoder 當前狀態（$Z_1$，不是 decoder 最終輸出），Key/Value 來自**整個** $H_{enc}$。

### Add & Norm + FFN

$$ H_{dec} = \text{LayerNorm}\big(Z_2 + \text{FFN}(Z_2)\big) \in \mathbb{R}^{seq \times d_{model}} $$

Decoder block 共 **3 個 Add & Norm**（masked self-attn 後、cross-attn 後、FFN 後），比 encoder 多一個，對應多出的 cross-attention sublayer；整個 block 重複堆疊 $N$ 層。

---

## Output Projection

$$ \text{Logits} = H_{dec}, W_{vocab}, \quad W_{vocab} \in \mathbb{R}^{d_{model} \times |V|} $$

$$ P(\text{word}) = \text{Softmax}(\text{Logits}) $$

$W_{vocab}$ 常與 $W_E$（token embedding）共享權重（tied embeddings），節省參數量。

---

## Training vs Inference：Decoder 輸入的差異

這是「full step」裡容易被忽略、但結構上不可省的一環：

|                     | Decoder 輸入                                | 計算方式                                                                 |
| ------------------- | ----------------------------------------- | -------------------------------------------------------------------- |
| **Training**        | Ground truth $y_{1:t-1}$（teacher forcing） | 所有時間步可平行計算，$P(y_1,\dots,y_n\|x) = \prod_{t=1}^n P(y_t\|y_{1...t-1})$ |
| **Inference (AT)**  | 模型自己上一步輸出 $\hat y_{1:t-1}$                | 必須逐步 autoregressive 生成，從 `<BOS>` 到 `<EOS>`                           |
| **Inference (NAT)** | 多個位置 `<BOS>` 平行輸出                         | $P(y_1,\dots,y_n\|x) = \prod_{t=1}^n P(y_t\|x)$，各位置條件獨立              |

**風險**：train 時輸入是乾淨的 ground truth，infer 時輸入是模型自己的（可能有誤的）輸出。一旦某步輸出偏差，該偏差會作為下一步的輸入被放大，尤其在小樣本 finetune、遇到訓練集未涵蓋的詞彙時，容易造成 hallucination——這正是 exposure bias 問題。

---

## 6. 完整流程總覽

```
Encoder（重複 N 層）:
  X_enc = TokenEmb(x) + PE(pos)
  Z1    = LayerNorm(X_enc + MultiHeadSelfAttn(X_enc))
  H_enc = LayerNorm(Z1 + FFN(Z1))

Decoder（重複 N 層；輸入依 training/inference 而異，見第 5 節）:
  X_dec = TokenEmb(y) + PE(pos)
  Z1    = LayerNorm(X_dec + MaskedMultiHeadSelfAttn(X_dec))
  Z2    = LayerNorm(Z1 + CrossAttn(Q=Z1, K=H_enc, V=H_enc))
  H_dec = LayerNorm(Z2 + FFN(Z2))

Output:
  Logits  = H_dec · W_vocab
  P(word) = Softmax(Logits)
```