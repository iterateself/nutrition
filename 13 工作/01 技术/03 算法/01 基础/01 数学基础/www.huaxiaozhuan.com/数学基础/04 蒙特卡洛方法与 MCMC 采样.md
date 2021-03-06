# 蒙特卡洛方法与 MCMC 采样

## 一、蒙特卡洛方法

1. 蒙特卡洛方法`Monte Carlo` 可以通过采用随机投点法来求解不规则图形的面积。

   求解结果并不是一个精确值，而是一个近似值。当投点的数量越来越大时，该近似值也越接近真实值。

2. 蒙特卡洛方法也可以用于根据概率分布来随机采样的任务。

### 1.1 布丰投针问题

1. 布丰投针问题是1777年法国科学家布丰提出的一种计算圆周率的方法：随机投针法。其步骤为：

   - 首先取一张白纸，在上面绘制许多条间距为$d$的平行线。

   - 取一根长度为$l,l\lt d$的针，随机地向纸上投掷$n$次，观测针与直线相交的次数，记做$m$。

   - 计算针与直线相交的概率$p=\frac{m}{n}$。可以证明这个概率$p=\frac{2l}{\pi\times d}$。因此有：

     $$\pi = 2 \frac{n\times l}{m\times d}$$

由于向纸上投针是完全随机的，因此用二维随机变量$(X,Y)$来确定针在纸上的具体位置。其中：

-$X$表示针的中点到平行线的距离，它是$[0,d/2]$之间的均匀分布。
-$Y$表示针与平行线的夹角，它是$[0,\frac{\pi}{2}]$之间的均匀分布。

<p align="center">
  <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20200603/DKVJdckGiqrT.png?imageslim">
</p>


当$X\lt \frac{l}{2} \sin Y$时，针与直线相交。

由于$X,Y$相互独立，因此有概率密度函数：

$$p(X=x,Y=y) = \begin{cases} \frac{4}{\pi d},& 0\le x\le d/2,0\le y\le \pi/2\\ 0,& \text{else} \end{cases}$$

因此，针与直线相交的概率为：

$$P\{X\lt \frac l2\sin Y\}=\int\int_{x\lt \frac l2 \sin y} p(x,y) dxdy=\int_{x=0}^{x=\frac l2\sin y}\int_{y=0}^{y= \pi/ 2} \frac {4}{\pi d} dxdy=\frac{2l}{\pi d}$$

根据$\frac{2l}{\pi d}=\frac{m}{n}$即可得证。

布丰投针问题中，蒙特卡洛方法是利用随机投点法来求解面积$\int\int_{x\lt \frac l2 \sin y} p(x,y) dxdy$。因为曲线的积分就是面积，这里的曲线就是概率密度函数$p(X,Y)$。

### 1.2 蒙特卡洛积分

1. 对于函数$f(x)$，其在区间$[a,b]$上的积分$\int_a^b f(x) dx$可以采用两种方法来求解：投点法、期望法。

2. 投点法求积分：对函数$f(x)$，对其求积分等价于求它的曲线下方的面积。

   此时定义一个常数$M$，使得$M\gt \max_{a \le x\le b} f(x)$，该常数在区间$[a,b]$上的面积就是矩形面积$M(b-a)$。

   随机向矩形框中随机的、均匀的投点，设落在函数$f(x)$下方的点为绿色，落在$f(x)$和$M$之间的点为红色。则有：**落在$f(x)$下方的点的概率等于$f(x)$的面积比上矩形框的面积** 。

   具体做法是：从$[a,b]$之间的均匀分布中采样$x_0$，从$[0,M]$之见的均匀分布中采样$y_0$，$(x_0,y_0)$构成一个随机点。

   - 若$y_0 \le f(x_0)$，则说明该随机点在函数$f(x)$下方，染成绿色。
   - 若$f(x_0) \lt y_0 \le M$，则说明该随机点在函数$f(x)$上方，染成红色。

   假设绿色点有$n_1$个，红色点有$n_2$个，总的点数为$n_1+n_2$，因此有：$\int_a^b f(x)dx = \frac{n_1}{n_1+n_2}\times M(b-a)$。

  <p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20200603/NG0HA5I0K2X6.png?imageslim">
  </p>
  

