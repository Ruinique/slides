# 局部块级共轭梯度法迭代求特征值



## <small>从最速下降法到<br>预处理共轭梯度法</small>



这段主要参考了 [共轭梯度法的通俗讲义](https://flat2010.github.io/2018/10/26/%E5%85%B1%E8%BD%AD%E6%A2%AF%E5%BA%A6%E6%B3%95%E9%80%9A%E4%BF%97%E8%AE%B2%E4%B9%89/)。

所谓的**二次型**，是关于向量的二次数值型函数，形如：

$$
f(x) = \frac{1}{2}x^TAx -b^Tx+c
$$
当 $A$ 为对称正定矩阵的时候，$f(x)$ 的最小值由 $Ax-b=0$ 的解处取到。



从图像上来看这也很显然，这个二次型函数的极值点就是我们梯度为 0 的点，我们对这个矩阵求导就可以有
$$
f'(x) = Ax-b
$$
就算 $A$ 不是对称矩阵，我们也有 $\frac{1}{2}(A+A^T)$ 是对称矩阵，依然可以转化成对称矩阵来去找到对应的解。



那么我们要找到这个点，就可以从任意一个点来下降到我们抛物面的底部。我们可以在刚开始任意选取一个方向，在这个方向上我们移动到让 $f(x)$ 下降最多的点。我们借助物理学家的观点，我们已经在这个方向上尽可能的移动了，任意存在正交分量的移动，分解到 $f(x)$ 方向上的移动都会使得我们的 $f(x)$ 增大，下一次移动的分量必然和我们本次移动的方向正交。



我们把和取到 $x$ 最小值对应的点向量叫做**误差向量**，记作 $e_i$ 。<br>把当前点和 $f(x)$  最小值的点之间的距离的向量叫做称之为**残差向量**，记作$r_i$。
<br>我们的残差向量就是所谓最速下降的方向。



<small>根据上面的推理，我们可以直接去求我们每次移动的步长：</small>
$$
\begin{aligned}
    r_i^T r_{i+1} &= r_i^T (b - A x_{i+1}) 
\\\ &= r_i^T (b - A (x_i + \alpha r_i)) 
\\\ &= r_i^T (b - A x_i + A \alpha r_i) 
\\\ &= r_i^T r_i + \alpha r_i^T A r_i 
\\\ &= 0
\end{aligned}
$$

可以解得我们的 $\alpha = -\frac{r_i^Tr_i}{r_i^TAr_i}$ 。



而且我们可以发现我们的残差向量直接存在一个直接的递推关系：

$$
r_{i+1}=r_i-\alpha_i A r_i
$$
我们只要按上面的公式去计算，就可以得到对应的残差向量了，而不依赖于中间结果 $x_i$。



> 关于特征值的一些杂谈
>
>我们又知道对称正定阵有 $n$ 个线性无关（正交）的特征向量，且任何一个向量都可以看成这些特征向量的合成。只要这些特征向量对应的特征值的绝对值都小于 1，我们最终就会逐渐减小，我们就可以利用（其实正定阵的特征值也都大于 0）对应的谱半径（也就是最大的特征值）去衡量我们的迭代速度，理论上越小迭代速度就越快，且必然迭代到 0。



那么对于我们理想的下降方法，应该是每一次都沿着对应的正交的特征向量的方向去走对应的步长，最后就可以以 $n$ 次（或者更少的）迭代，达到我们的最小值。

我们记每一次的搜索向量为 $d_i$ 可以得到：

$$
d_i^Te_{i+1} = 0
$$
我们对迭代的公式 $x_{i+1}=x_i+\alpha_id_i$ 做变形，可以得到 $e_{i+1}=e_i+\alpha_id_i$

联立得到 $\alpha_i=-\frac{d_i^Te_i}{d_i^Td_i}$



但是我们不知道 $e_i$ 还是没办法求出我们的答案，而我们都知道 $e_i$ 为啥其实直接就知道了对应的 $x$ 了。

所以说这样不太可行，我们用矩阵-向量正交来代替向量正交就可以了（也就是**共轭**）。

也就是 $d_i^TAd_j = 0$ 称之为 $d_i$ 和 $d_j$ 共轭。

我们这里不再要求搜索方向正交，只要求这两者共轭就可以了，这里我们的搜索方向不再直接沿着残差的方向，而是通过和我们的误差向量共轭来得到，对这样的递推公式求导也正好能得到这样的递归过程最速下降的点就是  $d_i^TAd_j = 0$ 的时候。



根据数学推导可以得到我们最终的误差向量可以写成 $n$ 个搜索向量的和，也就是用共轭方向发我们必然可以在 n 步之类递归到 0。

但是为了计算我们的 $d_i$ 我们得使用共轭格莱姆-施密特过程。



- Step:1 先找一组线性无关的向量。在这些向量里面任取一个，作为第一步的搜索方向向量（因为是第一步，不需要考虑正交性，因此可直接使用该向量作为搜索向量）。  
- Step:2 再在剩下的这些线性无关的向量中取一个出来进行向量分解。我们知道一个向量可以有很多种分解方式，那么怎么决定如何分解呢？首先找到与上一步的搜索向量成矩阵·向量正交的方向，该方向即作为这个要分解的向量的一个分量方向，这个方向的分量也将保留下来作为本轮迭代的搜索方向向量。其它方向的分量因为对我们没有用，自然就要消除掉。 



- Step:3 再在剩下的线性无关的向量中取一个出来进行向量分解。分解的方法就是，首先找一个与之前所有搜索向量均矩阵向量正交的方向，留下这个方向的向量作为本轮的搜索方向向量。其余的分量同样消除掉。  
- Step:4 重复第3)步的过程，直到所有的线性无关的向量都抽完为止。



