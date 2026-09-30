 https://hackmd.io/@HsuChiChen/probability#單-gt單變數變換連續型法2---分割區間法

|                                                         | $p_X(x) / f_X(x)$                                                    | $m_X(t)$                                  | $E[x]$              | $Var(x)$              |
| ------------------------------------------------------- | -------------------------------------------------------------------- | ----------------------------------------- | ------------------- | --------------------- |
| Bernoulli<br>$X \sim Ber(p)$                            | $p^x(1-p)^{1-x}$                                                     | $1-p+pe^t$                                | $p$                 | $p(1-p)$              |
| Binomial<br>$X \sim B(n, p)$                            | $\binom{n}{x}p^x(1-p)^{1-x})$                                        | $(1-p+pe^t)^n$                            | $np$                | $np(1-p)$             |
| Poisson<br>$X \sim Poi(\lambda), \lambda \triangleq np$ | $e^{-\lambda}\frac{\lambda^x}{x!}$                                   | $e^{\lambda(e^t-1)}$                      | $\lambda$           | $\lambda$             |
| Geometric<br>$X \sim G(p)$                              | $p^t(1-p)^{x-1}$ <br>                                                | $\frac{pe^t}{1-(1-p)e^t}$                 | $\frac{1}{p}$       | $\frac{1-p}{p^2}$     |
| Negative Binomial<br>$X \sim NB(r,p)$                   | $\binom{r}{p}p^r(1-p)^{x-r}$                                         | $(\frac{pe^t}{1-(1-p)e^t})^r$             | $r(\frac{1}{p})$    | $r\frac{1-p}{p^2}$    |
| Uniform<br>$X \sim U[a,b]$                              | $\frac{1}{b-a}$                                                      | $\frac{1}{b-a}\frac{1}{t}(e^{bt}-e^{at})$ | $\frac{a+b}{2}$     | $\frac{(b-a)^2}{12}$  |
| Gaussian<br>$X \sim N(\mu, \sigma^2)$                   | $\frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x-\mu)^2}{2\sigma^2}}$        | $e^{\mu t}+\frac{1}{2}\sigma^2t^2$        | $\mu$               | $\sigma^2$            |
| Exponential<br>$X \sim Exp(\lambda)$                    | $\lambda e^{-\lambda x}$                                             | $\frac{\lambda}{\lambda-t}$               | $\frac{1}{\lambda}$ | $\frac{1}{\lambda^2}$ |
| Gamma<br>$X \sim Gamma(\alpha, \lambda)$                | $\frac{\lambda^{\alpha}}{\Gamma{\alpha}}e^{-\lambda x}x^{\alpha -1}$ | $\frac{1}{(1-\beta t)^{\lambda}}$         | $\alpha \lambda$    | $\alpha \lambda^2$    |

[[Joint Distribution]]