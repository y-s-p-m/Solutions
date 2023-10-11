# Description

定义区间权值的 $s(l,r)$ 表示 $\displaystyle\min\limits_{l\le x<y\le r}\{|a_x-a_y|\}$

给定序列 $a$，求最大的 $s(l,r)-(r-l)$

$n\le 2\times 10^5,1\le a_i\le 2\times 10^5$

# Solution

$s(l,r)$ 可以通过 $\rm DP$ 得到：$s(l,r)=\min\{s(l,r-1),s(l+1,r),|a_l-a_r|\}$ 

全题的 Key observation 由抽屉原理得到： $s(l,r)\le \dfrac{m}{r-l+1}$

那么使用根号分治来做：

- 对于区间长度 $\le \sqrt n$ 的部分按照上面的 $\rm DP$ 式子暴力求 $s(l,r)$ 在更新答案

- 对于区间长度比较大的部分，可以枚举 $s$ 的数值，对于每个右端点计算满足条件的最小左端点 

	对于当前枚举的 $s=v$，设 $pos_i$ 表示最小的满足 $s(pos_i,i)=v$ 的位置，转移给 $s=v+1$ 可以通过消去差为 $v$ 的位置，即 $pos_i=max\{pos_{i-1},app_{a_i-v},app_{a_i+v}\}$

时间复杂度 $\Theta(n\sqrt m)$


# Code

```cpp
const int N=2e5+10,B=500;
const int inf=0x3f3f3f3f3f3f3f3f;
int n,m,K,a[N],S[N],lef[N],app[N];
signed main(){
	n=read(); m=read(); K=read();
	rep(i,1,n) a[i]=read(),S[i]=inf;
	int ans=0;
	for(int len=1;len<=B;++len){
		for(int i=1;i<=n-len;++i){
			ckmin(S[i],min(S[i+1],abs(a[i]-a[i+len])));
			if(len>=K-1) ckmax(ans,len*S[i]);		
		}
	}
	rep(j,0,B) lef[j]=1;
	for(int i=1;i<=n;++i){
		for(int j=0;j<=B;++j){
			if(i-lef[j]+1>=K) ckmax(ans,j*(i-lef[j]));
			ckmax(lef[j+1],lef[j]);
			if(a[i]+j<=m) ckmax(lef[j+1],app[a[i]+j]+1);
			if(a[i]-j>=1) ckmax(lef[j+1],app[a[i]-j]+1);
		}
		app[a[i]]=i;
	}
	print(ans);
	return 0;
}
```