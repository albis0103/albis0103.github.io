避免零機率問題，計算類別機率將每個類別加上常數 $\alpha$ （通常是 $1$ ）

未平滑的條件機率
$$P(\text{word}\; w|\text{category}\;c)=\frac{\text{count}(w,c)}{\sum\text{count}(c',c)}$$

Laplace Smoothing
$$P(\text{word}\; w|\text{category}\;c)=\frac{\text{count}(w,c)+\alpha}{\sum\text{count}(c',c)+\alpha\cdot V}$$
- $\alpha$ : 平滑係數
- $V$：詞彙比大小