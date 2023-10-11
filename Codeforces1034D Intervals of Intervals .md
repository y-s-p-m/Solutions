# Description

有 $n\le 3\times 10^5$ 个区间 $[a_i,b_i]$

定义区间的区间 $[l,r]$ 的价值是第 $l$ 个区间到第 $r$ 个区间的并的长度，找出 $k$ 个不同的区间的区间，使得总价值最大。

# Solution

二分第 $K$ 大并区间的并的长度，由于 并集大小 在区间个数增加的同时是不降的，可以双指针求出来每个右端点的最远左端点

此时需要快速求解一组区间的并的长度，发现上面过程和扫描线是吻合的，那么使用数据结构维护每个元素最后一次出现的位置，如果求 $[l,r]$ 的区间并集大小就是在数据结构上求有几个元素的最后一次出现 $\ge l$

乍一看甚至需要树套树了，但是转变成维护每个左端点的答案 $ans_l$，配合 `std::set` 维护所有区间的最后一次出现位置 $lst_i$，此时每次修改一段的 $lst_i$ 就是对 $ans$ 进行区间加

那么挪动指针是单点查询，这是平凡的

复杂度现在是 $\Theta(n\log V\log^2n)$ 的，需要找到冗余减少计算量

首先区间加法的形式在每次二分中是一样的，可以提前处理；其次区间加法可以差分，如果当前指针比差分开始的位置大就在指针的位置加权值而在 $i+1$ 处将增量撤销即可

最后求答案需要解决形如 “权值 $\ge x$ 的区间的权值之和” 这样一个问题，仍然套用求个数的过程，维护当前所有合法左端点的区间并之和

挪动指针时添加差分数组上的值，而在出现指针大于区间加的起始点的情况需要补加左端点个数个增量

总复杂度 $\Theta(n\log V)$

# Code

```cpp
const int N=3e5+10;
int n,K,l[N],r[N],val[N];
vector<pair<int,int> > incr[N];
set<tuple<int,int,int> > inter;
signed main(){
	n=read(); K=read();
	rep(i,1,n){
		int l=read(),r=read()-1;
		auto split=[&](const int x){
			auto iter=inter.lower_bound({x,0,0});
			if(iter!=inter.begin()){
				--iter;
				int l,r,id; 
				tie(l,r,id)=*iter;
				if(r>=x){
					inter.erase(iter);
					inter.insert({l,x-1,id});
					inter.insert({x,r,id});
				}
			}	
			return ;
		};
		split(l); split(r+1);
		auto iter=inter.lower_bound({l,0,0});
		int sumlen=0;
		while(1){
			if(iter==inter.end()) break;
			int L,R,id; tie(L,R,id)=*iter;
			if(L>r) break;
			incr[i].emplace_back(id+1,R-L+1);
			++iter;
			sumlen+=R-L+1;
			inter.erase(prev(iter));
		}
		if(sumlen<r-l+1){
			incr[i].emplace_back(1,r-l+1-sumlen);
		}
		inter.insert({l,r,i});
	}
	auto Num=[&](const int mid){
		int cnt=0,indic=0;
		rep(i,1,n) val[i]=0;
		for(int i=1;i<=n;++i){
			for(auto e:incr[i]){
				val[max(e.fir,indic)]+=e.sec;
				val[i+1]-=e.sec;
			}
			while(indic+1<=i&&val[indic+1]+val[indic]>=mid){
				++indic;
				val[indic]+=val[indic-1];
			}
			cnt+=indic;
		}
		return cnt;
	};
	auto Sum=[&](const int mid){
		int sum=0,cur=0,indic=0;
		rep(i,1,n) val[i]=0;
		for(int i=1;i<=n;++i){
			for(auto e:incr[i]){
				if(e.fir<=indic) cur+=e.sec*(indic-e.fir+1);
				val[max(e.fir,indic)]+=e.sec;
				val[i+1]-=e.sec;
			}
			while(indic+1<=i&&val[indic+1]+val[indic]>=mid){
				++indic;
				val[indic]+=val[indic-1];
				cur+=val[indic];
			}
			sum+=cur;
		}
		return sum;
	};
	int ans=0,l=0,r=1e9;
	while(l<=r){
		int mid=(l+r)>>1;
		if(Num(mid)>=K) ans=mid,l=mid+1;
		else r=mid-1;
	}
	print(Sum(ans+1)+(K-Num(ans+1))*ans);
	return 0;
}
```

题外话：这题注意到差分的左端点可以随便调花了非常非常久的时间