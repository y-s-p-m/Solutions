# A

如果有 $()$ 那么肯定是不合法的

有两种很简单的构造，`()()()()...()` 和 `((((...))))`，如果一个串是第一种构造的子串那么一定不是第二种构造的子串，反之亦然。

使用 python 取之

# B

把 $m\% k$ 的余数补齐，再把多出来的 $1$ 价格 regular coins $m$个一组f

使用 python 取之

# C

使用 SG 函数取之，注意到 $\text{mex}(\{0\})=1$ 而且你也只关注每个位置 $SG$ 函数是不是零，所以记录前缀 SG 信息即可。

注意 $\arg$ 前缀最小值不能作为答案

# D

设 $f_{i,j,k}$ 表示只考虑交换后的前 $i$ 个字符，其中有 $j$ 个 $0$ ，$01$ subsequence 有 $k$ 个

转移枚举 **最终结果串** 在第 $i$ 个位置是 $0$ 还是 $1$，前者使状态中 $j\leftarrow j+1$ ，后者使状态中 $k\leftarrow j+k$

如果这个位置不是 $0$ 且被钦定成了 $0$ 就要 $+1$。

比赛的时候注意到了这么个性质：交换 $10$ 可以让 $01$ 对子增加区间长度 -1 个，先编了一个贪心 WA 了，又编了一个 dp，但是没写。和 std 也不一样。 

# E

变成 $n-1$ 个节点的最短路，注意到第三类边形成若干团，于是建立虚点，现在图上边数变成 $n+|\Sigma|^2$，$0/1$ bfs 可以实现 $\Theta(qn)$ 的复杂度

一条路径如果不经过虚点则容易计算答案，否则可以预处理 $s->image,image->t$ 的最短路，枚举 $image$ 统计答案，最终复杂度 $\Theta(|\Sigma|^2(|\Sigma|^2+n+q)$

# F

观察一下 $(a,b),(x,y),ax+by$ ，你是不是觉得这和向量点积没什么区别？在 $(x,y)$ 保持一样的情况下，区别点积大小的方式是投影线段长度

考虑两个向量 $x_1=(a_1,b_1),x_2=(a_2,b_2)$ 的点积顺序，只有 $(a_1,b_1),(a_2,b_2)$ 的连线垂线满足以该直线上点为终点的向量与 $x_1,x_2$ 点积相同

那么可以 $\Theta(n^2)$ 计算出来任意两个向量的分界线。那么可以枚举所有可能的顺序，每种顺序可以通过前缀和来判断是否合法。

每次交换的括号后对前缀和的影响是一个区间加 $\pm1$ ，可以使用数据结构维护之。