这就要求我们保存之前所有的 $d_i$，这就是为什么共轭方向法在 CG 提出前实际上应用没有那么广的原因。



共轭方向法在每一轮迭代中所需的矩阵·向量乘积计算数量可以通过递归求解残差向量直接减少到一次，如下所示： 
$$ 
\begin{aligned}
    \vec{r}^{(i+1)} &= -A \vec{e}^{(i+1)} 
    \\\ &= -A \left( \vec{e}^{(i)} + \alpha^{(i)} \vec{d}^{(i)} \right) 
    \\\ &= \vec{r}^{(i)} - \alpha^{(i)} A \vec{d}^{(i)} 
\end{aligned}
$$



当我们利用残差的一些性质之后，我们可以直接利用残差向量去构造我们的搜索向量，而且残差向量正好张成一个 $Krylov$ 子空间，而且我们的搜索向量经由推导，正好又可以用我们的残差实现一个比较好实现的递推公式，而不再需要存储前面的搜索向量 $d_i$。

预处理里有几个比较常见的操作，比如对角阵预处理,我们利用这样的公式，得到了一个在没有舍入误差 n 步内必然收敛的一个公式。



<img src="https://ruinique-alibaba-oss.oss-cn-chengdu.aliyuncs.com/202412031655718.png"/>

上述的一个过程，实际上就是我们的 PCG 的处理。



## <small>利用 LOBPCG <br>求解偏特征值问题</small>



我们知道上面的 PCG 完成的对应优化目标，最后可以找到我们对应的 $\mathbf{A}\vec{x}=\vec{b}$ 的解。

那么回到我们的特征值问题，我们就有 
$$\mathbf{A}\vec{x}=\lambda\vec{x}$$ 这么一个方程，
我们要求的不只有我们的 $\lambda$ 还有 $x$。

那么其实我们只要把 $\lambda$ 当成一个已知的量，那么实际上我们就可以去利用 $CG$ 算法去优化我们要求解出的多个 $x$ 组成的矩阵。



然后我们又知道 $Rayleigh$ 商在 $x$ 接近特征向量的时候，是一个很好的对特征值的近似，毕竟有：

$$ R(\mathbf{A}, \vec{x}) = \frac{\vec{x}^T \mathbf{A} \vec{x}}{\vec{x}^T \vec{x}} = \lambda\frac{\vec{x}^T \vec{x}}{\vec{x}^T \vec{x}} = \lambda$$

>Rayleigh 商还具有一些良好的性质，比如我们的 Rayleigh 商的上确界和下确界恰好是最大和最小的特征值。

我们又可以用我们的 $\vec{x}$ 去代表我们的 $\lambda$。所以我觉得从本质上Rayleigh商是一个**非线性函数**, 所以其实求解特征值问题我们也可以看成一个非线性的 PCG 问题。



Rayleigh-Ritz方法是一种求解广义特征值问题的数值算法。

Rayleigh-Ritz 方法的核心思想是，在子空间 V 中求解 $V^T A V$ 的特征值和特征向量，以获得矩阵 $A$ 在该子空内的最佳近似。当你有 $m$ 个向量时，Rayleigh-Ritz 方法将会计算出 $m \times m$ 的矩阵 $V^T A V$ 的特征值和特征向量。

给定一个对称矩阵 $A \in \mathbb{R}^{n \times n}$，我们的目标是找到其部分或全部特征值以及对应的特征向量。我们考虑一个 $m$-维子空间 $V$，其中 $m < n$，并且这个子空间由一组线性无关的向量 $v_1, v_2, \dots, v_m$ 张成。

首先，定义投影算子 $P = VV^T$，其中 $V$ 是包含基向量 $v_i$ 的矩阵。对于任意的向量 $x \in V$，我们可以将其表示为 $x = Vy$，其中 $y \in \mathbb{R}^m$ 表示 $x$ 在子空间 $V$ 中的坐标。



接着，我们需要构造一个代表 $A$ 在子空间 $V$ 上行为的矩阵。为此，我们计算 Rayleigh quotient，它被定义：
 
