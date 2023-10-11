# Description

[link](https://www.luogu.com.cn/problem/P4747)

给定一个排列和一些询问，每个询问求给定 $l,r$，求包含当前区间的最小的完美子区间

完美子区间的定义为区间内的元素构成了值域连续的一段

$n\le 10^5,Q\le 10^5$

# Solution

## 前置

首先是关于这个题目的一些性质（但是我考试做这个题目的时候只剩下 $30\ min$ 了，显然是没有做探究性质这个工作的

	完美子区间的交还是完美子区间

## Method I

根据 $@i207M$ ，连续值域的问题可以考虑用离线之后线段树维护扫描线来解决

我们考虑用线段树维护扫描线

其实一个区间 $\ [\ l,r\ ]\ $ 是一个完美子序列的本质就是中里面有 $r-l$ 个相邻的数对

初始令线段数的每个下标的 $val_i=i$

从左到右来，如果当前位置的下一个值的位置是小于它的，那么$[1,pos_{a_i+1}]$区间加一 

小于也一样

然后如果 $val_l=r$ 那么区间$\ [\ l,r\ ]\ $ 就是完美子区间（满足本质）

代码是 $@i207M$ 的

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
	const int N=1e5+10;
	struct node{
		int l,r,val,pos,fl;
		#define fl(p) t[p].fl
 		#define l(p) t[p].l
		#define r(p) t[p].r
		#define val(p) t[p].val
		#define pos(p) t[p].pos//较大val出现的位置
	}t[N<<2];
	inline void push_up(int p)
	{
		val(p)=max(val(p<<1),val(p<<1|1));
		pos(p)=val(p<<1)>val(p<<1|1)?pos(p<<1):pos(p<<1|1);
		return ;
	}
	inline void push_down(int p)
	{
		if(!fl(p)) return ; 
		val(p<<1)+=fl(p); val(p<<1|1)+=fl(p);
		fl(p<<1)+=fl(p); fl(p<<1|1)+=fl(p);
		fl(p)=0;
		return ;
	}
	inline void update(int p,int l,int r)
	{
		if(l<=l(p)&&r(p)<=r) return val(p)++,++fl(p),void();
		int mid=(l(p)+r(p))>>1; push_down(p);
		if(l<=mid) update(p<<1,l,r); if(r>mid) update(p<<1|1,l,r);
		return push_up(p);
	}
	pair<int,int> ans[N];
	vector<pair<int,int> > g[N];
	priority_queue<pair<int,int> > q;
	int mx,id,a[N],p[N],n,T;
	inline void query(int p,int l,int r)
	{
		if(l<=l(p)&&r(p)<=r) 
		{
			if(mx<=val(p)) mx=val(p),id=pos(p);
			return ;
		}int mid=(l(p)+r(p))>>1; push_down(p);
		if(l<=mid) query(p<<1,l,r); if(r>mid) query(p<<1|1,l,r);
		return ;
	}//同时找到区间内最大数字和最大数字出现的位置
	inline void build(int p,int l,int r)
	{
		l(p)=l; r(p)=r; if(l==r) return val(p)=l,pos(p)=l,void();
		int mid=(l+r)>>1; build(p<<1,l,mid); build(p<<1|1,mid+1,r);
		return push_up(p);
	}
	inline bool judge(pair<int,int> x,int now)
	{	
		mx=0; query(1,1,x.first);//考虑到区间修改的次数，所以最大值应该当作val[l] 做就好了
		if(mx==now) return ans[x.second]=make_pair(id,now),1;
		return 0;
	}
	signed main()
	{
		n=read(); for(int i=1;i<=n;++i) a[i]=read(),p[a[i]]=i; T=read();
		for(int l,r,i=1;i<=T;++i) l=read(),r=read(),g[r].push_back(make_pair(l,i)); 
		build(1,1,n); 
		for(int i=1;i<=n;++i)
		{
			if(a[i]>1&&p[a[i]-1]<=i) update(1,1,p[a[i]-1]);
			if(a[i]<n&&p[a[i]+1]<=i) update(1,1,p[a[i]+1]); 
			int sz=g[i].size();
			for(int j=0;j<sz;++j) q.push(g[i][j]);
			while(!q.empty()) if(judge(q.top(),i)) q.pop(); else break;
		}
		for(int i=1;i<=T;++i) printf("%lld %lld\n",ans[i].first,ans[i].second);
		return 0;
	}
}
signed main(){return yspm::main();}
```

# Method II

对于那些 $l=r$ 的询问，直接特判掉

剩下的考虑一对相邻的点，如果这两个点都在这个区间里面出现了

那么 $[min_{pos_{L\to R}},max_{pos_{L \to R}}]$ 里面的点就都要在这个里面的

这样就是 $0 pts$ 暴力的做法的思想

那么现在是本题目的解法，所以我们考虑点对边建图

用 $E(x\to y)$ 表示如果点 $x$ 在序列里面了，那么 $y$ 也必须在序列里面

看上去是个点对对区间建边的操作，显而易见，线段树优化建图

然后考虑一个 $tarjan$ 把强联通分量缩起来，也就是如果这里面的那个东西出现之后那个强联通分量里面的东西也都要出现

然后对于每个询问，就考虑区间内的所有范围取并就做完了

但是口胡好轻松！写起来 $2444$ 不太好写也不太会写，留坑吧，回头再说

## Method III

析合树

留坑