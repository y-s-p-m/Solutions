# Description

[link](https://www.luogu.com.cn/problem/P2336)

概述：

给定 $n$ 个有姓有名的人的姓和名（数字串），然后给 $m$ 个询问

每次询问都是一个数字串，如果询问里的串是某个人的姓或者名，那么这个人就被点到了

求出每个人被点到了几次，和每次询问点到了几个人

# Solution

首先按照 [Sandy 的卡片](https://www.luogu.com.cn/problem/P2463) 这题的方法，我们可以把所有的串连起来

同时把 $height$ 数组 $ST$预处理出来

我们发现每个询问串如果在姓名里面出现了，那么显然会是一个后缀的前缀

后缀排序后，如果真的存在是子串的情况，那么会是一段后缀区间的子串

这个性质相当显然

然后我们二分求出这个区间（这里判断的方法就是 $lcp\ge len$ ），记为$[l,r]$

（以上是对于询问的处理）

然后看题目，每次询问点到了几个人

那我们标记每个后缀属于哪个人，然后问题变成了一段区间里有多少个不同的姓名

用 [HH的项链](https://www.luogu.com.cn/problem/P1972) 的各种解法均可处理这个问题

这里考虑树状数组吧，顺便把询问按照右端点排了序，还方便后面一问的

下一问要求每个点被哪些区间覆盖了

我们把所有的区间的贡献在开始的时候放上去

然后遍历整个连完了的串，如果当前位置已经超过询问的右端点就把询问的贡献消掉

接着考虑每个姓名的贡献，对于一个人，记录其后缀上次出现的位置，然后一个新的点，拿前缀和减去上次的就行了

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
namespace yspm{
	inline int read()
	{
		int res=0,f=1; char k;
		while(!isdigit(k=getchar())) if(k=='-') f=-1;
		while(isdigit(k)) res=res*10+k-'0',k=getchar();
		return res*f;
	}
	const int N=1e6+10;
	int sa[N],h[N][20],rk[N],x[N],y[N],lg[N],num,Q,n,m,cnt;
	int len[N],app[N],b[N],las[N],ans[N],pos[N];
	struct node{int l,r,id;}s[N];
	inline bool cmp(node a,node b){return a.r<b.r;}
	inline void qsort()
	{
		memset(y,0,sizeof(y));
		for(int i=1;i<=n;++i) y[rk[i]]++;
		for(int i=1;i<=m;++i) y[i]+=y[i-1];
		for(int i=n;i>=1;--i) sa[y[rk[x[i]]]--]=x[i];
		return ;
	}
	inline void get_sa()
	{
		m=5e5;
		for(int i=1;i<=n;++i) rk[i]=b[i],x[i]=i; qsort();
		for(int w=1,p=0;p<n;m=p,w<<=1)
		{
			p=0; for(int i=1;i<=w;++i) x[++p]=n-w+i;
			for(int i=1;i<=n;++i) if(sa[i]>w) x[++p]=sa[i]-w;
			qsort(); swap(rk,x); rk[sa[1]]=p=1;
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
			if(rk[i]==1){h[1][0]=k=0; continue;}
			if(k) --k; int j=sa[rk[i]-1];
			while(i+k<=n&&j+k<=n&&b[i+k]==b[j+k]) ++k; 
			h[rk[i]][0]=k;
		}
		for(int j=1;j<=18;++j)
		{
			for(int i=1;i+(1<<j)-1<=n;++i) h[i][j]=min(h[i][j-1],h[i+(1<<(j-1))][j-1]);
		} return ;
	}
	inline int query(int l,int r)
	{
		++l; int t=lg[r-l+1];
		return min(h[l][t],h[r-(1<<t)+1][t]);
	}
	struct BIT{
		int c[N];
		inline int lowbit(int x){return x&(-x);}
		inline void add(int x,int d){for(;x<=n;x+=lowbit(x)) c[x]+=d; return ;}
		inline int ask(int x){int res=0; for(;x;x-=lowbit(x)) res+=c[x]; return res;}
	}T;
	inline void calc(int id)
	{
		int l=1,r=rk[app[id]]-1,ans=rk[app[id]];
		while(l<=r) 
		{
			int mid=(l+r)>>1;
			if(query(mid,rk[app[id]])<len[id]) l=mid+1;
			else ans=mid,r=mid-1;
		}s[id-2*num].l=ans;
		l=rk[app[id]]+1,r=n,ans=rk[app[id]];
		while(l<=r)
		{
			int mid=(l+r)>>1;
			if(query(rk[app[id]],mid)<len[id]) r=mid-1;
			else ans=mid,l=mid+1;
		}s[id-2*num].r=ans; 
		return ;
	}
	signed main()
	{
		lg[0]=-1; lg[1]=0; for(int i=2;i<N;++i) lg[i]=lg[i>>1]+1;
		num=read(); Q=read();
		for(int i=1;i<=num;++i)
		{
			len[++cnt]=read(); app[cnt]=n+1;
			for(int j=1;j<=len[cnt];++j) b[++n]=read(),pos[n]=cnt; b[++n]=cnt+1e4;
			len[++cnt]=read(); app[cnt]=n+1;
			for(int j=1;j<=len[cnt];++j) b[++n]=read(),pos[n]=cnt; b[++n]=cnt+1e4;
		}
		for(int i=1;i<=Q;++i)
		{
			len[++cnt]=read(); app[cnt]=n+1;
			for(int j=1;j<=len[cnt];++j) b[++n]=read(),pos[n]=cnt; b[++n]=cnt+1e4;
		} 
		get_sa(); get_h(); 
		for(int i=1;i<=Q;++i) calc(2*num+i),s[i].id=i;
		sort(s+1,s+Q+1,cmp);
		for(int i=1,j=1;i<=Q;++i)
		{
			while(j<=s[i].r) 
			{
				if(pos[sa[j]]<=2*num) 
				{
					int tmp=(pos[sa[j]]+1)>>1;
					if(las[tmp]) T.add(las[tmp],-1);
					las[tmp]=j;
					T.add(j,1); 
				}++j;
			} ans[s[i].id]=T.ask(s[i].r)-T.ask(s[i].l-1);
		}
		for(int i=1;i<=Q;++i) printf("%d\n",ans[i]);
		memset(ans,0,sizeof(ans)); memset(las,0,sizeof(las));
		memset(T.c,0,sizeof(T.c));
		for(int i=1;i<=Q;++i) T.add(s[i].l,1);
		for(int i=1,j=1;i<=n;++i)
		{
			while(j<=Q&&s[j].r<i) T.add(s[j].l,-1),++j;
			if(pos[sa[i]]<=2*num) 
			{
				int t=(pos[sa[i]]+1)>>1; 
				ans[t]+=T.ask(i)-T.ask(las[t]);
				las[t]=i;
			}
		} for(int i=1;i<=num;++i) printf("%d ",ans[i]); puts("");
		return 0;
	}
}
signed main(){return yspm::main();}
```