$$R(x) = \frac{x^TAx}{x^Tx}$$

对于 $x = Vy$，我们有：

$$R(Vy) = \frac{y^TV^TAVy}{y^TV^TVy}$$



因为 $V$ 的列向量是线性独立的，所以 $V^TV$ 是非奇异的（假设 $V$ 是正交化的，则 $V^TV = I$）。因此，我们关注的是 $B = V^TAV$，这是一个 $m \times m$ 矩阵，它相当于映射了 $A$ 在子空间 $V$ 上的行为。

接下来，我们求解 $B$ 的特征值问题 $By = \lambda y$，这里 $\lambda$ 和 $y$ 分别是 $B$ 的特征值和特征向量。由于 $B$ 是 $A$ 在子空间 $V$ 上的投影，$B$ 的特征值可以作为 $A$ 的特征值的近似，并且 $V$ 的相应列与 $y$ 的乘积 $Vy$ 给出了 $A$ 的特征向量的近似



在 LOBPCG 算法中，$gramB$ 是广义特征值问题中 $B$ 矩阵在当前子空间的投影（即 Gram 矩阵）。该矩阵是通过计算 $\mathbf{B}_{\text{sub}} = \mathbf{V}^T\mathbf{B}\mathbf{V}$ 获得的。它是 $B$ 矩阵在当前子空间中的表示，包含了特征值和特征向量的关系。

当我们通过 $Rayleigh-Ritz$ 方法求解低维的广义特征值问题时，得到的特征值 $\lambda$ 实际上是 $\mathbf{B}$​ 的特征值，即在投影到子空间后的广义特征值问题中的特征值。

同理，我们的特征向量是 $gramA$ 的特征向量。



但是因为我们通过迭代应用预处理后的矩阵去计算我们广义 Krylov 子空间的基，如果我们直接全局地去利用残差去扩展解空间, 会让 Krylov 子空间的基以 $m$ 的速度增长, V 的规模会很大, 所以一般不直接一直用 $Rayleigh-Ritz$ 方法在全局上做,就要让问题变成局部的。



然后在论文里首先对原本的 $PCG$ 算法做了一个新的描述，从而更好地引入新的改进算法:

$$
\begin{aligned}
x(i+1) &= w(i) + \tau(i) x(i) + \gamma(i) x(i-1) 
\\\ w(i) &= T(B x(i) - \mu(i) A x(i)) 
\\\ \mu(i) &= \mu(x(i)), \quad \gamma(0) = 0
\end{aligned}
$$
这里的 $\mu$ 是求 Rayleigh 商来做近似，剩下的其实就是在残差的基础上加上了预处理包了一层, 多了层迭代。

<small>然后 $LOBPCG$ 论文里面改进了一下，因为 $x(i)$ 和 $x(i-1)$ 的值会越来越接近，所以最后比较接近的一对特征值，很容易算不出来。所以论文里加入了一层 $p(i)$ 的迭代。</small>



构造了这样的一组公式

$$
\begin{aligned}
x^{(i+1)} &= w^{(i)} + \tau^{(i)} x^{(i)} + \gamma^{(i)} p^i 
\\\ w(i) &= T(B x(i) - \mu(i) A x(i)) 
\\\ p^{(i+1)} &= w^{(i)} + \gamma^{(i)} p^{(i)} 
\\\ \mu(i) &= \mu(x(i)),\quad p^{(0)} = 0
\end{aligned}
$$



这里的 $p^{(i)}$ 容易看出来就是 $x^{(i+1)}-\tau^{(i)} x^{(i)}$ 只是在更新 $w(i)$ 以后直接就更新而已，这样看论文的实验效果可以有比较好的优化，但也没看到解释。

所以我们这里使用 $LOBPCG$ 算法，也就是我们不再使用整个 Krylov 子空间，而是使用当前的 $span\\{w_i,x_i,p_i\\}$



最后算法描述如下

**初始化**：选择初始向量x并保证正交。<br>
**迭代过程**：通过迭代逐步逼近特征值和特征向量。


- **计算 Rayleigh 商**：对每个向量计算Rayleigh商以获得特征值的近似。
- **计算残差**：计算残差以衡量当前近似的准确性。
- **正交化残差**：正交化残差以保证共轭性质。
- **应用预处理器**：通过预处理器加速收敛并提高稳定性。
- **Rayleigh-Ritz 方法**：在子空间中寻找特征值的近似。（因为有 $x^{(i+1)} = w^{(i)} + \tau^{(i)} x^{(i)} + \gamma^{(i)} p^i$,子空间就是 $span\\{w_i,x_i,p_i\\}$
- **更新 Ritz 向量**：根据Rayleigh-Ritz方法的结果更新特征向量的近似。
- **更新搜索方向p**：为下一次迭代更新搜索方向。
- **正交化搜索方向p**：正交化搜索方向以保证共轭性质。


- **结束迭代**：当满足收敛条件时结束迭代。