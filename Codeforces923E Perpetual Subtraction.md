# Description

- 黑板上有一个整数 $x$，初始时它的值为 $[0,n]$ 中的某一个整数，其中为 $i$ 的概率是 $p_i$。
- 每一轮你可以将 $x$ 完全随机地替换成 $[0,x]$ 中的一个数。
- 求 $m$ 轮后，对于每个 $i\in[0,n]$，$x=i$ 的概率。
- $n \le 10^5$，$m \le 10^{18}$，答案对 $998244353$ 取模。

# Solution

设 $f_{z,i}$ 表示经过 $z$ 轮得到数字 $i$ 的概率，显然有 $\displaystyle f_{0,i}=p_i,f_{z,i}=\sum_{j=i}^nf_{z-1,j}\frac{1}{j+1}$

使用 $\rm OGF$ 来刻画这个过程，那么写出来 $F_z(x)$ 看一看

$$F_z(x)=\sum_{i=1}^nf_{z,i}x^{i}=\sum_{i=1}^n\left(\sum_{j=i}^nf_{z-1,j}\frac{1}{j+1}\right)x^i=\sum_{j=1}^nf_{z-1,j}\frac1{j+1}\frac{1-x^{j+1}}{1-x}$$

省略了一步等比数列求和，问题不大

注意到 $\displaystyle \frac{x^{j+1}}{j+1}=\int^{x}_0 k^j\ dk,\frac{1}{j+1}=\int^{1}_0 k^j\ dk$ 两者相减就是改变积分下界

$$F_{z}(x)=\frac1{x-1}\sum_{j=1}^nf_{z-1,j}\int_{1}^xk^j\ dk$$

代换 $G_z(x)=F_z(x+1)$ 来避免 $\frac{1}{x-1}$ 和非零下界，那么： 

$$\begin{aligned}G_{z}(x)&=\frac1{x}\int^{x+1}_1\sum_{j=1}^nf_{z-1,j}k^j\ dk\\ &=\frac1{x}\int_{0}^xF_{z-1}(k+1) \ dk=\frac1{x}\int_{0}^xG_{z-1}(k) \ dk\\ [x^i]G_{z}(x)&=\frac{1}{i+1}[x^i]G_{z-1}(x)\end{aligned}$$

那么如果得到了 $G_0$ 可以快速得到 $G_m$，使用二项式定理和二项式反演即可得到 $F_0\to G_0$ 以及 $G_m\to F_m$ 的变换，具体而言：

$$G_0(x)=\sum_{i=1}^nf_{0,i}(x+1)^i=\sum_{i=1}^n f_{0,i}\sum_{j=0}^i \binom ij x^j=\sum_{i=0}^n x^i\sum_{j=i}^i f_{0,j} \binom ji $$

$$F_m(x)=\sum_{i=0}^n x^i\sum_{j=i}^n(-1)^{j-i}\binom{j}ig_{m,j}$$

全部是减法卷积

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
#define rep(i,a,b) for(int i=a;i<=b;++i)
const int mod=998244353;
inline int add(int x,int y,int Mod=mod){return x+y>=Mod?x+y-Mod:x+y;}
inline int del(int x,int y,int Mod=mod){return x-y<0?x-y+Mod:x-y;}
inline int mul(int x,int y,int Mod=mod){return x*y-x*y/Mod*Mod;}
inline void ckadd(int &x,int y,int Mod=mod){x=x+y>=Mod?x+y-Mod:x+y;}
inline void ckdel(int &x,int y,int Mod=mod){x=x-y<0?x-y+Mod:x-y;}
inline void ckmul(int &x,int y,int Mod=mod){x=x*y-x*y/Mod*Mod;}
inline int ksm(int x,int y,int Mod=mod){int res=1; for(;y;y>>=1,ckmul(x,x,Mod)) if(y&1) ckmul(res,x,Mod); return res;}
inline void approx(int val,int Mod=mod,int lim=1e5){int x=val,y=Mod,a=1,b=0; while(x>lim){swap(x,y); swap(a,b); a-=x/y*b; x%=y;} cout<<x<<"/"<<a<<endl; return ;}
const int N=5e5+10;
#define poly vector<int> 
int r[N],W[N];
int fac[N],ifac[N],inv[N],p[N],n,m;
inline void NTT(vector<int> &f,int lim,int opt){
	f.resize(lim);
	for(int i=0;i<lim;++i){
		r[i]=r[i>>1]>>1|((i&1)?(lim>>1):0);
		if(i<r[i]) swap(f[i],f[r[i]]);
	}
	for(int p=2;p<=lim;p<<=1){
		int len=p>>1; W[0]=1; W[1]=ksm(3,(mod-1)/p);
		if(opt==-1) W[1]=ksm(W[1],mod-2);
		for(int j=2;j<len;++j) W[j]=mul(W[j-1],W[1]);
		for(int k=0;k<lim;k+=p){
			for(int l=k;l<k+len;++l){
				int tt=mul(f[l+len],W[l-k]);
				f[l+len]=del(f[l],tt);
				ckadd(f[l],tt);
			}
		}
	}
	if(opt==-1) for(int i=0,tmp=ksm(lim,mod-2);i<lim;++i) ckmul(f[i],tmp);
	return ;
}
inline poly Mul(poly a,poly b){
	int n=a.size(),m=b.size(),lim=1;
	while(lim<=(n+m)) lim<<=1;
	NTT(a,lim,1); NTT(b,lim,1);
	for(int i=0;i<lim;++i) ckmul(a[i],b[i]);
	NTT(a,lim,-1);
	a.resize(n+m-1);
	return a;
}
signed main(){
	n=5e5; fac[0]=inv[0]=1;
	for(int i=1;i<=n;++i) fac[i]=mul(fac[i-1],i);
	ifac[n]=ksm(fac[n],mod-2);
	for(int i=n;i>=1;--i) ifac[i-1]=mul(ifac[i],i),inv[i]=mul(ifac[i],fac[i-1]);
	scanf("%lld%lld",&n,&m);
	for(int i=0;i<=n;++i) scanf("%lld",&p[i]);
	poly A(n+1),B(n+1),G(n+1);
	for(int i=0;i<=n;i++) A[i]=ifac[i],B[n-i]=mul(fac[i],p[i]);
	poly C=Mul(A,B);
	for(int i=0;i<=n;i++) G[i]=mul(ksm(inv[i+1],m),mul(ifac[i],C[n-i]));
	A.clear(); B.clear();
	A.resize(n+1); B.resize(n+1);
	for(int i=0;i<=n;i++){
		if(!(i&1)) A[i]=ifac[i];
		else A[i]=del(0,ifac[i]);
		B[n-i]=mul(fac[i],G[i]);
	} 
	C=Mul(A,B);
	for(int i=0;i<=n;i++) printf("%lld ",mul(C[n-i],ifac[i]));
	putchar('\n');
	return 0;
}
```

本题另有知识门槛比较高的相似对角化相关做法，比较关键的一点是注意到上三角矩阵的特征值就是对角线上元素