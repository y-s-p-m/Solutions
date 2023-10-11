# Description

[link](https://www.luogu.com.cn/problem/P2463)

# Solution

看到这种题，首先是两个数作差

然后我们把所有的差连起来，每个序列和每个序列中间加上一个极大值

然后求出来 $sa$ 和 $height$ 数组，并且把所

要求的就是在每 $n$ 个串的 $lcp$

（这里好像就可以直接$height$数组加上带优化的暴力）

这里考虑一下二分长度

如果有 $n$ 个 $height_i>=len$ 那么就当前的 $len$ 合法

$EOF$

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
	const int N=1e6+10;
	int rk[N],sa[N],n,m,x[N],y[N],b[N],id[N],cnt,minn=1e18+10,maxx,l,r=1e18+10;
	int s[1010][101],h[N],len[1010],num;
	inline void qsort()
	{
		memset(y,0,sizeof(y));
		for(int i=1;i<=n;++i) ++y[rk[i]];
		for(int i=1;i<=m;++i) y[i]+=y[i-1];
		for(int i=n;i>=1;--i) sa[y[rk[x[i]]]--]=x[i];
		return ;
	}
	inline void get_sa()
	{
		for(int i=1;i<=n;++i) rk[i]=b[i],x[i]=i; qsort();
		for(int w=1,p=0;p<n;m=p,w<<=1)
		{
			p=0; for(int i=1;i<=w;++i) x[++p]=n-w+i;
			for(int i=1;i<=n;++i) if(sa[i]>w) x[++p]=sa[i]-w;
			qsort(); swap(x,rk); rk[sa[1]]=p=1;
			for(int i=2;i<=n;++i) 
				if(x[sa[i-1]]==x[sa[i]]&&x[sa[i-1]+w]==x[sa[i]+w]) rk[sa[i]]=p;
				else rk[sa[i]]=++p;
		} return ;
	} 
	inline void get_h()
	{
		int k=0;
		for(int i=1;i<=n;++i)
		{
			if(rk[i]==1){k=h[1]=0; continue;}
			if(k) --k; int j=sa[rk[i]-1];
			while(i+k<=n&&j+k<=n&&b[i+k]==b[j+k]) ++k;
			h[rk[i]]=k;
		} return ;
	} 
	int st[N],top,vis[N];
	inline bool check(int x)
	{
		while(top) vis[st[top--]]=0;
		for(int i=1;i<=n;++i)
		{
			if(h[i]<x) while(top) vis[st[top--]]=0;
			if(!vis[id[sa[i]]]) 
			{
				st[++top]=id[sa[i]]; vis[id[sa[i]]]=1;
				if(top==num) return 1;
			}
		}
		return 0;
	}
	signed main()
	{
		n=read();
		for(int i=1;i<=n;++i)
		{
			len[i]=read();
			for(int j=1;j<=len[i];++j) s[i][j]=read(),maxx=j!=1?max(maxx,s[i][j]-s[i][j-1]):maxx;
			r=min(r,len[i]-1); 
		}
		for(int i=1;i<=n;++i)
		{
			for(int j=2;j<=len[i];++j)
			{
				b[++cnt]=s[i][j]-s[i][j-1];
				id[cnt]=i; minn=min(minn,b[cnt]);
			} b[++cnt]=++maxx;
		}
		for(int i=1;i<=cnt;++i) b[i]=b[i]-minn+1,m=max(m,b[i]);
		num=n; n=cnt; get_sa(); get_h(); 
		int ans=0;
		while(l<=r)
		{
			int mid=(l+r)>>1;
			if(check(mid)) l=mid+1,ans=mid;
			else r=mid-1;
		}cout<<ans+1<<endl;
		return 0;
	}
}
signed main(){return yspm::main();}
```