3. 期望法求积分：假设需要求解积分$I=\int_a^b f(x)dx$，则任意选择一个概率密度函数$p(x)$，其中$p(x)$满足条件：

   $$\int_a^b p(x) dx = 1\\ \text{if} \;f(x)\ne 0\;\text{then}\; p(x)\ne 0 ,\quad a\le x\le b$$

   令：

   $$f^*(x)=\begin{cases} \frac{f(x)}{p(x)},& p(x)\ne 0\\ 0,& p(x)=0 \end{cases}$$

   则有：$I=\int_a^b f(x)dx=\int_a^b f^*(x)p(x)dx$，它刚好是一个期望：设随机变量$X$服从分布$X\sim p(x)$，则$I=\mathbb E_{X\sim p}[f^*(X)]$。

   则期望法求积分的步骤是：

   - 任选一个满足条件的概率分布$p(x)$。
   - 根据$p(x)$，生成一组服从分布$p(x)$的随机数$x_1,x_2,\cdots,x_N$。
   - 计算均值$\bar I=\frac 1N\sum_{i=1}^N f^*(x_i)$，并将$\bar I$作为$I$的近似。

4. 在期望法求积分中，如果$a,b$均为有限值，则$p(x)$可以取均匀分布的概率密度函数：

  $$p(x) = \begin{cases} \frac {1}{b-a},& a\le x\le b\\ 0,& \text{else} \end{cases}$$

  此时$f^*(x) = (b-a) f(x)$，$\bar I = \frac{b-a}{N}\sum_{i=1}^Nf(x_i)$。

  其物理意义为：$\frac{\sum_{i=1}^Nf(x_i)}{N}$为在区间$[a,b]$上函数的平均高度，乘以区间宽度$b-a$就是平均面积。

  <p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20200603/lbJiHMl28rjh.png?imageslim">
  </p>
   

5. 对于期望$\mathbb E_p[f(X)]$，如果$p(x)$或者$f(x)$的表达式比较复杂，则也可以转化为另一个期望的计算。

   选择一个比较简单的概率密度函数$q(x)$，根据：

   $$\mathbb E_p[f(X)]=\int f(x) p(x)dx=\int f(x)\frac {p(x)}{q(x)}q(x)dx$$

   令$f^*(x) = f(x)\frac {p(x)}{q(x)}$，则原始期望转换为求另一个期望$\mathbb E_q[f^*(X)]$。此时可以使用期望法求积分的策略计算。

### 1.3 蒙特卡洛采样

1. 采样问题的主要任务是：根据概率分布$p(x)$，生成一组服从分布$p(x)$的随机数$x_1,x_2,\cdots$。

  - 如果$p(x)$就是均匀分布，则均匀分布的采样非常简单。

  - 如果$p(x)$是非均匀分布，则可以通过均匀分布的采样来实现。其步骤是：

    - 首先根据均匀分布$U(0,1)$随机生成一个样本$z_i$。

    - 设$\tilde P(x)$为概率分布$p(x)$的累计分布函数：$\tilde P(x)=\int_{-\infty}^x p(z) dz$。

      令$z_i=\tilde P(x_i)$，计算得到$x_i=\tilde P^{-1}(z_i)$，其中$\tilde P^{-1}$为反函数，则$x_i$为对$p(x)$的采样。

    <p align="center">
      <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20200603/lg108Kv963S8.png?imageslim">
    </p>
    

2. 通过均匀分布的采样的原理：假设随机变量$Z,X$满足$Z=\tilde P(X)$，则$X$的概率分布为：

  $$p_Z(z) \frac{d}{d x} \tilde P(x)$$

  因为$Z$是$[0,1]$上面的均匀分布，因此$p_Z(z)=1$；$\tilde P(x)$为概率分布$p(x)$的累计分布函数，因此$\frac{d}{d x} \tilde P(x)=p_X(x)$。因此上式刚好等于$p(x)$，即：$x_i$服从概率分布$p(x)$。

  这其中有两个关键计算：

  - 根据$p(x)$，计算累计分布函数$\tilde P(x)=\int_{-\infty}^x p(z) dz$。
  - 根据$z=\tilde P(x)$计算反函数$x=\tilde P^{-1}(z)$。

  如果累计分布函数无法计算，或者反函数难以求解，则该方法无法进行。

