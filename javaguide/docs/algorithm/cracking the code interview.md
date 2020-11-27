## 01

### 01.01. 判定字符是否唯一
一个是hashset直接
一个是把char排序，然后遍历
但是这两个都用到了库函数
好的做法是，利用ascii只有128个的限制
遍历数组，看是128个中的哪一个，并且做记录
这样能发现重复，并且，时间复杂度只有26n，也就是n
实现：
1. boolean 数组
2. 位运算。用两个long来装。
    每个char的值就是1左移char的int值位。
    判断有没有在，用&，没在的话放进去，用|
    
### 01.02. 判定是否互为字符重排
可以用128长度的int数组，来存放各个字符出现的次数。
然后做比较。

### 01.03. URL化
这题审题都审得有点够呛。
就是给了一个字符串，还有一个长度，这个长度自称是这个字符串的“真实”长度。
做的话可以先计算出整个的长度，然后从后往前放。


### 01.04. 回文排列
字符出现次数单数的只有1或0。

### 01.05. 一次编辑
从左往右遍历，比较current c1, current c2
出现相同就继续，
不同的话：
    插入：nxt c2要跟cur c1一样
    删除：cur c2要跟nxt c1一样
    替换：nxt c2要跟nxt c1不一样
这个思路的问题是，你只比较了前面的一样，没比较后面的一样。
所以按照长度的不同来分类会好一些。
长度相同的，就找到不同的那个，比较后面的是不是相同。
长度不同的，找到不同的那个，然后比较长的的后面，和短的的包含当前字符的后面。
对于边界。可以先做主支，再根据主支的条件去做边界。

### 01.06. 字符串压缩
直接遍历一遍。


### 01.07. 旋转矩阵
数学？

### 01.08. 零矩阵
可以用两个数组来存行和列的标识。
因为清过的行和列，就可以跳过不用再去处理了。

### 01.09. 字符串轮转
这个，如果s2是s1旋转的，那么s1一定出现在s2+s2里


### 03.01. 三合一
一个数组来实现三个栈？？怎么可能啊
。。题目看漏了，是还有栈的大小。

### 03.02. 栈的最小值
再维护一个最小值栈，放每次push进来当前的最小值。


### 03.03. 堆盘子
维护List of Stack.
pop永远拿list末尾的
要注意边界值，就是cap<=0

### 03.04. 化栈为队
队列，先进先出
就用两个栈来倒腾。
push就一直push，直到要pop了，就倒到第二个栈
就是第一个栈push，第二个栈pop，但是第二个栈必须是第一个栈倒过来的。


### 03.05. 栈排序
最小元素位于栈顶。
也就是维护一个单调递减的栈
要注意stck的pop和peek栈为空都是会抛异常的。


### 03.06. 动物收容所
用LinkedList, deque来做
注意dequecat的时候，用迭代器来
for(int[] animal : queue)
    queue.remove(animal);



## 动态规划

面试题 08.01. 三步问题
暴力法，深度优先，分别1，2，3，然后再递归算下面的
会有一个问题，重复的。
所以空间换时间
把结果都缓存起来，重复的就可以去查表。不用重复计算。
1有1种，2有2种，
n有dp[n-3]种+dp[n-2]+dp[n-1]种
============这里要注意两个问题，一个是初始值，一个是取模问题
初始值的话，我们拿3举例
3
2 1
1 2
1 1 1
如果是减掉当前步数是0的话，是要加一的，而不是0的，直接加上剪掉后的dp
所以要么把dp[0]设成1，要么就把1,2,3都先给出来。
反正就是把这个0的特殊操作给摘出去。
然后就是要取模。注意这个取模你得每步都取模，因为你不知道什么时候大于



面试题 08.11. 硬币
我的天啊这题有一个重复的问题
比如说n=6
1 11111
1 5
5 1
======后面两个就会重复
要去重的话，就一定会涉及到排序。
所以要有大小的限制。比如90，就得决定用25，然后1，2，3个分别和减掉的以及只用比25小的硬币排出来的总数
设f(i,n)表示的是最大包含（可以有，也可以没有）cents[i]的硬币，来组成总数为n的种数
所以f(i,n) = sum{ f(i-1, n-k(i-1)) }, k = 0, 1, 2, ...
Res = sum{f(k,n)} ,k遍历所有硬币

   1 2 3 4  //用前几个硬币
0  1 1 1 1 
1  1
2  1
3  1
4  1
5
6
7
8
9
10
11
//组成总数几分

代码：
```
class Solution {
    public int waysToChange(int n) {
        int M = 1000000007;
        int[] cents = new int[]{1, 5, 10, 25};
        int dp[][]= new int[cents.length][n+1];
        for(int i = 0; i < cents.length; i++){
            dp[i][0] = 1;
        }
        for(int i = 0; i <=n; i++){
            dp[0][i] = 1;
        }
        for(int total = 1; total <= n; total++){
            for(int centIdx = 1; centIdx < cents.length; centIdx++){
                int k = 0, cent = cents[centIdx];
                while(total - k * cent >= 0){
                    int restTotal = total - k * cent;
                    dp[centIdx][total] = (dp[centIdx][total] + dp[centIdx-1][restTotal]) % M;
                    k++;
                }
            }
        }
        return dp[cents.length-1][n];
    }
}

```
==============动态规划就是
你把dp公式给写好后，后面的你就照着格子慢慢算
是一个决策和执行的过程。慢慢算的时候你就不用去想问什么。
在写递归公式的时候想好就okay了
===============
这样的思路是朴素的动态规划。
会time exceed limitation
因为时间复杂度应该是n^3方
所以要想哪里有可以优化的部分。
=================
感觉好像没有可优化的部分。
哦有的！！
dp[centIdx][total] =
    dp[centIdx-1][total-0*cents[centIdx]]
    dp[centIdx-1][total-1*cents[centIdx]]
    dp[centIdx-1][total-2*cents[centIdx]]
    dp[centIdx-1][total-3*cents[centIdx]]
    dp[centIdx-1][total-...*cents[centIdx]]
    dp[centIdx-1][total-k*cents[centIdx]]            
然后你可以看到其实后面那部分，就相当于dp[centIdx][total-1*cents[centIdx]]的值
所以  
dp[centIdx][total] =
    dp[centIdx-1][total-0*cents[centIdx]] +
    dp[centIdx][total-1*cents[centIdx]]
从一个high level的逻辑上来理解，就是
dp[centIdx][total]等于
不用当前的centIdx硬币拼出total 加上
当前至少出一块centIdx的硬币，然后再去看剩下的total-cents[centIdx]用当前的所有硬币有几种拼法。
我觉得这个其实是，因为要搞顺序，所以一定要有一个大小的限制。也就是说，答案一定要排序并且不一样
所以这里，它是通过强制性地让centIdx参与一次，这样来达到一个区分的效果，来做去重。
但我觉得还是不太容易能够理解。

```
class Solution {
    static final int MOD = 1000000007;
    int[] coins = {25, 10, 5, 1};

    public int waysToChange(int n) {
        int[] f = new int[n + 1];
        f[0] = 1;
        for (int c = 0; c < 4; ++c) {
            int coin = coins[c];
            for (int i = coin; i <= n; ++i) {
                f[i] = (f[i] + f[i - coin]) % MOD;
            }
        }
        return f[n];
    }
}

```


































