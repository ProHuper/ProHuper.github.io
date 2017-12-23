---
layout:     post
title:      "数据挖掘十大算法 SVM"
subtitle:   "用python实现支持向量机分类"
date:       2017-12-23 20:00:00
author:     "Huper"
tags:
    数据挖掘&机器学习
---

在深度学习盛行前，支持向量机（SVM）可以说是机器学习中的主流技术，SVM于1995年正式发表，并于2000年前后在统计学习中掀起高潮。实际上，SVM的概念在上世纪60年代就已经出现，其理论也是在上世纪70年代左右相对成型，关于核函数的研究会更早。甚至直到深度学习如日中天的今天，SVM还是能展现出其优秀的性能，很多时候也是分类的主要选择，因为像深度学习这种data driven的学习方法，对数据量和数据处理能力的要求是非常高的，但是通常情况下SVM 只需要很小的数据就能找到数据之间分类的`超平面`，得到很不错的分类结果。

由于SVM这块涉及理论的东西比较多，实现起来也很麻烦，所以这篇博客还是整理了很久的，我主要是从`间隔`，`支持向量`，`核函数`，`SMO`，`软间隔`这几点来讲的，理论部分主要参考周大佬的《机器学习》。还有一些衍生出来的问题比如支持向量回归`SVR`等，就留到以后再说吧。

### 间隔与支持向量

我们假设有这样一个线性分类的问题：给定样本集$D=\\{(x_1,y_1),(x_2,y_2),...,(x_m,y_m)\\}$ ，$y_i\in\\{-1,+1\\}$，这是一个典型的二分类问题，在样本空间$D$中找到一个划分超平面，将不同类别的样本区分开。但是这样的超平面往往不止一个，如下图所示：

![none](/img/in-post/6.png)

那么我们应该选择哪个超平面来划分数据呢？按照正常的划分思想，我们要保证这个超平面要在数据集的“正中间”，就是图中加粗的那个超平面。这个超平面产生的分类结果对数据的局部扰动敏感度最低，鲁棒性和泛化能力都是最好的。

接下来我们详细研究这个超平面的相关性质，首先我们可以用如下的线程方程来描述它：


$$
\vec{w}^T\vec{x}+b=0
$$



$\vec{w}=(w_1;w_2;...;w_d)$是法向量（其实就是一个系数矩阵），决定了超平面的方向，b是偏移量，决定了超平面和原点之间的距离，因为给定超平面和偏移量就可以确定一个超平面。根据向量空间的相关知识，不难将样本空间任意点$\vec{x}$到超平$(\vec{w},b)$的距离表示为(向量表示有点麻烦，后面的我就不按照向量形式写了)：



$$
r=\frac{|w^Tx+b|}{||w||}
$$



对比二维数据里点到直线的距离很好理解这个公式，上面是一个线性表达式的绝对值，下面是一个向量的二范数。我们继续来看下具体该如何用这个超平面来划分数据：



$$
\begin{cases}\tag1
w^Tx_i+b\geq+1, && y_i=+1;\\
w^Tx_i+b\leq-1, && y_i=-1.
\end{cases}
$$



即对于给定样本$(x_i,y_i)\in D$，若$y_i=+1$，则$w^Tx_i+b>0$。若$y_i=-1$，则$w^Tx_i+b<0$。这里不考虑样本落在超平面上的情况。上式中等号成立的条件对应那些距离超平面最近的点，而所谓的`支持向量`，值得就是这些样本向量，另外我们可以将两个异类`支持向量`距离的和称为`间隔`，这个概念将直接引出`SVM`的优化问题，它可以表示为：



$$
\gamma=\frac{2}{||w||}
$$



这是怎么得到的？实际上就是两个超平面之间的距离（两个平面标准化表示的偏移量之差绝对值，除以方向向量的二范数）乘二，看下图就清楚了：

![none](/img/in-post/7.png)

就像之前说的，我们要找的一个“正中间”的超平面，就是说要使得`间隔`最大，当然是有$w，b$ 的参数限制的，也就是说现在的问题是一个带有约束的最优问题：



$$
max_{w,b}\frac{2}{||w||} \text{ }s.t.\text{ }y_i(w^Tx_i+b)\geq1,\text{ }i=1,2,...,m.
$$



后面的约束条件实际上就是对式$(1)$的改写。为了方便起见，一般会将上述问题转换为如下形式：



$$
min_{w,b}\frac{1}{2}{||w||^2} \text{ }s.t.\text{ }y_i(w^Tx_i+b)\geq1,\text{ }i=1,2,...,m.\tag2
$$



