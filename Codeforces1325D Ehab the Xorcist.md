# Description

[link](https://www.luogu.com.cn/problem/CF1325D)

给定整数 $u$ , $v$

求一个长度最小的数组 $a$

使得 $\sum a=v$ 同时 $xor\space a=u$

输出两行，第一行为长度，第二行为数组中的值

# Solution
 
首先考虑无解的情况，如果在比较高的位置上数的 $xor$ 为 $1$ 而和为 $0$ ，显然无解

所以 $u>v$ 时无解

然后对位考虑，如果二进制最后一位的值不一样，显然无解（没有可以进位啥的）

（这里就保证 $u$ 和 $v$ 的奇偶性相同了吧）

然后是 $u=v$ 的情形，这里特判一下 $u=0$ ，只要一行，剩下的输 $u$

首先一波操作：

令 $x=(v-u)>>1$

有一种方案是 $[x,x,u]$（挺神奇）

这是当下最小的情况

之后就是考虑两个数满足条件的构造了

这里有一个比较神的性质（可能是蒟蒻不知）

$a+b$ $=a \space xor \space b+2 \times(a \space and \space b)$

（证明分类讨论一下就可以）

我们发现如果满足$a+b=v$ 同时$a$ $xor$ $b=u$

则 $a$ $and$ $b=x$

再次对位考虑，如果$x$的某一位为$1$，那么 $a,b$ 同时为 $1$，$xor$为 $0$

否则对于 $xor$ 没有限制（$a,b$ $(1,0)$或$(0,0)$会有不同的$xor$结果）

所以我们看$u$ $and$ $x$ 的值

如果大于 $0$ 就不满足

否则就有一组解为 $[x,u+x]$

# Code

```cpp
signed main()
{
	int u=read(),v=read();
	if(u>v||(u&1)!=(v&1)) return puts("-1"),0;
	if(u==v){
		if(!u) puts("0");
		else printf("%lld\n%lld\n",1ll,u);
		return 0;
	}
	int x=(v-u)>>1;
	if(x&u) printf("3\n%lld %lld %lld",x,x,u);
	else printf("2\n%lld %lld\n",x,x^u);
	return 0;
}
```