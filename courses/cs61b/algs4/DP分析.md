>可以使用dp解决的问题的一般特征是：对于该问题，问题的解法很多，但是需要找到一个 `最大/最小/最优化某种属性` 的一个解，即从有限的解集中寻找有某种特性的那个解

## [01背包问题](https://www.acwing.com/problem/content/2/)
### 1.化整为零(状态表示) 
考虑dp数组中的每一个位置表示的原问题的什么子集、dp数组每个位置的值表示子集的什么属性、如何划分子集

### 2.化零为整(状态计算/状态转移)
要获得当前计算位置的dp值()，则需关注最后一个不同点
`dp[i][j]` 的计算可由两个子集得到，划分子集的依据是最后一个不同点，即选或不选第i个物品
由此分出两个子集
1. 不选第i个物品，则该子集的大小为dp[i-1][j]，即在j容量下只考虑前i-1个物品可选的最大价值
2. 选第i个物品，则必须在容量j中预留出第i个物品的容量大小w，然后考虑在剩下的容量下只选前i-1个物品可得到的最大价值，即为 `dp[i-1][j-w]` 

得到子集最大价值后即可计算当前位置最大价值 `dp[i][j] = max(dp[i-1][j], dp[i-1][j-w]+v)` (v为第i个物品的价值， w为第i个物品的容量)

```CPP
#include<iostream>
using namespace std;
int N,V;
int vo[1005];
int val[1005];
int dp[1005][1005];
int main(){
    cin>>N>>V;
    for(int i=1;i<=N;++i) cin>>vo[i]>>val[i];
    for(int i=1;i<=N;++i){
        for(int j=1;j<=V;++j)
            if(j<vo[i])dp[i][j] = dp[i-1][j];
            else dp[i][j] = max(dp[i-1][j],dp[i-1][j-vo[i]]+val[i]);
        }
    cout<<dp[N][V]<<endl;
    return 0;
}
```



## [完全背包问题](https://www.acwing.com/problem/content/3/)
### 1.化整为零(状态表示)
`dp[i][j]` 表示的是只考虑前i个物品(个数任选)，在容量为j的情况下可达到的最大价值

### 2.化零为整(状态计算/状态转移)
同样的，关注最后一个不同点，在完全背包中 `最后一个不同点` 指的是选择当前位置的物品的个数(对于 `dp[i][j]` ，指的是选择第i个物品的个数)
完全背包物品个数最大可取无穷，即可将当前的状态对应的集合分为无穷个子集，分别对应取0个 `物品i` 、取1个 `物品i` ...取k个 `物品i` ...
从而 `dp[i][j] = max(dp[i-1][j],dp[i-1][j-w]+v,dp[i-1][j-2*w]+2*v,...)` 
+ 可以注意到 `dp[i][j-w] = max(dp[i-1][j-w],dp[i-1][j-2*w]+v,dp[i-1][j-3*w]+2*v,...)` 
+ 从而 `dp[i][j] = max(dp[i-1][j],dp[i][j-w]+v)` (v为第i个物品的价值， w为第i个物品的容量)

```CPP
#include<iostream>
using namespace std;
int N,V;
int vo[1005];
int val[1005];
int dp[1005][1005];
int main(){
    cin>>N>>V;
    for(int i=1;i<=N;++i) cin>>vo[i]>>val[i];
    for(int i=1;i<=N;++i)
        for(int j=1;j<=V;++j){
            if(j<vo[i])dp[i][j] = dp[i-1][j];
            else dp[i][j] = max(dp[i-1][j],dp[i][j-vo[i]]+val[i]);
        }
        
    cout<<dp[N][V]<<endl;
}
```

## 区间DP问题
### 1.化整为零(状态表示)
以合并石子为例 `dp[i][j]` 表示合并区间[i, j]的数堆石子需要的最小代价

### 2.化零为整(状态计算/状态转移)
关注最后一个不同点，合并区间[i, j]的石子，可以从中任选一堆作为分界线将区间内的石子分为两大堆进行合并，即 `dp[[i][j] = max(dp[i][i]+dp[i+1][j]+S[j]-S[i-1],...,dp[i][k]+dp[k+1][j]+S[j]-S[i-1],...)` S[]为前缀和数组

## 最长公共子序列问题
### 1.化整为零(状态表示)
’dp[i][j]‘ 表示A[1]~A[i]与B[1]~B[j]的最大公共子序列长度

### 2.化零为整(状态计算/状态转移)
关注最后一个不同点，对于两个序列A[1]~A[i]、B[1]~B[j]，是否选择最后一个元素共有4种可能
即 `00 01 10 11` ，由此可将 `dp[i][j]` 划分为4个子集
+ 对于 00 A[i]和B[j]都不选，则为 `dp[i-1][j-1]`
+ 对于 01 选择B[j]而不选A[i]，则为 `dp[i-1][j]` 
+ 对于 10 选择A[i]而不选B[j]，则为 `dp[i][j-1]`
+ 对于 11 A[i]==B[j]时为 `dp[i-1][j-1]+1`，A[i]!=B[j]时为 `dp[i-1][j-1]`

此时 00 完全包含于 01 或 10
当A[i]==B[j]时 `dp[i][j] = max(dp[i-1][j],dp[i][j-1],dp[i-1][j-1]+1)` 
当A[i]!=B[j]时 `dp[i][j] = max(dp[i-1][j],dp[i][j-1])`
```CPP
#include<iostream>
using namespace std;
int n,m;
char a[1005], b[1005];
int dp[1005][1005];
int main(){
    cin>>n>>m>>a+1>>b+1;
    for(int i=1;i<=n;++i)
        for(int j=1;j<=m;++j){
            dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
            if(a[i]==b[j])dp[i][j] = max(dp[i][j], dp[i-1][j-1]+1);
        }
    cout<<dp[n][m]<<endl;
}
```