# Description

[link](https://www.luogu.com.cn/problem/P5369)

给定一个排列$\{a_{1\cdots n}\}$

求其所有排列的最大前缀和

$n\le 20$

# Solution

最大前缀和的本质是所有的补集前缀和都小于 $0$，然后这个排列的所有**真**后缀和都是大于 $0$ 的

注意是真后缀，所以正好处理掉了全是负数之类的情况

所以这题目可以考虑每个子集作为最大前缀和的方案数再乘上它本身的和

（这两步显然）

枚举这个序列的所有子集作为最大前缀和

然后是考虑该集合的补集有多少种排列方式使得所有前缀和都小于 $0$

再考虑一个集合有多少种排列的方式使得该集合的所有真后缀和都不小于 $0$

可以做到 $O(\log n)$ 求当前状态下的集合求和

所有前缀小于 $0$ 好做

然后考虑剩下的部分：

考虑到我们说的是真后缀，不是后缀

所以定义：$f_{S,0}$ 为集合元素的和小于 $0$ 然后满足真后缀都不小于 $0$

$f_{S,1}$ 表示集合元素都大于等于 $0$ 然后真后缀都不小于 $0$ 的方案数

转移？

$f_{S,0}$ 和 $f_{S,1}$ 分别加上那些满足条件的 $f_{S',1}$

最后还是一样统计答案……

剩下一个取模比较坑了……

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
namespace yspm{
	inline int read()
	{
		int res=0,f=1; char k;
		while(!isdigit(k=getchar())) if(k=='-') f=-1;
		while(isdigit(k)) res=res*10+k-'0',k=getchar();
		return res*f;
	}
	const int N=25,SZ=1<<20,mod=998244353;
	inline int add(int x,int y){return x+y>=mod?x+y-mod:x+y;}
	inline int del(int x,int y){return x-y<0?x-y+mod:x-y;}
	int a[N],n,s[SZ],f1[SZ][2],f2[SZ];
	inline int lowbit(int x){return x&(-x);}
	inline int getsum(int x)
	{
		int sum=0;
		for(int i=1;i<=n;++i)
		{
			if(x&(1<<(i-1))) sum+=a[i],x^=(1<<(i-1));
			if(!x) break;
		}return sum;
	}
	inline int calc(int x){int cnt=0; for(;x;x-=lowbit(x)) ++cnt; return cnt;}
	signed main()
	{
		n=read(); for(int i=1;i<=n;++i) a[i]=read();
		int S=(1<<n)-1;  
		for(int i=0;i<=S;++i) s[i]=getsum(i);  
		for(int i=1;i<=n;++i) if(a[i]>=0) f1[1<<(i-1)][1]=1; else f1[1<<(i-1)][0]=1;
		for(int st=1;st<=S;++st)
		{
			for(int i=1;i<=n;++i)
			{
				if(st&(1<<(i-1))) continue;
				if(s[st]+a[i]>=0) f1[st|(1<<(i-1))][1]=add(f1[st][1],f1[st|(1<<(i-1))][1]); 
				else f1[st|(1<<(i-1))][0]=add(f1[st][1],f1[st|(1<<(i-1))][0]);
			}
		} 
		f2[0]=1;
		for(int st=0;st<=S;++st)
		{
			if(!f2[st]) continue; 
			for(int i=1;i<=n;++i) 
			{
				if(st&(1<<(i-1))) continue;
				if(s[st|(1<<(i-1))]<0) f2[st|(1<<(i-1))]=add(f2[st],f2[st|(1<<(i-1))]); 
			}
		} 
		int ans=0;
		for(int i=0;i<=S;++i) ans=add(ans,f2[S^i]*add(s[i],mod)%mod*(f1[i][0]+f1[i][1])%mod),ans+=mod,ans%=mod;
		printf("%lld\n",ans);
		return 0;
	}
}
signed main(){return yspm::main();}
```