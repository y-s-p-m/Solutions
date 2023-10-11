最近发现自己喜欢上了最小割树（可能是感觉新鲜吧）

然后在网上找到了这个题

# Description

[link](https://www.luogu.com.cn/problem/P3729)

给定 $n$ 个点和 $m$ 条边，点带权，定义 $\{V\}$ 的安全程度为任意 $u,v$ 两点间的安全程度为两点间互不相交的链的个数的最小值

求点权和不小于询问的 $k$ 的任意连通块内最大的安全系数

$n\le 550 \ \ m\le 3000$

# Solution

看到“互不相交的链”，直觉转成最大流

然后转最小割

考虑解题需要任意两点的最小割，所以上最小割树

把询问从小到大排序，把最小割树上的边逆序排，然后我们用并查集合并树上的连通块，并且记录连通块内的点权和

对应更新答案即可

还是比较不错的题，转化出来就没啥了……（冲着算法找题选手捂脸）

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
	const int N=600,M=5010;
	struct node{
		int from,to,dis;
		bool operator<(const node e)const{return dis>e.dis;}
	}e[M<<1];
	int head[N],cnt,n,m,q;
	inline void add1(int u,int v,int w)
	{
		e[++cnt].from=u; e[cnt].to=v; e[cnt].dis=w;
		return ;
	}
	struct Graph{
		struct node{
			int nxt,to,lim,val;
		}e[M<<1];
		int head[N],cnt=1,col[N],tot,dep[N],S,T,t[N],a[N];
		queue<int> q;
		inline void adde(int u,int v,int w)
		{
			e[++cnt].val=w; e[cnt].to=v; e[cnt].nxt=head[u];
			return head[u]=cnt,void(); 
		}
		inline void add(int u,int v,int w){return adde(u,v,w),adde(v,u,0);}
		inline void getcol(int x)
		{
			col[x]=tot;
			for(int i=head[x];i;i=e[i].nxt)
			{
				int t=e[i].to; 
				if(e[i].lim&&col[t]!=tot) getcol(t); 
			} return ;
		}
		inline bool bfs()
		{
			while(q.size()) q.pop(); q.push(S);
			memset(dep,0,sizeof(dep)); dep[S]=1;
			while(q.size())
			{
				int fr=q.front(); q.pop(); 
				for(int i=head[fr];i;i=e[i].nxt)
				{
					int t=e[i].to; 
					if(e[i].lim&&!dep[t]) dep[t]=dep[fr]+1,q.push(t);
				}
			} return dep[T];
		}
		inline int dfs(int now,int in)
		{
			if(now==T) return in; int out=0;
			for(int i=head[now];i&&in;i=e[i].nxt)
			{
				int t=e[i].to; 
				if(dep[t]==dep[now]+1&&e[i].lim) 
				{
					int res=dfs(t,min(e[i].lim,in));
					e[i].lim-=res; e[i^1].lim+=res; 
					in-=res; out+=res;
				}
			}
			if(!out) dep[now]=0;
			return out;
		}
		inline int dinic()
		{
			for(int i=2;i<=cnt;++i) e[i].lim=e[i].val;
			int ans=0;
			while(bfs()) ans+=dfs(S,2e9+10); 
			return ans;
		}
		inline void build(int l,int r)
		{
			if(l==r) return ;
			int x=a[l],y=a[l+1]; S=x; T=y;
			int res=dinic(); tot++;
			add1(x,y,res); getcol(x);
			int cl=l,cr=r;
			for(int i=l;i<=r;++i) 
			{
				if(col[a[i]]==tot) t[cl++]=a[i];
				else t[cr--]=a[i];
			} 
			for(int i=l;i<=r;++i) a[i]=t[i];
			return build(l,cl-1),build(cr+1,r);
		}
	}G;
	int ans[N],sum[N],maxx,fa[N];
	inline int find(int x){return fa[x]==x?fa[x]:fa[x]=find(fa[x]);}
	struct ask{
		int id,val;
		bool operator <(const ask &qu) const{
			return val<qu.val;
		}
	}Q[N];
	int now=1,w[N];
	signed main()
	{
		n=read(); m=read(); q=read();
		for(int i=1;i<=q;++i) ans[i]=2e9+10;
		for(int i=1;i<=n;++i) w[i]=read();
		for(int i=1;i<=m;++i)
		{
			int x=read(),y=read(); 
			G.add(x,y,1); G.add(y,x,1);
		}
		for(int i=1;i<=q;++i) Q[i].val=read(),Q[i].id=i;
		sort(Q+1,Q+q+1);
		for(int i=1;i<=n;++i) G.a[i]=i;
		G.build(1,n); sort(e+1,e+cnt+1);
		for(int i=1;i<=n;++i) sum[i]=w[i],fa[i]=i,maxx=max(maxx,sum[i]);
		while(maxx>=Q[now].val&&now<=q) ans[Q[now].id]=0,now++;
		for(int i=1;i<=cnt;++i)
		{
			int u=e[i].from,v=e[i].to;
			u=find(u); v=find(v);
			fa[u]=v; sum[v]+=sum[u];
			maxx=max(maxx,sum[v]);
			while(maxx>=Q[now].val&&now<=q) ans[Q[now].id]=e[i].dis,now++;
		}
		for(int i=1;i<=q;++i)
		{
			if(ans[i]==0) puts("nan");
			else if(ans[i]==2e9+10) puts("Nuclear launch detected");
			else printf("%d\n",ans[i]);
		 } 
		return 0;
	}
}
signed main(){return yspm::main();}
```