也就是在限制条件下最小化$\|\|w\|\|^2$，前面的系数同样是为了便于求导，上式便是`SVM`的基本模型。

### 对偶问题

$(2)$式中得到的模型，实际上是一个凸（二次）优化问题，有很多现成的优化求解方法，但是还是可以对其进行进一步简化。我们可以使用`拉格朗日乘子法`求其对偶问题，给$(2)$式的个约束项添加一个`拉格朗日乘子`$\alpha_i\geq0$，可以得到$(2)$式对应的`拉格朗日函数`：



$$
L(\vec{w},b,\vec{a})=\frac{1}{2}{||w||^2}+\sum_{i=1}^m\alpha_i(1-y_i(w^Tx_i+b))\tag3
$$



在构造`拉格朗日函数`前，要首先将约束条件转换成小于等于或者大于等于0的形式。接着按照`拉格朗日函数`的求解步骤，分别求$L(\vec{w},b,\vec{a})$对参数$\vec{w}$和$\vec{b}$的偏导为零：



$$
\begin{cases}\tag4
\frac{\partial L}{\partial w}=0 \Rightarrow w=\sum_{i=1}^m\alpha_iy_ix_i\\
\frac{\partial L}{\partial b}=0 \Rightarrow 0=\sum_{i=1}^m\alpha_iy_i
\end{cases}
$$



这里为什么要使偏导为零呢？事实上这是`KKT`条件里的一个条件。假设我们有如下带有约束条件的优化问题：



$$
min_xf(x)\text{ } s.t.\text{ } g_i(x)\leq0,h_i(x)=0\\i=1,2,...,m,j=1,2,...,n
$$



那么其对应的`KKT`条件的完整要求有：

>- $L(x, a, b)$对求$x$导为零；(a，b分别为两个约束条件的拉格朗日乘子)
>- $h_i(x)=0$ ；(非不等式约束直接引入)
>- $a_i,b_i\geq0$；(拉格朗日乘子必须大于等于0)
>- $a_i*g_i(x)=0$ 。(不等式约束必须满足与乘子之积为0)

关于`拉格朗日乘子法`和`KKT`条件我之后也会进行详细总结，这里只是简单提一下，知道怎么用就行了。总之，我们的第一个`KKT`条件就是求`拉格朗日函数`对参数$\vec{w}$和$\vec{b}$的偏导为零之解，解出来第一个`KKT`条件之后，将$(4)$代入$(3)$便可得到$(2)$的对偶问题：



$$
max_\alpha\sum_{i=1}^n\alpha_i-\frac{1}{2}\sum_{i=1}^m\sum_{j=1}^m\alpha_i\alpha_jy_iy_j\vec{x}_i^T\vec{x_j}\tag5\\
s.t. \text{ }\sum_{i=1}^m\alpha_iy_i=0,\alpha_i\geq0,i=1,2,...,m
$$



这个就是我们最终需要的优化表达式，现在的目标就是求解参数$\vec{\alpha}$，这样我们就可以把预测模型表示为：



$$
f(x)=w^Tx+b=\sum_{i=1}^m\alpha_iy_ix_i^Tx+b \tag6
$$



求解式$(5)$有很多方法，后面要绍的`SMO`算法就是很常用的一种方法。

### 初看SMO算法

我们已经经历了原问题 -> 二次凸优化问题 -> 对偶问题的转变，现在只剩下求解$(5)$式了。在讲求解方法之前首先来说明一个`SVM`的有趣特性。之前已经解决了一个`KKT`条件了（偏导为0），因为在$(2)$式的约束中含有一个不等式约束：$y_i(w^Tx_i+b)\geq1$，所以我们还剩下如下的`KKT`条件：



$$
\begin{cases} \tag7
\alpha_i \geq 0;\\
y_if(x_i)-1\geq0;\\
\alpha_i(y_if(x_i)-1)=0.
\end{cases}
$$



通过$(7)$式里的条件不难发现，$\alpha_i=0$和$y_if(x_i)=1$，总会有一个满足。并且当$\alpha_i=0$时，当前样本不会出现在`拉格朗日函数`的累加项里，也就不会对超平面$f(x)$产生影响。当$\alpha_i>0$时，必有$y_if(x_i)=1$，说明当前样本在最大间隔的边界超平面上，也就是说当前样本是`支持向量`。这些都反映了`SVM`的一个重要特性：**最终模型仅与支持向量有关**。

