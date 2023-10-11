# Description

[link](https://www.luogu.com.cn/problem/P3321)

给定一个集合 $s$，和整数 $x,n,m$ （$n\le 10^9 \ \ m\le 8000 \ \ x \le m-1$）

保证 $\forall a\in \{S\},a\le m$

每次从集合里面随机选数，一共选 $n$ 次，求选出来的数的乘积 $\equiv x \ (mod\ m)$ 的方案数
 
# Solution

上手一个比较暴力的 $dp$ 

设 $f_{i,j}$ 是长度为 $i$ 的序列，乘积模数为 $j$ 的方案数

转移枚举每个可以被随机出来的数转移就好了

复杂度显然爆炸

$N \le 10^9 \ \ ?\ \ $ 矩阵乘法？

然而并不会……

----

有一个比较好的优化：

$$f_{2\times i,p}=\sum_{x\times y\equiv p(mod \ m)} f_{i,x}\times f_{i,y}$$

乘法取模并不能计算，怎么把乘法变加法？

套 $\log$ 用原根，即：

$$f_{2\times i,\log_{g} p}=\sum_{\log_{g}x \%m+\log _ {g} y\%m=\log_{g} p\% m } f_{i,\log_{g}x}\times f_{i,\log_{g}y}$$

直接 $NTT$ 卷上 $\log (n)$ 次即可

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
//#define int long long
namespace yspm{
	inline int read()
	{
		int res=0,f=1; char k;
		while(!isdigit(k=getchar())) if(k=='-') f=-1;
		while(isdigit(k)) res=res*10+k-'0',k=getchar();
		return res*f;
	}
	const int mod=1004535809,N=50010,L=8010;
	int g,d[100],cnt,n,m,x,sz,s[L],id[L];
	inline int ksm1(int x,int y,int mod)
	{
		int res=1; for(;y;y>>=1,x=1ll*x*x%mod) if(y&1) res=1ll*res*x%mod;
		return res;
	}
	inline int ksm(int x,int y)
	{
		int res=1; for(;y;y>>=1,x=1ll*x*x%mod) if(y&1) res=1ll*res*x%mod;
		return res;
	}
	inline bool check(int x,int p)
	{
		for(int i=1;i<=cnt;++i)
		{
			if(ksm1(x,(p-1)/d[i],p)==1) return 0;
		} return 1;
	}
	inline void find(int x)
	{	
		int t=x-1; cnt=0;
		for(int i=2;i*i<=t;++i) 
		{
			if(t%i==0) 
			{
				d[++cnt]=i; 
				while(t%i==0) t/=i;
			}
		} if(t>1) d[++cnt]=t;
		g=2; while(1) if(check(g,x)) break; else ++g;
		return ;
	}
	int A[N],r[N],B[N];
	inline int add(int x,int y){return x+y>=mod?x+y-mod:x+y;}
	inline int del(int x,int y){return x-y>=0?x-y:x-y+mod;}
	inline void NTT(int *f,int n,int opt)
	{
		for(int i=0;i<n;++i) if(i<r[i]) swap(f[i],f[r[i]]);
		for(int p=2;p<=n;p<<=1)
		{
			int len=p>>1,tmp=ksm(g,(mod-1)/p); 
			if(opt==-1) tmp=ksm(tmp,mod-2);
			for(int k=0;k<n;k+=p)
			{
				int buf=1;
				for(int l=k;l<k+len;++l)
				{
					int tt=1ll*buf*f[l+len]%mod;
					f[l+len]=del(f[l],tt);
					f[l]=add(f[l],tt);
					buf=1ll*buf*tmp%mod;
				}
			}
		} 
		if(opt==-1) for(int i=0;i<n;++i) f[i]=1ll*f[i]*ksm(n,mod-2)%mod;
		return ;
	}
	int len,sum,t[N];
	inline void mul(int *a,int *b,int *c)
	{	
		static int A[N],B[N],t[N];
		for(int i=0;i<len;++i) A[i]=a[i],B[i]=b[i];
		NTT(A,len,1); NTT(B,len,1);   
		for(int i=0;i<len;++i) A[i]=1ll*A[i]*B[i]%mod;  	  
		NTT(A,len,-1);
		for(int i=0;i<m-1;++i) t[i]=add(A[i],A[i+m-1]);
		for(int i=0;i<m-1;++i) c[i]=t[i];
		return ;
	}
	signed main()
	{
		n=read(); m=read(); x=read(); sz=read();  find(m); 
		for(int i=0;i<m-1;++i) id[ksm1(g,i,m)]=i; g=3;
		for(int i=1;i<=sz;++i) s[i]=read(),B[id[s[i]%m]]+=s[i]%m?1:0;
		len=1; sum=m<<1; while(len<=sum) len<<=1; 
		for(int i=0;i<len;++i) r[i]=r[i>>1]>>1|((i&1)?len>>1:0);
		
		A[id[1]]=1; 
		for(;n;n>>=1,mul(B,B,B))
		{
			if(n&1) mul(A,B,A);
		} 
		printf("%d\n",A[id[x%m]]);
		return 0;
	}
}
signed main(){return yspm::main();}
```
