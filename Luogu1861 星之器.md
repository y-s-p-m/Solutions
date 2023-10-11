# Description

[link](https://www.luogu.com.cn/problem/P1861)

现在有一个 $n \times m$ 的矩阵$A$，里面的每个元素$a_{i,j}$ 表示二元组$(i,j)$的位置有$a_{i,j}$ 颗星星

现在我们有一种操作，选定同一行或同一列的两个星组 $a$ ，把他们中的一颗星星向中间移动一个单位，该操作的贡献是两个位置的曼哈顿距离（意会一下，相当简单）

给定初始矩阵 $A$, 和末尾矩阵 $B$,保证 $B$ 是由 $A$ 进行一定的上述操作得到,求贡献和

# Solution

单看题目一脸懵逼……

直接上思路吧：（真的是闻所未闻的人类智慧）
 
定义一颗位于的二元组$(i,j)$的星星的 **“势能”** 为 $\frac{i^2+j^2}{2}$

我们考虑每一个移动对于两个位置的星星的势能的影响

$$E_0=\frac{x^2_1+y_1^2+x_2^2+y_2^2}{2}$$

$$E=\frac{(x_1+1)^2+y_1^2+(x_2-1)^2+y_2^2}{2}$$

$$\Delta E=x_2 \space - \space x_1$$

我们要的贡献就是$\Delta E$

然后我们发现这个移动跟操作的方式是无关的！！！！

所以这个题就做完了

$$ans=\sum^{n}_ {i=1}\sum^{m}_ {j=1} \frac{a_{i,j}* (i^2+j^2)}{2}-\sum^{n}_ {i=1}\sum^{m}_ {j=1} \frac{b_{i,j}* (i^2+j^2)}{2}$$

可以乘法分配律一下啥的

# Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
namespace yspm{
	inline int read()
	{
		int res=0,f=1; char k;
		while(!isdigit(k=getchar())) if(k=='-') f=-1;
		while(isdigit(k)) res=res*10+k-'0',k=getchar();
		return res*f;
	}
	const int N=210;
	int a[N][N],b[N][N],n,ans,m;
	signed main()
	{
		n=read(); m=read(); 
		for(int i=1;i<=n;++i) for(int j=1;j<=m;++j) a[i][j]=read(); 
		for(int i=1;i<=n;++i) for(int j=1;j<=m;++j) b[i][j]=read();
		for(int i=1;i<=n;++i) for(int j=1;j<=m;++j) ans+=(a[i][j]-b[i][j])*(i*i+j*j); 
		printf("%lld\n",ans>>1);
		return 0;
	}
}
signed main(){return yspm::main();}
```