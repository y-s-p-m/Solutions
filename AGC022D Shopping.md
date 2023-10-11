# Description

有 $n$ 个商场, 第 $i$ 个商场在数轴上的 $x_i$ 处, 你需要在第 $i$ 个商场花费连续的 $t_i$ 单位时间购物。现在有一趟火车会在 $0$ 到 $L$ 处往返, 行驶一单位距离要花费一单位时间。

你从 $0$ 时刻起在 $0$ 处上车, 只有在商场, $0$ 处或 $L$ 处才能下车, 问最少花费多少单位时间能在每一个商场都购完物后回到 $0$ 处。

$n\leqslant 3\times 10^5, 0<x<L\leqslant 10^9$.

# Solution

答案是 $2L$ 的倍数，于是将 $t_i$ 先缩到 $2L$ 以内

最劣的策略就是从 $1\sim n$ 逐个遍历，并让公交车走一圈回来再去下一个商场。先置答案为 $n+1$ 再减少之。

$t_i=0$ 的肯定可以直接带走。

记 $l_i=[t_i\le 2x_i],r_i=[t_i\le 2(L-x_i)]$，也可以理解为 $l$ 表示是否可以从右向左时进、左向右时出，$r$ 的含义类似。

特殊处理最靠右的商场，如果 $r_n=1$ 那么也可以在一个往返之内带回

剩余部分答案减少的形式为找到 $i<j$ 且 $l_i=r_j=1$，将 $i,j$ 配对减少绕行的一周。注意这里减少一周的过程在最后还是需要走出原点，这也就是特殊处理最后一个商场的原因。

对于 $l_i=1\ \land\ r_i=0$ 的商场可以**带来的一个信息**就是 $2x_i\le t_i\le 2(L-x_i)\Rightarrow x_i\le \dfrac{L}2$ ，那么它左边不存在 $l_i=0\ \land\ r_i=1$ 的商店。简而言之就是 $l_i\ \text{only}$ 右边不存在 $r_i\ \text{only}$，它们之间存在一条分界线

最靠左的 $l_i\ \text{only}$ 之前的 $l_i=r_i=1$ 和 $r_i\ \text{only}$ 配对，之后的 $l_i=r_i=1$ 和 $l_i\ \text{only}$ 配对，剩下的 $l_i=r_i=1$ 互相配对即可


# Code

```cpp
const int N=3e5+10;
int n,L,x[N],t[N],ans;
bool l[N],r[N];
signed main(){
	n=read(); L=read();
	for(int i=1;i<=n;++i) x[i]=read();
	for(int i=1;i<=n;++i){
		t[i]=read();
		ans+=t[i]/(2*L);
		t[i]%=2*L;
		if(!t[i]){
			--ans;
			continue;
		}
		l[i]=t[i]<=2*x[i];
		r[i]=t[i]<=2*(L-x[i]);
	}
	ans+=n+1-r[n];
	// n+1 -> last turn to bring the person back 
	int fix=n;
	int lcnt=0,rcnt=0;
	for(int i=1;i<n;++i){
		if(!l[i]&&!r[i]) continue;
		if(!r[i]){
			fix=i;
			break;
		}
		// bracket matches
		if(!l[i]&&lcnt) --lcnt,--ans;
		else if(l[i]) ++lcnt;
	}
	for(int i=n-1;i>=fix;--i){
		if(!l[i]&&!r[i]) continue;
		if(!l[i]) break;
		// bracket matches
		if(!r[i]&&rcnt) --rcnt,--ans;
		else if(r[i]) ++rcnt;
	}
	//lcnt,rcnt are both available choices
	ans-=(lcnt+rcnt)/2;
	print(2*L*ans);
	return 0;
}
```