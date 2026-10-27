## 1. Spatial Encoder

**功能**：對輸入序列中的每一幀獨立做空間特徵提取與下採樣，跨幀共享同一組權重。

**輸入輸出**：

- 輸入：$\mathbf{X} \in \mathbb{R}^{T \times C \times H \times W}$，$T$ 為輸入幀數，$C$ 通常為 1（灰階，如 Moving MNIST）或 3（RGB）
- 先 reshape 成 $(T \cdot C) \times H \times W$，把時間維度併入 batch 維度處理，讓每一幀都能以相同 2D 卷積獨立處理
- 輸出：$\mathbf{Z}_{enc} \in \mathbb{R}^{(T \cdot C_{hid}) \times H' \times W'}$，其中 $H' = H/2^{N_s}$，$N_s$ 為下採樣次數

**內部結構**：

- 由 $N_s$ 層的 **ConvSC（Spatial Convolution）模組**堆疊而成，每個 ConvSC 是：`Conv2d → GroupNorm → LeakyReLU`
- 下採樣策略：一般是奇數層 stride=2、偶數層 stride=1（或依實作交替），讓解析度逐步減半，同時保留部分不下採樣的層增加非線性深度
- 卷積核多為 3×3

**作用本質**：把原始像素空間映射到一個更緊湊、語義層次更高的特徵空間，同時降低後續 Mid-Xnet 要處理的空間解析度（因為時空聯合建模在原始解析度上計算量太大）。

---

## 2. Temporal Translator (Mid-Xnet)

**功能**：這是 SimVP 真正學習「動態演化」的核心模組，負責從過去 $T$ 幀的特徵推斷未來的時空特徵。

**輸入輸出**：

- 輸入：把 encoder 輸出的 $T$ 幀特徵沿 channel 維度拼接（reshape），得到 $\mathbf{Z}_{enc} \in \mathbb{R}^{(T \cdot C_{hid}) \times H' \times W'}$，此時「時間」已經完全融入 channel 維度，不再有獨立的時間軸
- 輸出：同形狀的 $\mathbf{Z}_{trans} \in \mathbb{R}^{(T \cdot C_{hid}) \times H' \times W'}$（在原版 SimVP，輸出幀數與輸入幀數相同，即 $T_{out} = T_{in}$）

**內部結構（Inception-like，如前所述）**：

- 是一個 **U-Net 形狀的 Encoder-Decoder**（注意：不是簡單堆疊，而是有跳接的沙漏結構）
- Encoder 側：由多層 **gInception (group Inception) 模組** 組成，每層先用 1×1 group conv 調整 channel，再並聯多個不同 kernel size（如 3, 5, 7, 9, 11）的 group convolution 分支，取代標準卷積以降低計算量，最後將各分支輸出相加（不是 concat）
- Decoder 側：對稱結構，並與 encoder 側對應層做 **skip connection**（concat 後接 1×1 conv 融合），這是典型 U-Net 設計，用來保留不同深度的時空資訊
- 每個 gInception block 後接 GroupNorm + LeakyReLU

**關鍵特性**：因為時間已被吸收進 channel 維度，這裡的卷積操作等價於同時對「空間鄰域」和「時間鄰域（不同幀對應的 channel 群組）」做混合，這就是 SimVP 用純 2D 卷積達成時空建模的核心機制——不需要 3D 卷積、不需要遞迴。

---

## 3. Spatial Decoder

**功能**：與 encoder 對稱，把 Mid-Xnet 輸出的低解析度時空特徵上採樣回原始 $H \times W$，重建出預測幀的像素值。

**輸入輸出**：

- 輸入：$\mathbf{Z}_{trans} \in \mathbb{R}^{(T \cdot C_{hid}) \times H' \times W'}$
- 輸出：$\hat{\mathbf{Y}} \in \mathbb{R}^{T \times C \times H \times W}$，reshape 回原始的時序幀格式

**內部結構**：

- 由 $N_s$ 層 **ConvSC（transposed 版）** 組成：`ConvTranspose2d（或 PixelShuffle）→ GroupNorm → LeakyReLU`，逐層將解析度加倍還原
- 與 Spatial Encoder 之間有 **skip connection**：encoder 最淺層（未下採樣前）的特徵會直接連到 decoder 最後一層，用 concat 方式融合，幫助恢復高頻細節（邊緣、紋理），避免因下採樣造成的資訊損失
- 最後一層通常接一個 1×1 conv 將 channel 數壓回 $C$（原始輸入的通道數），作為最終輸出

**作用本質**：Encoder 是「壓縮＋抽象」，Decoder 是「還原＋重建」，兩者透過 skip connection 形成一個完整的 U-Net-like autoencoder，中間插入 Mid-Xnet 負責真正的時序推理。

---

### 三者串接的資料流總結

$$\mathbf{X}_{T \times C \times H \times W} \xrightarrow{\text{Encoder（Share Framed）}} \mathbf{Z}_{enc} \xrightarrow{\text{reshape Time→channel}} \xrightarrow{\text{Mid-Xnet}} \mathbf{Z}_{trans} \xrightarrow{\text{reshape channel→時間}} \xrightarrow{\text{Decoder（+skip from encoder）}} \hat{\mathbf{Y}}_{T \times C \times H \times W}$$

需要我針對雷達回波外推任務，說明這三個模組在輸入通道數（例如多雷達站、多仰角）和 loss function 設計上需要做哪些調整嗎？