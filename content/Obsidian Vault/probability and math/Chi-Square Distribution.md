
$$f(x) = \frac{1}{\Gamma(\frac{n}{2})2^{\frac{2}{n}}}x^{\frac{n}{2}-1}e^{-\frac{x}{2}}, x≥0$$
- parameter: $n=1, 2, \cdots$ 
- mean: $n$
- variance: $2n$
- mgf: $(\frac{1}{1-2t})^{\frac{n}{2}}$ 

NOTE:
1. degree of freedom $n$ : The number of $Z \sim N(0, 1)$ that composition the $\chi^2_n$ , is mean can be freedom in $\mathbb{R}^n$ 
2. $Z_1, \cdots, Z_n \; i.i.d \; \sim N(0, 1)\;,\text{ then }Y=Z_1 + \cdots + Z_n\sim \chi^2_n$ 
3. $\chi^2_n = \Gamma(\frac{n}{2}, \frac{1}{2})$
$$\Gamma(\frac{n}{2}, \frac{1}{2})= \frac{\frac{1}{2}^{\frac{2}{n}}}{\Gamma(\frac{n}{2})}x^{\frac{n}{2}-1}e^{-\frac{x}{2}}=\frac{1}{\Gamma(\frac{n}{2})2^{\frac{2}{n}}}x^{\frac{n}{2}-1}e^{-\frac{x}{2}}$$
4. $X=Z^2 \sim \chi^2_1, Z\sim N(0, 1)$ 
5. $X_1, .., X_k$ be independent, $X_i \sim \chi_n^2$ , $Y = X_1 + \cdots + X_n \sim \chi^2_{n_1+\cdots n_k}$ 