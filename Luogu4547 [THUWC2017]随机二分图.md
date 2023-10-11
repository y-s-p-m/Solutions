# Description

[link](https://www.luogu.com.cn/problem/P4547)

# Solution

题目本质上要求的是 $n!$ 种完备匹配乘上各自出现的概率乘上 $2^n$ 之和

状态定义为 $f_{S,T}$ 为左部点的集合为 $S$，右部点的集合为 $T$ 的权值和

转移考虑枚举一条边

$$f_{S,T}=f_{S\ xor\  st,T \ xor\ ed}\times v_{st,ed}$$

固定枚举边的顺序可以进行转移，考虑到 $bitcount(S)=bitcount(T)$ 和数据范围，考虑记忆化搜索更为可行

同时还要注意的是要从按照二进制位从小到大进行转移，比较

最后来的瓶颈就是 $v_{st,ed}$ 了

第一种边是比较好做的，直接是 $\frac 1 2$

第二种边第三种边就不太好做

发现这里的式子只支持第一种边这种类别的，那么就考虑用期望线性性来拆边

先把两个边都用 $v=\frac 1 2$ 插入

然后考虑差异

第二类边需要再插入两边并集 $v=\frac 1 4$，第三类边需要再插入两边并集且 $v=-\frac 14 $

（后面的就是考虑共同出现的情况）

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define reg register
#define For(i,a,b) for(reg int i=a;i<=b;++i) 
#define Down(i,a,b) for(reg int i=b;i>=a;--i) 
namespace yspm{
    inline int read(){
        int res=0,f=1; char k;
        while(!isdigit(k=getchar())) if(k=='-') f=-1;
        while(isdigit(k)) res=res*10+k-'0',k=getchar();
        return res*f;
    }
    inline int p(int x){return (1<<(x-1));}
    const int N=1e5+10,mod=1e9+7;
    struct edge{
        int st,ed;
        long long val;
    }e[N];
    #define ll long long
    map<int,ll> f[N];
    int inv2=5e8+4,inv4=25e7+2,cnt,n,m;
    inline ll calc(int S,int T){
        if(!S) return 1;
        if(f[S].count(T)) return f[S][T];
        For(i,1,cnt){
            ll s=e[i].st,t=e[i].ed,v=e[i].val;
            if((S|s)!=S||(T|t)!=T||S>=(s<<1)) continue;
            f[S][T]=(f[S][T]+calc(S^s,T^t)*v%mod)%mod;
        }
        return f[S][T];
    }
    signed main(){
        n=read(); m=read(); 
        For(i,1,m){
            int opt=read();
            if(opt){
                int x1=read(),x2=read(),x3=read(),x4=read();
                e[++cnt]=(edge){p(x1),p(x2),inv2};
                e[++cnt]=(edge){p(x3),p(x4),inv2};
                if(x1==x3||x2==x4) continue;
                if(opt==1) e[++cnt]=(edge){p(x1)|p(x3),p(x2)|p(x4),inv4};
                else e[++cnt]=(edge){p(x1)|p(x3),p(x2)|p(x4),mod-inv4};
            }
            else{   
                e[++cnt].st=p(read());
                e[cnt].ed=p(read());
                e[cnt].val=inv2;
            }
        }
        printf("%lld\n",(1<<n)*calc((1<<n)-1,(1<<n)-1)%mod);
        return 0;
    }
}signed main(){return yspm::main();}
```