3. 对于复杂的概率分布$p(x)$，难以通过均匀分布来实现采样。此时可以使用`接受-拒绝采样` 策略。

  - 首先选定一个容易采样的概率分布$q(x)$，选择一个常数$k$，使得在定义域的所有位置都满足$p(x) \le k\times q(x)$。

  - 然后根据概率分布$q(x)$随机生成一个样本$x_i$。

  - 计算$\alpha_i =\frac{p(x_i)}{kq(x_i)}$，以概率$\alpha_i$接受该样本。

    具体做法是：根据均匀分布$U(0,1)$随机生成一个点$u_i$。如果$u_i \le \alpha_i$，则接受该样本；否则拒绝该样本。

    > 或者换一个做法：根据均匀分布$U(0,kq(x_i))$生成一个随机点，如果该点落在灰色区间（$(p(x_i),kq(x_i)])$）则拒绝该样本；如果该点落在白色区间（$[0,p(x_i)]$）则接受该样本。

  <p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20200603/MbhujuvHarn6.png?imageslim">
  </p>
   

4. `接受-拒绝采样` 在高维的情况下会出现两个问题：

   - 合适的$q$分布比较难以找到。
   - 难以确定一个合理的$k$值。

   这两个问题会导致拒绝率很高，无效计算太多。

## 二、马尔可夫链

1. 马尔可夫链是满足马尔可夫性质的随机过程。

   马尔可夫链$X_1,X_2,\cdots$描述了一个状态序列，其中每个状态值取决于前一个状态。$X_t$为随机变量，称为时刻$t$的状态，其取值范围称作状态空间。

   马尔可夫链的数学定义为：$P(X_{t+1}\mid X_t,X_{t-1},\cdots,X_1)=P(X_{t+1}\mid X_t)$。

### 2.1 马尔可夫链示例

1. 社会学家把人按照经济状况分成三类：下层、中层、上层。用状态 `1,2,3` 代表着三个阶层。社会学家发现：决定一个人的收入阶层的最重要因素就是其父母的收入阶层。

   - 如果一个人的收入属于下层，则他的孩子属于下层的概率是 0.65，属于中层的概率是 0.28，属于上层的概率是 0.07 。
   - 如果一个人的收入属于中层，则他的孩子属于下层的概率是 0.15，属于中层的概率是 0.67，属于上层的概率是 0.18 。
   - 如果一个人的收入属于上层，则他的孩子属于下层的概率是 0.12，属于中层的概率是 0.36，属于上层的概率是 0.52 。

   从父代到子代，收入阶层的变化的转移概率如下：

   |           | 子代阶层1 | 子代阶层2 | 子代阶层3 |
   | :-------: | :-------: | :-------: | :-------: |
   | 父代阶层1 |   0.65    |   0.28    |   0.07    |
   | 父代阶层2 |   0.15    |   0.67    |   0.18    |
   | 父代阶层3 |   0.12    |   0.36    |   0.52    |

2. 使用矩阵的表示方式，转移概率矩阵记作：

   $$\mathbf P= \begin{bmatrix} 0.65&0.28&0.07\\ 0.15&0.67&0.18\\ 0.12&0.36&0.52 \end{bmatrix}$$

   假设当前这一代人在下层、中层、上层的人的比例是概率分布$\vec\pi_0=(\pi_0(1),\pi_0(2),\pi_0(3))$，则：

   - 他们的子女在下层、中层、上层的人的概率分布是$\vec\pi_1=(\pi_1(1),\pi_1(2),\pi_1(3))=\vec\pi_0 \mathbf P$
   - 他们的孙子代的分布比例将是$\vec\pi_2=(\pi_2(1),\pi_2(2),\pi_2(3))=\vec\pi_1 \mathbf P=\vec\pi_0 \mathbf P^{2}$
   - ....
   - 第$n$代子孙在下层、中层、上层的人的比例是$\vec\pi_n=(\pi_n(1),\pi_n(2),\pi_n(3))=\vec\pi_{n-1} \mathbf P=\vec\pi_0 \mathbf P^{n}$

