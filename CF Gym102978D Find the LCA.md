# Description

给定一个长度为 $n$ 的序列 $a_i$，对于所有 $(n-1)!$ 个满足 $i\in [1,n]fa_i<i$ 的树，求

$$\sum_{\rm Tree} \prod_{x\in \rm subTree(LCA(n,n-1)} A_x$$

$$n\le 250000$$

# Solution

先对于每个 $k\in [1,n]$ 计算一个大小为 $k$ 的树，有多少树的形态使得树里面标号最大的两个点的 LCA 是树根

先考察一个单独的 $k$，容斥如果两个点选在了一个子树的情况，对于一个大小为 $siz$ 的子树会有 $siz^2$ 个方案

暴力的想法就是枚举在所有形态中大小为 $i$ 的子树出现了多少次，乘上自己形态数量，其它点的选择方案和选择标号的方案 从总方案中减去即可，结果表达为：

$$(k-2)^2(k-3)!-\sum_{l=1}^{k-3}\binom{k-3}{i}(i-1)!(k-3-i)!$$

后面的求和可以卷积处理，所以可以做到线对预处理 $\Theta(1)$ 查询

~~原来是这样子，那么场上很多人过就能解释了~~

给出结论：在 $k\ge 3$ 时上面表达式的值为 $\frac{(k-1)!}2$，证明考虑在合法方案和不合法方案中构造双射

我的构造是将一棵不合法的树重定根为 $k$，这样子原来 LCA(k,k-1) 现在是 LCA(1,k-1)，将这个 LCA 和其 $k-1$ 向子树砍下来接到 $1$ 那里，如果 LCA 是 $k-1$ 就只把这个点砍下来

看起来是题解的逆过程不过问题不大，那么至此表达式如下

$$\begin{aligned}Ans&=a_na_{n-1}\left[(n-2)!+\frac{(n-1)!}2\prod_{i=1}^{n-2}a_i+\sum_{i=3}^{n-1}(n-i-1)!\frac{(i-1)!}2[x^{i-2}] \sum_{i=2}^{n-2}F_i\right]\\
F_i&=(i-1)a_ix\prod_{j=i+1}^{n-2}(1+a_jx)
\end{aligned}$$

$F_i(x)$ 前缀的 $i-1$ 表示这个点作为子树根的时候它还得连接上 $[1,i-1]$ 的点，最后简记求解中的分治 $\rm NTT$ 的过程：

分治过程维护两个多项式 $F_{l,r},G_{l,r}$ 分别表示 $a_ix$ 只乘 $j\in[l,r]$ 中的 $1+a_jx$ 的乘积的和 以及区间 $1+a_ix$ 的乘积，最后的答案就是 $F_{2,n}$ 

合并区间时 $F_{l,r}\leftarrow F_{mid+1,r}+F_{l,mid}\times G_{mid+1,r}$，$G_{l,r} \to G_{l,mid}\times G_{mid+1}r$ 

感觉可能受制于分治乘法的套路式了，这个是稍加思考就能得到的

# Code

```cpp
inline pair<poly,poly> solve(int l,int r){
    if(r<l) return {{0},{0}};
    if(l==r){
        poly F={0,mul(l-1,a[l])},G={1,a[l]};
        return make_pair(F,G);
    } int mid=(l+r)>>1;
    pair<poly,poly>ls=solve(l,mid),rs=solve(mid+1,r);
    return make_pair(Plus(Mul(ls.fir,rs.sec),rs.fir),Mul(ls.sec,rs.sec));
}
signed main(){
    fac[0]=1; rep(i,1,250000) fac[i]=mul(fac[i-1],i);
    n=read(); rep(i,1,n) a[i]=read();
    int ans=fac[n-2],Mult=1;
    rep(i,1,n-2) ckmul(Mult,a[i]);
    ckadd(ans,mul(Mult,mul(inv2,fac[n-1])));
    poly F=solve(2,n-2).fir;
    for(int i=3;i<n;++i){
        int val=mul(inv2,mul(fac[i-1],fac[n-i-1]));
        ckadd(ans,mul(F[i-2],val));
    }
    print(mul(mul(a[n],a[n-1]),ans));
    return 0;
}
```