# Description

图上有 $n$ 个点，点有点权，点权互不相同，如果两个点点权成倍数关系，那么他们有边。  

现在希望删去一些点，来使图变成二分图，求出最少删去多少点。

$n,a_i\le 5\times 10^4$

# Solution

注意到偏序集合构成的有向图是非常特殊的，即 $x\to y,y\to z$ 就必然存在 $x\to z$，将它们转成无向边之后就构成了三元环

仍然站在有向图的视角考虑：如果要使得最终无向图没有三元环等价于在有向图上不存在同时拥有入度出度的点

这和可以直接套用 $\rm Dilworth$ 定理 $+$ 最小链覆盖 的 “不能有相邻边” 的问题有一定区别，但是可以通过拆点来做：将每个点拆成两个点表示只有入点 $x$ /只有出点 $x'$

**通过构造新的偏序集合来找到不可共存关系是本题要点**

此时偏序集合上的每条边表示这两个点不能共存，那么连边可以通过实际含义来确定

- 每个点连 $(x,x')$

- 偏序集合上的一条边 $x\to y$：只有 $x$ 只有出点，$y$ 只有入点合法，剩下三种连边

那么剩下的工作是找到最长反链，使用最小链覆盖即可解决

# Code

```cpp
const int inf=0x3f3f3f3f;
const int N=5e5+10,M=N*10;
int tot,S,T,mp[N];
struct Network_Flow{
	struct edge{int to,nxt,lim;}e[M];
	int head[N],dep[N],ecnt=1,cur[N];
	inline void adde(int u,int v,int w){
		e[++ecnt]={v,head[u],w}; head[u]=ecnt;
		return ;
	}
	inline void add(int u,int v,int w=1){return adde(u,v,w),adde(v,u,0);}
	inline bool bfs(){
		queue<int> q; q.push(S); 
		rep(i,1,tot) cur[i]=head[i],dep[i]=0; 
		dep[S]=1;
		while(q.size()){
			int fr=q.front(); q.pop();
			for(int i=head[fr];i;i=e[i].nxt) if(e[i].lim){
				int t=e[i].to; if(dep[t]) continue;
				dep[t]=dep[fr]+1; q.push(t);
			}
		} return dep[T];
	}
	inline int dfs(int x,int in){
		if(x==T) return in; int out=0;
		for(int i=head[x];i;cur[x]=i,i=e[i].nxt) if(e[i].lim){
			int t=e[i].to; if(dep[t]!=dep[x]+1) continue;
			int res=dfs(t,min(in,e[i].lim));
			e[i].lim-=res; e[i^1].lim+=res;
			in-=res; out+=res;
			if(!in) break;
		}
		if(!out) dep[x]=0; return out;
	}
	inline int dinic(){
		int sum=0;
		while(bfs()) sum+=dfs(S,inf);
		return sum;
	}
	inline void clear(){
		rep(i,1,tot) head[i]=0;
		ecnt=1;
		tot=0;
		return ;
	}
}F;
int n,a[N],in[N][2],out[N][2];
signed main(){
	int Test=read();
	while(Test--){
		n=read();
		rep(i,1,n) mp[a[i]=read()]=i;
		S=++tot; T=++tot;
		int Mx=0;
		for(int i=1;i<=n;++i){
			in[i][0]=++tot; in[i][1]=++tot;
			out[i][0]=++tot; out[i][1]=++tot;
			F.add(S,in[i][0]);
			F.add(S,in[i][1]);
			F.add(out[i][0],T);
			F.add(out[i][1],T);
			F.add(in[i][1],out[i][0]);
			ckmax(Mx,a[i]);
		}
		for(int i=1;i<=Mx;++i) if(mp[i]){
			for(int j=i*2;j<=Mx;j+=i) if(mp[j]){
				F.add(in[mp[i]][0],out[mp[j]][0]);
				F.add(in[mp[i]][1],out[mp[j]][0]);
				F.add(in[mp[i]][1],out[mp[j]][1]);
			}
		}
		print(F.dinic()-n);
		F.clear();
		rep(i,1,n) mp[a[i]]=0;
	}
	return 0;
}
```