然后来讲解求解$(5)$式的一个高效算法：`SMO`（Sequential Minimal Optimization），这里由于还没介绍`软间隔`， 先简单说一下`SMO`，具体的算法流程在后面。它的基本思路是首先固定$\alpha_i$之外的所有参数，然后求$\alpha_i$的上极值。由于我们有$\sum_{i=1}^m\alpha_iy_i=0$这个约束，说明固定其他参数的话，就可以用它们表示$\alpha_i$。但是冰雪聪明的你可能会发现这种思路有个问题，如果我们每只更新一个参数的话是不是有点问题呢？比如说我们现在要更新$\alpha_1$，固定除$\alpha_1$之外的所有参数后你会发现：$\alpha_1y_1=-\sum_{i=2}^m\alpha_iy_i$，所以它已经不是变量了，等于你自己意淫出来一套参数，就没有求解的意义了。所以我们需要每次更新两个参数，这样才能保证求解有意义，`SMO`算法实际上是在反复执行以下两个步骤直至收敛：

>- 选取两个待更新参数$\alpha_i$和$\alpha_j$，一般使用启发式选择法； 
>- 固定以上两个参数之外的所有参数，确定$\alpha_i$在式$(5)$极值条件下的值，然后用$\alpha_i$表示$\alpha_j$。

举个例子，比如说现在的参数列表是：$\alpha_1,\alpha_2,...,\alpha_m$。选取$\alpha_1,\alpha_2$作为待更新参数并固定参数$\alpha_3,\alpha_4,...,\alpha_m$。可以将约束条件写成如下形式：



$$
\alpha_1y_1+\alpha_2y_2=-\sum_{i=3}^m\alpha_iy_i
$$



由于剩下的参数已经固定了，所有上式的右半部分可以写成常数，即$\alpha_1y_1+\alpha_2y_2=\zeta$。把这个式子代入$(5)$中，原问题就变成一个关于$\alpha_1$的二次规划问题了。最后再来看下偏移量$b$的求解，我们用$S$表示支持向量的集合，由于对于任意支持向量$(x_s,y_s)$，有$y_sf(x_s)=1$，所以可以据此来用任意支持向量求得$b$，但是实际常常使用所有支持向量的的平均值来求解，即：



$$
b=\frac{1}{|S|}\sum_{s\in S}\left(\frac{1}{y_s}-\sum_{i\in S}\alpha_iy_ix_i^Tx_s\right)
$$



### 核函数

之前的问题一直研究的是线性可分的情况，有时候会遇到一些在当前空间内线性不可分的问题怎么办？比如下图所示的分类问题：

![none](/img/in-post/10.png)

这时候我们可以样本从原样本空间映射到一个更高维的样本空间中，使得样本在这个空间中线性可分。比如说对于上图二维空间中的样本，我们可以使用映射规则：$z_1=x_1^2,z_2=x_2^2,z_3=x_2$，把原样本映射到如下图所示的三维空间中：

![none](/img/in-post/9.png)

如果用$\phi(x)$来表示原样本$x$经过映射后的样本，那么我们最初的预测模型就变成了：



$$
f(x)=w^T\phi (x)+b
$$



接着转换成二次凸优化问题就变成了：



$$
min_{w,b}\frac{1}{2}{||w||^2} \text{ }s.t.\text{ }y_i(w^T\phi(x_i)+b)\geq1,\text{ }i=1,2,...,m.
$$



最后利用`拉格朗日乘子法`转换成对偶问题就变成了：



$$
max_\alpha\sum_{i=1}^n\alpha_i-\frac{1}{2}\sum_{i=1}^m\sum_{j=1}^m\alpha_i\alpha_jy_iy_j\phi(\vec{x_i})^T\phi(\vec{x_j})\\
s.t. \text{ }\sum_{i=1}^m\alpha_iy_i=0,\alpha_i\geq0,i=1,2,...,m
$$



流程和之前的一模一样，只需要把原样本替换成映射之后的样本就行了。注意到$\phi(\vec{x_i})^T\phi(\vec{x_j})$这个式子，一个矩阵乘法的 操作，由于映射后的样本空间维数可能很高，所以这个操作可能会很耗时，一般是不会直接计算的。我们可以找到一个这样的函数$k(\cdot,\cdot)$，使得：



$$
k(x_i,x_j)=\phi(x_i)^T\phi(x_j)
$$



这样原先的矩阵乘法操作就被大大简化了。于是现在的对偶问题就可以改写成：



$$
max_\alpha\sum_{i=1}^n\alpha_i-\frac{1}{2}\sum_{i=1}^m\sum_{j=1}^m\alpha_i\alpha_jy_iy_jk(\vec{x_i},\vec{x_j})\\
s.t. \text{ }\sum_{i=1}^m\alpha_iy_i=0,\alpha_i\geq0,i=1,2,...,m
$$



