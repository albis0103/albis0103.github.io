：為了了解自變數(independent)的數值或變化量對依變數(dependent)產生的數值或變化量(effect)

- 簡單線性回歸:一個 independent & dependent variable
- 多元回歸：超過兩個 independent variable 一個dependent variable
- 多變量回歸分析：多個 independent & dependent variable

note:

- correlation analysis: 描述兩變數間關係的方向跟強度，變數 $\in$ Random Variable
- Regression analysis：分析自變數跟依變數之間影響程度

$y_i=\beta_0 +\beta_1x_i + \epsilon_i, i= 1,..,n$

Regeression Param

- $\beta_0$：截距、常數項
- $\beta_1$：斜率
- $y_i, x_i, \epsilon_i：i^{th}$觀察 依變數\自變數\隨機變數(隨機誤差） 的觀測值

$E[y_i]=\beta_0 + \beta_1x_1$：

$i=1,..,n\\\beta_i$:已知

$\hat{y_i}=b_0+b_1x_1$

$b_i$：參數 $\beta_i$ 的估計值

參數估計方法: MSE(Mean Square Error)

$MSE = min \sum_{i=1}^n(y_i-\hat{y})^2\\=min \sum_{i=1}^n(y_i - b_0-b_1x_1)^2$

特徵

- $\epsilon_i=y_i-\hat{y}, E[\epsilon]=0$
    
    $\sum_{i=1}^n(y_i-\hat{y_i})^2=\sum_{i=1}^n(y_i-b_0-b_1x_1)^2=0$
    
- if $Cov(x_i, \epsilon_i)=0$, $x_i,\epsilon_i$無線性關係
    
    pf:
    $E[x_i \epsilon_i]-E[x_i]E[ \epsilon_i] \\ =E[x_i \epsilon_i]-E[x_i]*0=E[x_i \epsilon_i]$
    
- if $Cov(y_i, \epsilon_i)=0$, $y_i,\epsilon_i$無線性關係
    

SSE( Sum of Squares Error)誤差項平方和

$SSE=\sum_{i=1}^n(y_i-\hat{y_i})^2\\=\sum_{i=1}^n(y_i-b_0-b_1x_1)^2\\=\sum y_i^2-b_0\sum y_i-b_1\sum x_iy_i$

用$\bar{y}$估計y(依變數)

SST(Sum of Squares Total) $y_i-\bar{y}$差距

$SST=\sum_{i=1}^n(y_i-\bar{y})^2\\=\sum y_i^2 - 2 \bar{y} \sum y_i + n \bar{y}^2\\=\sum y_i^2 - 2 \bar{y}n \bar{y}\\=\sum y_i^2 -\frac{(\sum y)^2}{n}$

SSR(Sum of Squares Regression) $\hat{y}-\bar{y}$差距

$SST=\sum_{i=1}^n(\hat{y}-\bar{y})=SST-SSE\\= \frac{[\sum x_iy_i - \frac{[\sum x_iy_i-\frac{\sum x_i \sum y_i}{n}]^2}{}]^2}{\sum x_i^2 - \frac{(\sum x_i)^2}{n}}$

$$  
SST = SSR+SSE  
$$

判定係數 $R^2$ 回歸的平方和佔總平方和 比例

$R^2 = \frac{SSR}{SST}$