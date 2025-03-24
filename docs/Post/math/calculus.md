# 微积分
这里用来记录一些题目

基础篇 p116 例10

基础篇 p117 例11 (积分可以分开，也可以合起来)

下面这个注意整体是分式的导数
$$
\int \frac{xe^{-x}}{(1+e^{-x})^2} dx
$$

下面这个是一个三角换元
$$
\int \frac{1}{(x^2+1)^2} dx = ? 
$$

$$
令 x = tan(\theta), dx = sec^2(\theta)d(\theta)
$$

$$
= \int \frac{sec^2(\theta)}{(sec^2(\theta))^2} d(\theta) 
= \int \frac{1}{sec^2(\theta)}d(\theta) 
$$

$$
= \int cos^2(\theta)d(\theta) 
= \int \frac{1+cos(2\theta)}{2} d(\theta)
$$

$$
= \frac{\theta}{2} + \frac{sin(2\theta)}{4} + C
$$

$$
由于 sin(2\theta) = \frac{2tan(\theta)}{1+tan^2(\theta)} = \frac{2x}{1+x^2}
$$

$$
= \frac{arctan(x)}{2} + \frac{x}{2(1+x^2)} + C 
$$

有一个通项公式(尚未推导)

$$

I_m = \int \frac{1}{(x^2+a^2)^m} dx
$$

$$
由于分部积分法 \\
= \frac{x}{2a^2(m-1)(x^2+a^2)^{m-1}} + \frac{2m-3}{2a^2}I_{m-1}, m>1
$$

$$
最终I_1 = \frac{arctan(\frac{x}{a})}{a} + C
$$
