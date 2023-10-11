# Description

给定 $n,m,k,r,c,v$，求满足下述条件的大小为 $[n\times m]$，矩阵里面元素值域为 $[1,k]\cap \mathcal Z$ 的矩阵个数模 $998244353$ 的值

- 每行元素单调不降

- 每列元素单调不降

- $a_{r,c}=v$

$n,m\le 200,k\le 100$

# Solution

先尝试解决没有 $(r,c)$ 限制的数量统计问题，将 $\le i,>i$ 的元素在矩阵上的分布画一条从 **左下角的点** 到 **右上角的点** 的线

注意线画在点上而非格子上

这样子的线是可能相交的，但是不可能产生 **交错**，那么如果将第 $i$ 条线（即 $\le i,>i$ 的分布位置在矩阵上的分割线）向右 & 下平移 $i$ 个格子，那么现在的限制变成了路径不相交

有向图上的路径不相交问题可以使用 $\rm LGV$ 引理来解决，起点终点都是 $k$ 个，坐标也就是对应平移后的 **点坐标**

回到原问题则需要考虑 $a_{r,c}=v$ 的限制，根据 $r,c,v$ 三个变量确定网格图上的一个点 $p=(r+v-2,c+v-2)$（矩阵对应的网格图坐标范围是 $(0,0)\sim(n,m)$），强制有且仅有 $v-1$ 条边从 $p$ 的上面走过

使用变元的方法可以满足此类需求，即让每条经过 $p$ 点右上方的路径的权值乘积变成 $x$，这样将所有路径的边权乘起来那么合法的方案就是进行 $\rm LGV$ 引理得到的行列式的 $x^{v-1}$ 前面的系数

由于路径条数有限，那么乘积指数上界为 $k-1$，大可使用 $\rm Lagrange$ 插值公式还原多项式的方法来得到系数，那么现在的目的就是计算边权乘积之和，分别处理经过/不经过 $x$ 边对应的方案数

直接计算路径条数的方法就是找 $(r-d,c-d)[d\le \min(r,c)]$ 来作为中转点加和作为 $x$ 前面的系数

注意需要强制所有路径均不经过 $p$ 点本身，这是因为进行了路径平移，而这个格子里面已经有 $V$，所以前 $V-1$ 条路径都不应该经过这个格子的左上角

属于是 $\rm LGV$ 引理的关键应用题，非常有意思

# Code

```cpp
int fac[1010],ifac[1010],n,m,K;
const int N=310;
int a[310][310];
inline int C(int n,int m){return mul(fac[n],mul(ifac[m],ifac[n-m]));}
inline int Guass(){
    int ans=1;
    for(int i=1;i<=K;++i){
        if(!a[i][i]){
            for(int j=i+1;j<=K;++j) if(a[j][i]){
                swap(a[i],a[j]); 
                ans=del(0,ans);
                break;
            }
        }
        ckmul(ans,a[i][i]);
        int inv=ksm(a[i][i],mod-2);
        for(int j=i+1;j<=K;++j){
            int tmp=mul(a[j][i],inv);
            for(int k=i;k<=K;++k) ckdel(a[j][k],mul(a[i][k],tmp));
        }
    } return ans;
}
int sx[N],sy[N],ex[N],ey[N];

inline int calc(int sx,int sy,int ex,int ey){
    if(ex>sx||ey<sy) return 0;
    return C(sx-ex+ey-sy,ey-sy);
}
int coef1[N][N],coefx[N][N];
int r,c,v,x[N],y[N];
signed main(){
    n=1000; fac[0]=1;
    rep(i,1,n) fac[i]=mul(fac[i-1],i); 
    ifac[n]=ksm(fac[n],mod-2);
    Down(i,n,1) ifac[i-1]=mul(ifac[i],i);
    
    n=read(); m=read(); K=read();
    r=read(); c=read(); v=read();
    r+=v-2; c+=v-2; --K;
    for(int i=0;i<K;++i){
        for(int j=0;j<K;++j){
            //coordinate (n+i,i)->(j,m+j)
            coef1[i][j]=del(calc(n+i,i,j,m+j),mul(calc(n+i,i,r,c),calc(r,c,j,m+j)));
            for(int d=1;d<=r&&d<=c;++d){
                int x=r-d,y=c-d;
                int res=mul(calc(n+i,i,x,y),calc(x,y,j,m+j));
                ckadd(coefx[i][j],res);
                ckdel(coef1[i][j],res);
            }
        }
    }
    for(int i=0;i<=K;++i){
        memset(a,0,sizeof(a));
        for(int j=0;j<K;++j) for(int k=0;k<K;++k){
            a[j+1][k+1]=add(coef1[j][k],mul(coefx[j][k],i));
        }
        y[i]=Guass();
    }
    vector<int> dp(K+1);
    //x[i] = {0..K},y[i]
    for(int i=0;i<=K;++i){
        vector<int> tmp(K+1);
        tmp[0]=1;
        int coef=y[i];
        for(int j=0;j<=K;++j) if(i!=j){
            ckmul(coef,ksm(del(i,j),mod-2));
            vector<int> nxt(1+K);
            for(int t=0;t<=K;++t) ckdel(nxt[t],mul(j,tmp[t]));
            for(int t=0;t<K;++t) ckadd(nxt[t+1],tmp[t]);
            tmp=nxt;
        }
        for(int t=0;t<=K;++t) ckadd(dp[t],mul(tmp[t],coef));
    }
    print(dp[v-1]);
    return 0;
}
```