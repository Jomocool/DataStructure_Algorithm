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
        //初始化为全1，这样在*=时不会影响乘积
        //主要是不影响b[0]的计算，因为第一个for循环不处理b[0]，而第二个for循环必须*=，如果b数组初始化为0，则b[0]必然等于0
        vector<int>b(a.size(),1);

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
        //(x,y)x要找到匹配的y，使得(x+y)%k==0 => x%k+y%k=k||(x%k==0&&y%k==0)
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

#### [13. 罗马数字转整数](https://leetcode.cn/problems/roman-to-integer/)

```cpp
方法一：复合字符匹配
class Solution {
public:
    int romanToInt(string s) {
        //映射表
        unordered_map<string,int>mp={
            {"I",1},{"IV",4},
            {"V",5},{"IX",9},
            {"X",10},{"XL",40},
            {"L",50},{"XC",90},
            {"C",100},{"CD",400},
            {"D",500},{"CM",900},
            {"M",1000}
        };

        int i=0;
        int res=0;
        while(i<s.length()){
            if(i==s.length()-1){//最后一个字母，肯定是加
                res+=mp[string(1,s[i++])];
            }else{
                //只需判断当前字符s[i]是独自匹配还是和s[i+1]匹配，后者优先级更高，所以优先判断
                string valStr=s.substr(i,2);
                if(mp.count(valStr)){
                    res+=mp[valStr];
                    i+=2;
                }else{
                    res+=mp[string(1,s[i++])];
                }
            }
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);

方法二：单独字符匹配
逻辑更加清晰，如果下一个字符比当前字符大，说明当前字符是需要被减去的

class Solution {
public:
    int romanToInt(string s) {
        //映射表
        unordered_map<char,int>mp={
            {'I',1},
            {'V',5},
            {'X',10},
            {'L',50},
            {'C',100},
            {'D',500},
            {'M',1000}
        };

        int i=0;
        int res=0;
        while(i<s.length()){
            //最后一个字符，肯定是加，因为后面没有多余字符和它搭配
            if(i==s.length()-1){
                res+=mp[s[i++]];
                break;//及时break，不要继续往下面去判断了，有异常
            }

            if(mp[s[i]]<mp[s[i+1]]){
                //前小后大，就要和后面的字符搭配，值要减去前面小的对应值
                res-=mp[s[i++]];
            }else{
                res+=mp[s[i++]];
            }
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [67. 二进制求和](https://leetcode.cn/problems/add-binary/)

```cpp
方法一：模拟求和
class Solution {
public:
    string addBinary(string a, string b) {
        //翻转字符串a、b，从低位相加，同时方便计算结果
        reverse(a.begin(),a.end());
        reverse(b.begin(),b.end());
        string res;

        int sum=0;//数位上的数相加和
        int carry=0;//进位
        int aIdx=0;//遍历a
        int bIdx=0;//遍历b
        int aDigit=0;//a[aIdx]对应数值
        int bDigit=0;//b[bIdx]对应数值

        while(aIdx<a.length()&&bIdx<b.length()){
            aDigit=a[aIdx++]-'0';
            bDigit=b[bIdx++]-'0';
            sum=aDigit+bDigit+carry;
            res.push_back(sum%2+'0');
            carry=sum/2;
        }

        while(aIdx<a.length()){
            aDigit=a[aIdx++]-'0';
            sum=aDigit+carry;
            res.push_back(sum%2+'0');
            carry=sum/2;
        }

        while(bIdx<b.length()){
            bDigit=b[bIdx++]-'0';
            sum=bDigit+carry;
            res.push_back(sum%2+'0');
            carry=sum/2;
        }

        //如果还有进位，最高位添1
        if(carry)res.push_back('1');
        
        //翻转结果字符串
        reverse(res.begin(),res.end());

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(max(m,n))  其中m是a的长度，n是b的长度;
空间复杂度：O(1);
```

#### [434. 字符串中的单词数](https://leetcode.cn/problems/number-of-segments-in-a-string/)

```cpp
方法一：一次遍历

class Solution {
public:
    int countSegments(string s) {
        int i=0;
        int res=0;
        while(i<s.length()){
            //找到下一个字母
            while(i<s.length()&&s[i]==' ')i++;
            int tmp=i;//记录当前下标
            //找到下一个空格
            while(i<s.length()&&s[i]!=' ')i++;
            //如果i!=tmp，意味着i和tmp间有一个单词
            if(i!=tmp)res++;
        }
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [680. 验证回文串 II](https://leetcode.cn/problems/valid-palindrome-ii/)

```cpp
方法一：模拟
要丢掉的那一个字符必然是导致左右两边不对称的那一对字符的其中之一，然后只需判断丢掉字符后，剩余的字符串是否为回文串即可

class Solution {
public:
    bool isPalindrome(string& s,int left,int right){
        while(left<right){
            if(s[left]!=s[right])return false;
            left++;
            right--;
        }
        return true;
    }

    bool validPalindrome(string s) {
        int left=0;
        int right=(int)s.length()-1;
        while(left<right&&s[left]==s[right]){
            left++;
            right--;
        }
        //当前s[left]!=s[right]
        //1.删掉s[left]之后是回文串
        //2.删除s[right]之后是回文串
        //因为已经删掉了一个字符了，所以删掉之后的字符串必须是回文串，不能够再删除了
        return isPalindrome(s,left+1,right)||isPalindrome(s,left,right-1);
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [819. 最常见的单词](https://leetcode.cn/problems/most-common-word/)

```cpp
方法一：哈希表
步骤：
1.禁用列表转放入set：O(1)时间判断word是否存在于禁用列表中
2.哈希表mp记录词频
3.遍历paragraph，找到每个单词，注意单词都是由字母组成的，不是字母的字符直接pass，用ASCII码判断是否为26位大小写字母
4.将单词word转成小写，因为最终的结果都要是小写，比如ball实际上等价于BALL

class Solution {
public:
    void strToLower(string&s){
        for(auto&c:s){
            //tolower函数原型：int tolower(int c)
            //tolower函数的参数是值拷贝构造，返回参数转为小写后的字母，所以需要准备转为小写的字母去接收。toupper函数同理
            c=tolower(c);
        }
    }

    string mostCommonWord(string paragraph, vector<string>& banned) {
        //为了通过O(1)时间就能够判断当前单词是否在禁用列表中
        unordered_set<string>bannedSet(banned.begin(),banned.end());
        //记录词频
        unordered_map<string,int>mp;

        string resWord;//出现次数最多且不在禁用列表的单词

        int i=0;
        int n=paragraph.length();
        while(i<n){
            //不是字母都pass
            while(i<n&&!((paragraph[i]>='a'&&paragraph[i]<='z')||(paragraph[i]>='A'&&paragraph[i]<='Z')))i++;
            int start=i;
            //单词必须都是由字母组成的
            while(i<n&&((paragraph[i]>='a'&&paragraph[i]<='z')||(paragraph[i]>='A'&&paragraph[i]<='Z')))i++;
            
            //确保有单词
            if(i!=start){
                string word=paragraph.substr(start,i-start);
                strToLower(word);//全部转成小写字母
                if(!bannedSet.count(word)){//不在禁用列表中
                    mp[word]++;//词频加1
                    if(mp[word]>mp[resWord])resWord=word;//如果当前单词词频超过词频最高的单词，那么当前单词才是词频最高的单词
                }
            }
        }

        return resWord;
    }
};
时空复杂度分析:
时间复杂度：O(n+m);
空间复杂度：O(n+m);
```

#### [859. 亲密字符串](https://leetcode.cn/problems/buddy-strings/)

```cpp
方法一：一次遍历
长度不同，直接返回false，因为无论怎么交换都无法相同
cnt：记录s与goal不相同的字符个数
idx1：记录s与goal第一个不相同字符的下标
idx2：记录s与goal第二个不相同字符的下标
遍历一次后，有以下4种情况：
1. cnt>2：只能交换两个字符的情况下，s与goal必然无法相同，返回false
2. cnt==2：返回(s[idx1]==goal[idx2]&&s[idx2]==goal[idx1])
3. cnt==1：假设idx1与s中另一个下标为idx的字符交换
  3.1 s[idx1]==s[idx]：s[idx]仍然不等于goal[idx1]
  3.2 s[idx1]!=s[idx]：s[idx1]不等于goal[idx]
4. cnt==0：说明s和goal交换前就相同了，所以需要s中两个相同的字符交换即可，因为相同字符交换后，不影响二者的等价关系，所以需要一个记录词频的表，由于都是小写字母，所以用一个大小为26的数组记录即可

class Solution {
public:
    bool buddyStrings(string s, string goal) {
        //长度不同，无论怎么交换都没用
        if(s.length()!=goal.length())return false;

        vector<int>record(26);//记录词频
        int cnt=0;//记录不相同的地方有多少处
        int maxCnt=INT_MIN;//最高词频
        int idx1=-1;//记录第一处不相同的下标
        int idx2=-1;//记录第二处不相同的下标

        //开始遍历
        for(int i=0;i<s.length();i++){
            record[s[i]-'a']++;
            maxCnt=max(maxCnt,record[s[i]-'a']);
            if(s[i]!=goal[i]){
                cnt++;
                if(cnt==1)idx1=i;
                else if(cnt==2)idx2=i;
            }
        }

        if(cnt>2||cnt==1)return false;
        if(cnt==2)return s[idx1]==goal[idx2]&&s[idx2]==goal[idx1];

        //cnt==0的情况，看是否能够有两个相同的字符交换从而不影响二者的等价关系
        return maxCnt>=2;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);

总结：关键在于记录有几处不相同的地方，然后以2为分界线，分析各情况下交换两字符是否能够使二者相同
```



### Medium

#### [686. 重复叠加字符串匹配](https://leetcode.cn/problems/repeated-string-match/)

```cpp
方法一：模拟
通过长度来确定叠加次数的上下界，假设a和b的长度分别是m、n
下界：m/n（比如，abcd和abcdabcdabcd）
上界：m/n+1（比如，cdabcdabcdab）

class Solution {
public:
    int repeatedStringMatch(string a, string b) {
        if(a.empty())return -1;

        //aMul是a的叠加字符串
        string aMul=a;
        int res=1;

        while(aMul.size()<b.size()){
            aMul.append(a);
            res++;
        }
        //下界：如果当前aMul中已经能找到b了，直接返回当前叠加次数
        if(aMul.find(b)!=string::npos)return res;

        //上界：下界找不到，就再叠加一次
        aMul.append(a);
        res++;
        if(aMul.find(b)!=string::npos)return res;

        return -1;
    }
};
时空复杂度分析:
时间复杂度：O(m+n);
空间复杂度：O(m);
```

#### [6. N 字形变换](https://leetcode.cn/problems/zigzag-conversion/)

```cpp
方法一：模拟

class Solution {
public:
    string convert(string s, int numRows) {
        vector<string>vec(numRows);//用于存储N字形排列的字符串s
        bool down=true;//当前移动趋势，N字形有两种运动趋势：下或上
        string res;

        int i=0;
        while(i<s.length()){
            if(down){//从上到下处理各行字符串
                //对于从上到下就处理所有行，刚好是一条竖线
                for(int row=0;row<=numRows-1;row++){
                    if(i<s.length())vec[row].push_back(s[i++]);
                }
                //改变运动趋势
                down=!down;
            }else{//从下到上处理各行字符串
                //对于从下到上，就处理两竖线中间部分的斜线
                for(int row=numRows-2;row>0;row--){
                    if(i<s.length())vec[row].push_back(s[i++]);
                }
                //改变运动趋势
                down=!down;
            }
        }

        for(auto&str:vec){
            res.append(str);
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);

总结：本题的关键在于如何合理分配s[i]给从上到下和从下到上这两种趋势，既不能少插入s[i]，也不能多添加s[i]，所以需要一个合适的轨迹让二者配合，而轨迹的体现就在两个for循环的行分配上
```

#### [8. 字符串转换整数 (atoi)](https://leetcode.cn/problems/string-to-integer-atoi/)

```cpp
方法一：模拟
按照题目给的步骤一步步实现，然后注意字符串s中可能出现的字符以及应对这些字符的方式

class Solution {
public:
    int myAtoi(string s) {
        //空字符串返回0
        if(s.length()==0)return 0;
        int i=0;

        //1.丢弃无用的前导空格
        while(s[i]==' ')i++;

        //2.检查正负号
        int flag=1;//默认正号，因为正负号都没有的情况下就假定结果为正
        if(s[i]=='+')i++;
        else if(s[i]=='-'){
            i++;
            flag=-1;
        }

        //3.一直读入数字字符，直到非数字字符或字符串结尾
        int res=0;
        for(;i<s.length();i++){
            //非数字字符，结束读入
            if(!(s[i]>='0'&&s[i]<='9'))break;
            //读入前导0
            if(res==0&&s[i]=='0')continue;
            //在计算值之前，先预判是否会溢出（必须在计算值之前检测）
            bool isOvermin=res<INT_MIN/10||(res==INT_MIN/10&&s[i]>'8');
            bool isOvermax=res>INT_MAX/10||(res==INT_MAX/10&&s[i]>'7');
            //溢出的话就截断
            if(isOvermin)return INT_MIN;
            else if(isOvermax)return INT_MAX;
            //计算值
            res=res*10+flag*(s[i]-'0');
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [17. 电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

```cpp
方法一：回溯
class Solution {
public:
    vector<string>res;//结果集
    unordered_map<char,string>mp;//数字到字母的映射
    string path;//路径字符串
    
    void dfs(string&digits,int idx){
        //digits所有数字都处理完毕
        if(idx==digits.length()){
            res.push_back(path);
            return;
        }

        //处理当前数字digits[idx]的所有情况，即映射的所有字母都需要考虑一遍
        string str=mp[digits[idx]];//当前数字映射字母集合
        for(int i=0;i<str.length();i++){
            path.push_back(str[i]);
            dfs(digits,idx+1);//处理下一个数字
            path.pop_back();//回溯
        }
    }

    vector<string> letterCombinations(string digits) {
        if(digits.length()==0)return {};
        //初始化
        res.clear();
        mp.clear();
        path.clear();
        mp={
            {'2',"abc"},
            {'3',"def"},
            {'4',"ghi"},
            {'5',"jkl"},
            {'6',"mno"},
            {'7',"pqrs"},
            {'8',"tuv"},
            {'9',"wxyz"}
        };

        dfs(digits,0);

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(3^m × 4^n)  m是3个字母的数字个数，n是4个字母的数字个数，需要遍历每一种字母组合;
空间复杂度：O(m+n)  m+n是digits总长度，也是开辟的栈空间大小，哈希表可看作常数空间大小;

总结：
把每遍历一个数字，看成开辟一个新的栈，意味着树向下延伸一层。最后，有多少个叶子节点就代表着有多少种情况，而每种情况都是需要遍历的，所以这就是时间复杂度。有多少层就代表着最多要同时开辟多少个栈空间，所以这就是空间复杂度

综上，对于回溯算法，叶子节点个数（情况数）就是时间复杂度，树的高度（同时开辟栈空间大小）就是空间复杂度
```

#### [1513. 仅含 1 的子串数](https://leetcode.cn/problems/number-of-substrings-with-only-1s/)

> **选择10^9+7取模的原因**

![image-20230805095818798](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230805095818798.png)

> **虽然结果没有溢出，但是防止过程中的值溢出**

![image-20230805101428993](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230805101428993.png)

```cpp
方法一：找全1子串

对于字符串"11111"，可以按以下情况分析：
1.长度为1的全1子串数：有几个1则就有几个全1子串，即字符串长度
2.长度为2的全1子串数：可以看作以每个1为起点时，后面跟着一个1即可看作是一个长度为2的全1子串，且起点不同所以这些全1子串也不同，不会重复计数，但对于最后一个1，其后面没有多余的1，所以长度为2的全1子串数是字符串长度-1
3.长度为3的全1子串数：和上面同理，是字符串长度-2
4.长度为4的全1子串数：和上面同理，是字符串长度-3
5.长度为5的全1子串数：和上面同理，是字符串长度-4
综上，对于一个长度为n的全1字符串，其仅含1的子串数是n+(n-1)+(n-2)+...+1=(n*(n+1))/2

eg.字符串"0110111"
1.全1子串"11"：仅含1的子串数为3
2.全1子串"111"：仅含1的子串数为6
所以字符串"0110111"仅含1的子串数是9

class Solution {
public:
    //科学计数法转int
    static constexpr int P=int(1E9)+7;

    int numSub(string s) {
        int i=0;
        long long res=0;
        while(i<s.length()){
            while(i<s.length()&&s[i]=='0')i++;
            int tmp=i;
            while(i<s.length()&&s[i]=='1')i++;
            int n=i-tmp;//全1子串长度
            //1LL会在运算时把后面的临时数据扩容成long long类型，再在赋值给左边时转回int型
            res+=((1LL+n)*n)>>1;
            res%=P;
        }
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);

总结知识点：
1.科学技术法转int：int(1E9)
2.1LL：在运算时将后面的临时数据扩容成long long类型
```



### Hard



## 链表

### Easy

#### [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)

```cpp
方法一：双指针

class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode*newHead=new ListNode(-1);
        ListNode*cur=newHead;

        while(list1&&list2){
            //谁小谁先插入新链表
            if(list1->val<=list2->val){
                cur->next=list1;
                list1=list1->next;
            }else{
                cur->next=list2;
                list2=list2->next;
            }
            cur=cur->next;
        }

        //哪个链表还有多余节点，直接让其接到新链表后面
        if(list1)cur->next=list1;
        else if(list2)cur->next=list2;

        return newHead->next;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)

```cpp
方法一：模拟

class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(!head)return nullptr;
        
        ListNode*reverseHead=new ListNode(-1);
        ListNode*cur=head;
        ListNode*next=cur->next;
        while(cur){
            next=cur->next;
            cur->next=reverseHead->next;
            reverseHead->next=cur;
            cur=next;
        }
        return reverseHead->next;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```



### Medium

#### [237. 删除链表中的节点](https://leetcode.cn/problems/delete-node-in-a-linked-list/)

```cpp
方法一：和下一个节点值交换
关键：把要删去的节点变成能获取到其前驱节点的节点即可（即node之后的任意节点都行）

class Solution {
public:
    void deleteNode(ListNode* node) {
        //将要删掉的节点值改为其下一个节点值，因为node不会是最后一个节点，所以放心修改
        node->val=node->next->val;
        //然后跳过node->next直接跳到node->next->next即可
        node->next=node->next->next;
    }
};
时空复杂度分析:
时间复杂度：O(1);
空间复杂度：O(1);

总结：
一开始想的是将后面节点的值一个一个往前拷贝，然后最后让尾节点的前一个节点的next指向nullptr，但实际上不用这样，只需要让不需要的节点的前一个节点的next指向其后一个节点，这道题无法一开始就这样做的原因是没办法获取到node的前一个节点，但是node后面的所有节点都可以这样做了，都可以获取到其前面的一个节点
```



### Hard



## 数学

### Easy

#### [1518. 换水问题](https://leetcode.cn/problems/water-bottles/)

```cpp
方法一：模拟

class Solution {
public:
    int numWaterBottles(int numBottles, int numExchange) {
        int drinkedBottles=numBottles;//已经喝到的水
        int emptyBottles=drinkedBottles;//当前空水瓶数量

        //只有当空水瓶数量大于交换1瓶水所需要的空水瓶数量时，才可以继续换水
        while(emptyBottles>=numExchange){
            //可以换到的水加入到已经喝到的水
            drinkedBottles+=emptyBottles/numExchange;
            //当前空瓶子数量=前一次交换水瓶后剩余的空水瓶+交换水瓶的数量(这次喝的)
            emptyBottles=emptyBottles%numExchange+emptyBottles/numExchange;
        }

        return drinkedBottles;
    }
};
时空复杂度分析:
时间复杂度：O(b/e)  其中b=numBottles,e=numExchange;
空间复杂度：O(1);

方法二：数学
b=numBottles,e=numExchange
1.一定可以喝到b瓶水，得到b个空水瓶
2.每消耗e个空水瓶可以得到一瓶水（一个空水瓶），所以可以看作消耗(e-1)个空水瓶
综上，b-n(e-1)>=e => n<=(b-e)/(e-1)，所以要找到最大的n，但此时空水瓶数量是>=e的，因此最终答案是b+n+1
没有向老板借水的说法，所以不能欠老板水，因此最后一次换水时空水瓶数量应该大于等于e，而不能小于e，小于e就无法换水

疑点：为什么不是b-n(e-1)>=0
解释：因为不是有空水瓶就能换水，而是至少需要e个空水瓶才能换一瓶水，因此当空水瓶数量小于e时，就不用考虑换水了

class Solution {
public:
    int numWaterBottles(int numBottles, int numExchange) {
        int n=(numBottles-numExchange)/(numExchange-1);
        return numBottles<numExchange?numBottles:numBottles+n+1;
    }
};
时空复杂度分析:
时间复杂度：O(1);
空间复杂度：O(1);
```



### Medium



### Hard



## 哈希表

### Easy

#### [202. 快乐数](https://leetcode.cn/problems/happy-number/)

```cpp
方法一：哈希表
class Solution {
public:
    int powSum(int n){
        int sum=0;
        int digit=0;
        while(n>0){
            digit=n%10;
            sum+=digit*digit;
            n/=10;
        }
        return sum;
    }

    bool isHappy(int n) {
        //记录过程中的n，如果重复出现代表进入循环了，返回false
        unordered_set<int>s;
        while(n!=1){
            if(s.count(n))return false;
            s.insert(n);
            n=powSum(n);
        }
        return true;
    }
};
时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(logn);
```



### Medium

#### [204. 计数质数](https://leetcode.cn/problems/count-primes/)

```cpp
方法一：埃氏筛
定理：根据算数基本定理，所有合数都能够写成质数乘积的形式，意味着每个合数必然有一个质数因子
关键：
1.根据以上定理，由于因子一定小于等于自身，所以没被小于自身的质数乘积得到的话，就一定是质数了
2.从平方i*i开始，不需要从i*2开始，因为已经被前面的质数处理过了

class Solution {
public:
    int countPrimes(int n) {
        //初始化所有小于n的数为质数
        vector<bool>isPrime(n,true);
        int cnt=0;
        for(int i=2;i<n;i++){
            if(isPrime[i]){
                cnt++;
                if((long long)i*i<n){
                    //质数的倍数都是合数，标记为false
                    for(int j=i*i;j<n;j+=i){
                        isPrime[j]=false;
                    }
                }
            }
        }
        return cnt;
    }
};
时空复杂度分析:
时间复杂度：O(nloglogn);
空间复杂度：O(n);

总结：
从最小的质数2、3出发，其倍数就可以筛掉很多合数了，而不是这些倍数的数例如5、7、11、13……都是质数，然后再从这些质数出发（i*i出发，因为i*2开始的话都被前面的质数倍数处理完了，比如i*2是2的倍数，i*3是3的倍数，i*4是2的倍数，i*5是5的倍数，i*6是2的倍数，i*7是7的倍数，i*8是2的倍数，i*9是3的倍数……，而对于i*i，4、9、25、49……都是2、3、5、7自己本身的倍数，别的数处理不到）
```



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

