# Description

给定长度为 $n$ 的数列 $a$，求

$$\max\limits_{1\le i<j\le n}\{\rm LCM(a_i,a_j)\}$$

$n,a_i\le 10^5$

# Solution

$\rm LCM$ 先转化成乘积除以 $\rm gcd$ 的形式，枚举 $\rm gcd=t$ 并提取出来所有是该数倍数的数，下面叙述中的 $x$ 都表示除过 $t$ 得到的结果

从小到大枚举每个元素并将其设为 $\max\{a_i,a_j\}$ ，也就是说在被提取出来的所有数字中找到的最大的小于它的且和它互质的数

如果想要判定一些数字和某个数字 $x$ 是不是互质，可以通过 $[gcd(x,y)=1]=\sum\limits_{d|x,d|y}\mu(d)$ 来处理：

设 $cnt_x$ 表示 $x$ 的倍数在被提取出来的数字除 $\rm t$ 出现了多少次

先计算 $s=\sum\limits_{d|x}\mu (x)cnt_x$ 再二分找到最大的前缀满足只考虑该前缀在桶里面的贡献仍然可以得到 $s$ 

不难发现还要把桶可持久化所以复杂度太高了

通过从大到小枚举来减少冗余计算量，如果某个 $x$ 找到最大的 $z$ 满足 $\rm gcd\left(x,z\right)=1$ ，那么 $x<y<z$ 的所有 $y$ 在后续计算中将不被需要

那么维护一个栈以及和上面类似的桶，先算一次 $\sum\limits_{d|x}\mu (x)cnt_x$ 得到互质元素数量，再直接弹栈到不再存在互质元素即可

复杂度到了 $\Theta(n\log^2n)$ 但是还可以更优！

对于最优解 $(x,y)$ 那么一定存在一组 $a|x,b|y$ 且 $\rm gcd(a,b)=1,lcm(a,b)=lcm(x,y)$ 那么直接将每个数字的存在性表示为其所有倍数存在性的 或 结果，直接做 $\rm gcd=1$ 的部分即可

时间复杂度 $\Theta(n\log n)$ 

# Code

```cpp
const int N=1e5+10;
int stk[N],top;
int n;
bool a[N],isc[N];
int mu[N],pri[N],cnt;
vector<int> Div[N];
int ton[N];
signed main(){
	n=1e5; mu[1]=1;
	for(int i=2;i<=n;++i){
		if(!isc[i]) pri[++cnt]=i,mu[i]=-1;
		for(int j=1;j<=cnt&&pri[j]*i<=n;++j){
			isc[i*pri[j]]=1;
			if(i%pri[j]==0){
				mu[i*pri[j]]=0;
				break;
			}else mu[i*pri[j]]=-mu[i];
		}
	}
	n=read();
	for(int i=1;i<=n;++i) a[read()]=1;
	for(int i=1e5;i>=1;--i){
		for(int j=i;j<=1e5;j+=i){
			a[i]|=a[j];
			Div[j].emplace_back(i);
		}
	}
	int ans=0;
	for(int i=1e5;i>=1;--i) if(a[i]){
		ckmax(ans,i);
		int num=0;
		for(auto t:Div[i]) num+=ton[t]*mu[t];
		while(num){
			if(gcd(i,stk[top])==1) --num,ckmax(ans,i*stk[top]);
			for(auto t:Div[stk[top]]) --ton[t];
			--top;
		}
		for(auto t:Div[i]) ton[t]++;
		stk[++top]=i;
	}
	print(ans);
	return 0;
}
```