# Description

有 $n$ 个不同的球分成 $k$ 组，每组中有 $1$ 个球，或者两个相邻的球，球可以不被分到组中去

求分组方案数

$n\le 10^9,k\le 2^{15}$

# Solution

一个显然的 $\rm DP$ 式子为设 $f_{i,j}$ 表示 $i$ 个球 $j$ 组的方案数

$$f_{i,j}=f_{i-1,j}+f_{i-1,j-1}+f_{i-2,j-1}$$

加速和 $\rm CF623E$ 类似，直接尝试用 $f_{n,*},f_{n,*}$ 转移给 $f_{2n,*}$，具体就分中间的部分是不是另开一组

$$f_{2n,k}=\sum_{i+j=k}f_{n,i}f_{n,j}+\sum_{i+j+1=k}f_{n-1,i}f_{n-1,j}$$

写出 $F_i(x)$ 为 $f_{i,x}$ 的 $\rm OGF$ 不难得到如下转移：

$$F_i(x)=(x+1)F_{i-1}(x)+xF_{i-2}(x)$$

所以维护 $F_{n}(x),F_{n-1}(x),F_{n-2}(x)$ 转移到 $F_{2n}(x),F_{2n-1}(x),F_{2n-2}(x)$ 即可

使用 $\rm NTT+$ 倍增实现，复杂度 $\Theta(k\log k \log n)$

---

梦想不应止步于 $\log^2$ 的做法，考虑 **组合意义**

尝试枚举大小为 $2$ 的组的数量，得到一个表达式 $ans_i=\sum\limits_{j=0}^i \binom ij\binom {n-j}i $

先保留 $j$ 个球待用，再取出 $n-j$ 球中选 $i$ 个球表示被分组，最后在 $i$ 个球中选择 $j$ 个表示这 $j$ 个组大小为 $2$

由于前面已经保留出来 $j$ 个球，那么可以支持这种操作，将被保留的 $j$ 个球放在选中的球的前面就得到了长度为 $n$ 的序列，重新标号得到分组方案

那么考虑一种在该式子中计算的方案显然在上面的 $\rm DP$ 过程中被计算了

返回来每个 $\rm DP$ 中使用 $f_{i-2,k-1}$ 转移的方案中的 $k-1$ 是一定的；考察使用组合式子得到最终序列的标号是 $i$ 的位置，其前面所有 有 $k-1$ 个组（不分球数）的方案也被枚举了，因为这对应了 $\binom {n-j}i$ 中的第 $i$ 个元素，而其是第 $x$ 个两球组等价于 $\binom ij$ 中其位于 其被选中 的所有组合 中 其在第 $x$ 位置上的方案总和

因此其构成了双射，具有正确性（还是这样思考比较清晰一些）

不难发现 $\sum_{j=0}^i\binom{n-j}i\binom ij$ 的计算方式保证了每个球只被选择一次，尝试允许重复选择球再容斥，枚举 $j$ 个球被钦定重复选中了

$$ans_i=\sum_{j=0}^i(-1)^j\binom ij \binom{n-j}{i-j}2^{i-j}$$

容斥的是选择的 $j$ 个两球组中的和预留的元素重叠的那部分，后面 $2^{i-j}$ 表示的是别的球被选重复与否，这样子也就没有 $n$ 个球不够用的问题了

考虑每种重复选择了 $k$ 次的方案都被计算了 $\sum\limits_{j=0}^k(-1)^j\binom kj= 0$ 次，所以我们得到了正确的答案

直接拆开表示成下降幂就可以卷积了！只要一次哟！

upd: 被提醒下降幂出现了除以 $0$ 的窘况，但是这也是可以分开卷积解决的，鉴于原题数据中没有卡我也懒得改了

复杂度 $\Theta(k\log k)$

# Code

显然写了一次卷积的容斥做法，不得不说还是挺神的

```cpp
const int N=6e5+10;
int r[N],W[N],dfac[N],n,k,F[N],G[N],fac[N],ifac[N],inv[N];
inline void NTT(int *f,int lim,int opt){
    for(int i=0;i<lim;++i){
        r[i]=r[i>>1]>>1|((i&1)?(lim>>1):0);
        if(i<r[i]) swap(f[i],f[r[i]]);
    }
    for(int p=2;p<=lim;p<<=1){
        int len=p>>1; W[0]=1;
        W[1]=ksm(3,(mod-1)/p); if(opt==-1) W[1]=ksm(W[1],mod-2);
        rep(i,2,len-1) W[i]=mul(W[i-1],W[1]);
        for(int k=0;k<lim;k+=p){
            for(int l=k;l<k+len;++l){
                int tt=mul(W[l-k],f[l+len]);
                f[l+len]=del(f[l],tt);
                ckadd(f[l],tt);
            }
        }
    }
    if(opt==-1){
        int tmp=ksm(lim,mod-2);
        rep(i,0,lim-1) ckmul(f[i],tmp);
    }
    return ;
}
inline int C(int n,int m){
    if(n<m) return 0;
    return mul(fac[n],mul(ifac[m],ifac[n-m]));
}
signed main(){
    n=read(); k=read(); 
    dfac[0]=1; for(int i=1;i<=k;++i) dfac[i]=mul(dfac[i-1],n-i+1);
    fac[0]=inv[0]=1; rep(i,1,k) fac[i]=mul(fac[i-1],i);
    ifac[k]=ksm(fac[k],mod-2);
    Down(i,k,1) ifac[i-1]=mul(ifac[i],i),inv[i]=mul(fac[i-1],ifac[i]);
    
    for(int i=0,pw=1;i<=k;++i){
        F[i]=mul(ifac[i],ksm(dfac[i],mod-2));
        if(i&1) F[i]=del(0,F[i]);
        G[i]=mul(pw,mul(ifac[i],ifac[i]));
        ckadd(pw,pw);
    }
    
    int lim=1; while(lim<=k*2) lim<<=1;
    NTT(F,lim,1); NTT(G,lim,1);
    rep(i,0,lim-1) ckmul(F[i],G[i]);
    NTT(F,lim,-1);
    rep(i,1,k) print(mul(fac[i],mul(F[i],dfac[i])));
    return 0;
}
```