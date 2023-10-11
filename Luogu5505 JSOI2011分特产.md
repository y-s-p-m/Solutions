# Description

[link](https://www.luogu.com.cn/problem/P5505)

有 $n$ 个同学和 $m$ 种特产，要求分特产的时候不能有人没有特产

求方案数

$n,m\le10^3$

# Solution

一道容斥的上手的题目吧

设我们不需要考虑没有特产的情况，直接上插板法统计答案

$$f(i)=\prod^{m} _ {i=1}\binom{a_i+n-1}{n-1}$$

然后我们考虑这个要减去有人没有特产的情况

首先删去有一个人没有特产的情况，就是分给 $n-1$ 个人呗

这里我们发现如果直接 $f_n-f_{n-1}$ 显然是个假的做法

因为由定义，这个 $f_{n-1}$ 是有可能有$n-2$个人分到，$1$个人没有分到的

所以我们还得接着容斥

另：由于我们不知是哪 $i$ 个人没有被分到，所以还是得乘上一个 $\binom {n}{n-i}$

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
	const int N=3e3+10,mod=1e9+7;
	int inv[N],fac[N];
	inline void prework()
	{
		fac[0]=1; inv[0]=1; inv[1]=1;
		for(int i=1;i<N;++i) fac[i]=fac[i-1]*i%mod;
		for(int i=2;i<N;++i) inv[i]=(mod-mod/i)*inv[mod%i]%mod;
		for(int i=1;i<N;++i) inv[i]*=inv[i-1],inv[i]%=mod;
		return ; 
	}
	inline int C(int n,int m)
	{
		if(n<0||m<0||n<m) return 0; 
		return inv[n-m]*inv[m]%mod*fac[n]%mod;
	}
	int a[N],n,m,ans,f[N];
	signed main()
	{ 
		prework();
		n=read(); m=read(); 
		for(int i=1;i<=m;++i) a[i]=read();
		for(int i=1;i<=n;++i)
		{
			f[i]=1;
			for(int j=1;j<=m;++j) f[i]*=C(a[j]+i-1,i-1),f[i]%=mod;
		}
		for(int i=0;i<=n;++i)
		{
			if(i&1)
			{
				ans-=C(n,i)*f[n-i];
				ans+=mod; ans%=mod;
			}
			else 
			{
				ans+=C(n,i)*f[n-i]%mod;
				ans%=mod;
			}
		}cout<<ans<<endl;
		return 0;
	}
}
signed main(){return yspm::main();}
```

# Review

我们在解决容斥的题目的时候需要综合运用各种组合方法

同时精准找到重复信息然后进行枚举容斥