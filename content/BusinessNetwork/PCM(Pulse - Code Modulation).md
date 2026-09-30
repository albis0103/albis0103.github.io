

把類比訊號（連續時間、連續振幅的訊號）$x(t) 轉成數位訊號（離散時間、離散振幅的二進位序列），共三步：

1. **取樣（Sampling）**  
	- 以固定取樣頻率 f_s 取值，得到 $x[n] = x(nT_s)$，其中 $T_s = \frac{1}{f_s}$。  
	- 依 Nyquist–Shannon 取樣定理，需滿足 $f_s ≥ 2·f_{max}$ 才能無混疊地重建訊號。取樣前通常會先接抗混疊低通濾波器。
    
2. **量化（Quantization）**  
	- 把每個 $x[n]$ 映射到 $2^N$ 個離散準位之一（$N$是bit - depth）。
	    - 均勻量化的步階為 $\Delta = \frac{(x_{max} − x_{min})} { 2^N}$。
	    - 量化誤差介於 $±\frac{\Delta}{2}$ 之間。
	    - 對滿幅正弦波，SQNR ≈ 6.02·N + 1.76 dB，也就是每多 1 bit 約增加 6 dB。
3. **編碼（Encoding）**  
    - 把量化索引以 N-bit 二進位碼字輸出，常見格式有 signed integer 和 two's complement。
    

- note : 
	- bit - depth: 用$N$ bit表示，決定振幅方向解析度。not bandwidth($Hz$ )





## 位元率

$R = f_s × N × C$（C 為聲道數）
例如 CD 音訊：44,100 Hz × 16 bit × 2 ch = 1,411.2 kbps。

## 變體

- **LPCM（Linear PCM）**：均勻量化。WAV、AIFF 的標準格式。
- **非均勻 PCM**：使用 μ-law（北美、日本）或 A-law（歐洲、台灣）壓擴，把小振幅的量化誤差壓低。G.711 就是 8 kHz × 8 bit = 64 kbps。
- **DPCM**：編碼相鄰樣本的差值 x[n] − x̂[n]，利用樣本間的相關性來降低位元數。
- **ADPCM**：在 DPCM 的基礎上讓量化步階自適應調整，例如 G.726。

