# Description

一个大小为 $n\times m$ 的矩阵，每个格子可以为黑或白色，即共 $2^{nm}$ 种形态，记 $A_{1\sim n}$ 为每行第一个黑格子标号，如果没有就是 $m+1$，$B_{1\sim m}$ 为每列第一个格子标号，$C_{1\sim m}$ 为每列最后一个格子标号，如果这列没有格子分别为 $n+1/0$

求三元组 $(A,B,C)$ 的数量，$n\le 8000,m\le 200$

# Solution

设 $f_{j,i}$ 表示在 $j$ 列上放过黑色格子，这些黑色格子所在行的并集大小为 $i$

考虑新添加一列并放黑格子，这时候只能在最后一列上进行放置，让 $f_{j,i}$ 转移到：

- $f_{j+1,i}$，转移系数就是这一列上黑格子方案，考虑可以放几个格子得到系数为 $1+i+\binom{i}2$

- 枚举 $k,f_{j+1,i+k}$，在原有 $i$ 行基础上新 **插入** $k$ 行进来，这 $k$ 行上的第 $j+1$ 列都有黑色格子。这时由于原来的行上都有了黑色格子，那么这列上的黑色格子对其 $A_i$ 并不造成影响

    考虑列上的格子对 $B_{j+1},C_{j+1}$ 带来的影响，这等价于计算 $B_{j+1}-1,C_{j+1}+1$ 的不同对数，这等价于选出 $k+2$ 行并让最高/低两行作为作为白格子
    
    另一种做法是考察几个新增加行作为 $B_{j+1}/C_{j+1}$，殊途同归，得数都是 $\binom{i+k+2}{k+2}$

转移方程至此写作

$$f_{j+1,i}=\left(\binom{i+1}2+1\right)f_{j,i}+\sum_{k<i}\binom{i+2}{k}f_{j,k}$$

答案是 $\sum\limits_{i=0}^n\binom ni f_{m,i}$

直接使用 $\rm NTT$ 优化转移，时间复杂度 $\Theta(nm\log n)$

故事到这里结束我就不更博了，所以设 $f_{j,i}$ 的 $\rm EGF$ 为 $F_j(x)=\sum_{i\ge 0}\frac{f_{j,i}}{i!}x^i$ 丢进去继续化简（以下的 $f_{j,i}$ 均表示原来的 $\frac{f_{j,i}}{i!}$，与原来定义不同）：

$$\begin{aligned}&i!f_{j+1,i}=\left(\frac{i(i+1)}2+1\right)i!f_{j,i}+\sum_{k<i}\frac{(i+2)!}{(i+2-k)!k!}k!f_{j,k}\\
&f_{j+1,i}=\left(\frac{i(i+1)}2+1\right)f_{j,i}+\sum_{k\le i+2}\frac{(i+1)(i+2)}{(i+2-k)!}f_{j,k}-\frac{(i+1)(i+2)}2f_{j,i}-(i+1)(i+2)\left(f_{j,i+2}+f_{j,i+1}\right)\\
&f_{j+1,i}=\sum_{k\le i+2}\frac{(i+1)(i+2)}{(i+2-k)!}f_{j,k}-if_{j,i}-(i+1)(i+2)\left(f_{j,i+2}+f_{j,i+1}\right)
\end{aligned}$$

**求和时枚举不到 $i+2$ 就要减掉再变成 $e^x$**，$\rm GF$ 运算细节最关键，需要时刻注意缺项的问题

逐项考察，并使用导数进行系数配凑，第一项是：

$$[x^i]F_{j+1}(x)=(i+2)(i+1)[x^{i+2}]eF_{j}(x)\rightarrow[x^i]F_{j+1}(x)=[x^i]\left(eF_{j}(x)\right)'' $$

后面依次是 $-xF'_{j}(x)-F''_{j}(x)-\left[xF''_{j}(x)+2F'_{j}(x)\right]$