进而最后的分类模型就可以改写成：



$$
f(x)=w^T\phi(x)+b=\sum_{i=1}^m\alpha_iy_i\phi(x_i)^T\phi(x)+b=\sum_{i=1}^m\alpha_iy_ik(x_i,x)+b
$$



关于`核函数`的研究是非常成熟的，其相关理论远比这些复杂，以后再专门学习一下。这里先简单总结一下常用的`核函数`和`核函数`的特性。

|   核函数    |                   表达式                    |         参数         |
| :------: | :--------------------------------------: | :----------------: |
|   线性核    |          $k(x_i,x_j)=x_i^Tx_j$           |                    |
|   多项式核   |        $k(x_i,x_j)=(x_i^Tx_j)^d$         |      $d\geq0$      |
|   高斯核    | $k(x_i,x_j)=exp\left(-\frac{\|\|x_i-x_j\|\|^2}{2\sigma^2}\right)$ |     $\sigma>0$     |
|  拉普拉斯核   | $k(x_i,x_j)=exp\left(-\frac{\|\|x_i-x_j\|\|}{\sigma}\right)$ |     $\sigma>0$     |
| Sigmoid核 | $k(x_i,x_j)=tanh(\beta x_i^Tx_j+\theta)$ | $\beta>0,\theta<0$ |

核函数的特性：

>- 核函数的线性组合仍然是核函数；
>- 核函数的直积仍是核函数；
>- 对于核函数$k_1(\cdot,\cdot)$和任意函数$g(x)$，$k(x,z)=g(x)k_1(x,z)g(z)$仍是核函数。

### 软间隔问题

根据之前的假设，如果样本集在当前空间内线性不可分，我们可以试图将其映射到高维空间。但是有时候存在这样的情况：当前样本集大体上是线性可分的，但是部分数据存在扰动。也就是没办法找到一个完美的超平面，总有一些数据划分错误。具体情况如下图所示：

![none](/img/in-post/11.png)

这时候就要引入`软间隔`的概念了，之前的`间隔`可以称为`硬间隔`，因为它要求所有样本必须划分正确。在`软间隔`情况下，某些样本不必满足约束条件：$y_i(w^Tx_i+b)\geq1$。通常这种情况下的处理方式是给不满足条件的样本添加损失函数计算损失，并以最低损失作为优化目标。于是我们可以把之前的二次优化目标写成：


$$
min_{w,b}\frac{1}{2}{||w||^2}+C\sum_{i=1}^m\zeta_{0/1}(y_i(w^Tx_i+b)-1),\text{ }i=1,2,...,m.
$$


其中C>0，是个常数，它可以用来描述不满足约束的样本个数，当C为无穷大时表示所有样本满足约束，上式退化成`硬间隔`的优化目标。$\zeta_{0/1}$是0/1损失函数，当自变量小于0等时候为1，否则为0，这个损失函数比较简单，但是它非凸，不连续，数学性质不太好。实际中常常选择数学性质比较好，并且是0/1损失函数上界的`替代损失函数`比如：

>- hinge损失：$\zeta_{hinge}(z)=max(0,1-z)$;
>- 指数损失：$\zeta_{exp}(z)=exp(-z)$;
>- 对率损失：$\zeta_log(z)=log(1+exp(-z))$.

这几种替代损失函数的示意图如下：

![none](/img/in-post/12.png)

现在我们将损失函数替换成`hinge`，可以得到新的二次优化目标为：


$$
min_{w,b}\frac{1}{2}{||w||^2}+C\sum_{i=1}^m max(0,1-y_i(w^Tx_i+b)),\text{ }i=1,2,...,m.\tag8
$$


以这个式子作为最终的`软间隔`基本模型如何？不太好，因为它的约束条件不明显，不能直接对其使用`拉格朗日乘子法`，我们得想办法从中分离出一些有用的约束条件。这里引入`松弛因子`$\xi_i\geq0$，干什么的？既然`软间隔`允许某些样本有划分误差，我们就可以用$\xi_i$来表示这个误差容忍的范围，即$y_i(w^Tx_i+b)\geq1-\xi_i$，这种情况下`支持向量`满足$y_i(w^Tx_i+b)=1-\xi_i$。仔细一看，这个东西不就是我们需要的约束条件吗？现在就可以将$(8)$式改写成：


