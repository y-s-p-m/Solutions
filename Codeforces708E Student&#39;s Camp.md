# Description

有一个 $(n+2) \times m$ 的网格。

除了第一行和最后一行，其他每一行每一天最左边和最右边的格子都有 $p$ 的概率消失。

求 $k$ 天后，网格始终保持连通的概率。

$n,m \le 1.5 \times 10^3$，$k \le 10^5$，答案对 $10^9+7$ 取模。

# Solution

先计算 $\displaystyle P_i=\binom Ki(1-p)^{K-i}p^i$ 表示经过 $k$ 天后在单个方向有 $i$ 个格子消失的概率

那么设 $f_{i,l,r}$ 表示第 $i$ 行剩下 $l,r$ 且保证连通到第一行的概率，转移找上一行有交的就行了

设 $L_{i,j}$ 表示第 $i$ 行右端点不超过 $j$ 的所有区间 $f$ 值之和，$R_{i,j}$ 表示左端点不小于 $j$ 的区间 $f$ 值之和

由于矩形是对称的，那么 $L_{i,j}=R_{i,m-j+1}$ 

单个元素转移复杂度难以下降，同时对每个元素各自进行观察除了对称什么也得不到，所以只能尝试摸索前缀和转移是否可行

将单个元素转移式对两侧第二维求和得到

$$F_{i,r}=\sum_{l\le r}(S_{i-1}-L_{i-1,l-1}-L_{i-1,m-r})E_{l-1}E_{m-r}$$

其中 $S_{i}$ 表示第 $i$ 行所有元素的 $\rm dp$ 值求和的结果，拆开分别做前缀和即可

时间复杂度 $\Theta(mn)$

# Code

```cpp
const int N=2010;
int n,p,c[N],d[N][N],dp[N];
signed main(){
	n=read();
	int a=read(),b=read();
	p=mul(a,ksm(b,mod-2));
	d[1][0]=d[1][1]=1;
	for(int i=2;i<=n;i++){
		d[i][0]=1;
		for(int j=1;j<=i;j++)
			d[i][j]=(d[i-1][j]*ksm(mod+1-p,j)+d[i-1][j-1]*ksm(p,i-j))%mod;
	}
	for(int i=1;i<=n;i++){
		int res=0;
		for(int j=1;j<i;j++) ckadd(res,mul(c[j],d[i][j]));
		c[i]=del(1,res);
	}
	for(int i=1;i<=n;i++){
		int res=0;
		for(int j=1;j<i;j++){
			int tmp=(dp[j]+j*(i-j)+dp[i-j]+j*(j-1)/2)%mod;
			res=(res+c[j]*d[i][j]%mod*tmp)%mod;
		}
		res=(res+c[i]*d[i][i]%mod*(i*(i-1)/2))%mod;
		dp[i]=res*ksm(mod+1-c[i]*d[i][i]%mod,mod-2)%mod;
	}
	print(dp[n]);
	return 0;
}
```