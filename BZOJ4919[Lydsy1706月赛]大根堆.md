# Description

[link](http://www.lydsy.com/JudgeOnline/problem.php?id=4919)

一棵树，点带权，求我们把这棵树转化成一个大根堆最多能有多少节点

即对于任意两个点i,j，如果i在树上是j的祖先，那么$v_i>v_j$

$n\le 2\times 10^5\ \ \ \ \ v_i\le 10^9$

# Solution

状态：$f_{x,i}$ 表示 $x$ 点为根的子树中最大权值为 $i$ 的时候最大点数

在对点 $dfs$ 的时候我们把每个点的儿子的权值线段树合并一下就可以维护信息了

最后是看当前点对于我们的 $f$ 有什么影响 

我们发现我们这个点能带来的更新答案是 $f[x][val[x]-1]+1$

然后我们看看这个题哪里要改，如果$f_{x,p}$ 已经比可更新值大就不用更新了

最关键的性质：对于一个点 $x$ 它的 $f$ 数组是单调不降的（因为权值递增诶……）

在 $dfs$ 的时候我们二分一下可以改的值，（具体看代码，易懂）

然后在权值线段树上改改就行了

这里 $trick$ 标记永久化 ：对于线段树上的每个点，如果这个点的值被更新了（就是$+1$） 

我们不需要下传标记，直接在查询的时候加上父链信息和即可


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
	const int N=200010;
	struct node{
		int sum,ls,rs;
		#define ls(p) t[p].ls
		#define rs(p) t[p].rs
		#define sum(p) t[p].sum
	}t[N<<5];
	int rt[N],tot;
	inline void update(int &p,int l,int r,int st,int ed,int val)
	{
		if(!p) p=++tot;
		if(l<=st&&ed<=r) return sum(p)+=val,void();
		int mid=(st+ed)>>1;
		if(l<=mid) update(ls(p),l,r,st,mid,val);
		if(r>mid) update(rs(p),l,r,mid+1,ed,val);
		return ;
	} 
	inline int query(int p,int l,int r,int x)
	{
		if(!p||!x) return 0;
		int mid=(l+r)>>1;
		if(x<=mid) return query(ls(p),l,mid,x)+sum(p);
		else return query(rs(p),mid+1,r,x)+sum(p);
	} 
	inline int merge(int x,int y,int l,int r)
	{
		if(!x||!y) return x+y;
		if(l==r) return sum(x)+=sum(y),x;
		int mid=(l+r)>>1,p=++tot; 
		sum(p)=sum(x)+sum(y);
		ls(p)=merge(ls(x),ls(y),l,mid);
		rs(p)=merge(rs(x),rs(y),mid+1,r);
		return p;
	}
	vector<int> g[N];
	int n,p[N],b[N],m;
	inline void dfs(int x)
	{
		int siz=g[x].size();
		for(int i=0;i<siz;++i)
		{
			int t=g[x][i]; dfs(t); 
			rt[x]=merge(rt[x],rt[t],1,m);
		}
		int val=query(rt[x],1,m,p[x]-1)+1;
		if(val<=query(rt[x],1,m,p[x])) return ;
		int l=p[x],r=m;
		while(l<r)
		{
			int mid=(l+r+1)>>1;
			if(query(rt[x],1,m,mid)<val) l=mid;
			else r=mid-1;
		}
		update(rt[x],p[x],l,1,m,1);
		return ;
	}
	signed main()
	{
		n=read(); 
		for(int i=1;i<=n;++i)
		{
			p[i]=read(); b[++m]=p[i];
			int fa=read(); g[fa].push_back(i);
		}
		sort(b+1,b+m+1); m=unique(b+1,b+m+1)-b-1;
		for(int i=1;i<=n;++i) p[i]=lower_bound(b+1,b+m+1,p[i])-b;
		dfs(1); 
		printf("%d\n",query(rt[1],1,m,m));
		return 0;
	}
}
signed main(){return yspm::main();} 
```

# Review

请认准标记永久化