$$
min_{w,b}\frac{1}{2}{||w||^2}+C\sum_{i=1}^m\xi_i\tag9 \\
s.t. \text{ }y_i(w^Tx_i+b)\geq1-\xi_i,\xi_i\geq0,i=1,2,...,m
$$


$(9)$式用来做`软间隔`的基本模型就舒服多了，简化了优化表达式，并且添加约束条件，`可以使用拉格朗日乘子法`。类似处理$(2)$式的思路，我们还是来求其对偶问题，首先构造`拉格朗日函数`，由于这次有两个约束条件，所以现在需要两组`拉格朗日乘子`，最终的表达式如下：


$$
L(w,b,\xi,\alpha,\beta)=\frac{1}{2}{||w||^2}+C\sum_{i=1}^m\xi_i\\
+\sum_{i=1}^m\alpha_i(1-\xi_i-y_i(w_Tx_i+b))-\sum_{i=1}^m\beta_i\xi_i
$$


这里在乘上因子之前，将约束条件都转换成了小于等于零的形式。这个`拉格朗日函数`对应的`KKT条件`就比较多了，但是在求解对偶问题时，我们只需要偏导为0的`KKT`条件，即：


$$
\begin{cases}\tag{10}
\frac{\partial L}{\partial w}=0 \Rightarrow w=\sum_{i=1}^m\alpha_iy_ix_i\\
\frac{\partial L}{\partial b}=0 \Rightarrow 0=\sum_{i=1}^m\alpha_iy_i\\
\frac{\partial L}{\partial \xi}=0 \Rightarrow C=\alpha_i+\beta_i
\end{cases}
$$


相比`硬间隔`，前两项都没有变，只是多出来了第三项。将$(10)$代回`拉格朗日函数`最终得到$(9)$式的对偶问题为：


$$
max_\alpha\sum_{i=1}^n\alpha_i-\frac{1}{2}\sum_{i=1}^m\sum_{j=1}^m\alpha_i\alpha_jy_iy_j\vec{x}_i^T\vec{x_j}\tag{11}\\
s.t. \text{ }\sum_{i=1}^m\alpha_iy_i=0,C\geq\alpha_i\geq0,i=1,2,...,m
$$


exo me ？相比$(5)$，`软间隔`的对偶问题仅仅是对$\alpha_i$多了一个上界约束！我们现在再来看下剩下的`KKT`条件，看看能不能找到`软间隔`模型的特点：


$$
\begin{cases} \tag{12}
\alpha_i \geq 0;\\
\beta_i \geq 0;\\
\xi_i\geq0;\\
\beta_i\xi_i=0;\\
y_if(x_i)-1+\xi_i\geq0;\\
\alpha_i(y_if(x_i)-1+\xi_i)=0.
\end{cases}
$$


>此时，$\alpha_i=0$和$y_if(x_i)=1-\xi_i$，总会有一个满足：
>
>- 当$\alpha_i=0$时，当前样本不会出现在`拉格朗日函数`的累加项里，也就不会对超平面$f(x)$产生影响。
>- 当$\alpha_i>0$时，必有$y_if(x_i)=1-\xi_i$，说明当前样本是`支持向量`，这时候再来看$\alpha_i$的上界:
>  - 若$\alpha_i<C$，则$\beta_i\geq0,\xi_i=0$，样本在最大间隔边界上，还是`支持向量`。
>  - 若$\alpha_i = C$，则$\beta_i = 0$，这时候又要看$\xi_i$：
>    - 若$\xi_i\leq1$，则样本落在最大间隔内部，也就是容错范围内。
>    - 若$\xi_i>1$，则样本落在最大间隔外部，样本被错误分类。

无论是容错范围内还是错误分类的样本都比较少，所以这时候还是可以认为`硬间隔`的模型只与其`支持向量`有关。

### 再看SMO算法

前面讲`SMO`参数选择的时候比较粗略，因为当时还没有提出`软间隔`，模型并不完整。现在结合`软间隔`得出的最新模型讲一下`SMO`中参数选取的具体过程。前面假设我们固定了两个参数，就可以得到：


$$
\alpha_1y_1+\alpha_2y_2=-\sum_{i=3}^m\alpha_iy_i=\zeta \tag{13}
$$


在分析问题之前，我们可以先将$\alpha_1,\alpha_2$的关系图绘制如下：

![none](/img/in-post/8.png)

我们知道在我们现在这个二分类问题里：$y_i=\pm1$，因此可以划分出以下情况：

**y1和y2异号**

