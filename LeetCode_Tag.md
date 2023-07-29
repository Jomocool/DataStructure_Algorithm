## 数组

### Easy

#### [414. 第三大的数](https://leetcode.cn/problems/third-maximum-number/)

```cpp
方法一：一次遍历
class Solution {
public:
    int thirdMax(vector<int>& nums) {
        //不能用INT_MIN的原因是nums[i]可能等于INT_MIN，最后判断时无法确定是最初赋值的INT_MIN，还是数组里的元素
        long firstMaxNum=LONG_MIN;//第一大的值
        long secondMaxNum=LONG_MIN;//第二大的值
        long thirdMaxNum=LONG_MIN;//第三大的值

        //一定是前三名之外的值才可以改变前三名的名次，所以对于n的值需要有别于前三大的值
		//因此，n不仅要大于超过的那个值，还必须小于未超过的值，如果等于这二者的其中之一，名次不会有任何变化，即不会进入任何一个if分支
        for(auto &n:nums){
            if(n>firstMaxNum){
                //跑步比赛中，前三名之外的选手超过了第一名，第一名变成了第二名，第二名变成了第三名，且只有后面的选手会受影响
                //赋值顺序不能修改
                thirdMaxNum=secondMaxNum;
                secondMaxNum=firstMaxNum;
                firstMaxNum=n;
            }
            //n未超过第一名，但超过了第二名，此时n成为第二名，第二名成为第三名，因为如果此时n就是第一名(即有相同值)，名次不应该改变
            else if(firstMaxNum>n&&n>secondMaxNum){
                thirdMaxNum=secondMaxNum;
                secondMaxNum=n;
            }
            //n未超过第二名，但超过了第三名，此时n成为第三名
            else if(secondMaxNum>n&&n>thirdMaxNum){
                thirdMaxNum=n;
            }
        }

        //如果不存在第三大的值，就返回最大值
        return thirdMaxNum==LONG_MIN?firstMaxNum:thirdMaxNum;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);

方法二：有序集合
set是一个有序容器，元素按升序排列

class Solution {
public:
    int thirdMax(vector<int>& nums) {
        set<int>s;

        for(auto &n:nums){
            s.insert(n);
            if(s.size()>3){//因为只需要前三大的数，所以有序集合只需保存三个值即可
                s.erase(s.begin());
            }
        }
		
        //set会自己查重，如果s的大小小于3，说明没有第三大的数，返回最大值
        //begin是指向最小元素的迭代器，rbegin是指向最大元素的迭代器(end左移一个的迭代器)
        return s.size()==3?*s.begin():*s.rbegin();
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [605. 种花问题](https://leetcode.cn/problems/can-place-flowers/)

```cpp
class Solution {
public:
    bool canPlaceFlowers(vector<int>& flowerbed, int n) {
        //给flowerbed前后各添加一个地块，简洁化代码，且不会影响种植
        flowerbed.insert(flowerbed.begin(),0);
        flowerbed.push_back(0);
        int size=flowerbed.size();

        int i=1;
        while(i<=size-2){
            //当前地块已经种了花，相邻地块就不能种花了，所以跳转到下下个地块
            if(flowerbed[i]==1)i+=2;
            else{//当前地块没种花，如果相邻地块也没有花，则可以种下一朵花
                if(flowerbed[i-1]==0&&flowerbed[i+1]==0){
                    n--;
                    i+=2;
                }else{
                    i++;
                }
            }
        }

        //n<=0说明可以种下n朵花
        return n<=0;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [628. 三个数的最大乘积](https://leetcode.cn/problems/maximum-product-of-three-numbers/)

```cpp
方法一：排序
先将数组排序，排完序后，最大值要么是最后三个最大值乘积，要么是前两个最小值（负数）相乘后再和最大值相乘的乘积，且必须选两个负数，偶数个负数相乘才能得到正数，这些绝对值最大的数相乘后才有可能得到最大值乘积。关键思路就是，让三个绝对值最大的去相乘，看谁的乘积更大，如果没有负数，那必然就是最后三个最大值相乘的乘积最大，只有一个负数的情况也是如此，大于等于两个负数的情况下比较才有悬念

class Solution {
public:
    int maximumProduct(vector<int>& nums) {
        sort(nums.begin(),nums.end());
        
        int n=nums.size();
        int res1=nums[n-3]*nums[n-2]*nums[n-1];
        int res2=nums[n-1]*nums[0]*nums[1];

        return max(res1,res2);
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(logn)  主要是快排的额外空间开销;

方法二：线性扫描
从上面的方法可以看出，其实我们只需要知道前三个最大的数，而前两个最小的数，用五个数记录即可

class Solution {
public:
    int maximumProduct(vector<int>& nums) {
        int firstMaxVal=INT_MIN;
        int secondMaxVal=INT_MIN;
        int thirdMaxVal=INT_MIN;
        int firstMinVal=INT_MAX;
        int secondMinVal=INT_MAX;

        //即使是相同的值也可以重复利用
		/*
		因此，当n=1000，fitstMaxVal=1000，secondMaxVal=thirdMaxVal=INT_MIN时，此时意味着有两个1000，这两个1000都可以用到乘积
        中，所以secondMaxVal=1000
        */
        //最小值也同理
        for(auto&n:nums){
            if(n>firstMaxVal){
                thirdMaxVal=secondMaxVal;
                secondMaxVal=firstMaxVal;
                firstMaxVal=n;
            }else if(n>secondMaxVal){
                thirdMaxVal=secondMaxVal;
                secondMaxVal=n;
            }else if(n>thirdMaxVal){
                thirdMaxVal=n;
            }

            if(n<firstMinVal){
                secondMinVal=firstMinVal;
                firstMinVal=n;
            }else if(n<secondMinVal){
                secondMinVal=n;
            }
        }

        int res1=firstMaxVal*secondMaxVal*thirdMaxVal;
        int res2=firstMaxVal*firstMinVal*secondMinVal;

        return max(res1,res2);
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [643. 子数组最大平均数 I](https://leetcode.cn/problems/maximum-average-subarray-i/)

```cpp
方法一：滑动窗口

class Solution {
public:
    double findMaxAverage(vector<int>& nums, int k) {
        double sum=0;
        double maxSum=INT_MIN;//窗口值最大和

        for(int i=0;i<k;i++){
            sum+=nums[i];
        }

        maxSum=sum;

        for(int right=k;right<nums.size();right++){
            sum-=nums[right-k];
            sum+=nums[right];
            maxSum=max(maxSum,sum);
        }

        return maxSum/k;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [674. 最长连续递增序列](https://leetcode.cn/problems/longest-continuous-increasing-subsequence/)

```cpp
方法一：一次遍历

class Solution {
public:
    int findLengthOfLCIS(vector<int>& nums) {
        int maxLen=INT_MIN;
        int start=0;//每个递增序列的起始下标
        while(start<nums.size()){
            int index=start;
            //遍历递增序列
            while(index+1<nums.size()&&nums[index]<nums[index+1]){
                index++;
            }
            maxLen=max(maxLen,index-start+1);//计算最大长度
            //由于是连续递增序列，只要出现了nums[index]<nums[index+1]，前面的连续递增序列就被断开了，新的起点就是index+1
            start=index+1;
        }

        return maxLen;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [697. 数组的度](https://leetcode.cn/problems/degree-of-an-array/)

```cpp
方法一：哈希表
题目翻译过来实际上就是，找到包含所有（数组中出现频次最高的数）的最短连续子数组，比如说出现频次最高的数是2，那就找到包含所有2的最短连续子数组

class Solution {
public:
    int findShortestSubArray(vector<int>& nums) {
        unordered_map<int,int>counts;
        for(int i=0;i<nums.size();i++){
            counts[nums[i]]++;
        }

        int maxCount=0;//度

        for(const auto&pair:counts){
            maxCount=max(maxCount,pair.second);
        }

        int minLen=INT_MAX;
        for(const auto&pair:counts){//可以有多个满足maxCount频次的元素
            if(pair.second==maxCount){
                int maxCountNum=pair.first;
                int left=0;
                int right=nums.size()-1;
                while(nums[left]!=maxCountNum)left++;
                while(nums[right]!=maxCountNum)right--;
                minLen=min(minLen,right-left+1);
            }
        }
        
        return minLen;
    }
};
时空复杂度分析:
时间复杂度：O(n^2)  最坏情况下，每个元素出现频次都一样;
空间复杂度：O(n);

优化：
上面代码可优化的地方：
1.找maxCount，可以在计算频次时就记录了
2.两个while循环可以省略，因为可以看作是寻找出现频次最高的数的第一次出现的位置和最后一次出现的位置

关键：
1.记录元素的频次：用于判断该元素是否为定义数组度的关键元素
2.记录元素第一次和最后一次出现的位置：用于计算连续子数组最多可缩小到的范围，因为子数组必须至少包含所有关键元素

class Solution {
public:
    int findShortestSubArray(vector<int>& nums) {
        //key:元素
        //value[0]:频次
        //value[1]:第一次出现的位置
        //value[2]:最后一次出现的位置
        unordered_map<int,vector<int>>map;
        int maxCount=INT_MIN;

        for(int i=0;i<nums.size();i++){
            //如果已经在map中了，只需更新频次和最后一次出现的位置
            if(map.count(nums[i])){
                map[nums[i]][0]++;
                map[nums[i]][2]=i;
            }else{//第一次进入map中，初始化
                map[nums[i]]={1,i,i};
            }
            //记录最大频次，也就是数组的度
            maxCount=max(maxCount,map[nums[i]][0]);
        }

        int minLen=INT_MAX;//最短连续子数组长度
        for(auto &pair:map){
            //如果当前元素的频次等于数组的度，则是符合要求的元素，计算最短子数组长度
            if(pair.second[0]==maxCount){
                minLen=min(minLen,pair.second[2]-pair.second[1]+1);
            }
        }

        return minLen;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [717. 1 比特与 2 比特字符](https://leetcode.cn/problems/1-bit-and-2-bit-characters/)

```cpp
发现规律，1总是要和后面一个数字组成2比特字符，而0只能自己组成1比特字符，所以遍历bits数组，如果遇到1，则跳2个下标；遇到0，则只跳1个下标。待该组成的字符都组成完毕后，是否能够留下最后一个0独自组成1比特字符，判断标志是能否到达最后一个0所在下标

class Solution {
public:
    bool isOneBitCharacter(vector<int>& bits) {
        int i=0;
        while(i<bits.size()){
            //如果能够处于最后一个0，说明可以使其独自组成第一种字符，符合要求，返回true
            if(i==bits.size()-1)return true;
            if(bits[i]==1)i+=2;//如果是1，必须和后面的0或1组成第二种字符，所以跳两个下标
            else i++;//如果是0，只能组成第一种字符，跳一个下标
        }
        return false;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [724. 寻找数组的中心下标](https://leetcode.cn/problems/find-pivot-index/)

```cpp
方法一：一次遍历

class Solution {
public:
    int pivotIndex(vector<int>& nums) {
        //记录数组总和后，就不需要再遍历去计算右边元素总和了，直接减去左边元素总和即可
        int sum=0;//数组总和
        for(int i=0;i<nums.size();i++){
            sum+=nums[i];
        }

        int leftSum=0;//左侧所有元素相加和
        for(int i=0;i<nums.size();i++){
            //i的左侧元素和等于右侧元素和，则i是中心坐标
            if(leftSum==sum-leftSum-nums[i])return i;
            leftSum+=nums[i];
        }

        return -1;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [747. 至少是其他数字两倍的最大数](https://leetcode.cn/problems/largest-number-at-least-twice-of-others/)

```cpp
方法一：一次遍历
找最大和次大的数的下标，判断最大的数是否大于等于次大的数的两倍

class Solution {
public:
    int dominantIndex(vector<int>& nums) {
        //只有一个数，没有其他数，直接返回该数下标即可，确保后面两个下标值都不为-1
        if(nums.size()==1)return 0;
		
        //引入最大值和次大值是为了方便先初始化最大值和次大值的下标，否则等于-1时，不好处理越界
        int firstMaxVal=INT_MIN;//最大值
        int secondMaxVal=INT_MIN;//次大值
        int firstMaxIndex=-1;//最大值下标
        int secondMaxIndex=-1;//次大值下标
		
        for(int i=0;i<nums.size();i++){
            if(nums[i]>firstMaxVal){
                secondMaxVal=firstMaxVal;
                secondMaxIndex=firstMaxIndex;
                firstMaxVal=nums[i];
                firstMaxIndex=i;
            }
            else if(nums[i]>secondMaxVal){
                secondMaxVal=nums[i];
                secondMaxIndex=i;
            }
        }

        //如果最大值大于等于次大值的两倍，则返回其坐标，否则返回-1
        return firstMaxVal>=secondMaxVal*2?firstMaxIndex:-1;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [830. 较大分组的位置](https://leetcode.cn/problems/positions-of-large-groups/)

```cpp
方法一：一次遍历
从左到右遍历，如果出现连续3个及以上相同的字符，就将该[start,end]加入到结果集中，遍历的顺序其实也是起始下标的顺序，所以无需再排序

class Solution {
public:
    vector<vector<int>> largeGroupPositions(string s) {
        vector<vector<int>>res;//结果集
        int start=0;//起始下标

        while(start<s.length()){
            //定义结束下标，从起始下标开始遍历
            int end=start;
            //有相同的字符就往后走，注意越界
            while(end+1<s.length()&&s[end]==s[end+1])end++;
            //如果有3个及以上相同的连续字符，将起始下标和结束下标加入结果集
            if(end-start+1>=3)res.push_back({start,end});
            //重新定义下一个起始下标
            start=end+1;
        }
        
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [888. 公平的糖果交换](https://leetcode.cn/problems/fair-candy-swap/)

![image-20230726121723195](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230726121723195.png)

```cpp
方法一：哈希表（列方程）
只用交换一个糖果盒
关键：sumA-x+y=sumB+x-y => x=y+(sumA-sumB)/2

class Solution {
public:
    vector<int> fairCandySwap(vector<int>& aliceSizes, vector<int>& bobSizes) {
        vector<int>res;
        int sumA=accumulate(aliceSizes.begin(),aliceSizes.end(),0);
        int sumB=accumulate(bobSizes.begin(),bobSizes.end(),0);
        int delta=(sumA-sumB)/2;

        //记录alice各盒的糖果数，方便以O(1)的时间复杂度获取
        unordered_set<int>s(aliceSizes.begin(),aliceSizes.end());
        for(auto&y:bobSizes){
            int x=y+delta;
            if(s.count(x)){
                res=vector<int>{x,y};
                break;
            }
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n+m);
空间复杂度：O(m);
```

#### [914. 卡牌分组](https://leetcode.cn/problems/x-of-a-kind-in-a-deck-of-cards/)

```cpp
方法一：最大公约数
各组相同元素的个数为count，则需要找到各组count的最大公约数，确保有一个x能够使各组元素平均分配，且x>=2

class Solution {
public:
    bool hasGroupsSizeX(vector<int>& deck) {
        int cnt[10000]={0};
        int x=0;
        for(auto&n:deck)cnt[n]++;

        //找所有count的最大公约数x，如果x是1了，则已经不符合要求了
        for(auto&n:cnt){
            if(n){
                x=gcd(n,x);
                if(x==1)return false;
            }
        }

        return true;
    }
};
时空复杂度分析:
时间复杂度：O(nlogc)  n是deck的大小，logc是数组中数的范围;
空间复杂度：O(n);
```

#### [941. 有效的山脉数组](https://leetcode.cn/problems/valid-mountain-array/)

```cpp
方法一：爬坡

class Solution {
public:
    bool validMountainArray(vector<int>& arr) {
        int n=arr.size();
        //arr.length()<3就不是山脉数组了
        if(n<3)return false;

        //先到山顶
        int i=0;
        while(i+1<n&&arr[i]<arr[i+1])i++;
        //如果山顶在第一个下标，则只有下坡没有上坡，无效的山脉数组
        //如果山顶在最后一个下标，则只有上坡没有下坡，也是无效的山脉数组
        if(i==n-1||i==0)return false;

        //下山
        while(i+1<n&&arr[i]>arr[i+1])i++;
        
        //最终如果都是下坡，则可以到达最后一个下标，那么就是有效的山脉数组
        return i==n-1;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [989. 数组形式的整数加法](https://leetcode.cn/problems/add-to-array-form-of-integer/)

```cpp
方法一：逐位相加
class Solution {
public:
    vector<int> addToArrayForm(vector<int>& num, int k) {
        int carry=0;//进位
        int i=num.size()-1;//遍历num
        int digit=0;//k各个数位位上的数
        int sum=0;
        while(i>=0){
            if(k==0&&carry==0)break;//k=0并且进位也是0，就没有继续逐位相加的必要了
            digit=k%10;//获取k当前数位上的数
            sum=num[i]+digit+carry;//计算和
            num[i]=sum%10;//将结果存入num中
            carry=sum/10;//计算进位
            k/=10;//k除10，方便获取下一个数位上的数
            i--;//方便获取num下一个数位上的数
        }

        //出循环后还需要考虑以下三种情况
        //1.二者都加完了，但是进位没处理
        //2.k未加完：在num前面插入
        //3.num未加完：不用做处理，因为本来就是在num原地上相加
        //2必须在1之前处理，这样就不需要在后面额外又处理一次最高的进位了

        //第二种情况
        while(k>0){
            digit=k%10;
            sum=digit+carry;
            num.insert(num.begin(),sum%10);
            carry=sum/10;
            k/=10;
        }

        //第一种情况，前面两个循环已经把num和k都加完了
        if(carry){
            num.insert(num.begin(),carry);
        }

        return num;
    }
};
时空复杂度分析:
时间复杂度：O(max(n,logk));
空间复杂度：O(1);
```

#### [1128. 等价多米诺骨牌对的数量](https://leetcode.cn/problems/number-of-equivalent-domino-pairs/)

```cpp
方法一：哈希表
class Solution {
public:
    int iterativeAdd(int n){
        //等差数列求和公式
        return (n*(n-1))/2;
    }

    int numEquivDominoPairs(vector<vector<int>>& dominoes) {
        //key:多米诺骨牌
        //value:多米诺骨牌出现的次数，[1,2],[2,1]看作同一组多米诺骨牌
        map<vector<int>,int>mp;
        int count=0;
        for(auto&vec:dominoes){
            //当前二元对的逆序对，可能已经存在于mp中，不用重复另开一组多米诺骨牌
            vector<int>tmp={vec[1],vec[0]};
            if(mp.count(tmp))mp[tmp]++;
            else{
                mp[vec]++;
            }
        }

        for(auto&pair:mp){
            //如果一组多米诺骨牌出现n次，就会有(n-1)+(n-2)+...+1种(i,j)组合
            count+=iterativeAdd(pair.second);
        }

        return count;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);

方法二：二元组表示+计数（更节省时间和空间的哈希表表示法）
将二元对用统一的格式表示，换算成二位数，因为都是0~9的数，因此不会超过100，比如[1,2],[2,1]可以用12表示，这样就不用哈希表了
class Solution {
public:
    int numEquivDominoPairs(vector<vector<int>>& dominoes) {
        vector<int>counts(100,0);
        int val=0;
        int res=0;
        for(auto&vec:dominoes){
            val=vec[0]<vec[1]?vec[0]*10+vec[1]:vec[1]*10+vec[0];
            res+=counts[val];//这样加的恰好是(n-1)+(n-2)+...+1
            counts[val]++;
        }
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

方法一开始用的是unordered_map，出现如下问题：

> error: call to implicitly-deleted default constructor of ‘unordered_map＜vector<int>, int＞‘ m；

`unordered_map`中用`std::hash`来计算`key`，但是C++中没有给`vector<int>`做**Hash**的函数，所以不能用`vector<int>`作为`unordered_map`的key。

但是`map`可以，`map`里面是通过操作符`<`来比较大小，而`vector<int>`是可以比较大小的，所以，`map`在这里用是可以的

### Medium

#### [581. 最短无序连续子数组](https://leetcode.cn/problems/shortest-unsorted-continuous-subarray/)

![微信截图_20200921203355.png](https://pic.leetcode-cn.com/1600691648-ZCYlql-%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20200921203355.png)

```cpp
方法一：一次遍历
	将数组nums看作numsA，numsB，numsC三段，nums大小为n
	numsA:[0,left-1]有序
	numsB:[left,right]无序
	numsC:[right+1,n-1]有序
	所以可以将题目转为找left和right

规律：
    numsB的任意元素大于numsA[left-1]，numsB的任意元素小于numsC[right+1]
    1.从左到右遍历nums，记录当前遍历过的所有元素中的最大值(maxVal)下标，numsB[right]是最后一个满足小于maxVal的元素
    2.从右到左遍历nums，记录当前遍历过的所有元素中的最小值(minVal)下标，numsB[left]是最后一个满足大于minVal的元素
	两个遍历可以在一个for循环中实现

卡点：
	一开始没考虑到有相同元素的存在，认为left就是第一个nums[i]>nums[i+1]的i，right则是最后一个nums[i]>nums[i+1]的i+1，有重复元素后，发现这样子的话left就不准确了，left始终指向重复元素的最后一个，实际上应该指向重复元素的第一个，因为它们是一起的，都需要重新排序

class Solution {
public:
    int findUnsortedSubarray(vector<int>& nums) {
        int n=nums.size();
        int left=-1;
        int right=-1;
        int maxVal=INT_MIN;
        int minVal=INT_MAX;

        for(int i=0;i<n;i++){
            //nums[right]是最后一个小于maxVal的元素
            if(nums[i]<maxVal)right=i;
            else maxVal=nums[i];

            //nums[left]是最后一个大于minVal的元素
            if(nums[n-1-i]>minVal)left=n-1-i;
            else minVal=nums[n-1-i];
        }

        //right==-1，说明nums已经是升序数组了
        return right==-1?0:right-left+1;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [665. 非递减数列](https://leetcode.cn/problems/non-decreasing-array/)

```cpp
方法一：贪心算法
经过自己的理解后，想到的另一个解决问题的角度：
对于nums[i]>nums[i+1]：（要么调小nums[i]，要么调大nums[i+1]，尽可能调小而不是调大影响后面的）
其实nums[i]的修改范围属于[nums[i-1],nums[i+1]]，可以根据这两个值的大小来选择修改哪个值
1.当nums[i+1]>=nums[i-1]时，nums[i]有可选择的值修改，就尽量修改nums[i]的值，减少对后面的影响
2.当nums[i+1]<nums[i-1]时，nums[i]没有可选择的值修改，就只能修改nums[i+1]，并令其尽可能小，但最小也要等于nums[i]
在有可选择范围的时候就调小nums[i]，调小到最小值否则调大nums[i+1]，但也只能调大到最小值

class Solution {
public:
    bool checkPossibility(vector<int>& nums) {
        int cnt=0;
        for(int i=0;i<nums.size()-1;i++){
            if(nums[i]>nums[i+1]){
                if(i==0)nums[i]=nums[i+1];
                else if(nums[i+1]>=nums[i-1])nums[i]=nums[i-1];
                else if(nums[i+1]<nums[i-1])nums[i+1]=nums[i];
                cnt++;//进入当前if分支后，必然会修改一次值，修改次数加1
            }
        }
        return cnt<=1;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [840. 矩阵中的幻方](https://leetcode.cn/problems/magic-squares-in-grid/)

![image-20230725115449164](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230725115449164.png)

```cpp
方法一：找规律
3阶幻方的解是固定的，有以上8种
共同点：
1.中间元素都是5
2.角上都是偶数
3.中点都是奇数
同一行的解可以通过旋转得到，第一行镜像，可以得到第二行，也就是说，3阶幻方本质上只有一个解，其余都是旋转镜像的结果

步骤：
1.遍历中间的部分，只有是5的时候，才判断以它为中心的3阶矩阵是否为幻方
2.由于5已经比对过了，只需要对比其他元素，由于解的旋转不变特性，将8个元素按顺序存放，方便后面比较
3.解的编码，将前面部分的放在后面就达到了旋转效果；镜像只需反向迭代即可
4.输入的数组首位元素和8、6、2、4逐个比较决定可能的解，从编码的数组中取出正向镜像两个解比较是否其一，就可以确定是否为幻方

class Solution {
public:
    vector<int>m{8,1,6,7,2,9,4,3,8,1,6,7,2,9,4,3};

    bool isMagic(vector<int>&vec){
        //因为只有偶数会在角落，所以只需遍历偶数和vec[0]比较即可
        //如果vec[0]不是偶数，说明不是幻方，因为幻方的左上角必定是偶数
        for(int i=0;i<8;i+=2){
            if(m[i]==vec[0]){
                //判断是否为以m[i]为左上角元素的幻方，正向遍历是旋转，逆向遍历是镜像，二者只要是其一即可
                //因为无论是旋转还是镜像，都是幻方解的一种体现
                return vec==vector<int>(m.begin()+i,m.begin()+i+8)||
                       vec==vector<int>(m.rbegin()+7-i,m.rbegin()+15-i);
            }
        }
        return false;
    }

    int numMagicSquaresInside(vector<vector<int>>& grid) {
        //方便在以某个坐标为中心的情况下，计算其四周的坐标
        //k∈[0,7]：di[k]和dj[k]一起使用，比如di[0]、dj[0]，以中心坐标向左和上各移动一格，获取到左上角的坐标，以此类推
        int di[8]={-1,-1,-1,0,1,1,1,0};
        int dj[8]={-1,0,1,1,1,0,-1,-1};
        int count=0;

        //i<grid.size()-1而不是i<=grid.size()-2
        //因为grid.size()是unsigned int，当其等于1时，grid.size()-2不会是负数，不方便和int类型的i作比较，会出现异常
        for(int i=1;i<grid.size()-1;i++){
            for(int j=1;j<grid[0].size()-1;j++){
                if(grid[i][j]==5){
                    vector<int>around(8);//以[i][j]为中心的四周元素，共8个
                    for(int k=0;k<8;k++){
                        around[k]=grid[i+di[k]][j+dj[k]];
                    }
                    if(isMagic(around))count++;
                }
            }
        }
        return count;
    }
};
时空复杂度分析:
时间复杂度：O(n*m);
空间复杂度：O(1);

方法二：暴力法
1.网格的总和是45，因为是1到9的不同数字
2.每列每行的和都是15，因为有3行，45/3=15
3.对角线的和也是15，因为和行列一样
4.总共有4个穿过中心的线，和为4*15=60，整个网格的和为45，而中心值被额外加了3次，其余值只被加了一次，所以中心值=(60-45)/3=5
减少额外时间，即如果中心值不等于5，则肯定不是幻方

class Solution {
public:
    bool isMagic(vector<int>&vec){
        //1~9每个数必须出现一次，且只能出现一次
        vector<int>record(16,0);
        for(int i=0;i<9;i++){
            record[vec[i]]++;
        }
        //只需考虑1~9
        for(int i=1;i<10;i++){
            if(record[i]!=1)return false;
        }
		
        //各行各列各对角线和必须为15
        vector<int>row(3);//各行和
        vector<int>col(3);//各列和
        vector<int>diagonal(2);//各对角线和

        for(int i=0;i<3;i++){
            row[i]=vec[i*3]+vec[i*3+1]+vec[i*3+2];
            if(row[i]!=15)return false;
        }

        for(int i=0;i<3;i++){
            col[i]=vec[i]+vec[i+3]+vec[i+6];
            if(col[i]!=15)return false;
        }

        diagonal[0]=vec[0]+vec[4]+vec[8];
        diagonal[1]=vec[2]+vec[4]+vec[6];
        if(diagonal[0]!=15||diagonal[1]!=15)return false;

        return true;
    }

    int numMagicSquaresInside(vector<vector<int>>& grid) {
        int count=0;
        int di[9]={-1,-1,-1,0,0,0,1,1,1};
        int dj[9]={-1,0,1,-1,0,1,-1,0,1};

        for(int i=1;i<grid.size()-1;i++){
            for(int j=1;j<grid.size()-1;j++){
                if(grid[i][j]==5){
                    vector<int>around(9);
                    for(int k=0;k<9;k++){
                        around[k]=grid[i+di[k]][j+dj[k]];
                    }
                    if(isMagic(around))count++;
                }
            }
        }

        return count;
    }
};
时空复杂度分析:
时间复杂度：O(n*m);
空间复杂度：O(1);
```

#### [849. 到最近的人的最大距离](https://leetcode.cn/problems/maximize-distance-to-closest-person/)

```cpp
方法一：双指针
1.找到两个中间只有0的1，且距离要最远，然后坐在它们中间即可，最大距离要除2
2.特殊情况，坐在边界，最大距离不用除2

class Solution {
public:
    int maxDistToClosest(vector<int>& seats) {
        //由于是找两个相距最远的相邻1，先把特殊情况考虑了，即第一个1左边全是0和最后一个1右边全是0的情况
        int left=0;
        while(seats[left]==0)left++;
        int right=seats.size()-1;
        while(seats[right]==0)right--;
		
        //seats.size()是unsigned int类型，而max()函数必须是两个同类型的值比较，所以需要先转为int类型
        int maxDis1=max(left,(int)seats.size()-1-right);//特殊情况的最大距离

        int maxDis2=INT_MIN;//两个相邻1的最大距离
        //此时left是第一个1，right是第二个1
        while(left<right){
            //找到下一个空位置
            while(left<right&&seats[left]==1)left++;
            int i=left;
            //找到下一个相邻1，然后才可以计算中间的距离
            while(left<right&&seats[left]==0)left++;
            maxDis2=max(maxDis2,left-(i-1));
        }

        return max(maxDis1,maxDis2/2);
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [剑指 Offer 66. 构建乘积数组](https://leetcode.cn/problems/gou-jian-cheng-ji-shu-zu-lcof/)

```cpp
方法一：一次遍历
先求a数组的总乘积allMul，然后b[i]=allMul/a[i]，但是有问题，因为如果a[i]=0，不好处理

解决方法：
记录0的个数zeroCnt，分情况讨论：
1.zeroCnt==0：求A数组的总乘积allMul，然后B[i]=allMul/A[i]
2.zeroCnt==1：求A数组删掉该0之后的总乘积allMul，还要记录0的下标zeroIdx，最后b[zeroIdx]=allMul，其它都是0
3.zeroCnt>1：b数组全为0

class Solution {
public:
    vector<int> constructArr(vector<int>& a) {
        vector<int>b(a.size(),0);//返回的b数组
        int zeroCnt=0;//0的个数
        int zeroIdx=-1;//唯一0的下标
        int allMul=1;//总乘积

        for(int i=0;i<a.size();i++){
            if(a[i]==0){
                //记录0的个数和下标
                zeroCnt++;
                zeroIdx=i;
            }else{
                allMul*=a[i];
            }
        }
        
        if(zeroCnt==0){//没有0，普通处理
            for(int i=0;i<a.size();i++){
                b[i]=allMul/a[i];
            }
        }
        else if(zeroCnt==1){//有一个0，单独处理zeroIdx上的乘积即可，其余全是0
            b[zeroIdx]=allMul;
        }
        //0的个数大于1个，则不需要额外处理，直接返回全0即可

        return b;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);

方法二：正反遍历
由于b[i]抛弃的是a[i]，所以只需要正序遍历（从左到右）得到a[0]~a[i-1]的乘积，再反序遍历（从右到左）得到a[i+1]~a[a.size()-1]的乘积
class Solution {
public:
    vector<int> constructArr(vector<int>& a) {
        int mul=1;
        vector<int>b(a.size(),1);//初始化为全1，这样在*=时不会影响乘积

        for(int i=1;i<a.size();i++){
            mul*=a[i-1];
            b[i]*=mul;
        }

        //重置mul
        mul=1;
        for(int i=a.size()-2;i>=0;i--){
            mul*=a[i+1];
            b[i]*=mul;//注意是*=，而不是=，不然就覆盖掉前面的乘积了
        }

        return b;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

```cpp
方法一：双指针（贪心算法）
left、right指针移动规则：
1. height[left]<=height[right]：left++
2. height[left]>height[right]:right--
谁矮移动谁
解释：
面积=min(height[left],height[right])*(right-left)
假设此时height[left]<height[right1]，假设移动指向高的柱子那个指针，移动right1至right2，会出现以下情况
1. height[right1]<=height[right2]，此时宽度一定是缩小了，而高度不变，依旧是height[left]，所以面积缩小
2. height[right1]>height[right2]
  2.1 height[right2]<=height[left]：此时高度是更小的height[right2]，面积肯定也更小了
  2.2 height[right2]>height[left]：此时宽度一定是缩小了，而高度不变，依旧是height[left]，所以面积缩小
综上，移动指向高的那个指针，只会让面积更小，而移动矮的那个指针，可能可以使下一个柱子的高度更高，然后更加充分地利用原先那个高柱子的优势

class Solution {
public:
    int maxArea(vector<int>& height) {
        int left=0;
        int right=height.size()-1;
        int max_Area=INT_MIN;

        while(left<right){
            int h=height[left]<=height[right]?height[left++]:height[right--];//谁矮谁移动
            int w=right-left+1;//此时的left或right已经指向了下一个柱子，而不是当前计算面积的柱子，所以宽度要加1
            max_Area=max(max_Area,h*w);
        }

        return max_Area;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [1497. 检查数组对是否可以被 k 整除](https://leetcode.cn/problems/check-if-array-pairs-are-divisible-by-k/)

```cpp
方法一：按余数统计
关键点1：x、y的和如果能够被k整除，说明x%k+y%k==k，所以需要余数为x%k的需要和余数为y%k的搭配
关键点2：如何让负数取模k的余数处于[0,k-1]的范围？xk=(x%k+k)%k即可，x是正数也可以这样计算，不影响余数
eg.(-5)%3=-2：(-5)=(-3)*1+(-2)，也可以是
   ((-5)%3+3)%3=1：(-5)=(-3)*2+1

class Solution {
public:
    bool canArrange(vector<int>& arr, int k) {
        //下标是0~k-1，恰好对应余数
        vector<int>remainder(k,0);
        //统计余数
        for(auto&x:arr){
            remainder[(x%k+k)%k]++;
        }

        for(int i=1;i<k;i++){
            //余数i要和余数k-i搭配，这样才能被k整除，所以它们数量要相同
            if(remainder[i]!=remainder[k-i])return false;
        }
        
        //余数0只能和余数0搭配，因为余数k取模k之后就是余数0，所以余数0的数量必须是偶数
        return remainder[0]%2==0;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```



### Hard



## 字符串

### Easy



### Medium



### Hard



## 链表

### Easy



### Medium



### Hard



## 数学

### Easy



### Medium



### Hard



## 哈希表

### Easy



### Medium



### Hard



## 二分查找

### Easy



### Medium



### Hard



## 栈

### Easy



### Medium



### Hard



## 双指针

### Easy



### Medium



### Hard



## 贪心算法

### Easy



### Medium



### Hard



## 回溯算法

### Easy



### Medium



### Hard



## 动态规划

### Easy



### Medium



### Hard



## 优先搜索

### Easy



### Medium



### Hard



## 树

### Easy



### Medium



### Hard

