# Description

[link](http://www.lydsy.com/JudgeOnline/problem.php?id=4026)

区间乘积$\varphi$

# Solution

这个题值得记录的地方是巧妙的使用了欧拉函数的性质

$$\varphi(\prod^r_{i=l} a_i)\ = \ \prod^r_{i=l}a_i \prod\frac{p_i-1}{p_i}$$

然后是个线性筛素数 $+$ 线性求逆元

最后按照 $HH$ 的项链那样记录该质数上一个出现的位置就行了

记得乘上区间积

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
	const int N=1e5+10,M=1e6+1000,mod=1e6+777;
	int pri[M],fl[M],cnt,inv[M],las[N],id[M];
	inline void prework()
	{
		for(int i=2;i<M;++i)
		{
			if(!fl[i]) 
			{
				pri[++cnt]=i;
				for(int j=i*i;j<M;j+=i) fl[j]=1; 
			}
		}
		for(int i=1;i<=cnt;++i) id[pri[i]]=i;
		inv[0]=1; inv[1]=1; 
		for(int i=2;i<M;++i) inv[i]=mod-mod/i*inv[mod%i]%mod;
		return ;
	}
	int n,m,a[N],rt[N],tot,mul[N],lans;
	struct node{
		int ls,rs,sum;
		#define ls(p) t[p].ls
		#define rs(p) t[p].rs
		#define sum(p) t[p].sum
	}t[N*80];
	inline void update(int &p,int pre,int l,int r,int x,int val)
	{
		p=++tot; t[p]=t[pre]; (sum(p)*=val)%=mod;
		if(l==r) return ;
		int mid=(l+r)>>1;
		if(x<=mid) update(ls(p),ls(pre),l,mid,x,val);
		else update(rs(p),rs(pre),mid+1,r,x,val);
		return ;
	}
	inline int query(int p,int l,int r,int x)
	{
		if(l>=x) return sum(p); if(r<x) return 1;
		int mid=(l+r)>>1;
		return query(ls(p),l,mid,x)*query(rs(p),mid+1,r,x)%mod; 
	}
	signed main()
	{
		prework(); n=read(); m=read(); mul[0]=1; sum(0)=1;
		for(int i=1;i<=n;++i) a[i]=read(),mul[i]=mul[i-1]*a[i]%mod;
		for(int i=1;i<=n;++i)
		{
			rt[i]=rt[i-1];
			for(int j=1;pri[j]*pri[j]<=a[i];++j)
			{
				if(a[i]%pri[j]==0)
				{
					while(a[i]%pri[j]==0) a[i]/=pri[j];
					if(las[j]) 
					{
						update(rt[i],rt[i],1,n,las[j],pri[j]*inv[pri[j]-1]%mod);
						update(rt[i],rt[i],1,n,i,(pri[j]-1)*inv[pri[j]]%mod);
					}
					else 
					{
						update(rt[i],rt[i],1,n,i,(pri[j]-1)*inv[pri[j]]%mod);
					}
					las[j]=i;
				}
			}
			if(a[i]!=1)
			{
				if(las[id[a[i]]]) update(rt[i],rt[i],1,n,las[id[a[i]]],a[i]*inv[a[i]-1]%mod);
				update(rt[i],rt[i],1,n,i,(a[i]-1)*inv[a[i]]%mod);
				las[id[a[i]]]=i;
			}
		}
		while(m--)
		{
			int l=read()^lans,r=read()^lans;
			lans=query(rt[r],1,n,l)*mul[r]%mod*inv[mul[l-1]]%mod;
			printf("%lld\n",lans);
		}
		return 0;
	}
}
signed main(){return yspm::main();} 
```