: 常用於RLHF, 用來限制 Trainable policy model 不要偏離 Reference policy model 太遠的正則化項
- $\pi_\theta$ :Trainable policy, Training currently model, parameter is $\theta$ 
- $\pi_{ref}$ :Reference, the fixed model weight, between SFT and RL training stage
- $V$ :Embedding model

$D_{KL}(\pi_\theta(\cdot |s)||\pi_{ref}(\cdot |s))=E_{ \pi_\theta}[log\pi_\theta(x)-log\pi_{ref}(x)] =\sum_{x \in V} \pi_\theta(x|s)log\frac{\pi_\theta(x|s)}{\pi_{ref}(x|s)}$ 

RLHF/ PPO Objective function
$Obj=E[reward(x, y)]-\alpha \cdot D_{KL}(\pi_\theta||\pi_{ref})$ 
- $reward(x, y)$:score from reward model
- $\alpha$: KL penalty 係數(coefficient),控制正則化強度

-- problem: $D_{KL}$ ~ Uniform Distribution, $\pi_{ref}$ not certain True
add weight factor $\hat{J_{\theta_{ref}}}(s)^\beta$ 
- $\hat{J_{\theta_{ref}}}(s) \in [0, 1]$ : 
	- $H(\pi_{ref}(\cdot | s))$ low: reference policy 很確定答案（常見詞, ..etc）
	- $H(\pi_{ref}(\cdot | s))$ high: reference policy 不確定答案，decrese KL penalty,  $\hat{J_{\theta_{ref}}}(s)^\beta \rightarrow 0$ 
- $\beta$ : 控制$\hat{J_{\theta_{ref}}}(s)$ slope

$L_{KL}=E_{s,x \sim \pi_\theta}[\hat{J_{\theta_{ref}}}(s) \cdot log\frac{\pi_\theta(x|s)}{\pi_{ref}(x|s)}]$  
$\hat{J_{\theta_{ref}}}(s)=\frac{H_{max}-H(\pi_{ref}(\cdot | s))}{H_{max}}$ 