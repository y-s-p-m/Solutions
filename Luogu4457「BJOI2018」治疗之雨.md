# Description

有 $m+1$个数：第一个为 $p$，最小值为 $0$，最大值为 $n$；剩下 $m$ 个都是无穷，没有最小值或最大值。你可以进行任意多轮操作，每轮操作如下：

在不为最大值的数中等概率随机选择一个（如果没有则不操作），把它加一；

进行 $k$ 次这个步骤：在不为最小值的数中等概率随机选择一个（如果没有则不操作），把它减一。

现在问期望进行多少轮操作以后第一个数会变为最小值 $0$。

$$n\le 1500,m,k\le 10^9$$

# Solution

按照分手是祝愿的做法：定义 $f_i$ 为剩下 $i$ 滴血的期望轮数

转移考虑每次的增减方式，因为只需要考虑第一个数的值，所以

$$f_{i}=\frac{(f_{i-1}+1)\times m^k} {(m+1)^{k+1}}+\sum_{p=0}^k (f_{i+p}+1)\times(\frac{m}{m+1})^{k-p+1}$$

这东西可以用高斯消元来做，把式子一推就是个没有分数的式子了

然后发现上面的式子是有问题的：没有考虑可以是任意 $k$ 个地方掉血，所以正经的式子可以写成：

$$f_{i}=\frac{(f_{i-1}+1)\times m^k} {(m+1)^{k+1}}+\sum_{p=0}^k (f_{i+p}+1)\times\frac{m^{k-p+1} \times \binom k i}{(m+1)^{k-p+1}}$$

这题目只想到了这里，然后发现这个式子高斯消元完和初始量就没关系了，初始化的细节还是不太会想

发现这是个概率期望题目，所以状态定义是有问题的：应该考虑逆着做，就是定义 $f_i$ 为有 $i$ 滴血被杀死的期望轮数,$g_i$ 表示一轮里面打出伤害大于等于 $i$ 的概率,式子大概差不多：

$$f_{i}=\frac{m}{m+1}(g_{i}+\sum_{j=0}^{i-1} (f_{i-j}+1) \frac{m^{k-j+1} \times \binom k i}{(m+1)^{k-j+1}})+\frac{1}{m+1}(g_{i+1}+\sum_{j=0}^{i} (f_{i+1-j}+1) \frac{m^{k-j+1} \times \binom k i}{(m+1)^{k-j+1}})$$

注意 $f_n$ 就不考虑后面一半了

然后发现这个式子有个有趣的地方：$f_{i-j}+1$ 

那么后面的概率部分可以拿出来，就能跟前面的 $g_i$ 合并了

然后长成这个样子：

$$f_{i}=1+\frac{m}{m+1}\sum_{j=0}^{i-1} f_{i-j} P_{j}+\frac{1}{m+1} \sum_{j=0}^{i} f_{i+1-j}P_j$$

其中 $P_j$ 表示一轮内打掉 $k$ 滴血的概率

$$P_i=\frac{\binom k i m^{k-i}}{(m+1)^{k}}$$

然后直接答案就是 $f_p$ 就做完了

时间复杂度 $O(n^3)$ 期望得分：$60\to 80\ pts$

---

下面是卡过的方法：

观察矩阵的形式：每行都是一些连续有值的和一些连续没有值的

然后我们发现这东西比较好的在于我们可以每次直接把 $a_{i,i},a_{i,i+1}$ 消成 $1$

顺着仅更新下面行的 $a_{j,i+1},a_{j,n}$ 就好了

$upd:orz\ skyh$

因为这个矩阵有值的部分是一个直角梯形，那么可以考虑让每个 $x_i=a\times x_1 +b$ 

然后全怼到第 $n$ 行里面消元，再反解其他的就完成了 

这样得到的矩阵是一个类似于这样的

$$a_{1,1},a_{1,2},0,0\cdots$$

$$0,a_{2,2},a_{2,3},0\cdots$$

$$0,0,a_{3,3},a_{3,4}\cdots$$

最后一行没有多出来的一个有数的值，所以可以直接计算出答案，最后返回去带入就做完了

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define reg register
#define For(i,a,b) for(reg int i=a;i<=b;++i) 
#define Down(i,a,b) for(reg int i=a;i>=b;--i) 
namespace yspm{
    inline int read()
    {
        int res=0,f=1; char k;
        while(!isdigit(k=getchar())) if(k=='-') f=-1;
        while(isdigit(k)) res=res*10+k-'0',k=getchar();
        return res*f;
    }
    const int mod=1e9+7;
    inline int add(int x,int y){return x+y>=mod?x+y-mod:x+y;}
    inline int del(int x,int y){return x-y<0?x-y+mod:x-y;}
    inline int mul(int x,int y){return 1ll*x*y-1ll*x*y/mod*mod;}
    inline int min(int x,int y){return x<y?x:y;}
    inline int max(int x,int y){return x>y?x:y;}
    inline int ksm(int x,int y)
    {
        int res=1; 
        for(;y;y>>=1,x=mul(x,x)) if(y&1) res=mul(res,x);
        return res;
    }
    const int N=1510;
    int inv[N],ans[N],n,p,m,k,P[N],a[N][N];
    inline int calc(int mx,int now,int d)
    {
        int ans=0; 
        while(now>0) 
        {
            if(now<mx) now++;
            now-=d; ++ans;
        }return ans;
    }
    int cnt;
    inline void work()
    {
        n=read(); p=read(); m=read(); k=read();
        if(!k) return puts("-1"),void(); 
        if(k==1&&!m) return puts("-1"),void();
        if(!m) return printf("%d\n",calc(n,p,k)),void();
        memset(a,0,sizeof(a));
        int now=1,t1=ksm(m+1,mod-2);
        For(i,0,n) P[i]=0,ans[i]=0;
        
        For(i,0,min(n,k)) P[i]=mul(now,mul(ksm(t1,k),ksm(m,k-i))),now=mul(now,mul(inv[i+1],k-i));
        For(i,1,n-1) 
        {
            a[i][i]=1; a[i][n+1]=1;
            For(j,0,i-1) a[i][i-j]=del(a[i][i-j],mul(P[j],mul(m,t1))); 
            For(j,0,i) a[i][i-j+1]=del(a[i][i-j+1],mul(P[j],t1));
        }
        a[n][n]=1; a[n][n+1]=1;
        For(i,0,n-1) a[n][n-i]=del(a[n][n-i],P[i]);
        For(i,1,n) 
        {
            int tmp=ksm(a[i][i],mod-2); a[i][i]=1; a[i][n+1]=mul(a[i][n+1],tmp); 
            if(i!=n) a[i][i+1]=mul(a[i][i+1],tmp);
            For(j,i+1,n) 
            {
                tmp=a[j][i],a[j][i]=0; 
                a[j][i+1]=del(a[j][i+1],mul(tmp,a[i][i+1]));
                a[j][n+1]=del(a[j][n+1],mul(tmp,a[i][n+1]));
            }
        }ans[n]=a[n][n+1];
        Down(i,n-1,1) ans[i]=del(a[i][n+1],mul(a[i][i+1],ans[i+1]));
        return printf("%d\n",ans[p]),void();
    }
    signed main()
    {
        inv[0]=inv[1]=1; 
        For(i,2,N-1) inv[i]=del(mod,mul(mod/i,inv[mod%i])); 
        int T=read(); while(T--) work();
        return 0;
    }
}
signed main(){return yspm::main();}
```