3. 假设初始概率分布为$\pi_0=(0.72,0.19,0.09)$，给出前 14 代人的分布状况：

   ```
     0 0.72 0.19 0.09
   ```

   ```
     1 0.5073 0.3613 0.1314
   ```

   ```
     2 0.399708 0.431419 0.168873
   ```

   ```
     3 0.34478781 0.46176325 0.19344894
   ```

   ```
     4 0.3165904368 0.4755635827 0.2078459805
   ```

   ```
     5 0.302059838985 0.482097475693 0.215842685322
   ```

   ```
     6 0.294554638933 0.485285430346 0.220159930721
   ```

   ```
     7 0.290672521545 0.486874112293 0.222453366163
   ```

   ```
     8 0.288662659788 0.487677173087 0.223660167125
   ```

   ```
     9 0.28762152488 0.488086910874 0.224291564246
   ```

   ```
     10 0.287082015513 0.488297220381 0.224620764107
   ```

   ```
     11 0.286802384833 0.488405577077 0.22479203809
   ```

   ```
     12 0.286657431274 0.488461538107 0.224881030619
   ```

   ```
     13 0.286582284718 0.488490482311 0.22492723297
   ```

   ```
     14 0.28654332537 0.488505466739 0.224951207891
   ```

   可以看到从第 9 代开始，阶层分布就趋向于稳定不变。

4. 如果换一个初始概率分布为$\vec\pi_0=(0.51,0.34,0.15)$，给出前 14 代人的分布状况：

   ```
   xxxxxxxxxx
   ```

   ```
     0 0.51 0.34 0.15
   ```

   ```
     1 0.4005 0.4246 0.1749
   ```

   ```
     2 0.345003 0.459586 0.195411
   ```

   ```
     3 0.31663917 0.47487142 0.20848941
   ```

   ```
     4 0.3020649027 0.4818790066 0.2160560907
   ```

   ```
     5 0.294550768629 0.48521729983 0.220231931541
   ```

   ```
     6 0.290668426368 0.486853301457 0.222478272175
   ```

   ```
     7 0.288659865019 0.487671049342 0.223669085639
   ```

   ```
     8 0.28761985994 0.488085236095 0.224294903965
   ```

   ```
     9 0.287081082851 0.488296834394 0.224622082755
   ```

   ```
     10 0.286801878943 0.488405532034 0.224792589023
   ```

   ```
     11 0.286657161801 0.488461564615 0.224881273584
   ```

   ```
     12 0.286582142693 0.488490512087 0.224927345221
   ```

   ```
     13 0.28654325099 0.488505487331 0.224951261679
   ```

   ```
     14 0.286523087645 0.488513240994 0.224963671362
   ```

   可以发现到第 8 代又收敛了。

5. 两次不同的初始概率分布，最终都收敛到概率分布$\vec\pi=(0.286 , 0.489, 0.225)$。 这说明收敛的行为和初始概率分布$\vec\pi_0$无关，而是由概率转移矩阵$\mathbf P$决定的。

   计算$\mathbf P^{n}$：

   ```
   xxxxxxxxxx
   ```

   ```
     0 [[ 0.65  0.28  0.07]
   ```

   ```
        [ 0.15  0.67  0.18]
   ```

   ```
        [ 0.12  0.36  0.52]]
   ```

   ```
     1 [ [ 0.4729  0.3948  0.1323]
   ```

   ```
         [ 0.2196  0.5557  0.2247]
   ```

   ```
         [ 0.1944  0.462   0.3436]]
   ```

   ```
     ...
   ```

   ```
     18 [[ 0.28650397  0.48852059  0.22497545]
   ```

   ```
         [ 0.28650052  0.48852191  0.22497757]
   ```

   ```
         [ 0.28649994  0.48852213  0.22497793]]
   ```

   ```
     19 [[ 0.28650272  0.48852106  0.22497622]
   ```

   ```
         [ 0.28650093  0.48852175  0.22497732]
   ```

   ```
         [ 0.28650063  0.48852187  0.2249775 ]]
   ```

   ```
     20 [[ 0.28650207  0.48852131  0.22497661]
   ```

   ```
         [ 0.28650115  0.48852167  0.22497719]
   ```

   ```
         [ 0.28650099  0.48852173  0.22497728]]
   ```

   ```
     21 [[ 0.28650174  0.48852144  0.22497682]
   ```

   ```
         [ 0.28650126  0.48852163  0.22497712]
   ```

   ```
         [ 0.28650118  0.48852166  0.22497717]]
   ```

   ```
     ...
   ```

   可以看到 ：

   $$\mathbf P^{18}=\mathbf P^{19}=\cdots=\begin{bmatrix} 0.286& 0.489 & 0.225 \\ 0.286 & 0.489 & 0.225 \\ 0.286& 0.489 & 0.225 \\ \end{bmatrix}$$

   发现当$n$足够大的时候， 矩阵$\mathbf P^{n}$收敛且每一行都稳定收敛到$\vec\pi=(0.286 , 0.489, 0.225)$。

