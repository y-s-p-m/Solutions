# Description

有一个 $2^k\times 2^k$ 的矩阵 $A$，行列从 $0$ 开始标号，有两个长度为 $t$ 的序列 $x,y$。

你需要通过若干次操作使得整个矩阵 $A$ 全为 $0$。

一次操作需要三个参数 $(p,q,w)$，会给所有 $A[(x_i+p)\bmod 2^k][(y_i+q)\bmod 2^k]$ 异或上 $w$。

求最小的操作次数。

$k\le 9,t\le 99$ 且 $t$ 为奇数。

# Solution

设矩阵 $F$ 满足在所有 $(x_i,y_i)$ 处值为 $1$ 而其它位置为 $0$

考察一下题目中给出的操作的形式和**二维循环卷积**是类似的，那么可以定义一种新的乘法：

$$F\times G=H\Leftrightarrow H_{i,j}=\oplus_{a,b}\oplus_{c,d} [a+b\equiv i \bmod 2^k][c+d\equiv j \bmod 2^k]F_{a,c}\times G_{b,d}$$

此时问题变成了找到一个矩阵 $G$ 满足 $F\times G=A$，其中 $A$ 是题目中给定的矩阵

观察到上面定义的 $F$ 矩阵在进行**乘方运算时在异或运算的配合下**有非常好的性质：除了自乘之外所有乘法结果会被 $\oplus$ 两次，也就是说 $F^{2^i}$ 满足在且仅在 $F[2^ix\% 2^{k}][2^iy\% 2^{k}]$ 处为 $1$

那么不难发现 $F^{-1}=F^{2^k-1}$ ,暴力进行乘法即可

注意到 $F^{2^i}$ 中只有 $t$ 个元素有值，复杂度就是 $\Theta(2^{2k}kt)$

# Code

```cpp
const int N=512;
int k,n,m,a[N][N],f[2][N][N];
int x[N],y[N];
signed main(){
	k=read(); n=(1<<k);
	int cur=0;	
	for(int i=0;i<n;++i) for(int j=0;j<n;++j) f[cur][i][j]=read();
	m=read();
	rep(i,1,m) x[i]=read()-1,y[i]=read()-1;
	for(int e=0;e<k;++e){
		for(int i=0;i<n;++i) for(int j=0;j<n;++j) f[cur^1][i][j]=0;
		for(int t=1;t<=m;++t){
			for(int i=0;i<n;++i){
				for(int j=0;j<n;++j){
					f[cur^1][(i+x[t])&(n-1)][(j+y[t])&(n-1)]^=f[cur][i][j];
				}
			}
		}
		rep(i,1,m) x[i]=(x[i]*2)&(n-1),y[i]=(y[i]*2)&(n-1);
		cur^=1;
	}
	int ans=0;
	for(int i=0;i<n;++i) for(int j=0;j<n;++j) if(f[cur][i][j]) ++ans;
	print(ans);
	return 0;
}
```