前面的使用复合函数求导法则直接得到，整理得到下式，可能是微分方程：

$$F_{j+1}(x)=e^{x}F_{j}(x)+(2e^x-x-2)F'_{j}(x)+(e^x-x-1)F''_j(x)$$

解决这类问题的一个手段是 $dp_{J,i,j}$ 表示 $\sum_{i=0}^{m}\sum_{j=0}^m f_{J}x^je^{ix}$

根据方程进行转移，注意这里对 $(e^{jx})'=je^{jx}$，那么需要据此处理一阶/二阶导数的真正系数，和百鸽笼一题中没有导数的转移式子还不一样

具体而言，由于导数是对整体进行的，那么以二阶导为例： $(x^je^{ix})''=i^2e^{ix}x^j+2ije^{ix}x^{j-1}+j(j-1)e^{ix}x^{j-2}$，所以系数需要附加

最后统计答案时有个简单做法是不再逐 $[x^i]$ 求贡献再累加，而是再乘 $e^{x}$ 将所有项都直接贡献到 $[x^n]\left(eF_m(x)\right)$

时间复杂度 $\Theta(m^3+n)$，完全可以出到 $m=300,n=10^7,mod=10^9+7$

实现的时候由于系数比较多容易漏，需要认真仔细一些

这是让我学会函数系数在 $\rm GF$ 下求导等计算的一题，简单记录纪念一下

# Code

代码里面变量名称和叙述中有所不同，但是这不关键

```cpp
const int maxN=310;
int f[2][maxN][maxN],N,M,cur,fac[8010],ifac[8010];
struct node{int de,dx,dv;}; vector<node>vec[2];
inline void push(int id,int be,int bx,int bv){
    for(auto t:vec[id]){
        ckadd(f[cur^1][be+t.de][bx+t.dx],mul(bv,t.dv));
    }
    return ;
}
signed main(){
    fac[0]=1;
    for(int i=1;i<=8000;++i) fac[i]=mul(fac[i-1],i);
    ifac[8000]=ksm(fac[8000],mod-2);
    Down(i,8000,1) ifac[i-1]=mul(ifac[i],i);
    N=read(); M=read(); 
    f[0][0][0]=1;
    vec[0].pb((node){1,0,2}); 
    vec[0].pb((node){0,1,mod-1}); 
    vec[0].pb((node){0,0,mod-2});
    
    vec[1].pb((node){1,0,1}); 
    vec[1].pb((node){0,1,mod-1}); 
    vec[1].pb((node){0,0,mod-1});
    //f[cur][i][j] -> e^{xi}x^j
    for(int m=1;m<=M;++m){
        for(int i=0;i<=m;++i){
            for(int j=0;j<=m;++j) if(f[cur][i][j]){
                if(j>=1){
                    push(0,i,j-1,mul(f[cur][i][j],j));
                    push(1,i,j-1,mul(f[cur][i][j],2*i*j));
                }
                if(j>=2){
                    push(1,i,j-2,mul(f[cur][i][j],j*(j-1)));
                }
                ckadd(f[cur^1][i+1][j],f[cur][i][j]);
                push(0,i,j,mul(f[cur][i][j],i));
                push(1,i,j,mul(f[cur][i][j],i*i));
            }
        }
        rep(i,0,m) rep(j,0,m) f[cur][i][j]=0;
        cur^=1;
    }
    int ans=0;
    for(int i=0;i<=M;++i){
        for(int j=0;j<=N&&j<=M;++j) if(f[cur][i][j]){
            ckadd(ans,mul(f[cur][i][j],mul(ksm(i+1,N-j),ifac[N-j])));
        }
    }
    print(mul(ans,fac[N]));
    return 0;
}
```

玻璃碎片也能成为水晶雕塑，我摸索了这么久也最终会了 $\rm GF$ 的计算方法，再次感谢 zero4338 的指导