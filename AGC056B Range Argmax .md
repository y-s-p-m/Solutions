# Description

给定 $m$ 个区间 $[l_i,r_i]$ ，求有多少个 $\{x_1\dots x_m\}$ 满足存在一个 $1\sim n$ 的排列 $p$ 使得 $\forall \ i\in[1,m],x_i=\arg\max\{p_{l_i}\dots p_{r_i}\}$

$n\le 300$

# Solution

找到**最小**的 $k$ 满足存在一个 $x_i=k$ 且所有包含 $k$ 的区间的 $x_i$ 都是 $k$，这里如果任选 $k$ 会导致重复计算

由于本题是数下标组而不是元素本身，那么可以将剩下待确定的 $x_i$ 变成子问题 $[1,k),(k,n]$

需要注意的是，$(k,n]$ 这部分的子问题和原问题是完全一致的，都是要求被 $[L,R]$ 包含的 $[l_i,r_i]$ 的 $x_i$ 且没有额外的限制

但是 $[1,k)$ 这部分需要满足所有包含 $k$ 的元素的区间的最大值必须要是 $k$，也就是在 $[1,k)$ 这个子区间选择新的 $k'$ 时要满足该元素处在全局所有包含 $k$ 的区间的最小左端点以右，否则上一步 $k$ 的选择是非法的

也就是说，如果预处理 `mn[l][r][k]` 表示所有 $[l,r]$ 包含的 $[l_i,r_i]$ 中包含 $k$ 的最小值那么 $mn[l][r][k]\le k'< k$

设 $f_{L,R,K}$ 表示区间 $[L,R]$ 的局部问题中选中的 $k\ge K$ 的情况下会生成多少种 $x_i$，本质上是一个后缀和，而不对 $k$ 加以限制的信息就可以取 $f_{L,R,L}$

此时转移式便成了： $f_{L,R,K}=f_{L,K-1,mn[L][R][K]}\times f_{K+1,R,K+1}+f_{L,R,K+1}$

# Code

```cpp
const int N=310;
int mn[N][N][N],dp[N][N][N],n,m;
bool vis[N][N];
signed main(){
	n=read(); m=read();
	rep(i,1,m){
		int l=read(),r=read();
		vis[l][r]=1;
	}
	for(int i=n;i>=1;--i){
		for(int j=i;j<=n;++j){
			for(int k=i;k<=j;++k){
				mn[i][j][k]=k;
				if(k>=i+1) ckmin(mn[i][j][k],mn[i+1][j][k]);
				if(k<=j-1) ckmin(mn[i][j][k],mn[i][j-1][k]);
				if(vis[i][j]) mn[i][j][k]=i;
			}
		}
	}
	rep(i,0,n) dp[i+1][i][i+1]=1;
	for(int i=n;i>=1;--i){
		for(int j=i;j<=n;++j){
			for(int t=j;t>=i;--t){
				dp[i][j][t]=add(dp[i][j][t+1],mul(dp[i][t-1][mn[i][j][t]],dp[t+1][j][t+1]));
			}
		}
	}
	print(dp[1][n][1]);
	return 0;
}
```