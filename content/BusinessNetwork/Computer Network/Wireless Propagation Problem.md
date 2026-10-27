#### EMI (Electromagnetic interference)
：**同頻干擾（co-channel）**，遇到相同頻率到無線電裝置會產生干擾

#### Inverse Square Law Attenuation
：**能量和距離平方成反比 $(\frac{1}{r})^2$**，接收率會隨距離衰減
- 已量測一參考點源 功率密度 $S_1$ ，距離 $r_1$ ，推測另一點 $S_2$  $$S_2 = S_1\cdot (\frac{r_1}{r_2})^2$$
	- 因為點源在距離 r 的功率密度：$S(r)=\frac{P_tG_t}{(4\pi r^2)}$，兩個距離 r₁、r₂ 相除，$\frac{P_tG_t}{4\pi}$ 抵消，所以 $\frac{S_2}{S_1}=(\frac{r_1}{r_2})^2$

- **dB表示**： $S_2(dB)=S_1(dB)-20\log_{10}(\frac{r_2}{r_1})$ 

#### Absorptive Attenuation
：電磁波（訊號）**穿過介質時，能量被轉成熱而損失**，造成訊號強度偏弱$$P(r)=P_0\cdot e^{-\alpha \cdot r}$$
- $\alpha$：會隨頻率變化，由材料的複介電係數決定

#### Shadow Zones
：傳輸時訊號（高頻）因沒辦法穿越障礙物造成傳輸死角

#### Multi-Path inference
：訊號因為反射、繞射、散射，會經不同的延遲跟相位到接收端，導致接收器可能發生疊加干擾