### 2.2 平稳分布

1. 马尔可夫链定理：如果一个非周期马尔可夫链具有转移概率矩阵$\mathbf P$，且它的任何两个状态是联通的，则有：

   $$\lim_{n\rightarrow \infty} \mathbf P^{n}=\begin{bmatrix} \pi {(1)}&\pi {(2)}&\cdots&\pi {(j)}&\cdots\\ \pi {(1)}&\pi {(2)}&\cdots&\pi {(j)}&\cdots\\ \vdots&\vdots&\ddots&\vdots&\vdots\\ \pi {(1)}&\pi {(2)}&\cdots&\pi {(j)}&\cdots\\ \vdots&\vdots&\ddots&\vdots&\vdots \end{bmatrix}\\ \pi {(j)} =\sum_{i=0}^{\infty} \pi {(i)} P_{i,j}$$

   其中：

   -$1,2,\cdots,j,\cdots$为所有可能的状态。

   -$P_{i,j}$是转移概率矩阵$\mathbf P$的第$i$行第$j$列的元素，表示状态$i$转移到状态$j$的概率。

   - 概率分布$\vec\pi$是方程$\vec\pi\mathbf P= \vec\pi$的唯一解，其中$\vec\pi=(\pi {(1)},\pi {(2)},\cdots,\pi {(j)},\cdots),\sum_{j=0}^{\infty}\pi {(j)} =1$。

     称概率分布$\vec \pi$为马尔可夫链的平稳分布。