假设此时y1=1，y2=-1，代入(13)有$\alpha_1-\alpha_2=\zeta$。因为我们并不知道等式右边常数的正负，所以这时两个参数满足的关系可能符合上图中的红线或者蓝线。如果它们满足红线的关系，$\alpha_2$的取值范围就是$(-\zeta,C)$；如果他们满足蓝线的关系，$\alpha_2$的取值范围就是$(0,C-\zeta)$。如果我们分别用L和H表示$\alpha_2$取值的上下界，它们可以表示为：


$$
\begin{cases}\tag{14}
L=max(0,\alpha_2-\alpha_1)\\
H=min(C,C+\alpha_2-\alpha_1)
\end{cases}
$$


假设此时y1=-1，y2=1，按照同样的思路，发现此时$\alpha_2$的上下界仍然为(14)。因此当y1和y2异号时$\alpha_2$的取值范围为(14)。注意这里$\alpha_2$的取值范围会用到自己。实际上由于我们现在是在更新$\alpha_2$，所有等号右边的值都是旧值，也就是$\alpha_1^{old},\alpha_2^{old}$，后面我就不特殊注明了。 

**y1和y2同号**

当y1和y2同号时，同样按照前面的分析方法，可以发现无论它们都是+1，还是都是-1，$\alpha_2$的取值范围都是：


$$
\begin{cases} \tag{15}
L=max(0,\alpha_1+\alpha_2-C) \\
H=min(C,\alpha_1+\alpha_2)
\end{cases}
$$


由于我们现在已经固定了从第三个参数往后的所有参数，所以我们现在可以将(11)中的对偶问题改写成：


$$
min_{\alpha_1,\alpha_2}W(\alpha_1,\alpha_2)=\frac{1}{2}K_{11}\alpha_1^2+\frac{1}{2}K_{22}\alpha_2^2+y_1y_2K_{12}\alpha_1\alpha_2\tag{16}\\
-(\alpha_1+\alpha_2)+y_1\alpha_1\sum_{i=3}^my_i\alpha_iK_{i1}+y_2\alpha_2\sum_{i=3}^my_i\alpha_iK_{i2} \\
s.t. \text{ }\alpha_1y_1+\alpha_2y_2=\zeta,0\leq\alpha_i\leq C,i=1,2.
$$


其中$K_{ij}=K(x_i,x_j)$是核函数，如果不需要映射的话，它就等于样本内，下面具体来讲如何更新$\alpha_1,\alpha_2$。大体思路是：

> 1. 首先求**沿着约束方向，未经剪辑**（即不考虑约束不等式$,0\leq\alpha_i\leq C$）时$\alpha_2$的最优解，记作$\alpha_2^{new,unc}$；
> 2. 然后再添加剪辑条件，得到最终的最优解$\alpha_2^{new}$；
> 3. 利用$\alpha_2^{new}$计算$\alpha_1^{new}$。

具体来说，对于第一步，我们首先引入预测误差：


$$
E_i=f(x_i)-y_i=\left(\sum_{j=1}^m\alpha_iy_iK(x_j,x_i)+b \right)-y_i, i=1,2
$$


为了证明方便，在此引入记号$v_i=\sum_{j=3}^m\alpha_iy_iK(x_i,x_j)$，这样就可以把(16)中的目标函数写成：


$$
W(\alpha_1,\alpha_2)=\frac{1}{2}K_{11}\alpha_1^2+\frac{1}{2}K_{22}\alpha_2^2+y_1y_2K_{12}\alpha_1\alpha_2 \tag{17}\\
-(\alpha_1+\alpha_2)+y_1\alpha_1v_1+y_2\alpha_2v_2
$$


这个表达式两个参数混杂，我们先简化一下，由于有$\alpha_1y_1+\alpha_2y_2=\zeta$和$y_i^2=1$，所以有$\alpha_1=(\zeta-y_2\alpha_2)y_1$，进而可以将(17)改写成：


$$
W(\alpha_2)=\frac{1}{2}K_{11}(\zeta-\alpha_2y_2)^2+\frac{1}{2}K_{22}\alpha_2^2+y_2K_{12}(\zeta-\alpha_2y_2)\alpha_2 \\
-(\zeta-\alpha_2y_2)y_1-\alpha_2+v_1(\zeta-\alpha_2y_2)+y_2v_2\alpha_2
$$


现在完全就是一个单变量的最优问题，直接求导：


$$
\frac{\partial W}{\partial \alpha_2}=K_{11}\alpha_2+K_{22}\alpha_2-2K_{12}\alpha_2-K_{11}\zeta y_2\\
+K_{12}\zeta y_2+y_1y_2-1-v_1y_2+y_2v_2
$$


