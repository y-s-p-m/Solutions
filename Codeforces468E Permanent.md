# Description

给定一个 $n\times n$ 的矩阵，初始时矩阵的每个元素都为 $1$。又给出 $k$ 个三元组 $(x,y,z)$，表示将矩阵的 $(x,y)$ 位置变成 $z$。你需要求出完成这些变换之后矩阵的积和式。对 $10^9+7$ 取模。

$n\le 10^5,k\le 50$

# Solution

$w=(w-1)+1$ 。对于一种合法的选择特殊位置的方案（符合排列），设其选择 $m$ 个特殊位置，则对答案的贡献为特殊位置的 $w$ 减一后乘积再乘 $(n-m)!$，因为没有被命中的列的排列可以任选。

考虑处理特殊位置，将其转化为行和列的匹配，那么匹配数量就是上面的 $m$ 。在状压 $\rm DP$ 时不用另开一维。

$\Theta(k2^{k})$ 的想法不再赘述。

注意到在一个点选择出边的时候，影响的只是这个联通块里面的点。可以对于每个联通块分别处理来将点数降到 $k$，同时将二分图的点数小的一侧压进状态里，复杂度降到 $\displaystyle\Theta(k2^{\frac{k}2})$

不利用二分图的性质那么还有一个做法：求出来原图一棵生成树，并枚举非树边是否选择。树边的选择情况是可以通过树形 $\rm DP$ 来统计的，记录子树里面有多少条边被选入匹配即可，复杂度 $\Theta(2^{m-n}n^2)$ 。

揉起来可以做到 $\Theta(k^2n^{\frac{1}3})$ ，可以通过

# Code

```cpp
const int N=2e6+10;
int n,K,sum[N],tmp[N],res[N],fac[N];
vector<pair<int,int> > G[N];
bool vis[N];
int cnt_edge;
vector<int> node;
inline void expand(int x){
	if(vis[x]) return ;
	vis[x]=1;
	node.emplace_back(x); cnt_edge+=G[x].size();
	for(auto t:G[x]) expand(t.fir);
}
namespace Bitmask_DP{
	int id[N],idx[N],idy[N],cnty,cntx;
	int dp[1<<20];
	inline void work(){
		cntx=0,cnty=0;
		for(auto x:node){
			if(x<=n){
				idx[++cntx]=x;
				id[x]=cntx;
			}else{
				idy[++cnty]=x;
				id[x]=cnty;
			}
		}
		if(cntx<cnty){
			for(int i=1;i<=cnty;++i){
				swap(idx[i],idy[i]);
				swap(id[idx[i]],id[idy[i]]);
			}
			swap(cntx,cnty);
		}
		int U=1<<cnty; --U;
		dp[0]=1;
		for(int i=1;i<=cntx;++i){
			for(int s=U;s>=0;--s) if(dp[s]){
				dp[s]%=mod;
				for(auto t:G[idx[i]]){
					if(s>>(id[t.fir]-1)&1) continue;
					dp[s|(1<<(id[t.fir]-1))]+=dp[s]*t.sec%mod;	
				}
			}
		}
		rep(s,0,U) if(dp[s]) res[__builtin_popcount(s)]+=dp[s],dp[s]=0;
		rep(i,1,cntx) id[idx[i]]=0,idx[i]=0;
		rep(i,1,cnty) id[idy[i]]=0,idy[i]=0;
		return ;
	}	
}
namespace Brute_Force{
	struct DSU{
		int fa[N];
		inline void init(int n){rep(i,1,n) fa[i]=i; return ;}
		inline int find(int x){return fa[x]==x?x:fa[x]=find(fa[x]);}
		inline void merge(int x,int y){fa[find(x)]=find(y); return ;}
	}Dsu;
	struct edge{int u,v,w;}e[110];
	int ecnt,idcnt;
	vector<pair<int,int> > tr[N];
	bool mark[N];
	int dp[300][300][2],tmp[300][2],id[N],siz[300];
	inline void dfs(int x,int fat){
		memset(dp[x],0,sizeof(dp[x]));
		dp[x][0][mark[x]]=siz[x]=1;
		for(auto e:tr[x]){
			int t=e.fir; if(t==fat) continue;
			dfs(t,x);
			for(int i=0;i<=siz[x];++i){
				for(int j=0;j<=siz[t];++j){
					tmp[i+j+1][1]+=dp[x][i][0]*dp[t][j][0]%mod*e.sec%mod;
					tmp[i+j][0]+=dp[x][i][0]*(dp[t][j][0]+dp[t][j][1])%mod;
					tmp[i+j][1]+=dp[x][i][1]*(dp[t][j][0]+dp[t][j][1])%mod;
				}
			}
			siz[x]+=siz[t];
			for(int i=0;i<=siz[x];++i){
				dp[x][i][0]=tmp[i][0]%mod;
				dp[x][i][1]=tmp[i][1]%mod;
				tmp[i][0]=tmp[i][1]=0;
			}
		}
	}
	inline void work(){
		idcnt=ecnt=0;
		for(auto x:node) id[x]=++idcnt;
		Dsu.init(idcnt);
		rep(i,1,idcnt) tr[i].clear();
		for(auto x:node){
			if(x>n) break;
			for(auto t:G[x]){
				int u=id[x],v=id[t.fir];
				if(Dsu.find(u)!=Dsu .find(v)){
					Dsu.merge(u,v);
					tr[v].emplace_back(u,t.sec);
					tr[u].emplace_back(v,t.sec);
				}else{
					e[++ecnt]={u,v,t.sec};
				}
			}
		}
		int S=1<<ecnt; --S;
		for(int s=0;s<=S;++s){
			bool legal=1;
			int coef=1;
			for(int i=1;i<=ecnt;++i) if(s>>(i-1)&1){
				if(mark[e[i].u]||mark[e[i].v]){
					legal=0;
					break;
				}
				ckmul(coef,e[i].w);
				mark[e[i].u]=mark[e[i].v]=1;
			}
			if(legal){
				dfs(1,0);
				int Bas=__builtin_popcount(s);
				for(int i=0;i<=siz[1];++i){
					int val=add(dp[1][i][0],dp[1][i][1]);
					res[Bas+i]+=val*coef%mod;
				}
			}
			rep(i,1,ecnt) mark[e[i].u]=mark[e[i].v]=0;
		}
		return ;
	}
}
signed main(){
	n=read(),K=read();
	for(int i=1;i<=K;++i){
		int x=read(),y=read(),z=read();
		ckdel(z,1);
		G[x].emplace_back(y+n,z);
		G[y+n].emplace_back(x,z);
	}
	sum[0]=1;
	for(int x=1;x<=n+n;++x) if(G[x].size()&&!vis[x]){
		cnt_edge=0; node.clear();
		expand(x);
		sort(node.begin(),node.end());
		cnt_edge/=2;
		if(cnt_edge*2>node.size()*3) 
		Bitmask_DP::work();
		else 
		Brute_Force::work();
		for(int i=0;i<=K;++i) res[i]%=mod;
		for(int i=0;i<=K;++i) for(int j=0;i+j<=K;++j) tmp[i+j]+=sum[i]*res[j]%mod;
		rep(i,0,K) sum[i]=tmp[i]%mod,res[i]=tmp[i]=0; 
	}
	int ans=0; fac[0]=1;
	for(int i=1;i<=n;++i) fac[i]=mul(fac[i-1],i);
	for(int i=n;i>=0;--i) ans+=mul(sum[i],fac[n-i]);
	cout<<ans%mod<<endl;
	return 0;
}
```