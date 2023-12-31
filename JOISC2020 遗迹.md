# Description

[https://loj.ac/p/3276](https://loj.ac/p/3276)

给定一个有 $2n$ 个建筑的街道，建筑的高度的多重集等于多重集 $\{1,1,2,2,\dots n,n\}$

会发生 $n$ 次地震，每次地震会使得每个高度 $i$ 的建筑中非最右侧建筑的高度减少 $1$，减至 $0$ 则不再减

最后有 $n$ 个高度为 $1\sim n$ 的建筑会被保留，给定位置集合，求有多少初始高度排列能得到呢？

$n\le 600$

# Solution

设每个数字第一次出现为 $\rm fir_i$ 第二次出现类似的为 $\rm sec_i$

考虑给定一组 $A$ 求得最后的 $B$ 的过程，以下两种是等价的

- 从 $n$ 到 $1$ 扫每个 $i$，维护集合 $S$，初始为空，每次将 $S\leftarrow S\cup\{fir_i,sec_i\}$ 并弹出最大元素作为这个位置最后保留的 $pos_i$，那么得到的 $B$ 就是所有 $pos_i$ 的并集

- 从 $2n$ 到 $1$ 扫描每个位置，维护长度为 $n$ 的序列 $B$ 初始为全 $0$，对于 $i$ 找到 $\le A_i$ 仍为 $0$ 的 $B_j$ 并将 $B_j$ 置为 $i$

我只想到了第一个过程所以这题没得做，执行第二个过程并设 $f_{i,j}$ 表示从后往前走到了 $i$，前面的放置满足 $\rm mex=j+1$ 的方案数，初始化 $f_{2n+1,0}=1$，答案为 $f_{1,n}$

讨论 $i$ 是否属于题目中所给的 $B$ 有两种转移，设 $B_j\ge i$ 的数量为 $s_i$，$\ge i$ 中不在 $B$ 中的数字数量为 $des_i$，简记 $len_i=2n-i+1$：

- $i\notin B$ 时比较容易，这时候 $A_i$ 的取值只能是 $[1,j]$ 否则会让 $j$ 增加，那么 $f_{i+1,j}$ 贡献给 $f_{i,j}$ 的系数是没有被两次放置的数字，形式化表示为 $j-des_{i+1}$

- $i\in B$ 时分 $A_i=j+1$ 与否进行讨论，如果 $A_i>j+1$ 时不会对第二维造成变化，先 $f_{i,j}\leftarrow f_{i+1,j}$ 

    如果 $A_i=j+1$ 枚举 $f_{i+1,j}$ 贡献到的 $f_{i,j+k}(k>0)$ 首先选出来 $suf_i-j-1$ 个数字中的 $k$ 个最后变成了 $(j+1,j+k]$（左边是开区间的原因是 这些被选的点的下标 $\ge i+1$），即先附加 $\binom{suf_i-j-1}{k-1}$

    同时将高度为 $i$ 的两根柱子看作是本质不同的柱子来转移，最后除 $2^n$，不难发现这和 $i\notin B$ 的转移是吻合的，所以这时候 $A_i$ 的取值有 $k+1$ 种即 $(j+1,j+k]$ 中的一个或者 $j+1$ 的两个

    剩下的就考虑 $k-1$ 个数字分配高度使得最终形成长度为 $k-1$ 的连续段的方案数
    
    朴素的 dp 的做法就是 $g_{i,j}$ 表示 $j$ 个位置填 $\le i$ 的数字并使其合法的方案数，那么 $g_{i,j}=2jg_{i-1,j-1}+(j-1)jg_{i-1,j-2}+g_{i-1,j}$ 

    含义就是在高度 $i$ 填 $0/1/2$ 个柱子，这里注意每个高度都有两个本质不同的柱子了所以要在 $g_{i-1,j-1}$ 处乘 $2$ 即 $g_{i-1,j-2}$ 的系数 $\binom j 2$ 也乘 $2$

    预处理即可

时间复杂度 $\Theta(n^3)$

# Code

```cpp
const int N=1210;
int n,dp[N][N],suf[N],fac[N],ifac[N],f[N];
inline int C(int n,int m){return mul(fac[n],mul(ifac[m],ifac[n-m]));}
bool vis[N];
signed main(){
    f[0]=fac[0]=1;
    rep(i,1,1200) fac[i]=mul(fac[i-1],i);
    ifac[1200]=ksm(fac[1200],mod-2);
    Down(i,1200,1) ifac[i-1]=mul(ifac[i],i);
    n=read(); 
    for(int i=1;i<=n;++i){
        int pos=read();
        vis[pos]=1;
        f[i]=mul(fac[i+1],mul(ifac[i+2],C(2*(i+1),i+1)));
        ckmul(f[i],fac[i]);
    }
    dp[n<<1|1][0]=1;
    Down(i,2*n,1){
        suf[i]=suf[i+1]+vis[i];
        if(!vis[i]){
            int des=2*n-i-suf[i];
            for(int j=des+1;j<=suf[i];++j) dp[i][j]=mul(dp[i+1][j],j-des);
        }else{
            for(int j=0;j<=suf[i+1];++j) if(dp[i+1][j]){
                ckadd(dp[i][j],dp[i+1][j]);
                for(int k=1;j+k<=suf[i];++k){
                    int addi=mul(f[k-1],mul(k+1,C(suf[i+1]-j,k-1)));
                    ckadd(dp[i][j+k],mul(dp[i+1][j],addi));
                }
            }
        }
    }
    print(mul(dp[1][n],ksm((mod+1)/2,n)));
    return 0;
}
```

这题真的做完了吗？

另附 $g_{i,i}=Cat_{i+1}\times(i!)$ 的简单解释

考虑 $g_{n,n}$ 的实际含义，将 $1\sim 2n$ 写到序列上，中间挑 $n$ 个数字出来，限制 $pre_{2k}\ge k$，将有数的位置看成 $+1$，空位置看成 $-1$ 那么变成了偶数位置 $\ge 0$，这就是卡特兰数第 $n+1$ 项，可以用不经过直线 $y=-1$ 来理解

更直白地（来自 happyguy656）：偶数位置的限制 $\ge 0$ 再添加进去奇数位置 $\ge -1$ 那么就都有限制了，再在序列开头加一个左括号，结尾加一个右括号就变成了偶数位置 $\ge 0$，奇数位置 $\ge 1$，就是正宗卡特兰数了，序列长度 $2n+2$，所以是第 $n+1$ 项

最后乘阶乘的原因是这 $n$ 个数字是等价的，轮换也无妨

[在 LOJ 讨论区里 nealchen 写了这样一个结论](https://loj.ac/d/2491)：长度为 $n$ 的括号序列的合法子序列的数量为 $Cat_{n+1}$，考察了最后一个括号的匹配者并使用了 $\rm OGF$ 相同来证明的，有没有组合意义解释呢？