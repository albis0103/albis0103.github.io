### 基本原理

ML 本質是一個函數，目標是找出最佳參數：

- **Training（訓練）**：透過資料找出模型參數的過程
- **Inference（推論）**：使用已找到參數的模型進行預測

如何表示擁有數百萬參數的函數？→ **Neural Network（神經網路）**

模型能力比喻：

- 參數量 = 天資（先天條件）
- 資料量 = 後天努力（訓練資料）

###  LLM 的生成機制（Autoregressive）

ChatGPT 本質是「文字接龍」，每次依前文預測下一個 token：

$$  
P(x_1, x_2, ..., x_T) = \prod_{t=1}^TP(x_t|x_1, x_2, .., x_{t-1})\\=P(x_1) *P(x_2|x_1)*P(x_3|x_1,x_2)...  
$$

### 三階段訓練流程

| 階段      | 名稱                                | 方法                       | 資料來源                                                                                             |
| ------- | --------------------------------- | ------------------------ | ------------------------------------------------------------------------------------------------ |
| Phase 1 | **Pretrain**                      | Self-supervised learning | Web crawler 大量無標注文字。<br>目標函數是Self-Autoregression Model (ex.gpt)或 Masked Language Model (ex.BERT) |
| Phase 2 | **Instruction Fine-Tuning (IFT)** | Supervised learning      | 人工標注 (instruction → answer) pair                                                                 |
| Phase 3 | **RLHF / RLAIF**                  | Reinforcement Learning   | 人類/AI 偏好回饋，用強化學習方式調整(PPO、DPO演算法)                                                                 |

**下圍棋類比：**

- Phase 1 & 2：看棋譜學習
- Phase 3：發現贏了，提升這些棋步的機率



**LLM 所需的語言知識：**

- 語言知識（Linguistic Knowledge）：1M 資料即足夠
- 詞彙知識（Lexical Knowledge）：越多越好
- 世界知識（World Knowledge）：複雜、多層次，需大量資料

###  Fine-tuning 方法

**Instruction Fine-Tuning 實作路徑：**

1. Phase 1：用 web crawler 資料做 pretrain（可用 LLaMA 初始參數）
2. Phase 2：用 ChatGPT reverse engineering 做 self-instruct 產生標注資料

**Adapter-based Fine-Tuning（如 LoRA）：**

- 凍結原始參數，只update 新訓練的少量參數

**Fine-tuning 策略演進：**

- 打造一堆專才（如 BERT）
- 打造一個通才（如 Google FLAN，收集大量標注資料）

**Pretrain 的遷移能力：** 因為餘弦相似、語料夠大，模型能舉一反三

**RLHF(Reinforcement Learning from Human Feedback)**
steps:
1. SFT(Supervised Fine-Tuning):在pre-training model 用人工標注 (instruction → answer) pair, 透過Supervised learning 產出$\pi_\theta$ policy
2. Reward Model training
3. PPO: 用RM score 做reward, 加入KL penalty 以 PPO 更新policy
		[[KL penalty (Kullback-Leibler)]]
	$max_\theta E[reward(x, y)]-\alpha \cdot D_{KL}(\pi_\theta||\pi_{ref})$ 
**Hallucination**
- Intrinsic: 與Source矛盾
	- Parametric Knowledge Bias：模型參數化知識的Time-boundedness and Long-tail entity 覆蓋不足
	- Algorithmic Bias: 訓練/推論 分布不一致, training used ground truth , but ouput $P(y_t|y_{<t})$ 
- Extrinsic：完全沒有Source依據
	- Sampling Error: top-k/temperature..etc抽樣策略引入不同錯誤分布
	- Confirmation Bias: 人類認知偏誤
reason
- AT(Autoregression):$P(x_t|x<t)$ 只保證local optimize not global optimize
- RLHF標記偏好聽起來自信、完整的回答

### Transformer 架構

語言模型演化：N-gram → Feed-forward Network → RNN → **Transformer**

[[Transformer Full Step]]

**Transformer step：**
![[Pasted image 20260716092648.png]]

1. **Tokenization**：文字 → token（token list 映射到 embedding vector，768D）
    
    ![[Pasted image 20260716092357.png]]
    
2. **Input Layer**：
    
    - Word Embedding（word2vec）
        
    - Positional Embedding（每個位置對應特定向量）
        ![[Pasted image 20260716092447.png]]
        
3. **Attention**：計算所有詞對所有詞的相關性，再加權求和
    
    - GPT 使用 **Causal/Masked Attention**（只考慮左側詞）
    - 前面 attention 計算結果存於 **KV-Cache** 加速推論
4. **Feedforward**：彙整多個 attention module 輸出為單一 embedding
    
    - FFN(x) = Linear → ReLU/GELU → Linear
5. **Output Layer**：linear transformer + softmax → 機率分布
    

### 模型透明度與解釋性

Gen-AI 是 black box，但可從三個維度理解：

|概念|定義|例子|
|---|---|---|
|**Transparency**|開源程度（參數/程式碼/訓練方法）|開源 vs 閉源模型|
|**Interpretable**|思維過程透明可追蹤|Decision Tree ✓ / Transformer ✗|
|**Explainable**|找出影響輸出的關鍵輸入（無標準答案）|Transformer ✓（可做但非完美）|
![[Pasted image 20260716092736.png]]
**分析技術：**

- Shallow Layer：掃描語言結構
- Deep Layer：捕捉語義標籤（正面/負面）
- Hidden State：token embedding 背後隱藏的資訊（如詞性）
- Probing：找出深層 hidden state
- t-SNE / PCA：投影到二維平面觀察