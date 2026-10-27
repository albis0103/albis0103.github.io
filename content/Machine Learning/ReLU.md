
**ReLU(Rectified Linear Unit)**
	$ReLU(x)=max(0, x)$

- problem: Dying ReLU: some neural input negative value will never be activation
```juliaplots
f(x) = x > 0 ? x : 0
title = ReLU(x)
x_lable = x
y_lable = ReLU(x)
xmin = -5
xmax = 5
```

**Leaky ReLU**
	$\text{Leaky ReLU(x)}=\begin{cases}x, & \text{if x} ≥ 0 \\ \alpha x , & \text{if x} < 0 \end{cases}$ 
		$\alpha$ is constant, comment value 0.01
```juliaplots
f(x) = x > 0 ? x : 0.1*x

xmin = -5
xmax = 5
title = Leaky ReLU
x_label = x
y_label = f(x)
```


**PReLU(Parametric ReLU)**

$PReLU(x)=\begin{cases} x, & \text{if x≥0} \\ \alpha x, & \text{if x < 0}\end{cases}$ 
$\alpha$ : learnable parameter, not fixed constant
```juliaplots
f(x) = x > 0 ? x : 0.1*x

xmin = -5
xmax = 5
title = Leaky ReLU
x_label = x
y_label = f(x)
```

