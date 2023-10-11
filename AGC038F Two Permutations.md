# Description

给定两个 $0 \sim (n - 1)$ 的排列 $\{p_0, p_1, \ldots , p_{n - 1}\}$ 和 $\{q_0, q_1, \ldots , q_{n - 1}\}$，要求构造两个 $0 \sim (n - 1)$ 的排列 $\{A_0, A_1, \ldots , A_{n - 1}\}$ 和 $\{B_0, B_1, \ldots , B_{n - 1}\}$，且必须满足： $A_i$ 要么等于 $i$，要么等于 $p_i$，$B_i$ 要么等于 $i$，要么等于 $q_i$。

最大化 $A_i \ne B_i$ 的下标 $i$ 的数量。

$n\le 10^5$

# Solution

将答案置为 $n$ 再减少。$p_i=q_i=i$ 的 $i$ 必然让答案减少 $1$ 。

根据每个位置在两个排列上是否选择走置换环分情况讨论：

- $p_i=i$ 或者 $q_i=i$，另一个选择不走置换环会导致答案减少

- $p_i=q_i$ 都走或者都不走置换环会导致答案减少

- $p_i\neq q_i$ ，都不走会导致答案减少

用最小割来刻画答案的减量。如果 $p,q$ 中两个置换分到了和 $S/T$ 连通的同一个集合则视为置换环转的情况相异，将 $p$ 中转的置换分入 $S$ 集合，$q$ 中转的置换分入 $T$ 集合，连边如下：

- $p_i=i,q_i\neq i$ 则 $q_i$ 所在置换不旋转（分到 $S$ 集合中）时答案减少，于是对应该置换的点和 $T$ 连流量为 $1$ 的边

- $p_i\neq i,q_i=i$ 同上，只不过变成了 $p$ 中的点和 $S$ 连

- $p_i,q_i\neq i,p_i\neq q_i$，如果 $p_i$ 所在置换对应的点和 $T$ 联通（不转），$q_i$ 所在置换对应的点和 $S$ 联通（也是不转） 就需要让答案减少，那么从 $q$ 中点向 $p$ 中点连边即可

- $p_i,q_i\neq i,p_i= q_i$，同时不转在上面刻画过了，同时转也类似，那么从 $p$ 中点向 $q$ 中点连边即可 

跑最大流，时间复杂度 $\Theta(n\sqrt n)$ ，当然需要当前弧优化

# Code

```cpp
const int N=4e5+10;
int n,p[N],q[N],idp[N],idq[N],tot;
int head[N],cur[N],dep[N],S,T,ecnt=1;
struct edge{int to,nxt,lim;}e[N<<2];
inline void adde(int u,int v,int w){
	e[++ecnt]={v,head[u],w}; head[u]=ecnt;
	return ;
}
inline void add(int u,int v,int w){return adde(u,v,w),adde(v,u,0);}
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
	for(int &i=cur[x];i;i=e[i].nxt) if(e[i].lim){
		int t=e[i].to; if(dep[t]!=dep[x]+1) continue;
		int res=dfs(t,min(in,e[i].lim));
		e[i].lim-=res; e[i^1].lim+=res;
		in-=res; out+=res;
		if(!in) break;
	}
	if(!out) dep[x]=0; return out;
}
int main() {
    n=read();
    for(int i=1;i<=n;++i) p[i]=read()+1;
    for(int i=1;i<=n;++i) q[i]=read()+1;
    S=++tot; T=++tot;
	for(int i=1;i<=n;++i) if(!idp[i]){
		int x=i;
		++tot;
		while(!idp[x]) idp[x]=tot,x=p[x];
	}
	for(int i=1;i<=n;++i) if(!idq[i]){
		int x=i;
		++tot;
		while(!idq[x]) idq[x]=tot,x=q[x];
	}
	int ans=n;
    for (int i=1;i<=n;++i){
	    if(p[i]==i||q[i]==i){
			if(p[i]==i&&q[i]==i){
				--ans;
				continue;
			}
			if(p[i]==i) add(idq[i],T,1);
        	if(q[i]==i) add(S,idp[i],1);
		}else{
			if(p[i]!=q[i]) add(idq[i],idp[i],1);
        	else add(idq[i],idp[i],1),add(idp[i],idq[i],1);
		}
	}
	while(bfs()) ans-=dfs(S,2*n);
    print(ans);
	return 0;
}

```