：傳統最小平方法(OLS, [[Least Squares Method]]) 加入 penalty term，限制model weight

$$L=\sum_{i=1}^n(y_i-\hat y_i)^2+\lambda\sum_{j=1}^pw_j^2$$

$$J(w)=\frac{1}{2}\sum_{i=1}^N(w^T\phi(x_i)-t_i)^2+\frac{\lambda}{2}w^Tw$$