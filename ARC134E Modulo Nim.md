# Description

现在在黑板上写了 $n$ 个数 $A_1,\dots A_n$ ，有两个人轮流来修改这些数字并进行博弈。如果所有数字的最大值是 $0$ 当前操作者胜利。否则在 $[1,\max]$ 中选择一个数 $m$ 并将所有数字改为它模 $m$ 的余数

给定 $a_1\dots a_n$ ，求有多少 $\{A_i\}$ 满足 $\forall i\in[1,n],1\le A_i\le a_i$ 且 $\{A_i\}$ 在博弈中先手必胜？

$n\le 200,\max a_i\le 200$

# Solution

将局面转化为不可重集合（不过在最后统计局面数量时不能将相同的 $a_i$ 只处理一个）

$\{1\}$ 局面是必败态。特判 $\exists i\in[1,n],a_i=1$

先尝试相对简单的策略，比如选择 $m=2,3,4$ :

- $m=2$，此时特殊处理 $\{2\}$ 是必败态。可重集合中存在奇数则是必胜态。

下面讨论中将默认集合中没有奇数，且最大值 $\ge 3$

- $m=3$ ，如果集合中必然存在 $\equiv 1,2 \pmod 3$ 的数字，因为 $\{1\},\{2\},\{1,2\}$ 的胜负手已定，如果不是两个余数都存在那么可以直接获胜。否则不会选择 $m=3$

下述讨论中无特殊情况将要求集合中存在模 $3$ 为 $1$ 和余 $2$ ，且这些元素都是偶数。

- $m=4$ ，不是 $4$ 倍数的偶数将导致后手失败，这里仍然出现了一个必败特例 $\{4,8\}$ ，模 $5,6,7,8$ 进入必胜态，模 $1,2,3,4$ 一概讨论失效了

限制更紧了，需要满足集合里面元素是 $12$ 的倍数

注意到 $A_{\max}=200$， $12$ 的倍数只有 $16$ 个。于是可以将这 $16$ 个数字是否存在压成二进制，先做一次 $\rm DP$ 即在 $a_i$ 的限制下每种集合有多少选法，对于那些 $\rm SG$ 值不合法的从所有方案中减去。

$\{1\},\{2\}$ 方案数都是 $1$ ，$\{4,8\}$ 集合的方案是 $2$ 的次幂减去所有数全选 $4,8$ 

求 $SG$ 值可以暴力递归，并在不是所有数都为 $12$ 的倍数的边界直接判断，于是递归到的状态数和存下来的状态数同级

# Code

```cpp
const int N=210;
map<vector<int>,int> SG;
int All=1,a[N],n;
int dp[2][1<<16];
inline int calc_SG(vector<int> vec){
	if(vec.size()==1&&vec[0]<=2) return 0;
	if(vec.size()==2&&vec[0]==4&&vec[1]==8) return 0;
	for(auto v:vec) if(v%12) return 1;
	if(SG.count(vec)) return SG[vec];
	int Mx=vec.back();
	for(int m=5;m<=Mx;++m){
		vector<int> nxt;
		for(auto v:vec) if(v%m) nxt.emplace_back(v%m);
		sort(nxt.begin(),nxt.end());
		nxt.erase(unique(nxt.begin(),nxt.end()),nxt.end());
		if(!calc_SG(nxt)) return SG[vec]=1;
	}
	return SG[vec]=0;
}
signed main(){
	n=read(); 
	int Mx=0,Mn=210;
	rep(i,1,n){
		ckmul(All,a[i]=read());
		ckmax(Mx,a[i]);
		ckmin(Mn,a[i]);
	}
	--All;
	if(All<0) All=mod-1;
	if(Mn==1){
		if(Mx==1) puts("0");
		else print(All);
		return 0;
	}
	int ill=1;
	for(int i=1;i<=n;++i){
		if(a[i]<4) ill=0;
		else if(a[i]>=8) ckadd(ill,ill);
	}
	--All;
	All-=ill;
	if(Mn>=4) ++All;
	if(Mn>=8) ++All;
	int cur=0;
	dp[cur][0]=1;
	Mx/=12;
	int U=(1<<Mx)-1;
	for(int i=1;i<=n;++i){
		for(int s=0;s<=U;++s) if(dp[cur][s]){
			for(int j=1;j*12<=a[i];++j){
				ckadd(dp[cur^1][s|(1<<(j-1))],dp[cur][s]);	
			}
			dp[cur][s]=0;
		}
		cur^=1;
	}
	for(int s=1;s<=U;++s){
		vector<int> vec;
		for(int j=1;j<=Mx;++j) if(s>>(j-1)&1) vec.emplace_back(j*12);
		if(!calc_SG(vec)) All-=dp[cur][s];
	}
	print((All%mod+mod)%mod);
	return 0;
}
```