然后，使导数为0，并代入约束$\alpha_1^{old}y_1+\alpha_2^{old}y_2=\zeta$，得到：


$$
(K_{11}+K_{22}-2K_{12})\alpha_2^{new,unc}=y_2(y_2-y_1+\zeta K_{11}-\zeta K_{12}+v_1-v_2)\\
=y_2\left[ y_2-y_1+\zeta K_{11}-\zeta K_{12}+\left( f(x_1)-\sum_{j=1}^2y_j\alpha_jK_{1j}-b\right) -\left(  f(x_2)-\sum_{j=1}^2y_j\alpha_jK_{2j}-b\right)\right] \\
=y_2((K_{11}+K_{22}-2K_{12})\alpha_2^{old}y_2+y_2-y_1+f(x_1)-f(x_2)) \\
=(K_{11}+K_{22}-2K_{12})\alpha_2^{old}+y_2(E_1-E_2)
$$


注意到$K_{11}+K_{22}-2K_{12}$（也就是$\|\|\phi(x_1)\phi(x_2)\|\|^2$）是常量，如果我们把它表示为$\eta$，就可以最终得到未经剪辑的$\alpha_2$新值：


$$
\alpha_2^{new,unc}=\alpha_2^{old}+\frac{y_2(E_1-E_2)}{\eta}
$$


完美！！！，第二步，我们来加入对$\alpha_2$的上下界约束条件，结合(14)和(15)，有：


$$
\alpha_2^{new}=\begin{cases}
H, && \alpha_2^{new,unc}>H \\
\alpha_2^{new,unc}, && L\leq \alpha_2^{new,unc}\leq H \\
L, && \alpha_2^{new,unc} < L
\end{cases}
$$


也就是要限制更新值在上下界内，最后一步，直接计算$\alpha_1$的更新值就行了（用新旧两个约束条件联立求解）：


$$
\alpha_1^{new}=\alpha_1^{old}+y_1y_2(\alpha_2^{old}-\alpha_2^{new})
$$


参数更新后还需要更新偏移量b，这里不用专门去更新特征矩阵w，因为w可以用参数和样本表示，进行分类的时候不用它也可以，b值是要不断更新并计算的。因为我们有$y_1=\sum_{i=1}^m\alpha_iy_iK_{i1}+b$，所以直接把已有参数代入就可以解得：


$$
b_1^{new}=y_1-\sum_{i=3}^m\alpha_iy_iK_{i1}-\alpha_1^{new}y_1K_{11}-\alpha_2^{new}y_2K_{21}
$$


这个含参累加项怎么办？别急，先看看之前计算误差的表达式，把含有$\alpha_1,\alpha_2$的两项从中分离出来就有：


$$
E_1=\sum_{i=3}^m\alpha_iy_iK_{i1}+\alpha_1^{old}y_1K_{11}+\alpha_2^{old}y_2K_{21}+b_{old}-y_1
$$


代回更新$b_1$的表达式，跟部分常量项说再见：


$$
b_1^{new}=-E_1-y_1K_{11}(\alpha_1^{new}-\alpha_1^{old})-y_2K_{21}(\alpha_2^{new}-\alpha_2^{old})+b^{old}
$$


同样的思路，可以根据误差$E_2$和第二组样本计算得到： 


$$
b_2^{new}=-E_2-y_1K_{12}(\alpha_1^{new}-\alpha_1^{old})-y_2K_{22}(\alpha_2^{new}-\alpha_2^{old})+b^{old}
$$


值得注意的是，如果$\alpha_1,\alpha_2$都满足$>0,<C$，那么这两个更新的偏移量是相等的。如果等于$\alpha_1,\alpha_2$等于0或者C，那么这两个更新偏移量范围内的所有值都满足`KKT`条件的偏移量这时候取两者平均值作为新的偏移就行了。

### python实现

不得不说，`SVM`这块的理论是真的多，看了三天头发掉了好多。并且讲了这么多才把基础的东西说完了。代码实现我是参考书上写的，功能不是很全面，以后再慢慢完善。

