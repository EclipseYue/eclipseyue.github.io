# 2025美赛纪实

## 1.20 tex细节

* .cls文件中有\newcommand和一些宏定义信息，可以修改

* 需要查明编译顺序才能把某些??编译出来(也可以添加setting.json)

* 引用配套的\cite{}和\bibitem{}


## 1.21 

### 阅读论文

### 算法

#### 元胞自动机
参考[元胞自动机](https://blog.csdn.net/Hjh1906008151/article/details/130313590)。

元胞空间 邻居模式(冯诺依曼 Moore 扩展moore)

**步骤：**
* 建立假设、简化条件。
* 设置初始状态和边界。
* 推导状态转方程和条件。
* 根据时间迭代。

**例子：**
* 奇偶游戏
* 生灭游戏
* 森林火灾

#### LSTM
参考[LSTM](https://blog.csdn.net/mary19831/article/details/129570030)

RNN示意图

RNN梯度爆炸(长时间序列的缺点)

>损失函数 $L = \frac{1}{2} (Y-O)^2$
> 损失函数的偏导： 
> $$
\frac{\partial L_{t}}{\partial W_{x}}=\sum_{p=1}^{t} \frac{\partial L_{t}}{\partial D_{t}} \times \frac{\partial O_{t}}{\partial S_{t}} \times\left(\prod_{i=t+1-k}^{t} \frac{\partial \tanh \left(\theta_{i}\right)}{\partial \theta_{i}}\right) X_{t+1-k} W_{s}^{k-1}
>$$ 
> 注意到最后的$W_s^{k-1}$容易出现梯度消失/梯度爆炸

所以引入LSTM

![An image caption](assets\images\LSTM.png)

* 遗忘门：根据输入信息和隐藏信息得出遗忘信号，用于使$C_{t-1}$遗忘
$$
f_t = \sigma( W_f \cdot[h_{t-1},x_{i}] + b_f)
$$
* 输入门：之前留下的信息和现在需要记住的信息相加，得到新的记忆状态
$$
i_t = \sigma(W_i \cdot [h_{t-1},x_i] + b_i)
\\
C_i =tanh (W_c \cdot [h_{t-1},x_t]+b_f)
$$
* 输出门：整合$C_t$得到最终输出(其中$h_t$代表实际)
$$
C_t = f_t * C_{t-1} + i_t * \bar{C_t}
\\
\sigma_t = \sigma(W_o \cdot [h_{t-1},x_t]+ b_o)
\\
h_t = o_t*tanh(C_t)  
$$

参数理解：
[LSTM参数理解](https://blog.csdn.net/baidu_38963740/article/details/117197619)

时间序列理解：
[https://zhuanlan.zhihu.com/p/36455374]





