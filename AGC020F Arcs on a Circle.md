# Description

你有一个长度为$C$的圆，你在上面画了$N$个弧。弧$i$有长度$l_i$。

每一条弧$i$随机均匀地放置在圆上：选择圆上的一个随机实点，然后出现一条以该点为中心的长度为$l_i$的弧。弧是独立放置的。例如，它们可以相互交叉或包含。

现在问你圆的每一个实点被至少一个弧覆盖的概率是多少？注意一条弧覆盖了它的两个端点。

$n\le 6,c\le 50$

# Solution

$n$ 的数量级小的时候变数就多了

一个常用的思想是将最长的一个圆弧的起点视为原点来避免跨原点的状态记录，而剩余的 $n-1$ 个点的小数部分一定不是 $0$ 且互不相同

现在的困境就在于判定圆弧相交，那么可以 **枚举小数部分大小关系**

那么将 $n$ 个左右端点确定成成 $n\times C$ 个，不同的整数部分对应 $n$ 个不同的小数部分

由于枚举了小数部分的大小关系，那么每个元素有确定的 $C$ 个左端点可以使用

状压 $\rm DP$ 来计算有多少种方案最终覆盖了整个圆即可：设 $f_{i,S}$ 表示前 $i$ 个单位的圆弧完成了覆盖，使用了 $S$ 集合中的元素的方案数

按照左端点的顺序从小到大转移，每次添加左端点对应的元素，并将第一维根据添加的线段进行更新即可

注意新的第一维对 $nC$ 取 $\min$


# Code

```cpp
int n,c;
int f[310][70],l[20],id[20];
signed main(){
	n=read(); c=read();
	rep(i,1,n) l[i]=read();
	sort(l+1,l+n+1);
	rep(i,1,n-1) id[i]=i;
	double sum=0;
	int U=1<<(n-1); --U;
	do{
		memset(f,0,sizeof(f));
		f[l[n]*n][0]=1;
		for(int i=1;i<=n*c;++i) if(i%n){
			int now=id[i%n];
			for(int j=i;j<=n*c;++j){
				for(int S=0;S<=U;++S) if(!(S>>(now-1)&1)){
					int pos=max(i+l[now]*n,j);
					ckmin(pos,n*c);
					f[pos][S|(1<<(now-1))]+=f[j][S];
				}
			}
		}
		sum+=f[c*n][U];
	}while(next_permutation(id+1,id+n));
	rep(i,1,n-1) sum/=c*i;
	printf("%.14lf\n",sum);
	return 0;
}
```