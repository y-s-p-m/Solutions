# Solution

发现每个点的修改是对子树有影响的，所以这是个区间改，单点查，所以考虑差分，转化为区间查，单点加

使用主席树维护第 $k$ 大，接着上树状数组维护 $dfn$ 序即可

实现的时候可以参考下方链接里面的内容来减少码量

[https://www.luogu.com.cn/blog/jiqimao/p4175-ti-xie](https://www.luogu.com.cn/blog/jiqimao/p4175-ti-xie)

$Code:$

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
	const int N=80000+10;
	inline int lowbit(int x){return x&(-x);}
	vector<int> g[N];
	int fa[N][20],lg[N],dep[N],b[N<<1],m,T,in[N],out[N];
	struct node{
		int id,k,x,y,val,opt;
	}q[N];
	int c1,c2,c3,c4,t1[30],tim,t2[30],t3[30],t4[30],n,p[N];
	struct tree{
		int ls,rs,val;
		#define ls(p) t[p].ls
		#define rs(p) t[p].rs
		#define val(p) t[p].val
	}t[20000010];
	int rt[N<<2],tot;
	inline void push_up(int p){return val(p)=val(ls(p))+val(rs(p)),void();}
	inline void update(int &p,int l,int r,int x,int v)
	{
		if(!p) p=++tot;
		if(l==r) return val(p)+=v,void();
		int mid=(l+r)>>1;
		if(x<=mid) update(ls(p),l,mid,x,v);
		else update(rs(p),mid+1,r,x,v);
		return push_up(p);
	}
	inline int query(int l,int r,int k)
	{
		if(l==r) return l;
		int mid=(l+r)>>1,sum=0;
		for(int i=1;i<=c3;++i) sum+=val(rs(t3[i]));
		for(int i=1;i<=c4;++i) sum+=val(rs(t4[i]));
		for(int i=1;i<=c2;++i) sum-=val(rs(t2[i]));
		for(int i=1;i<=c1;++i) sum-=val(rs(t1[i]));
		if(k>sum)
		{
			for(int i=1;i<=c1;++i) t1[i]=ls(t1[i]);
			for(int i=1;i<=c2;++i) t2[i]=ls(t2[i]);
			for(int i=1;i<=c3;++i) t3[i]=ls(t3[i]);
			for(int i=1;i<=c4;++i) t4[i]=ls(t4[i]);
			return query(l,mid,k-sum);
		}
		else
		{
			for(int i=1;i<=c1;++i) t1[i]=rs(t1[i]);
			for(int i=1;i<=c2;++i) t2[i]=rs(t2[i]);
			for(int i=1;i<=c3;++i) t3[i]=rs(t3[i]);
			for(int i=1;i<=c4;++i) t4[i]=rs(t4[i]);
			return query(mid+1,r,k);
		}
	}
	inline int lca(int x,int y)
	{
		if(dep[x]<dep[y]) swap(x,y);
		while(dep[x]>dep[y]) x=fa[x][lg[dep[x]-dep[y]]-1];
		if(x==y) return x;
		for(int i=19;i>=0;--i) if(fa[x][i]!=fa[y][i]) x=fa[x][i],y=fa[y][i];
		return fa[x][0];
	}
	inline int ask(int x,int y,int k)
	{
		c1=0,c2=0,c3=0,c4=0; 
		int l=lca(x,y);
		for(int i=in[fa[l][0]];i;i-=lowbit(i)) t1[++c1]=rt[i];
		for(int i=in[l];i;i-=lowbit(i)) t2[++c2]=rt[i];
		for(int i=in[x];i;i-=lowbit(i)) t3[++c3]=rt[i];
		for(int i=in[y];i;i-=lowbit(i)) t4[++c4]=rt[i];
		return query(1,m,k);
	}
	inline void dfs(int x,int fat)
	{
		dep[x]=dep[fat]+1; fa[x][0]=fat; in[x]=++tim; 
		for(int i=1;(1<<i)<=dep[x];++i) fa[x][i]=fa[fa[x][i-1]][i-1];
		int siz=g[x].size();
		for(int i=0;i<siz;++i) if(g[x][i]!=fat) dfs(g[x][i],x);
		return out[x]=tim,void();
	}
	
	inline void change(int x,int pos,int val)
	{
		for(int i=x;i<=n;i+=lowbit(i)) update(rt[i],1,m,pos,val);
		return ;
	}
	signed main()
	{
		for(int i=1;i<N;++i) lg[i]=lg[i-1]+((1<<lg[i-1])==i);
		n=read(); T=read();
		for(int i=1;i<=n;++i) p[i]=read(),b[++m]=p[i];
		for(int i=1;i<n;++i) 
		{
			int u=read(),v=read();
			g[u].push_back(v); g[v].push_back(u); 
		}
		for(int i=1;i<=T;++i)
		{
			q[i].opt=read();
			if(q[i].opt) q[i].x=read(),q[i].y=read(),q[i].k=q[i].opt;
			else q[i].id=read(),q[i].val=read(),b[++m]=q[i].val;
		}
		sort(b+1,b+m+1); m=unique(b+1,b+m+1)-b-1;
		for(int i=1;i<=n;++i) p[i]=lower_bound(b+1,b+m+1,p[i])-b;
		for(int i=1;i<=T;++i) if(!q[i].opt) q[i].val=lower_bound(b+1,b+m+1,q[i].val)-b;
		dfs(1,0);
		for(int i=1;i<=n;++i) change(in[i],p[i],1),change(out[i]+1,p[i],-1);
		for(int i=1;i<=T;++i)
		{
			if(q[i].opt) 
			{
				int l=lca(q[i].x,q[i].y);
				if(dep[q[i].x]+dep[q[i].y]-dep[l]*2+1<q[i].k) puts("invalid request!");
				else printf("%d\n",b[ask(q[i].x,q[i].y,q[i].k)]);
			}
			else
			{
				change(in[q[i].id],p[q[i].id],-1),change(out[q[i].id]+1,p[q[i].id],1);
				p[q[i].id]=q[i].val;
				change(in[q[i].id],p[q[i].id],1); change(out[q[i].id]+1,p[q[i].id],-1);
			}
		}
		return 0;
	}
}
signed main(){return yspm::main();}
```
