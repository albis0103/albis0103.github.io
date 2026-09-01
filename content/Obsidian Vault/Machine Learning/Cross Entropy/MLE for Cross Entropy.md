MLE(Maximum Likelihood Estimate)目標：

資料集：{ $(x_1, y_1),..,(x_n,y_n)$}

給模型樣本x預設y的最大聯合概率

$L(\theta)=max_\theta \prod_iq(y_i|x_i;\theta)$
$=max_\theta \sum_i log(q(y_i|x_i))$
$=min-\sum_i log(q(y_i|x_i))$