2. 注意，在马尔可夫链定理中：

   - 马尔可夫链的状态不要求有限，可以是无穷多个。

   - 非周期性在实际任务中都是满足的。

   - 两个状态的连通指的是：状态$i$可以通过有限的$n$步转移到达$j$(并不要求从状态$i$可以直接一步转移到状态$j$）。

     马尔可夫链的任何两个状态是联通的含义是：存在一个$n$，使得矩阵$\mathbf P^{n}$中的任何一个元素的数值都大于零。

3. 从初始概率分布$\vec\pi_0$出发，在马尔可夫链上做状态转移，记时刻$i$的状态$X_i$服从的概率分布为$\vec\pi_i$， 记作$X_i \sim \vec \pi_i$，则有：

   $$X_0 \sim \vec\pi_0\\ X_1 \sim \vec\pi_1\\ \cdots\\ X_n \sim \vec\pi_n\\ X_{n+1} \sim \vec \pi_{n+1}\\ \cdots$$

   假设到达第$n$步时，马尔可夫链收敛，则有：

   $$X_n \sim \vec\pi\\ X_{n+1} \sim \vec\pi\\ \cdots$$

   所以$X_n,X_{n+1},X_{n+2},\cdots$都是同分布的随机变量（当然它们并不独立）。

   如果从一个具体的初始状态$x_0$开始，然后沿着马尔可夫链按照概率转移矩阵做调整，则得到一个转移序列$x_0,x_1,\cdots,x_{n},x_{n+1},\cdots$。

   根据马尔可夫链的收敛行为，当$n$较大时，$x_{n},x_{n+1},\cdots$将是平稳分布$\vec \pi$的样本。

4. 定理：如果非周期马尔可夫链的转移矩阵$\mathbf P$和某个分布$\vec \pi$满足：$\pi(i)P_{i,j}=\pi(j)P_{j,i}$，则$\vec \pi$是马尔可夫链的平稳分布。

   这被称作马尔可夫链的细致平稳条件`detailed balance condition` ，其证明如下：

   $$\pi(i)P_{i,j}=\pi(j)P_{j,i} \rightarrow \sum_{i=1}^{\infty}\pi(i)P_{i,j}=\sum_{i=1}^{\infty}\pi(j)P_{j,i}= \pi(j)\sum_{i=1}^{\infty}P_{j,i}=\pi(j) \rightarrow \vec\pi\mathbf P=\vec\pi$$

   .

## 三、MCMC 采样

1. 概率图模型中最常用的采样技术是马尔可夫链蒙特卡罗方法`Markov Chain Monte Carlo:MCMC`。

   给定连续随机变量$X \in \mathcal X$的概率密度函数$\tilde p(x)$，则$X$在区间$\mathbb A$中的概率可以计算为：

   $$P(\mathbb A)=\int_\mathbb A\tilde p(x)dx$$

   如果函数$f: \mathcal X \longmapsto \mathbb R$， 则可以计算$f(X)$的期望：$\mathbb E_{X \sim \tilde p(x)}[f(X)]=\int f(x)\tilde p(x)dx$。

   - 如果$X$不是一个单变量，而是一个高维的多元变量$\vec X$，且服从一个非常复杂的分布，则对于上式的求积分非常困难。为此，`MCMC`先构造出服从$\tilde p$分布的独立同分布随机变量$\mathbf {\vec x}_1,\mathbf {\vec x}_2,\cdots,\mathbf {\vec x}_N$， 再得到$\mathbb E_{\vec X\sim \tilde p(\mathbf {\vec x})}[f(\vec X)]$的无偏估计：

     $$\mathbb E_{\vec X\sim \tilde p(\mathbf {\vec x})}[f(\vec X)]=\frac 1N \sum_{i=1}^{N}f(\mathbf {\vec x}_i)$$

   - 如果概率密度函数$\tilde p(\mathbf {\vec x})$也很复杂，则构造服从$\tilde p$分布的独立同分布随机变量也很困难。`MCMC` 方法就是通过构造平稳分布为$\tilde p(\mathbf {\vec x})$的马尔可夫链来产生样本。

### 3.1 MCMC 算法

1. `MCMC` 算法的基本思想是：先设法构造一条马尔可夫链，使其收敛到平稳分布恰好为$\tilde p$。然后通过这条马尔可夫链来产生符合$\tilde p$分布的样本。最后通过这些样本来进行估计。

   这里马尔可夫链的构造至关重要，不同的构造方法将产生不同的`MCMC`算法。`Metropolis-Hastings:MH`算法是`MCMC`的重要代表。

2. 假设已经提供了一条马尔可夫链，其转移矩阵为$\mathbf Q$。目标是另一个马尔科夫链，使转移矩阵为$\mathbf P$、平稳分布是$\tilde p$。

   通常$\tilde p(i)Q_{i,j} \ne \tilde p(j)Q_{j,i}$，即$\tilde p$并不满足细致平稳条件不成立。但是可以改造已有的马尔可夫链，使得细致平稳条件成立。

   引入一个函数$\alpha(i,j)$，使其满足：$\tilde p(i)Q_{i,j}\alpha(i,j)=\tilde p(j)Q_{j,i}\alpha(j,i)$。如：取$\alpha(i,j)=\tilde p(j)Q_{j,i}$，则有：

   $$\tilde p(i)Q_{i,j}\alpha(i,j)=\tilde p(i)Q_{i,j}\tilde p(j)Q_{j,i}=\tilde p(j)Q_{j,i}\tilde p(i)Q_{i,j}=\tilde p(j)Q_{j,i}\alpha(j,i)$$

   令：$Q^{\prime}_{i,j}=Q_{i,j}\alpha(i,j),Q^{\prime}_{j,i}=Q_{j,i}\alpha(j,i)$，则有$\tilde p(i)Q^{\prime}_{i,j}=\tilde p(j)Q^{\prime}_{j,i}$。其中$Q^{\prime}_{i,j}$构成了转移矩阵$\mathbf Q^{\prime}$。而$\mathbf Q^{\prime}$恰好满足细致平稳条件，因此它对应的马尔可夫链的平稳分布就是$\tilde p$。

3. 在改造$\mathbf Q$的过程中引入的$\alpha(i,j)$称作接受率。其物理意义为：在原来的马尔可夫链上，从状态$i$以$Q_{i,j}$的概率跳转到状态$j$的时候，以$\alpha(i,j)$的概率接受这个转移。

   - 如果接受率$\alpha(i,j)$太小，则改造马尔可夫链过程中非常容易原地踏步，拒绝大量的跳转。这样使得马尔可夫链遍历所有的状态空间需要花费太长的时间，收敛到平稳分布$\tilde p$的速度太慢。

   - 根据推导$\alpha(i,j)=\tilde p(j)Q_{j,i}$，如果将系数从 1 提高到$K$，则有：

     $$\alpha^{*}(i,j)=K\tilde p(j)Q_{j,i}=K\alpha (i,j)\\ \alpha^{*}(j,i)=K\tilde p(i)Q_{i,j}=K\alpha (j,i)$$

     于是：$\tilde p(i)Q_{i,j}\alpha^{*}(i,j)=K\tilde p(i)Q_{i,j}\alpha(i,j)=K\tilde p(j)Q_{j,i}\alpha(j,i)=\tilde p(j)Q_{j,i}\alpha^{*}(j,i)$。因此，即使提高了接受率，细致平稳条件仍然成立。

4. 将$\alpha(i,j),\alpha(j,i)$同比例放大，取：$\alpha(i,j)=\min\{\frac{\tilde p(j)Q_{j,i}}{\tilde p(i)Q_{i,j}},1\}$。

   - 当$\tilde p(j)Q_{j,i}= \tilde p(i)Q_{i,j}$时：$\alpha(i,j)=\alpha(j,i)=1$，此时满足细致平稳条件。
   - 当$\tilde p(j)Q_{j,i}\gt \tilde p(i)Q_{i,j}$时：$\alpha(i,j)=1,\alpha(j,i)=\frac{\tilde p(i)Q_{i,j}}{\tilde p(j)Q_{j,i}}$，此时满足细致平稳条件。
   - 当$\tilde p(j)Q_{j,i} \lt \tilde p(i)Q_{i,j}$时：$\alpha(i,j)=\frac{\tilde p(j)Q_{j,i}}{\tilde p(i)Q_{i,j}},\alpha(j,i)=1$，此时满足细致平稳条件。

5. `MH` 算法：

   - 输入：

     - 先验转移概率矩阵$\mathbf Q$
     - 目标分布$\tilde p$

   - 输出： 采样出的一个状态序列$\{x_0,x_1,\cdots,x_{n},x_{n+1},\cdots\}$

   - 算法：

     - 初始化$x_0$

     - 对$t=1,2,\cdots$执行迭代。迭代步骤如下：

       - 根据$Q(x^{*} \mid x_{t-1})$采样出候选样本$x^{*}$，其中$Q$为转移概率函数。

       - 计算$\alpha(x^{*} \mid x_{t-1})$：

         $$\alpha( x^{*} \mid x_{t-1})=\min \left(1,\frac{\tilde p( x^{*})Q( x _{t-1} \mid x^{*})}{\tilde p( x _{t-1})Q( x^{*} \mid x _{t-1})} \right)$$

       - 根据均匀分布从$(0,1)$内采样出阈值$u$，如果$u \le \alpha( x^{*} \mid x_{t-1})$，则接受$x^{*}$， 即$x_{t}= x^{*}$。否则拒绝$x^{*}$， 即$x_{t}= x_{t-1}$。

     - 返回采样得到的序列$\{x_0,x_1,\cdots,x_{n},x_{n+1},\cdots\}$

   > 注意：返回的序列中，只有充分大的$n$之后的序列$\{ x_{n}, x_{n+1},\cdots\}$才是服从$\tilde p$分布的采样序列。

### 3.2 Gibbs 算法

1. `MH`算法不仅可以应用于一维空间的采样，也适合高维空间的采样。

   对于高维的情况，由于接受率$\alpha$的存在（通常$\alpha \lt 1$），`MH`算法的效率通常不够高，此时一般使用 `Gibbs` 采样算法。

2. 考虑二维的情形：假设有概率分布$\tilde p(x,y)$，考察状态空间上$x$坐标相同的两个点$A(x_1,y_1),B(x_1,y_2)$，可以证明有：

   $$\tilde p(x_1,y_1)\tilde p(y_2\mid x_1)=\tilde p(x_1)\tilde p(y_1\mid x_1)\tilde p(y_2\mid x_1)\\ \tilde p(x_1,y_2)\tilde p(y_1\mid x_1)=\tilde p(x_1)\tilde p(y_2\mid x_1)\tilde p(y_1\mid x_1)$$

   于是$\tilde p(x_1,y_1)\tilde p(y_2\mid x_1)=\tilde p(x_1,y_2)\tilde p(y_1\mid x_1)$。则在$x=x_1$这条平行于$y$轴的直线上，如果使用条件分布$\tilde p(y\mid x_1)$作为直线上任意两点之间的转移概率，则这两点之间的转移满足细致平稳条件。

   同理：考察$y$坐标相同的两个点$A(x_1,y_1),C(x_2,y_1)$， 有$\tilde p(x_1,y_1)\tilde p(x_2\mid y_1)=\tilde p(x_2,y_1)\tilde p(x_1\mid y_1)$。在$y=y_1$这条平行于$x$轴的直线上，如果使用条件分布$\tilde p(x\mid y_1)$作为直线上任意两点之间的转移概率，则这两点之间的转移满足细致平稳条件。

   可以构造状态空间上任意两点之间的转移概率矩阵$\mathbf Q$： 对于任意两点$A=(x_A,y_A),B=(x_B,y_B)$， 令从$A$转移到$B$的概率为$Q(A\rightarrow B)$：

   - 如果$x_A=x_B=x^{*}$，则$Q(A\rightarrow B)=\tilde p(y_B\mid x^{*})$。
   - 如果$y_A=y_B=y^{*}$，则$Q(A\rightarrow B)=\tilde p(x_B\mid y^{*})$。
   - 否则$Q(A\rightarrow B)=0$。

   采用该转移矩阵$\mathbf Q$，可以证明：对于状态空间中任意两点$A,B$，都满足细致平稳条件：

   $$\tilde p(A)Q(A\rightarrow B)=\tilde p(B)Q(B\rightarrow A)$$

   于是这个二维状态空间上的马尔可夫链将收敛到平稳分布$\tilde p(x,y)$，这就是吉布斯采样的原理。

3. `Gibbs` 算法：

   - 输入：目标分布$\tilde p(\mathbf{\vec x})$，其中$\mathbf{\vec x}\in \mathbb R^n$

   - 输出： 采样出的一个状态序列$\{\mathbf{\vec x}_0,\mathbf{\vec x}_1,\cdots\}$

   - 算法步骤：

     - 初始化：随机初始化$\mathbf{\vec x}_0,t=0$。

     - 执行迭代，迭代步骤如下：

       - 随机或者以一定次序遍历索引$1,2,\cdots,n$。遍历过程为（设当前索引为$i$）：

         - 将$x_{1},\cdots,x_{i-1},x_{i+1},\cdots,x_{n}$保持不变，计算条件概率：$\tilde p(x_{i}\mid x_{1}=x_{t,1},\cdots,x_{i-1}=x_{t,i-1},x_{i+1}=x_{t,i+1},\cdots,x_{n}=x_{t,n})$。

           > 该条件概率就是状态转移概率$Q(A\rightarrow B)$

         - 根据条件概率$\tilde p(x_{i}\mid x_{1}=x_{t,1},\cdots,x_{i-1}=x_{t,i-1},x_{i+1}=x_{t,i+1},\cdots,x_{n}=x_{t,n})$对分量$x_{i}$进行采样，假设采样结果为$x_{t,i}^*$，则获得新样本$\mathbf{\vec x}_{t+1}=(x_{t,1},\cdots,x_{t,i-1},x_{t,i}^*,x_{t,i+1},\cdots,x_{t,n})^T$。

         - 令$t\leftarrow t+1$，继续遍历。

     - 最终返回一个状态序列$\{\mathbf{\vec x}_0,\mathbf{\vec x}_1,\cdots\}$。

4. 吉布斯采样`Gibbs sampling` 有时被视作`MH`算法的特例，它也使用马尔可夫链获取样本。