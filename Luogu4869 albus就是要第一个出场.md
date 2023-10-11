# Description

[link](https://www.luogu.com.cn/problem/P4869)

求一个数在一个序列的线性基表示的所有数中的排名

$n\le 10^5$

# Solution

线性基的另外一种应用

和查询一个排名为 $k$ 的数也不那么类似……

我们将所有的数都插入到线性基里面

有主元的位置记录来，如果当前的那个和有主元的位置与起来不为 $0$ ，那就排名加上主元的排名

其实就是 $1<<x$

因为那个位置有异或起来答案是 $0$ 的结果

然后我们发现集合里面有 $n$ 个数，线性基里面有 $siz$ 个数，中间必然异或起来有重复的答案

剩下的是一个结论，就是序列里面能异或出来一个数，重复的方案为 $2^{n-siz}$

$siz$ 为线性基的大小

[关于这东西怎么理解……](https://www.luogu.com.cn/blog/szxkk/solution-p4869)

在线性基里面的数字一共有 $siz$ 个，然后异或的答案一共是有 $2^{siz}$ 个

序列里面能异或出来的答案也就只有 $2^{siz}$ 个（这里的正确性待考究）

那么每异或出来一个答案，然后就可以找到其他的方案使得其异或值为 $0$ （就是其它的数的异或值）

然后方案就是$2^{n-siz}$ 

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
	const int N=100010,mod=10086;
	int a[N],n,q,p[100],st[50],siz,ans;
	inline int ksm(int x,int y)
	{
		int res=1;
		for(;y;y>>=1,x=x*x%mod) if(y&1) res=res*x%mod;
		return res;
	}
	inline void insert(int x)
	{
		for(int i=30;i>=0;--i) 
		{
			if(!(x>>i)) continue;
			if(!(~p[i])) { p[i]=x; break;}
			else x^=p[i];
		}return ;
	}
	signed main()
	{
		memset(p,-1,sizeof(p));
		n=read(); for(int i=1;i<=n;++i) a[i]=read(),insert(a[i]); q=read();
		for(int i=0;i<=30;++i) if(~p[i]) st[siz++]=i; 
		for(int i=0;i<siz;++i) if((q>>st[i])&1) ans|=(1ll<<i);
		printf("%lld\n",(ans*ksm(2,n-siz)%mod+1)%mod);
		return 0;
	}
}
signed main(){return yspm::main();}
```