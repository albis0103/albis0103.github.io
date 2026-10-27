勇於解釋模型預測結果

- Feature importance: Feature attribution方法之一，衡量每個特徵影響層度
**LIME(Local Interpretable Model-agnoistic Explanations)**
	- 想了解黑盒子模型 $f$ 中樣本 $x$，透過把 $x$ 附近資料隨機擾動，觀察對結果影響($\pi_x$, 為kernel base distance)，用擾動過後樣本建立新的資料集，並訓練簡單的可解釋模型
	- $G$：可解釋模型的集合
	- $\mathcal{L}(f, g, \pi_x)$：$g$ 在 $\pi_x$ 加權下對 $f$ 的誤差
	- $\Omega(g)$：懲罰項
	問題：只保證區域性無法解釋全域模型
$$\xi(x) = \arg\min_{g \in G}  \mathcal{L}(f, g \pi_x) + \Omega(g)$$ 
		
**SHAP(SHapley Additive exPlanations)**
：基於博弈論Shapley解釋模型的預測，將所有特徵視為合作遊戲的"玩家"，模型預測視為"報酬"，計算每個特徵在所有可能集合中對預測結果貢獻的加權平均$\phi_i$ 
- 透過可加性，加總 == 模型輸出
$$f(x) = \phi_0 + \sum_{i=1}^{M} \phi_i$$
- $\phi_0$為 基準期望值$E[f(x)]$ 