```python
import numpy as np
import matplotlib.pyplot as plt
import re

# 加载数据
def load_data(file):
    data = []
    label = []
    f = open(file, 'r')
    for line in f.readlines():
        data_info = line.split('\t')
        data.append([float(data_info[0]), float(data_info[1])])
        label.append(float(data_info[2]))
    return data, label

# 选择待更新参数
def select_para(i, m):
    j = i
    while j == i:
        j = int(np.random.uniform(0, m))
    return j

# 调整参数的值
def adjust_para(aj, h, l):
    if aj > h:
        aj = h
    if l > aj:
        aj = l
    return aj

"""
一个简单的SMO算法，参数对选取方式为：

遍历alpha集合选取第一个，另一个在剩下的参数中随机选取。

参数：数据集，标签集，常数c，松弛因子xi，最大循环次数t。
"""

def simple_smo(data, label, c, xi, t):

    data = np.mat(data)
    # label转换成矩阵后还要再转置成列向量
    label = np.mat(label).transpose()

    m, n = np.shape(data)
    # 初始化拉格朗日乘子
    alphas = np.mat(np.zeros((m, 1)))
    # 初始化偏移量和轮数为0
    b, temp_t = 0, 0
    while temp_t < t:
        changed = 0
        for i in range(m):
            # 根据转换后的模型计算分类值  (1x100)x(100x2)x(2x1)
            fxi = float(np.multiply(alphas, label).T*data*data[i, :].T) + b
            ei = fxi - float(label[i])
            # 满足约束条件
            if (label[i] * ei < -xi) and (alphas[i] < c) or (label[i] * ei > xi) and (alphas[i] > 0):
                j = select_para(i, m)
                fxj = float(np.multiply(alphas, label).T*data*data[j, :].T) + b
                ej = fxj - float(label[j])
                alphai = alphas[i].copy()
                alphaj = alphas[j].copy()

                if label[i] != label[j]:
                    l = max(0, alphas[j] - alphas[i])
                    h = min(c, c + alphas[j] - alphas[i])

                else:
                    l = max(0, alphas[j] + alphas[i] - c)
                    h = min(c, alphas[j] + alphas[i])

                if l == h:
                    continue

                eta = 2.0*data[i, :]*data[j, :].T-data[i, :]*data[i, :].T-data[j, :]*data[j, :].T

                if eta >= 0:
                    continue

                alphas[j] -= label[j]*(ei-ej)/eta
                alphas[j] = adjust_para(alphas[j], h, l)

                if abs(alphas[j] - alphaj) < 0.00001:
                    continue

                alphas[i] += label[j]*label[i]*(alphaj-alphas[j])

                b1 = b-ei-label[i]*(alphas[i]-alphai)*data[i, :]*data[i, :].T\
                     - label[j]*(alphas[j]-alphaj)*data[i, :]*data[j, :].T

                b2 = b-ej-label[i]*(alphas[i]-alphai)*data[i, :]*data[j, :].T\
                     - label[j]*(alphas[j]-alphaj)*data[j, :]*data[j, :].T

                if (0 < alphas[i]) and (c > alphas[i]):
                    b = b1

                elif (0 < alphas[j]) and (c > alphas[j]):
                    b = b2

                else:
                    b = (b1 + b2) / 2.0

                changed += 1

        if changed == 0:
            temp_t += 1

    return b, alphas


def draw(data, label, alphas, b):
    data = np.array(data)
    n = np.shape(data)[0]
    x1 = []
    y1 = []
    x2 = []
    y2 = []
    for i in range(n):
        if int(label[i]) == 1:
            x1.append(data[i, 0])
            y1.append(data[i, 1])
        else:
            x2.append(data[i, 0])
            y2.append(data[i, 1])
    plt.scatter(x1, y1, s=30, c='red')
    plt.scatter(x2, y2, s=30, c='green', marker='s')
    x = np.arange(2.0, 8.0, 0.1)
    ws = cal_weight(alphas, data, label)
    y = [(i * ws[0] + np.array(b)[0]) / -ws[1] for i in x]
    plt.plot(x, y)
    plt.xlabel('X1')
    plt.ylabel('X2')
    plt.show()


def cal_weight(alpha, data, labels):
    data = np.mat(data)
    labels = np.mat(labels).transpose()
    m, n = np.shape(data)
    w = np.zeros((n, 1))
    for i in range(m):
        w += np.multiply(alpha[i]*labels[i], data[i, :].T)
    return w


if __name__ == '__main__':
    data_set, label_set = load_data("E:\io\data.txt")
    b, alphas = simple_smo(data_set, label_set, 0.6, 0.001, 40)
    draw(data_set, label_set, alphas, b）
```

结果如下：

![none](/img/in-post/13.png)

### 参考

**周志华《机器学习》**

**李航《统计学学习方法》**

[tonglin0325的博客](https://www.cnblogs.com/tonglin0325/p/6078439.html)

[SVM中的对偶问题、KKT条件以及对拉格朗日乘子求值得SMO算法](http://blog.csdn.net/huangynn/article/details/38760197)