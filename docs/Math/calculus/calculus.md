# 微积分

这里记录微积分中的典型题、解题技巧和易错点。当前重点是积分技巧，尤其是换元、分部积分和递推公式。

## 积分技巧

### 分式导数结构

基础篇 p116 例10。

下面这个题要注意：整体形式接近一个分式的导数，不能只看分子局部。

$$
\int \frac{xe^{-x}}{(1+e^{-x})^2} dx
$$

### 三角换元

基础篇 p117 例11。积分可以拆开处理，也可以合并后再换元。

题目：

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

## 递推公式

下面是同类积分的通项递推公式，后续需要补完整推导。

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

## 重要结论

### 高斯积分

$$
\int_{-\infty}^{+\infty} e^{-x^2} dx = \sqrt{\pi}
$$

## 待补充

- 分部积分法常见套路。
- 三角换元适用形式。
- 高斯积分推导。
- 多元微积分和梯度相关内容。
