# Description

给定正整数 $n$ 和一个长度为 $n$ 的序列 $a$，问有多少长度为 $n$ 的排列 $p$，满足对于任意 $i$ 有 $p_i=a_i$ 或 $p_{p_i}=a_i$。

答案对 $10^9+7$ 取模。

$n \leq 10^5$。

# Solution

$p$ 序列的方案数和统计 $a$ 序列每个联通块能形成多少种子序列 的乘积相同

考虑 $p$ 序列的置换环允许连向原图中距离为 $2$ 的点之后会发生什么：

- 每条边仍然是连向其原来的终点，在 $a$ 序列中的表现是一个简单环

- 全部连向距离为 $2$ 的点：那么对于大小为偶数的环会形成两个环；大小为奇数的环仍然形成一个环，大小不为 $1$ 时形态发生改变

- 形成一个环套树，由于要还原成序列的需求，每个非环部分一定是链

对于所有简单环，统计每种环长的出现个数可以写一个 $\rm DP$ 来统计方案：设 $f_i$ 表示使用了 $i$ 个同大小的环的方案数

转移可以枚举是在第二种情况中简单的改变了形态还是说使用了两个相同的环进行了重拼接，注意两个长度为 $i$ 的环重拼接有 $i$ 中方案

对于每个环套树，找到每个支链，尝试将其揉到环里面去

揉的方案分链顶在 $p$ 中直接指向还是间隔指向，但是间隔指向需要满足链长小于 这个链与相邻的链在环上的距离

全部跑暴力统计即可，时间复杂度 $\Theta(n)$

# Code

```cpp
const int N=1e5+10;
int n,ans=1;
int deg[N],a[N],dp[N];
bool cir[N];
int mark[N],ch[N],ton[N];
signed main(){
	n=read();
	rep(i,1,n) deg[a[i]=read()]++;
	rep(i,1,n) if(!mark[i]){
		int x=i;
		while(!mark[x]) mark[x]=i,x=a[x];
		if(mark[x]==i){
			while(!cir[x]) cir[x]=1,x=a[x];
		}
	}
	for(int i=1;i<=n;++i) if(deg[i]>cir[i]+1) puts("0"),exit(0);
	for(int i=1;i<=n;++i) if(!deg[i]){
		int x=i,len=0;
		while(!cir[x]) ++len,x=a[x];
		ch[x]=len;
	}
	for(int i=1;i<=n;++i) if(cir[i]){
		int len=0;
		int firlen=0,pos=0,lst=0;
		int x=i;
		while(cir[x]){
			cir[x]=0; ++len;
			if(ch[x]){
				if(!pos) pos=len,firlen=ch[x];
				else{
					if(ch[x]>len-lst) puts("0"),exit(0);
					if(ch[x]<len-lst) ckadd(ans,ans);
				}
				lst=len;
			}
			x=a[x];
		}
		if(!pos) ton[len]++;
		else{
			if(firlen>len-lst+pos) puts("0"),exit(0);
			if(firlen<len-lst+pos) ckadd(ans,ans);	
		}
	}
	for(int i=1;i<=n;++i) if(ton[i]){
		dp[0]=1;
		dp[1]=(i>1&&(i&1))+1;
		for(int j=2;j<=ton[i];++j){
			dp[j]=((i>1&&(i&1))+1)*dp[j-1];
			(dp[j]+=i*(j-1)*dp[j-2])%=mod;
		}
		ckmul(ans,dp[ton[i]]);
	}
	print(ans);
	return 0;
}
```