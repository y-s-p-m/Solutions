# Description

现有一个长度为 $n$ 的非负整数数组 $\{a_i\}$ 。小 L 定义了一种神奇变换：
$$f_k=\left(\sum_{i=1}^na_i^k\right)\bmod 998244353$$
小 L 计划用变换生成的序列 $f$ 做一些有趣的事情，但是他并不擅长算乘法，所以来找你帮忙，希望你能帮他尽快计算出 $f_{1\dots n}$。

保证输入的 $a_i$ 满足 $0\le a_i\le10^9$。在一个测试文件中，保证有 $\sum n\le 4\times 10^5$。

# Solution

很久没写多项式题了，所以这题用来写写板子。

题其实没啥难的，乍一看是求若干 $f_k$ 的异或和，但是你需要的信息还是网格状的，所以你变换一下考量的维度，就能发现每个 $i$ 你需要所有的次幂和。

等比数列求和，得到一个封闭，封闭形式展开，你就又得到一行对答案的贡献

再看看发现你把这个封闭都加起来再一块展开就行了，突出一个并行计算。那么你发现到现在你需要通分一下。

最最直观的想法是对于每个 $i$ 求 $\displaystyle [x^i]\sum_{i=1}^n \frac{1}{1-a_ix}$，我拿我的板子写了一下这个分治发现死活过不去。

点开洛谷题解发现还有另外一种刻画的方法，就是你求 $[x^{i-1}]F(x)$ ，也就是把系数往小平移一位。这时候你表达一下通分的式子就是

$$\dfrac{\sum_{i=1}^n a_i\prod_{j\neq i}1-a_jx}{\prod_{i=1}^n(1-a_ix)}$$

他们说分母求导之后取负就是分母，我也觉得很神奇，不知道是不是我的表达式方式有问题导致把这个错过了。这样你省略了分治乘法中维护分子的部分，常数削减至原来的不多于四成，可以通过。

upd 我纯shaber。我的那个求导也是分子好像

# Code

因为没有缺省源了，所以放完整代码。

```cpp
#include<bits/stdc++.h>
#define fir first
#define sec second
#define rep(i,a,b) for(int i=a;i<=b;++i)
#define vi vector<int> 
using namespace std;
template<typename T>inline void ckmax(T &x,T y){x=x<y?y:x;}
template<typename T>inline void ckmin(T &x,T y){x=x>y?y:x;}
template<typename T=int>inline T read(){
	T res=0,f=1; char k;
	while(!isdigit(k=getchar())) if(k=='-') f=-1;
	while(isdigit(k)) res=res*10+k-'0',k=getchar();
	return res*f;
}
template<typename T>inline void print(T x,bool fl=1){
	if(x<0) putchar('-'),x=-x; 
	if(x>=10) print(x/10,0);
	putchar(x%10+'0');
	if(fl) putchar('\n');
}
const int N=1e6+10;
const int mod=998244353;
inline int add(int x,int y,int Mod=mod){return x+y>=Mod?x+y-Mod:x+y;}
inline int del(int x,int y,int Mod=mod){return x-y<0?x-y+Mod:x-y;}
inline int mul(int x,int y,int Mod=mod){return 1ll*x*y-1ll*x*y/Mod*Mod;}
inline void ckadd(int &x,int y,int Mod=mod){x=x+y>=Mod?x+y-Mod:x+y;}
inline void ckdel(int &x,int y,int Mod=mod){x=x-y<0?x-y+Mod:x-y;}
inline void ckmul(int &x,int y,int Mod=mod){x=1ll*x*y-1ll*x*y/Mod*Mod;}
inline int ksm(int x,int y,int Mod=mod){int res=1; for(;y;y>>=1,ckmul(x,x,Mod)) if(y&1) ckmul(res,x,Mod); return res;}
int r[N],W[N],inv[N],n,a[N];
inline void NTT(vi &f,int lim,int opt){
	f.resize(lim);
	rep(i,0,lim-1){
		r[i]=r[i>>1]>>1|((i&1)?(lim>>1):0);
		if(i<r[i]) swap(f[i],f[r[i]]);
	}
	for(int p=2;p<=lim;p<<=1){
		int len=p>>1;
		W[0]=1; W[1]=ksm(3,(mod-1)/p);
		if(opt==-1) W[1]=ksm(W[1],mod-2);
		rep(j,2,len-1) W[j]=mul(W[j-1],W[1]);
		for(int k=0;k<lim;k+=p){
			for(int l=k;l<k+len;++l){
				int tt=mul(f[l+len],W[l-k]);
				f[l+len]=del(f[l],tt);
				ckadd(f[l],tt);
			}
		}
	}
	if(opt==-1){
		int invl=ksm(lim,mod-2);
		rep(i,0,lim-1) ckmul(f[i],invl);
	}
}
inline vi Mul(vi A,vi B){
	int n=A.size(),m=B.size();
	int lim=1;
	while(lim<=(n+m)) lim<<=1;
	NTT(A,lim,1);
	NTT(B,lim,1);
	rep(i,0,lim-1) ckmul(A[i],B[i]);
	NTT(A,lim,-1);
	A.resize(n+m-1);
	return A;
}
inline vi Plus(vi A,vi B){
	int n=A.size(),m=B.size();
	if(n<m) swap(n,m),swap(A,B);
	for(int i=0;i<m;++i) ckadd(A[i],B[i]);
	return A;
}
inline vi conquer(int l,int r){
	if(l==r) return {1,del(0,a[l])};
	int mid=(l+r)>>1;
	return Mul(conquer(l,mid),conquer(mid+1,r));
}
inline vi Inv(vector<int> f,int len){
	if(len==1) return {ksm(f[0],mod-2)};
	vector<int> G0=Inv(f,(len+1)>>1);
	int lim=1; while(lim<=(len<<1)) lim<<=1;
	vector<int> tmp,G;
	rep(i,0,len-1) tmp.push_back(f[i]);
	NTT(tmp,lim,1); NTT(G0,lim,1);
	rep(i,0,lim-1) G.push_back(mul(G0[i],del(2,mul(G0[i],tmp[i]))));
	NTT(G,lim,-1); G.resize(len);
	return G;
}

signed main(){
	int T=read();
	while(T--){
		n=read();
		for(int i=1;i<=n;++i) a[i]=read();
		vi zi,mu=conquer(1,n);
		zi.resize(mu.size());
		for(int i=0;i<zi.size()-1;++i) zi[i]=del(0,mul(mu[i+1],i+1));
		int ans=0;
		zi=Mul(zi,Inv(mu,mu.size()));
		for(int i=0;i<n;++i) ans^=zi[i];
		print(ans);
	}
	return 0;
}
```