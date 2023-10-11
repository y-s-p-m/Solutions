# Description

[link](https://www.luogu.com.cn/problem/P4480)

一个餐厅在相继的 $n$ 天里，每天需用的餐巾数不尽相同。假设第 $i$天 $(i=1,2,...,n)$ 需要 $r_i$ 块餐巾。餐厅可以在任意时刻购买新的餐巾，每块餐巾的费用为 $p$ 。

使用过的旧餐巾，则需要经过清洗才能重新使用。把一块旧餐巾送到清洗店A，需要等待 $m_1$ 天后才能拿到新餐巾，其费用为 $c_1$ ；把一块旧餐巾送到清洗店B，需要等待 $m_2$ 天后才能拿到新餐巾，其费用为 $c_2$ 。

例如，将一块第 $k$ 天使用过的餐巾送到清洗店A清洗，则可以在第 $k+m_1$ 天使用。

请为餐厅合理地安排好 $n$ 天中餐巾使用计划，使总的花费最小。

$n\le2000$

加强版 $n\le 200000$

# Solution

## Method I

考虑费用流解决这个问题

首先对每天拆点

买进餐巾： $S\to i_1$ 建 $(inf,p)$ 

快洗：$i_2\to (i+m_1+1)_ 1$ 建 $(inf,c_1)$

慢洗：$i_2\to(i+m_2+1)_ 1$ 建 $(inf,c_2)$

向汇点：$i\to T$ 建 $(a_i,0)$ 

每天用脏的：$S\to i_2$ 建 $(a_i,0)$

向下一天转移的毛巾 $i_2\to (i+1)_ 2 $ 建 $(inf,0)$ 

然后最小费用最大流的板子套上就能过掉

```cpp
const int inf=0x3f3f3f3f3f3f3f3f;
const int N=4010,M=100010;
struct node{
	int to,nxt,lim,cost;
}e[M<<1];
int head[N],cnt=1,ans,incf[N],n,pre[N],d[N],c[N],s,t,c1,c2,m2,m1,p,mx;
queue<int> q; bool vis[N];
inline void adde(int u,int v,int w,int c){
	e[++cnt].to=v; e[cnt].nxt=head[u]; e[cnt].lim=w; e[cnt].cost=c;
	return head[u]=cnt,void();
}
inline void add(int u,int v,int w,int c){return adde(u,v,w,c),adde(v,u,0,-c);}
inline bool spfa(){
	while(q.size()) q.pop(); memset(d,0x3f,sizeof(d)); 
	q.push(s); d[s]=0; vis[s]=1; incf[s]=inf; 
	while(!q.empty()){
		int fr=q.front(); q.pop(); vis[fr]=0;
		for(int i=head[fr];i;i=e[i].nxt) if(e[i].lim){
			int nt=e[i].to; 
			if(d[nt]>d[fr]+e[i].cost){
				d[nt]=d[fr]+e[i].cost; pre[nt]=i; incf[nt]=min(incf[fr],e[i].lim);
				if(!vis[nt]) vis[nt]=1,q.push(nt);
			}
		}
	} return d[t]!=inf;
} 
signed main(){
	n=read(); m1=read(); m2=read(); p=read(); c1=read(); c2=read();  
	for(int i=1;i<=n;++i) c[i]=read();
	s=n<<1|1; t=s+1;
	for(int i=1;i<=n;++i){
		if(i+m1+1<=n) add(i+n,i+m1+1,inf,c1);
		if(i+m2+1<=n) add(i+n,i+m2+1,inf,c2);
		if(i+1<=n) add(i+n,i+1+n,inf,0);
		add(s,i,inf,p);
		add(s,i+n,c[i],0);
		add(i,t,c[i],0);
	}
	while(spfa()){
		int x=t; 
		while(x!=s){
			e[pre[x]].lim-=incf[t];
			e[pre[x]^1].lim+=incf[t];
			x=e[pre[x]^1].to;
		} mx+=incf[t],ans+=incf[t]*d[t];
	} printf("%lld\n",ans);
	return 0;
}

```

## Method II

首先关注到如果买进一定的餐巾那么可以构造出一种方案使得在当前餐巾数量合法的前提下花费最小

（这里直接上个贪心啥的模拟一下就好了，并不是很好实现，我看了 $std$）

然后观察每个餐巾数量都取 $min$ 花费的前提下，这个花费和数量的关系

如果买多了不洗不优，反之如果买少了，要不是不合法，要不就会让洗的次数过多？（这里其实有点迷惑的，因为好像这个关系如果洗的多的话然后能优一点？）

上面的留坑

花费最小值和购买量存在函数关系

假令买少了也不优，那么这个函数是个单谷的

所以直接上个三分求一求

```cpp
const int inf=1e18+10,N=2e5+10;
int l,ans=inf,n,r,m1,m2,c1,a[N],c2,p;
struct node{int num,tim;}; 
deque<node>qx,qk,qm;
inline int calc(int x){
	int las,now,res=p*x; 
	while(qx.size()) qx.pop_front(); 
	while(qk.size()) qk.pop_front();
	while(qm.size()) qm.pop_front();
	for(int i=1;i<=n;++i){
		while(qx.size()&&qx.front().tim+m2<=i) qk.push_back(qx.front()),qx.pop_front();
		while(qk.size()&&qk.front().tim+m1<=i) qm.push_back(qk.front()),qk.pop_front();
		las=a[i],now=min(x,las);
		x-=now; las-=now;
		while(las&&qm.size()){
			now=min(qm.back().num,las);
			las-=now; res+=c1*now;
			if(now==qm.back().num) qm.pop_back();
			else qm.back().num-=now;
		}
		while(las&&qk.size()){
			now=min(las,qk.back().num);
			las-=now; res+=c2*now;
			if(now==qk.back().num) qk.pop_back();
			else qk.back().num-=now;
		}
		if(las) return 1e18+10;
		qx.push_back((node){a[i],i});
	} return res;
}
signed main(){
	n=read(); m1=read(); m2=read(); c1=read(); c2=read(); p=read();
	if(m1<m2) swap(m1,m2),swap(c1,c2); //m1 -> slow
	if(c1>c2) c1=c2,m1=m2;
	for(int i=1;i<=n;++i) r+=(a[i]=read());
	while(l<=r){
		int mid=(l+r)>>1;
		int a1=calc(mid),a2=calc(mid+1);
		if(a1<a2) ans=a1,r=mid-1;
		else ans=a2,l=mid+1;
	}printf("%lld\n",ans);
	return 0;
}
```
