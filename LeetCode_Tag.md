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

#### [581. ![微信截图_20200921203355.png](https://pic.leetcode-cn.com/1600691648-ZCYlql-%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20200921203355.png)

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

#### [387. 字符串中的第一个唯一字符](https://leetcode.cn/problems/first-unique-character-in-a-string/)

```cpp
方法一：哈希表
由于只有小写字母，所以可以用大小为26的数组记录索引

思路：
初始化数组数据为字符串长度，遍历字符串，第一次遇到的字符就记录下标，第二次遇到就标记为-1，这样就可以确定不是唯一字符了，然后再遍历数组，找到除了-1之外的最小值，就是唯一字符的下标了（不能在记录下标同时记录最小下标，因为可能没有唯一字符，全为-1）

class Solution {
public:
    int firstUniqChar(string s) {
        int n=s.length();
        vector<int>vec_mp(26,n);
        int min_Idx=n;

        for(int i=0;i<n;i++){
            if(vec_mp[s[i]-'a']==n){//第一次遇到s[i]，记录下标
                vec_mp[s[i]-'a']=i;
            }else{//不是第一次遇到s[i]
                vec_mp[s[i]-'a']=-1;
            }
        }

        for(int i=0;i<26;i++){
            if(vec_mp[i]!=-1)min_Idx=min(min_Idx,vec_mp[i]);
        }

        return min_Idx==n?-1:min_Idx;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [594. 最长和谐子序列](https://leetcode.cn/problems/longest-harmonious-subsequence/)

```cpp
方法一：哈希表
由于最大值和最小值的差值等于1，意味着序列中只能有两种数，x和x+1，所以用哈希表记录各个数出现的次数，然后遍历哈希表，如果key=x，那么其对应的子序列长度是mp[x]+mp[x+1]，即x和x+1出现的次数之和，如何没有x+1，则不考虑x

卡点：
我卡在了没有意识到对于最小值和最大值相差为1的序列，实际上只有两个值，即x和x+1，所以只需要记录各个数的词频即可

class Solution {
public:
    int findLHS(vector<int>& nums) {
        int maxLen=0;
        unordered_map<int,int>mp;
        for(auto&n:nums){
            mp[n]++;
        }
        for(auto&pair:mp){
            int x=pair.first;
            if(mp.count(x+1))maxLen=max(maxLen,mp[x]+mp[x+1]);
        }
        return maxLen;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
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
每个数都会有属于自己的循环数，所以用set记录当前n是否出现过，如果出现过，代表进入循环了，无法得到1，否则早退出了

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

#### [205. 同构字符串](https://leetcode.cn/problems/isomorphic-strings/)

![image-20230808223855470](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230808223855470.png)

```cpp
方法一：双向映射哈希表
如果只有英文字母组合，那就用数组当哈希表即可，比如a映射b，就是vec['a'-'a']='b'。但是s和t是由任意有效的ASCII字符组成的

思路：
1.哈希表mp记录映射关系
2.考虑不同字符映射到同一个字符的情况，所以相当于映射和被映射的字符是一对一的关系，因此需要另一张表记录反向映射

class Solution {
public:
    bool isIsomorphic(string s, string t) {
        if(s.length()!=t.length())return false;

        unordered_map<char,char>mp_st;//s[i]->t[i]
        unordered_map<char,char>mp_ts;//t[i]->s[i]
        for(int i=0;i<s.length();i++){
            if(!mp_st.count(s[i])){
                //t[i]映射到s[i]，但是t[i]已经被映射过了，再被s[i]映射的话就不合规了
                if(mp_ts.count(t[i]))return false;
                //互相都没映射，首次建立映射关系
                else{
                    mp_st[s[i]]=t[i];
                    mp_ts[t[i]]=s[i];
                }
            }else{
                //对于主动映射字符s[i]就要检查是否映射多个字符了
                //对于被映射的字符t[i]就要检查是否被多个字符映射了
                //s[i]映射字母不等于t[i]或者t[i]映射字母不等于s[i](包括了t[i]无映射字母的情况)
                if(mp_st[s[i]]!=t[i]||mp_ts[t[i]]!=s[i])return false;
            }
        }
        return true;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [290. 单词规律](https://leetcode.cn/problems/word-pattern/)

```cpp
方法一：双向映射哈希表
和205同构字符串类似，只不过是字符和字符串的爽想要映射，但是思路是一样的

class Solution {
public:
    bool wordPattern(string pattern, string s) {
        unordered_map<char,string>mp_ps;//pattern[i]->word
        unordered_map<string,char>mp_sp;//word->pattern[i]

        int i=0;
        int j=0;
        int m=pattern.length();
        int n=s.length();
        while(i<m&&j<n){
            int start=j;//记录当前单词起始下标
            while(j<n&&s[j]!=' ')j++;//遍历当前单词
            string word=s.substr(start,j-start);//获取当前单词
            //判断映射情况
            if(!mp_ps.count(pattern[i])){
                //pattern[i]没有映射字符串，但是当前单词已经被映射过了，再被pattern[i]映射的话就不合规了
                if(mp_sp.count(word))return false;
                //双向映射
                mp_ps[pattern[i]]=word;
                mp_sp[word]=pattern[i];
            }else{
                if(mp_ps[pattern[i]]!=word||mp_sp[word]!=pattern[i])return false;
            }
            i++;
            j++;//跳过空格
        }
        return i>m-1&&j>n-1;//所有字符和单词的都映射完了，且合规
    }
};
时空复杂度分析:
时间复杂度：O(max(m,n));
空间复杂度：O(m+n);
```

#### [599. 两个列表的最小索引总和](https://leetcode.cn/problems/minimum-index-sum-of-two-lists/)

```cpp
方法一：哈希表

class Solution {
public:
    vector<string> findRestaurant(vector<string>& list1, vector<string>& list2) {
        //以O(1)的时间获取list2中各餐厅和其下标
        unordered_map<string,int>mp2;
        for(int i=0;i<list2.size();i++){
            mp2[list2[i]]=i;
        }

        int minIdxSum=INT_MAX;//最小下标和
        vector<string>res;//结果集

        for(int i=0;i<list1.size();i++){
            string canteen=list1[i];
            //如果list2有当前餐厅并且索引和更小，则清空结果集，重新加入餐厅
            if(mp2.count(canteen)&&i+mp2[canteen]<minIdxSum){
                minIdxSum=i+mp2[canteen];//更新最小索引和
                res.clear();
                res.push_back(canteen);
            }
            //如果list2有当前餐厅并且索引和相等，则直接加入餐厅
            else if(mp2.count(canteen)&&i+mp2[canteen]==minIdxSum){
                res.push_back(canteen);
            }
        }
        
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(m+n);
空间复杂度：O(m);
其中m是list1的大小，n是list2的大小
```

#### [645. 错误的集合](https://leetcode.cn/problems/set-mismatch/)

```cpp
方法一：哈希表
思路：
用哈希表记录各个数值出现的频率，然后从1遍历到n，如果中间某个值对应的value是0，代表这是缺失的值；而如果value是2，代表这是重复出现的值

class Solution {
public:
    vector<int> findErrorNums(vector<int>& nums) {
        int n=nums.size();
        int repeatVal=0;//重复值
        int lostVal=0;//缺失值
        unordered_map<int,int>mp;
        for(auto&n:nums){
            mp[n]++;
        }

        for(int i=1;i<=n;i++){
            if(!mp.count(i))lostVal=i;
            if(mp[i]==2)repeatVal=i;
        }

        return {repeatVal,lostVal};
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);

一开始的错误想法：
数组排序后，nums[i]应该等于i+1，如果不等，代表nums[i]是重复出现的值，并且i+1是丢失的整数。但实际上nums[i]不一定是重复出现的值。比如[1,3,4,5,5]，nums[1]!=1+1，会以为重复出现的值是3，但实际上重复出现的值是5，这是因为缺失的值和重复出现的值是没有任何关系的，都是随机的
```

#### [884. 两句话中的不常见单词](https://leetcode.cn/problems/uncommon-words-from-two-sentences/)

```cpp
方法一：哈希表
两张哈希表分别记录s1和s2个单词出现的次数

class Solution {
public:
    vector<string> uncommonFromSentences(string s1, string s2) {
        unordered_map<string,int>mp1;
        unordered_map<string,int>mp2;
        vector<string>res;

        int index1=0;
        while(index1<s1.length()){
            int start=index1;
            while(index1<s1.length()&&s1[index1]!=' ')index1++;
            string word=s1.substr(start,index1-start);
            mp1[word]++;
            index1++;
        }
        int index2=0;
        while(index2<s2.length()){
            int start=index2;
            while(index2<s2.length()&&s2[index2]!=' ')index2++;
            string word=s2.substr(start,index2-start);
            mp2[word]++;
            index2++;
        }

        for(auto&p1:mp1){
            if(p1.second==1&&!mp2.count(p1.first))res.push_back(p1.first);
        }
        for(auto&p2:mp2){
            if(p2.second==1&&!mp1.count(p2.first))res.push_back(p2.first);
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n+m);
空间复杂度：O(n+m);
n是s1的长度，m是s2的长度
```

#### [1207. 独一无二的出现次数](https://leetcode.cn/problems/unique-number-of-occurrences/)

```cpp
方法一：哈希表+哈希集合
由于只有2001个数，可以使用数组记录每个数的出现次数，再用哈希集合记录各出现次数然后找是否有重复的

class Solution {
public:
    bool uniqueOccurrences(vector<int>& arr) {
        int count[2001]={0};
        for(auto&n:arr){
            count[n+1000]++;
        }

        unordered_set<int>s;
        for(int i=0;i<2001;i++){
            if(count[i]==0)continue;//不算0
            if(s.count(count[i]))return false;
            s.insert(count[i]);
        }

        return true;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
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

#### [720. 词典中最长的单词](https://leetcode.cn/problems/longest-word-in-dictionary/)

```cpp
方法一：哈希集合
关键：
1.排序words，使得字典序小的单词在前，并且相似的单词聚在一起
2.最长的单词必须是由单个字母累加而来的，比如world就必须由[w,wo,wor,worl,world]累加得到

class Solution {
public:
    string longestWord(vector<string>& words) {
        sort(words.begin(),words.end());//排序words
        unordered_set<string>s;//记录有效的累加单词
        s.insert("");//用于处理单字母删去一个字母后变为空字符串的情况
        string res;//结果字符串
        for(auto&word:words){
            if(s.count(word.substr(0,word.size()-1))){
                res=word.length()>res.length()?word:res;
                s.insert(word);//能够从单字母累加上来的单词才是有效的，才能加入s中
            }
        }
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(n);

方法二：字典树
从根节点到叶子节点的一条路径代表着该条路径涉及的字母由上到下组合起来的单词是由单个字母一个一个字母累加而来的
关键：
1.每个节点维护一个大小为26的子节点数组children，因为由26个小写字母
2.isEnd用于判断是否存在从根节点到当前节点组成的单词，方便判断下一个单词是否可以由当前单词累加一个字母得到

class Trie{
public:
    Trie(){
        this->children=vector<Trie*>(26,nullptr);
        this->isEnd=false;
    }

    void insert(const string&word){
        Trie*node=this;
        for(const auto&c:word){
            int index=c-'a';
            if(node->children[index]==nullptr){
                node->children[index]=new Trie();
            }
            node=node->children[index];
        }
        node->isEnd=true;
    }

    bool search(const string&word){
        Trie*node=this;
        for(const auto&c:word){
            int index=c-'a';
            //如果不存在字母c的路径或者该单词不是有其他单词添加一个字母而来的，返回false
            /*
            对于第二个条件：对于路径上所有节点(除叶子节点外)，isEnd必须都为true
            因为既然是由其他单词累加一个字母得到，比如world，就必须存在单词[w,wo,wor,worl]
            所以路径上的节点的isEnd由于在insert中的处理，都会被置为true
            */
            if(node->children[index]==nullptr||!node->children[index]->isEnd){
                return false;
            }
            node=node->children[index];
        }
        return node!=nullptr&&node->isEnd;
    }

private:
    vector<Trie*>children;
    bool isEnd;
};

class Solution {
public:
    string longestWord(vector<string>& words) {
        Trie trie;
        for(auto& word:words){
            trie.insert(word);
        }
        string res;
        for(auto& word:words){
            if(trie.search(word)){
                if(word.length()>res.length()||word.length()==res.length()&&word<res){
                    res=word;
                }
            }
        }
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(nm);
空间复杂度：O(nm);
n是words大小，m是最长words[i]的长度
```

#### [970. 强整数](https://leetcode.cn/problems/powerful-integers/)

```cpp
方法一：枚举+哈希集合

class Solution {
public:
    vector<int> powerfulIntegers(int x, int y, int bound) {
        if(bound<2)return {};//x^i和y^i最小值是2
        if(x==1){
            if(y==1)return {2};//x，y都是1，只能得到2
            else swap(x,y);//保证只能y是1，方便后面处理死循环
        }

        vector<int>res;//结果集
        unordered_set<int>s;//记录已加入结果集的值，避免重复添加
        for(int i=0;pow(x,i)<bound;i++){
            for(int j=0;pow(x,i)+pow(y,j)<=bound;j++){
                if(j>0&&y==1)break;//如果y==1，要注意避免死循环，因为1^n=1，处理一次就够了
                int powSum=pow(x,i)+pow(y,j);
                if(!s.count(powSum)){
                    res.push_back(powSum);
                    s.insert(powSum);
                }
            }
        }
        return res;
    }
};
时空复杂度分析:
时间复杂度：O((logbound)^2);
空间复杂度：O((logbound)^2);
```

#### [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

```cpp
方法一：滑动窗口+哈希集合
思路：
因为是找子串，所以需要一个滑动窗口来考虑所有可能的子串，而无重复字符就是滑动窗口变化的条件

class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_set<char>st;//记录当前无重复子串中的字符
        int maxLen=0;//如果是空字符串，不会进入while循环，直接返回0
        int start=0;
        int end=0;

        while(end<s.length()){
            //不断的加入新字符，直到有重复字符的出现
            while(end<s.length()&&!st.count(s[end])){
                st.insert(s[end]);
                end++;
            }
            //当前s[end]是重复字符，s[start,end-1]子串是无重复字符子串
            maxLen=max(maxLen,end-start);
            //不断删去前面的字符，直到[start,end]中没有重复字符s[end]
            while(start<end&&st.count(s[end])){
                st.erase(s[start]);
                start++;
            }
        }

        return maxLen;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)

```cpp
方法一：快排思想
这里的快排思想是找一个pivot，然后小于pivot的在其右边，大于pivot的在其左边，函数partion返回值是排完序后的pivot所在下标，假设其当前下标是n，由于nums[0~n-1]虽然没排好序，但是都大于pivot，所以pivot就是nums第n+1大的值，因此要找第k大的值，只需要在某次partion时找到返回值为k-1的下标pos，此时nums[pos]就是nums第k大的值

class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        int left=0,right=nums.size()-1;
        int res=0;
        while(left<=right){
            int pos=partion(nums,left,right);
            if(pos==k-1){
                res=nums[pos];
                break;
            }else if(pos<k-1){
                left=pos+1;
            }else{
                right=pos-1;
            }
        }
        return res;
    }

private:
    int partion(vector<int>&nums,int left,int right){
        int pivot=nums[left];
        int l=left+1,r=right;
        while(l<=r){
            if(nums[l]<pivot&&nums[r]>pivot){
                swap(nums[l++],nums[r--]);
            }
            if(nums[l]>=pivot){
                l++;
            }
            if(nums[r]<=pivot){
                r--;
            }
        }
        //此时nums[r]>=pivot而nums[l]<=pivot，由于pivot左边的元素要比其大，所以交换nums[r]过去
        swap(nums[left],nums[r]);
        //r是pivot当前的下标，代表着pivot是第r+1大的值
        return r;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(logn);

方法二：优先队列（小顶堆）
维护数组nums中最大的k个数，其中这k个数中，最小的那个就是第k大的数。过程中如果队列元素数量超过k个，删掉最小的那个，因为此时队列中有k+1个数，他都比其他k个数小，至多只可能是第k+1大的数，不可能是第k大的数

class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        //小顶堆，维护k个元素，遍历玩nums后可以保证堆顶是第k大的数，因为数组中比其大的k-1个数都是堆里了
        priority_queue<int,vector<int>,greater<int>>p_que;
        for(auto&n:nums){
            p_que.push(n);
            if(p_que.size()>k)p_que.pop();
        }
        return p_que.top();
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(k);
```

#### [347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)

```cpp
方法一：优先队列+哈希表
优先队列：左边是队尾，右边是队头
小根堆：左大右小
大根堆：左小右大
求前K大：要把小的删去，队列只能删队头，所以小的在队头，即在右边，因此是小根堆，p1在p2左边，所以排序算法是p1.second>p2.second
求前K小：要把大的删去，队列只能删队头，所以大的值在队头，即右边，因此是大根堆，p1在p2左边，所以排序算法是p1.second<p2.second
关键：把队列想象成队尾在左，队头在右，即左进右出。队头就代表着大小根堆的根

class Solution {
public:
    //自定义比大小
    struct cmp{
        bool operator()(pair<int,int>&p1,pair<int,int>&p2){
            return p1.second>p2.second;
        }
    };

    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int,int>mp;//记录元素词频
        priority_queue<pair<int,int>,vector<pair<int,int>>,cmp>pq;//存储前k个高频次所对应的元素
        vector<int>res;//结果集

        //记录词频
        for(auto& n:nums){
            mp[n]++;
        }

        //维护前k个高频元素
        for(auto& pair:mp){
            pq.push(pair);
            if(pq.size()>k)pq.pop();
        }

        //遍历小顶堆，将元素加入结果集中
        while(!pq.empty()){
            res.push_back(pq.top().first);
            pq.pop();
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(nlogk);
空间复杂度：O(n);
```

#### [380. O(1) 时间插入、删除和获取随机元素](https://leetcode.cn/problems/insert-delete-getrandom-o1/)

```cpp
方法一：哈希表+数组
哈希表：key存储元素值，value值可以存储各元素对应索引
数组：方便通过取模来获取随机数，删除元素时，可以通过哈希表获取元素对应索引，然后将该值和最后一个值交换，最后再删去最后一个元素即可

class RandomizedSet {
public:
    RandomizedSet() {
        //随机数随时间而变
        srand((unsigned)time(NULL));
    }
    
    bool insert(int val) {
        if(mp.count(val))return false;//已存在，返回false
        int idx=nums.size();//新元素下标
        nums.emplace_back(val);
        mp[val]=idx;//记录新元素及其对应下标
        return true;
    }
    
    bool remove(int val) {
        if(!mp.count(val))return false;//如果不存在val，返回false
        int idx=mp[val];//获取val下标
        int last=nums.back();//获取数组最后一个值
        nums[idx]=last;//将最后一个值备份到即将删去的元素所在下标
        mp[last]=idx;//并更新最后一个元素的下标
        nums.pop_back();//删去最后一个元素，val被最后一个元素覆盖，直接删去最后一个元素即可
        mp.erase(val);//哈希表中也删去val对应的记录
        return true;
    }
    
    int getRandom() {
        //通过对数组大小随机取模获取到0~nums.size()-1的数，这样每个下标被得到的概率是相同的
        return nums[rand()%nums.size()];
    }

private:
    vector<int>nums;
    unordered_map<int,int>mp;
};

本题的关键在于用数组存储元素，哈希表记录元素及其对应下标，这样可以通过常数时间获取到元素下标，然后通过其和最后一个元素交换，最后直接删去最后一个元素即可，
```

#### [451. 根据字符出现频率排序](https://leetcode.cn/problems/sort-characters-by-frequency/)

```cpp
方法一：哈希表+优先队列

class Solution {
public:
    struct compare{
        bool operator()(const pair<char,int>&p1,const pair<char,int>&p2){
            return p1.second<p2.second;
        }
    };

    string frequencySort(string s) {
        unordered_map<char,int>mp;
        priority_queue<pair<char,int>,vector<pair<char,int>>,compare>pq;
        string res;

        for(auto&c:s){
            mp[c]++;
        }

        for(auto&pair:mp){
            pq.push(pair);
        }

        while(!pq.empty()){
            char c=pq.top().first;
            int count=pq.top().second;
            res+=string(count,c);
            pq.pop();
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n+klogk);
空间复杂度：O(n+k);
```

#### [648. 单词替换](https://leetcode.cn/problems/replace-words/)

```cpp
方法一：哈希集合
哈希集合可以保证常数时间验证词根

class Solution {
public:
    string replaceWords(vector<string>& dictionary, string sentence) {
        string res;
        unordered_set<string>st(dictionary.begin(),dictionary.end());
        int n=sentence.length();

        int idx=0;
        while(idx<n){
            string word;
            //遍历当前单词，看是否有对应词根，并且由于是一个个从短到长，可以保证是最短词根
            while(idx<n&&sentence[idx]!=' '){
                word.push_back(sentence[idx]);
                //word是词根，词根后面的字母就可以直接跳过了
                if(st.count(word)){
                    res+=word+" ";
                    while(idx<n&&sentence[idx]!=' ')idx++;
                    idx++;//越过空格，或者保证处理完最后一个具有词根的单词后，idx是n+1，方便后面判断
                    break;
                }
                idx++;
            }
            //退出循环有三种情况：
            //1.idx==n (最后一个单词 但word还未处理)
            //2.sentence[idx]==' ' (word还未处理)
            //3.idx==n+1 (最后一个单词 且word已处理)
			
            //如果idx==n+1，说明最后一个单词具有词根，已经处理完了，直接break
            if(idx==n+1)break;
            //1.不是最后一个单词，则sentence[idx]==' '
            //2.是最后一个单词，则idx==n，如果idx=n+1说明最后一个单词具有词根，不用在这处理
            else if(idx==n||sentence[idx]==' '){//越界的先在前面判断
                res+=word+" ";
                idx++;
            }
        }

        res.pop_back();//删去最后一个多余的空格
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(m);
```

#### [692. 前K个高频单词](https://leetcode.cn/problems/top-k-frequent-words/)

```cpp
方法一：哈希表

class Solution {
public:
    bool static compare(const pair<string,int>&p1,const pair<string,int>&p2){
        if(p1.second==p2.second)return p1.first<p2.first;
        return p1.second>p2.second;
    }

    vector<string> topKFrequent(vector<string>& words, int k) {
        unordered_map<string,int>mp(words.size());
        for(auto&word:words)mp[word]++;

        vector<pair<string,int>>vec(mp.begin(),mp.end());
        sort(vec.begin(),vec.end(),compare);

        vector<string>res;
        res.reserve(k);
        auto beg=vec.begin();

        while(k--){
            res.push_back(beg->first);
            beg++;
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(nlogk);
空间复杂度：O(n);

方法二：优先队列+哈希表
class Solution {
public:
    struct compare{
        bool operator()(const pair<string,int>&p1,const pair<string,int>&p2){
            if(p1.second==p2.second)return p1.first>p2.first;
            return p1.second<p2.second;
        }
    };

    vector<string> topKFrequent(vector<string>& words, int k) {
        unordered_map<string,int>mp;
        priority_queue<pair<string,int>,vector<pair<string,int>>,compare>pq;
        vector<string>res;

        for(auto&word:words){
            mp[word]++;
        }

        for(auto&pair:mp){
            pq.push(pair);
        }
		
        //由于是取k次优先队列头（而没有删除），所以频次大的或同频次但字典序小的要放在右边，因此要注意排序规则
        while(k--){
            res.emplace_back(pq.top().first);
            pq.pop();
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(nlogk);
空间复杂度：O(n);
```

#### [718. 最长重复子数组](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/)

> `数组或字符串的动态规划为什么状态表的含义都必须是以某个特定的元素作为截止元素？`
>
> 在动态规划中，状态表的含义以特定的元素作为截止元素，是为了将问题分解成子问题，并且避免重复计算。动态规划常用于解决具有重叠子问题特性的问题，通过存储子问题的结果，可以避免重复计算，提高算法的效率。
>
> 比如，对于数组nums1{3,2,1,4}和nums2{3,2,1,0}，要找它们的最长重复子数组长度，就可以先找{3,2,1}和{3,2,1}的最长重复子数组长度，依次类推下去，就把原本需要重复计算的结果记录下来。对于原始数组nums1和nums2分别是以各自的最后一个元素结尾，而对于子问题{3,2,1}和{3,2,1}也必须以各自的最后一个元素结尾才是等价的子问题，这就是为什么状态表的涵义都必须是以某个元素作为最后一个元素处理，因为不能擅自改变原始数据

```cpp
方法一：动态规划

class Solution {
public:
    int findLength(vector<int>& nums1, vector<int>& nums2) {
        int n=nums1.size();
        int m=nums2.size();
        
		//经典套路：
        //dp[i][j]:以nums1[i-1]结尾的子数组和以nums2[j-1]结尾的子数组的最长公共子数组长度
        
        //必须准确地考虑到以每一个数字结尾的情况，这样才能处理到所有的情况
        //如果是按照子数组nums1[0,i-1]和子数组nums2[0,j-1]最长公共子数组长度来理解，
        //dp[i][j]意义不清，因为不是求子序列，而是子数组，没有删除元素这一说法，必须是连续的，
        //所以需要确切的结尾元素来区分各dp[i][j]的涵义
        vector<vector<int>>dp(n+1,vector<int>(m+1,0));
        int maxLen=0;

        for(int i=1;i<=n;i++){
            for(int j=1;j<=m;j++){
                if(nums1[i-1]==nums2[j-1]){
                    dp[i][j]=dp[i-1][j-1]+1;//子数组必须是连续的，所以看这两个子数组之前是否也相同
                    maxLen=max(maxLen,dp[i][j]);
                }
                //不相等的话就不连续了，而子数组是默认连续的，因此就是0
            }
        }

        return maxLen;
    }
};
时空复杂度分析:
时间复杂度：O(nm);
空间复杂度：O(nm);
```



### Hard



## 二分查找

### Easy

#### [69. x 的平方根](https://leetcode.cn/problems/sqrtx/)

```cpp
方法一：二分法

class Solution {
public:
    int mySqrt(int x) {
       int left=1;
       int right=x;

       while(left<=right){
           int mid=left+((right-left)>>1);
           if((long)mid*mid==x){//转成long，防止溢出
               return mid;
           }
           //mid只能处于[left,right]范围内，当前范围都是经过过前面的筛选后得到的
           //因此小于left或大于right，都不是最终答案，因为其平方要么过小、要么过大
           
           //left==right时，mid=left=right，如果mid*mid!=x，即将退出循环，会有以下两种情况：
           //1.mid*mid>x:right=mid-1后，此时right*right<x，但是(right+1)*(right+1)>x，返回right
           //2.mid*mid<x:left=mid+1后，此时left*left>x，但是(left-1)*(left-1)>x，应当返回left-1
           //  刚好等于right
           //所以退出循环后，应当返回right
           else if((long)mid*mid>x){
               right=mid-1;
           }else{
               left=mid+1;
           }
       }
       
       //退出循环后，right<left，返回小的那个；如果返回大的left，left*left>x，不符合题意
       return right;
    }
};
时空复杂度分析:
时间复杂度：O(logx);
空间复杂度：O(1);
```

#### [278. 第一个错误的版本](https://leetcode.cn/problems/first-bad-version/)

```cpp
方法一：二分查找

// The API isBadVersion is defined for you.
// bool isBadVersion(int version);

class Solution {
public:
    int firstBadVersion(int n) {
        int low=1;
        int high=n;
        while(low<high){
            int mid=low+((high-low)>>1);
            if(isBadVersion(mid)){
                high=mid;//继续向前判断前面是否有更小的错误版本
            }else{
                low=mid+1;
            }
        }
        return high;
    }
};
时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```

#### [350. 两个数组的交集 II](https://leetcode.cn/problems/intersection-of-two-arrays-ii/)

```cpp
方法一：哈希表

class Solution {
public:
    vector<int> intersect(vector<int>& nums1, vector<int>& nums2) {
        unordered_map<int,int>mp1;
        unordered_map<int,int>mp2;
        vector<int>res;

        for(auto&n:nums1)mp1[n]++;
        for(auto&n:nums2)mp2[n]++;

        for(auto&p1:mp1){
            if(mp2.count(p1.first)){
                int cnt=min(p1.second,mp2[p1.first]);
                res.insert(res.end(),cnt,p1.first);
            }
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(m+n);
空间复杂度：O(m+n);
```

#### [367. 有效的完全平方数](https://leetcode.cn/problems/valid-perfect-square/)

```cpp
方法一：二分查找

class Solution {
public:
    bool isPerfectSquare(int num) {
        int low=1;
        int high=num;
        while(low<=high){
            int mid=low+((high-low)>>1);
            if((long)mid*mid==num){
                return true;
            }else if((long)mid*mid<num){
                low=mid+1;
            }else{
                high=mid-1;
            }
        }
        return false;
    }
};
时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```

#### [374. 猜数字大小](https://leetcode.cn/problems/guess-number-higher-or-lower/)

```cpp
方法一：二分查找

/** 
 * Forward declaration of guess API.
 * @param  num   your guess
 * @return 	     -1 if num is higher than the picked number
 *			      1 if num is lower than the picked number
 *               otherwise return 0
 * int guess(int num);
 */

class Solution {
public:
    int guessNumber(int n) {
        int low=1;
        int high=n;
        while(low<=high){
            int num=low+((high-low)>>1);
            if(guess(num)==0){
                return num;
            }else if(guess(num)==1){
                low=num+1;
            }else{
                high=num-1;
            }
        }
        return -1;
    }
};
时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```

#### [392. 判断子序列](https://leetcode.cn/problems/is-subsequence/)

```cpp
方法一：双指针

class Solution {
public:
    bool isSubsequence(string s, string t) {
        int sIdx=0;
        int tIdx=0;
        int m=s.length();
        int n=t.length();
        while(tIdx<n){
            //如果当前t[tIdx]和s[sIdx]匹配，就后移sIdx，去匹配下一个
            if(s[sIdx]==t[tIdx])sIdx++;
            if(sIdx==m)return true;//s的所有字符都被按序匹配完了，就说明s是t的子序列
            tIdx++;//无论匹不匹配，tIdx都要后移，不匹配就相当于删去当前t[tIdx]，不考虑它
        }
        return m==0;//空字符串必然是t的子序列
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [441. 排列硬币](https://leetcode.cn/problems/arranging-coins/)

```cpp
方法一：二分查找
利用等差数列求和公式，能够以常数时间获取到n个完整行的总硬币数

class Solution {
public:
    int arrangeCoins(int n) {
        int low=1;
        int high=n;
        while(low<=high){
            int mid=low+((high-low)>>1);
            //mid*(mid+1)/2:mid个完整行的硬币数量
            if((long)mid*(mid+1)/2==n){
                return mid;
            }else if((long)mid*(mid+1)/2<n){
                low=mid+1;
            }else{
                high=mid-1;
            }
        }
        return high;
    }
};
时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```

#### [704. 二分查找](https://leetcode.cn/problems/binary-search/)

```cpp
方法一：二分查找

class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left=0;
        int right=nums.size()-1;
        while(left<=right){
            int mid=left+((right-left)>>1);
            if(nums[mid]==target){
                return mid;
            }else if(nums[mid]<target){
                left=mid+1;
            }else{
                right=mid-1;
            }
        }
        return -1;
    }
};
时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```

#### [744. 寻找比目标字母大的最小字母](https://leetcode.cn/problems/find-smallest-letter-greater-than-target/)

```cpp
方法一：二分查找

class Solution {
public:
    char nextGreatestLetter(vector<char>& letters, char target) {
        int left=0;
        int right=letters.size()-1;
        while(left<=right){
            int mid=left+((right-left)>>1);
            if(letters[mid]<=target){
                left=mid+1;
            }else{
                //letters[mid]>target，不能简单地就让right=mid-1，因为我们要找的字母也是大于target的，所以需要判断其是否满足条件
                //判断mid前一个(如果存在)字母是否小于等于target，如果是，当前mid对应字母就符合要求了
                if(mid>0&&letters[mid-1]<=target)return letters[mid];
                right=mid-1;
            }
        }
        //while循环中都没有返回结果，说明不存在或者target<=letters[0]，都返回letters[0]
        return letters[0];
    }
};
时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```



### Medium

#### [167. 两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)

```cpp
方法一：双指针

class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int left=0;
        int right=numbers.size()-1;
        vector<int>res;

        while(left<right){//保证left和right不等， 即不会重复使用同一个元素
            int sum=numbers[left]+numbers[right];
            if(sum==target){
                res={left+1,right+1};
                break;
            }else if(sum<target){//过小
                left++;
            }else{//过大
                right--;
            }
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);

方法二：二分查找
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        for(int i=0;i<numbers.size();i++){
            //left=i+1:保证在i后面找到对应值，避免重复使用同一元素，同时保证mid>i
            //一开始我是在整个numbers寻找target-numbers[i]，但这样会出现的问题：
            //1.重复计算了，因为numbers[0,i-1]的元素肯定都和numbers[i]匹配过了
            //2.由于有相同值的情况，而如果是在整个numbers使用二分查找，每次返回的都是同一个下标，不好区分
            int left=i+1,right=numbers.size()-1;
            while(left<=right){
                int mid=left+((right-left)>>1);
                if(numbers[mid]+numbers[i]==target){
                    return {i+1,mid+1};
                }else if(numbers[mid]+numbers[i]<target){
                    left=mid+1;
                }else{
                    right=mid-1;
                }
            }
        }
        return {-1,-1};
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(1);
```

#### [475. 供暖器](https://leetcode.cn/problems/heaters/)

```cpp
方法一：排序+二分查找
思路：
排序heaters数组，遍历houses数组，依次找到每个房屋左边和右边最近的加热器，计算房屋与这两个加热器的距离，其中最小值是可覆盖"当前房屋"的最小半径，即加热器的加热半径至少要是这个值，当前房屋才能被覆盖，而对于所有房屋的最小覆盖半径中的最大值，就是整个heaters数组能够覆盖整个houses数组的最小值

class Solution {
public:
    int findRadius(vector<int>& houses, vector<int>& heaters) {
        int minRadius=0;
        int n=heaters.size();
        sort(heaters.begin(),heaters.end());
        
        for(int& house:houses){
            int right=upper_bound(heaters.begin(),heaters.end(),house)-heaters.begin();//house右边最近的加热器
            int left=right-1;//house左边最近的加热器
            //right>=n，house右边没有加热器
            int rightDistance = right>=n?INT_MAX:heaters[right]-house;
            //left<=-1，house左边没有加热器
            int leftDistance = left<0?INT_MAX:house-heaters[left];

            minRadius=max(minRadius,min(leftDistance,rightDistance));
        }
        return minRadius;
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(1);
```

#### [29. 两数相除](https://leetcode.cn/problems/divide-two-integers/)

```cpp
方法一：类二分查找
思路：
1.判断边界情况
2.统一符号，都是负数。为什么不是正数？：INT_MIN转为正数为溢出

class Solution {
public:
    int divide(int dividend, int divisor) {
        if(dividend==0)return 0;
        if(dividend==INT_MIN&&divisor==-1)return INT_MAX;
        if(dividend==INT_MIN&&divisor==1)return INT_MIN;//由于res是正数，暂时可能会是INT_MAX+1，溢出了，所以提前判断

        bool isNeg=(dividend>0)^(divisor>0);
        if(dividend>0)dividend=-dividend;
        if(divisor>0)divisor=-divisor;

        int res=0;
        vector<int>candidates;
        candidates.push_back(divisor);
        //1/10/100/1000……(二进制)个divisor
        while(candidates.back()>=dividend-candidates.back()){
            candidates.push_back(candidates.back()+candidates.back());
        }
        
        for(int i=candidates.size()-1;i>=0;i--){
            if(dividend<=candidates[i]){
                res+=(1<<i);
                dividend-=candidates[i];
            }
        }

        return isNeg?-res:res;
    }
};
时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(logn);
n=dividend

方法二：位运算
class Solution {
public:
    int divide(int dividend, int divisor) {
        if(dividend==0)return 0;
        if(dividend==INT_MIN&&divisor==-1)return INT_MAX;

        long x=dividend;
        long y=divisor;

        bool isNeg=(x>0)^(y>0);
        //转成正数，因为负数的符号位1，在右移完之后，会被当作值来处理，影响结果。
        if(x<0)x=-x;
        if(y<0)y=-y;

        long res=0;
        for(int i=31;i>=0;i--){
            if((x>>i)>=y){//x有2^i个y
                res+=(1<<i);
                x-=y<<i;
            }
        }

        return isNeg?-res:res;
    }
};
时空复杂度分析:
时间复杂度：O(1);
空间复杂度：O(1);
```

#### [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

```cpp
方法一：二分查找

class Solution {
public:
    //最右边
    int rightBinarySearch(vector<int>&nums,int target){
        int left=0;
        int right=nums.size()-1;
        while(left<=right){
            int mid=left+((right-left)>>1);
            if(nums[mid]==target){
                if(mid==nums.size()-1||nums[mid+1]>target)return mid;
                left=mid+1;
            }else if(nums[mid]<target){
                left=mid+1;
            }else{
                if(mid>0&&nums[mid-1]==target)return mid-1;
                right=mid-1;
            }
        }
        return -1;
    }
	
    //最左边
    int leftBinarySearch(vector<int>&nums,int target){
        int left=0;
        int right=nums.size()-1;
        while(left<=right){
            int mid=left+((right-left)>>1);
            if(nums[mid]==target){
                if(mid==0||nums[mid-1]<target)return mid;
                right=mid-1;
            }else if(nums[mid]>target){
                right=mid-1;
            }else{
                if(mid<nums.size()-1&&nums[mid+1]==target)return mid+1;
                left=mid+1;
            }
        }
        return -1;
    }

    vector<int> searchRange(vector<int>& nums, int target) {
        int leftPos=leftBinarySearch(nums,target);
        int rightPos=rightBinarySearch(nums,target);
        return{leftPos,rightPos};
    }
};
时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```

#### [153. 寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/)

```cpp
方法一：二分查找
关键：
找到区分左右递增子数组的元素，即nums[0]和nums[n-1]，其实只要找到两子数组的最值即可
左子数组的最小值就是nums[0]，最大值不清楚
右子数组的最大值就是nums[n-1]，最小值不清楚，因为是我们要找的。
而左子数组的所有元素必然都大于等于nums[0]，右子数组的所有元素必然也都小于等于nums[n-1]。因此，通过互斥事件可推出，小于nums[0]的元素肯定在右子数组，大于nums[n-1]的元素肯定在左子数组

class Solution {
public:
    int findMin(vector<int>& nums) {
        int n=nums.size();
        //左边递增子数组的最小值大于右边递增子数组的最大值，如果出现左边值小于右边值的情况，说明整个数组单调递增
        //或者只有一个元素
        if(nums[0]<nums[n-1]||n==1)return nums[0];

        //上面条件不成立，意味着nums中有左右两个递增子数组
        int left=0;
        int right=n-1;
        while(left<=right){
            int mid=left+((right-left)>>1);
            //由于mid要么在左边递增子数组要么在右边递增子数组，所以下面两个条件至少有一个且只有一个成立
            if(nums[mid]>nums[n-1]){//在左边递增子数组中
                //数组中如果出现前值大于后值的情况，说明就是两个子数组的交界处，此时右边的值就是最小值
                if(nums[mid]>nums[mid+1])return nums[mid+1];
                left=mid+1;
            }else if(nums[mid]<nums[0]){//在右边递增子数组中
                if(nums[mid-1]>nums[mid])return nums[mid];
                right=mid-1;
            }
        }
        return -1;
    }
};
时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```

#### [162. 寻找峰值](https://leetcode.cn/problems/find-peak-element/)

```cpp
方法一：二分查找

class Solution {
public:
    int findPeakElement(vector<int>& nums) {
        int n=nums.size();
        if(n==1)return 0;
        else if(n==2)return nums[0]>nums[1]?0:1;

        //至少有3个元素
        int left=0;
        int right=n-1;
        while(left<=right){
            //分析各种可能的情况：1.mid下标边界问题 2.nums[mid]和两边的值的大小关系
            int mid=left+((right-left)>>1);
            if(mid==0&&nums[mid]>nums[mid+1]){
                return mid;
            }else if(mid==n-1&&nums[mid]>nums[mid-1]){
                return mid;
            }else if(mid>0&&mid<n-1&&nums[mid]>nums[mid-1]&&nums[mid]>nums[mid+1]){
                return mid;
            }else{//上面的情况都不符合
                if(mid==0)left=mid+1;
                else if(mid==n-1)right=mid-1;
                else{
                    if(nums[mid]<nums[mid+1]){//因为nums[mid]<nums[mid+1]不符合，那就往大的值收缩
                        left=mid+1;
                    }else{
                        right=mid-1;
                    }
                }
            }
        }
        return -1;
    }
};
时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```

#### [287. 寻找重复数](https://leetcode.cn/problems/find-the-duplicate-number/)

<img src="https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230902134503373.png" alt="image-20230902134503373" style="zoom:80%;" />

<img src="https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230902133458907.png" alt="image-20230902133458907"  />

```cpp
方法一：二分查找
在nums中，小于等于nums[i]的数应该是nums[i]个，假设实际值为cnt
1. cnt<=nums[i]:重复值>nums[i]
2. cnt>nums[i]:重复值<=nums[i]
通过以上两种情况，可以用二分查找法加快搜索速度
不要被无序数组迷惑了

class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        int n=nums.size();
        int low=1;
        int high=n-1;
        int res=0;

        while(low<=high){
            int cnt=0;
            int mid=low+((high-low)>>1);
            for(int i=0;i<n;i++){
                cnt+=nums[i]<=mid;
            }
            if(cnt<=mid){
                low=mid+1;
            }else{
                res=mid;//让res记录mid，直到最后一个mid就是最终结果，结束循环后直接返回
                high=mid-1;
            }
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(1);

方法二：位运算
把nums中所有数看作二进制数，先计算nums中第i位是1的数的数量x，再计算[1,n]中n个数第i位是1的数的数量y，如果x>y，代表重复的那个数的第i位是1，考虑完所有数位之后，重复的那个数哪些数位是1都可以知道，也就可以计算出它的值了

本质上就是看数组nums和数组{1,2,3,……,n}对于某个数量计算的不同来找到重复数的特征，比如位运算方法就是找到重复数在当前数位上的数字是1，计算到第0位之后，重复数所有数位上的1都找到了，自然它的值就得出来了

class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        int n=nums.size(),ans=0;
        
        //确认二进制下最高位是哪一位
        int bit_max=31;
        while(((n-1)>>bit_max)==0){
            bit_max--;
        }

        for(int bit=bit_max;bit>=0;bit--){
            int x=0,y=0;
            for(int i=0;i<n;i++){
                if((nums[i]>>bit)&1){
                    x++;
                }
                if(i>=1&&(i>>bit)&1){
                    y++;
                }
            }

            //代表重复的数的bit位上是1
            if(x>y){
                ans|=1<<bit;
            }
        }

        return ans;
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(1);

方法三：快慢指针（Floyd判圈算法）
把索引和nums元素都看作普通值，然后nums[i]代表着i->nums[i]，即i下一个节点是nums[i]

思路：
1.转成环图
2.用Floyd判圈算法找到环的入口，即重复值(入度为2)，因为有两个索引值指向它，所以才会有环

class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        int slow=0,fast=0;
        //不能先判断while循环条件，因为一开始的slow和fast必然相等，这样永远无法进入循环
        do{
            slow=nums[slow];
            fast=nums[nums[fast]];//一次走两步
        }while(slow!=fast);

        slow=0;
        while(slow!=fast){
            slow=nums[slow];
            fast=nums[fast];
        }
        
        return slow;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [378. 有序矩阵中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-sorted-matrix/)

![](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230902141948251.png)

![image-20230902142828618](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230902142828618.png)

```cpp
方法一：优先队列

class Solution {
public:
    int kthSmallest(vector<vector<int>>& matrix, int k) {
        int n=matrix.size();
        //优先队列(大根堆)
        priority_queue<int,vector<int>,less<int>>pq;
        for(int i=0;i<n;i++){
            for(int j=0;j<n;j++){
                pq.push(matrix[i][j]);
                if(pq.size()>k){
                    pq.pop();
                }
            }
        }
        return pq.top();
    }
};
时空复杂度分析:
时间复杂度：O((n^2)logn);
空间复杂度：O(k);

方法二：二分查找

class Solution {
public:
    //找小于等于mid的元素数量cnt，并判断是否cnt>=k
    bool check(vector<vector<int>>&matrix,int mid,int k,int n){
        //初始位置在matrix[n-1][0]，因为小于等于mid的都在矩阵的左下角模块
        int i=n-1;
        int j=0;
        int cnt=0;
        while(i>=0&&j<n){
            //一列一列处理，同时利用到了矩阵行和列的非递增性
            if(matrix[i][j]<=mid){//=mid很关键，代表mid存在于matrix中
                cnt+=i+1;
                j++;//mid>=matrix[i][j]>=matrix[i-k][j](k<=i)
            }else{
                //mid<matrix[i][j]<=matrix[i][j+k](k<n-j)
                //所以当前行第j列及其后面的都大于mid，因此不用考虑当前行了，i--
                i--;
            }
        }
        return cnt>=k;
    }

    int kthSmallest(vector<vector<int>>& matrix, int k) {
        int n=matrix.size();
        int left=matrix[0][0];
        int right=matrix[n-1][n-1];

        while(left<right){
            int mid=left+((right-left)>>1);
            if(check(matrix,mid,k,n)){
                right=mid;
            }else{
                left=mid+1;
            }
        }
        
        //left一定存在于matrix中，原因如下：
        //假设第k小的元素是m，第k+1小的元素是s，则[m,s)都符合答案
		//但是由于left是第一个符合条件的答案，即m，所以left才是我们想要的答案
        return left;
    }
};
时空复杂度分析:
时间复杂度：O((nlog(r-l));
空间复杂度：O(1);
```

#### [436. 寻找右区间](https://leetcode.cn/problems/find-right-interval/)

![image-20230903152733890](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230903152733890.png)

```cpp
方法一：二分查找
思路：
把起点及其在intervals对应下标作为一个整体都放到一个集合中，然后按起点值从小到大排序，然后遍历intervals的终点，找到第一个大于或等于它的起点，即lower_bound

class Solution {
public:
    vector<int> findRightInterval(vector<vector<int>>& intervals) {
        //{start[i],i}
        vector<pair<int,int>>startIntervals;
        int n=intervals.size();
        for(int i=0;i<n;i++){
            startIntervals.emplace_back(intervals[i][0],i);
        }
        sort(startIntervals.begin(),startIntervals.end());

        vector<int>ans(n,-1);
        for(int i=0;i<n;i++){
            auto it=lower_bound(startIntervals.begin(),startIntervals.end(),make_pair(intervals[i][1],0));
            if(it!=startIntervals.end()){
                ans[i]=it->second;
            }
        }

        return ans;
    }
};
时空复杂度分析:
时间复杂度：O((nlogn);
空间复杂度：O(n);

方法二：双指针

思路：
起点数组和终点数组从小到大排序

class Solution {
public:
    vector<int> findRightInterval(vector<vector<int>>& intervals) {
        vector<pair<int,int>>startIntervals;
        vector<pair<int,int>>endIntervals;
        int n=intervals.size();

        for(int i=0;i<n;i++){
            startIntervals.emplace_back(intervals[i][0],i);
            endIntervals.emplace_back(intervals[i][1],i);
        }
        sort(startIntervals.begin(),startIntervals.end());
        sort(endIntervals.begin(),endIntervals.end());

        int i=0;
        int j=0;
        vector<int>ans(n,-1);
        while(i<n&&j<n){
            //每个终点都要有一个右边界起点，所以是定住终点的指针j，去找合适的起点，因此移动起点的指针i
            while(i<n&&endIntervals[j].first>startIntervals[i].first){
                i++;
            }
            if(i<n){
                ans[endIntervals[j].second]=startIntervals[i].second;
            }
            j++;
        }
        return ans;
    }
};
时空复杂度分析:
时间复杂度：O((nlogn);
空间复杂度：O(n);
```

#### [454. 四数相加 II](https://leetcode.cn/problems/4sum-ii/)

```cpp
方法一：哈希表
nums1和nums2一组，nums3和nums4一组，将时间复杂度从n^4降到n^2
mp[val]:nums1和nums2组成值为val的组合数

class Solution {
public:
    int fourSumCount(vector<int>& nums1, vector<int>& nums2, vector<int>& nums3, vector<int>& nums4) {
        int res=0;
        unordered_map<int,int>mp;
        for(auto&a:nums1){
            for(auto&b:nums2){
                mp[a+b]++;
            }
        }

        for(auto&c:nums3){
            for(auto&d:nums4){
                int target=-(c+d);
                if(mp.count(target)){
                    res+=mp[target];
                }
            }
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O((n^2);
空间复杂度：O(n^2);
```

#### [792. 匹配子序列的单词数](https://leetcode.cn/problems/number-of-matching-subsequences/)

```cpp
方法一：二分查找

class Solution {
public:
    int numMatchingSubseq(string s, vector<string>& words) {
        vector<vector<int>>pos(26);
        for(int i=0;i<s.length();i++){
            pos[s[i]-'a'].emplace_back(i);
        }

        int res=words.size();
        for(auto&w:words){
            if(w.length()>s.length()){
                res--;
                continue;
            }
			
            //不从0开始，因为是upper_bound，是严格大于。
            //如果s[0]==w[0]，但upper_bound无法找到，因为不是严格大于
            //但如果使用lower_bound，p可能会原地不动
            int p=-1;//当前指向s的指针遍历到的位置，初始化时还没遍历，所以不能指向0，只能指向1。
            for(auto&c:w){
                auto &ps=pos[c-'a'];//用引用，不用拷贝，减少时间
                //s中p下标之后第一个出现字符c的下标
                //upper_bound可以防止s[p]被重复利用，当前p能够被找到意味着已经被上一个字符c使用过了，所以不能重复再被使用一次
                //因此使用upper_bound，在大于下标p的情况下去找当前字符c
                auto it=upper_bound(ps.begin(),ps.end(),p);
                if(it==ps.end()){
                    res--;
                    break;
                }
                p=*it;
            }
        }

        return res;
    }
};
```



### Hard



## 栈

### Easy

#### [225. 用队列实现栈](https://leetcode.cn/problems/implement-stack-using-queues/)

```cpp
方法一：双队列

class MyStack {
public:
    queue<int>q1;
    queue<int>q2;

    MyStack() {}
    
    void push(int x) {
        q1.push(x);
    }
    
    int pop() {
        //把q1的除了尾端的其余元素放到q2暂存
        while(q1.size()>1){
            q2.push(q1.front());q1.pop();
        }
        //获取尾端元素，即栈顶
        int val=q1.front();q1.pop();
        //把放在q2暂存的元素放回q1
        while(!q2.empty()){
            q1.push(q2.front());q2.pop();
        }
        return val;
    }
    
    int top() {
        while(q1.size()>1){
            q2.push(q1.front());q1.pop();
        }
        int val=q1.front();q1.pop();
        while(!q2.empty()){
            q1.push(q2.front());q2.pop();
        }
       	//之前先出队列是为了保证元素的顺序正确
        q1.push(val);
        return val;
    }
    
    bool empty() {
        return q1.empty();
    }
};
```

#### [682. 棒球比赛](https://leetcode.cn/problems/baseball-game/)

```cpp
方法一：栈
根据ops[i]的操作，来处理分数：
1."x":直接入栈
2."+":弹栈一次，计算前两次得分和，然后再把弹栈出来的分数和得分和压栈
3."D":计算前一次得分的两倍，直接压栈
4."C":前一次分数无效，直接弹栈就好

class Solution {
public:
    int calPoints(vector<string>& operations) {
        stack<int>stk;
        int res=0;
        for(auto&op:operations){
            if(op=="+"){
                int preScore=stk.top();stk.pop();
                int newScore=preScore+stk.top();
                stk.push(preScore);
                stk.push(newScore);
            }else if(op=="D"){
                int newScore=stk.top()*2;
                stk.push(newScore);
            }else if(op=="C"){
                stk.pop();
            }else{
                //string转int
                int flag=1;
                int num=0;
                if(op[0]=='-')flag=-1;
                for(int i=0;i<op.length();i++){
                    if(op[i]=='+'||op[i]=='-')continue;
                    num=num*10+(op[i]-'0');
                }
                stk.push(flag*num);
            }
        }
        
        while(!stk.empty()){
            res+=stk.top();stk.pop();
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(nm);
空间复杂度：O(n);
其中，n是操作数，m是最大分数字符串长度
```

#### [844. 比较含退格的字符串](https://leetcode.cn/problems/backspace-string-compare/)

```cpp
方法一：栈

class Solution {
public:
    bool backspaceCompare(string s, string t) {
        stack<char>stk;
        string sRes;
        string tRes;

        for(auto&c:s){
            if(!stk.empty()&&c=='#')stk.pop();
            else if(c!='#') stk.push(c);//不能是else，不然当栈空时会push('#')
        }
        while(!stk.empty()){
            sRes.push_back(stk.top());stk.pop();
        }

        for(auto&c:t){
            if(!stk.empty()&&c=='#')stk.pop();
            else if(c!='#') stk.push(c);
        }
        while(!stk.empty()){
            tRes.push_back(stk.top());stk.pop();
        }

        return sRes==tRes;
    }
};
时空复杂度分析:
时间复杂度：O(max(n.m));
空间复杂度：O(max(n,m));
n==s.length(),m==t.lengh()
```

#### [1047. 删除字符串中的所有相邻重复项](https://leetcode.cn/problems/remove-all-adjacent-duplicates-in-string/)

```cpp
方法一：栈

class Solution {
public:
    string removeDuplicates(string s) {
        stack<char>stk;
        for(auto&c:s){
            if(!stk.empty()){
                //如果当前字符c和前一个字母相同，弹栈且c不压栈，这样就做到消除两个字母了
                if(c==stk.top()){
                    stk.pop();
                    continue;
                }
            }
            stk.push(c);
        }

        string res;
        while(!stk.empty()){
            res.push_back(stk.top());stk.pop();
        }
		
        //出栈顺序和字符串顺序相反，所以需要反转
        //反转字符串比把栈顶元素插入到res前面时间效率更高，因为插入到res前面需要不断开辟新的空间，所以采用反转字符串的方式。
        reverse(res.begin(),res.end());
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```



### Medium

#### [155. 最小栈](https://leetcode.cn/problems/min-stack/)

```cpp
方法一：双栈

class MinStack {
public:
    stack<int>stk1;
    stack<int>stk2;
    int minVal;

    MinStack() {
        minVal=INT_MAX;
    }
    
    void push(int val) {
        minVal=min(minVal,val);
        stk1.push(val);
    }
    
    void pop() {
        int val=stk1.top();stk1.pop();
        //stk2用于暂存stk1的元素，然后重新计算最小之后，再返回stk1中
        if(minVal==val){
            minVal=INT_MAX;
            while(!stk1.empty()){
                minVal=min(minVal,stk1.top());
                stk2.push(stk1.top());stk1.pop();
            }
            while(!stk2.empty()){
                stk1.push(stk2.top());stk2.pop();
            }
        }
    }
    
    int top() {
        return stk1.top();
    }
    
    int getMin() {
        return minVal;
    }
};
```

#### [71. 简化路径](https://leetcode.cn/problems/simplify-path/)

```cpp
方法一：栈
文件名：压栈
.：忽略
..：弹栈

class Solution {
public:
    string simplifyPath(string path) {
        stack<string>stk;
        int idx=0;
        int n=path.length();
        while(idx<n){
            while(idx<n&&path[idx]=='/')idx++;
            if(idx==n)break;//如果不break，可能会stk中会多一个"/"
            int start=idx;
            while(idx<n&&path[idx]!='/')idx++;
            string filename="/"+path.substr(start,idx-start);
            if(filename=="/.")continue;
            else if(filename=="/.."){
                if(!stk.empty())stk.pop();
            }
            else stk.push(filename);
        }
        if(stk.empty())return "/";//空路径返回根目录即可
		
        //stk中目录顺序是反的，所以需要另一个辅助栈将其反转
        string res;
        stack<string>tmp;
        while(!stk.empty()){
            tmp.push(stk.top());stk.pop();
        }
        while(!tmp.empty()){
            res+=tmp.top();tmp.pop();
        }
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [394. 字符串解码](https://leetcode.cn/problems/decode-string/)

```cpp
方法一：双辅助栈
思路：
一个栈用于存放数字，另一个栈用于存放字符串

class Solution {
public:
    string decodeString(string s) {
        stack<int>stk_num;
        stack<string>stk_str;
        int num=0;
        string res;
        int n=s.length();

        for(int i=0;i<n;i++){
            if(isalpha(s[i])){
                res.push_back(s[i]);
            }else if(isdigit(s[i])){
                num=num*10+(s[i]-'0');
            }else if(s[i]=='['){
                stk_num.push(num);
                //先把前面解码得到的字符串res放入栈，等下就可以直接对stk_str.top()操作，再重置res。保证后入栈的在后面
                stk_str.push(res);
                //重置
                num=0;
                res="";
            }else{//s[i]==']'
                //cnt是当前res的频次
                int cnt=stk_num.top();stk_num.pop();
                for(int k=0;k<cnt;k++)stk_str.top()+=res;//保证先入栈的在前面
                res=stk_str.top();stk_str.pop();//res是'['前的字符串，所以处理完']'后，应该把解码后的stk.top()赋值给res
            }
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [636. 函数的独占时间](https://leetcode.cn/problems/exclusive-time-of-functions/)

```cpp
方法一：栈

class Solution {
public:
    vector<int> exclusiveTime(int n, vector<string>& logs) {
        vector<int>res(n,0);
        //pair<flag,开始运行的时间戳>
        stack<pair<int,int>>stk;

        for(auto&log:logs){
            char type[10];
            int flag,timestamp;
            sscanf(log.c_str(),"%d:%[^:]:%d",&flag,type,&timestamp);
            if(type[0]=='s'){
                if(!stk.empty()){
                    //计算当前调用函数的独占时间，因为即将悬挂
                    res[stk.top().first]+=timestamp-stk.top().second;
                }
                stk.emplace(flag,timestamp);
            }else{//end:此时栈顶一定是对应的函数start信息
                res[stk.top().first]+=timestamp-stk.top().second+1;stk.pop();
                //栈顶函数结束时间戳也是栈底悬挂函数起始时间戳
                if(!stk.empty())stk.top().second=timestamp+1;
            }
        }
        return res;
    }	
};
时空复杂度分析:
时间复杂度：O(nm);
空间复杂度：O(n);
```

#### [739. 每日温度](https://leetcode.cn/problems/daily-temperatures/)

```cpp
方法一：单调栈
保证栈从栈底到栈顶的温度temperatures[i]（存下标，既能获取温度也能获取下标）是单调递减的顺序，然后如果当前温度大于栈顶温度，就说明当前温度大于栈顶温度，即res[stk.top()]=i

class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        stack<int>stk;
        vector<int>res(temperatures.size(),0);
        for(int i=0;i<temperatures.size();i++){
            while(!stk.empty()&&temperatures[i]>temperatures[stk.top()]){
                res[stk.top()]=i-stk.top();stk.pop();
            }
            stk.push(i);
        }
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [856. 括号的分数](https://leetcode.cn/problems/score-of-parentheses/)

```cpp
方法一：栈
括号的匹配一般都是栈，因为栈的FILO很符合括号的匹配顺序

思路：
eg."(()(()))"
stk初始化时先压入0
0:'('，栈压入0，stk{0,0}
1:'('，栈压入0，stk{0,0,0}
2:')'，弹栈，计算分数，stk{1,0}
3:'('，栈压入0，stk{0,1,0}
4:'('，栈压入0，stk{0,0,1,0}
5:')'，弹栈，计算分数，stk{1,1,0}
6:')'，弹栈，计算分数，stk{3,0}
7:')'，弹栈，计算分数，stk{6}
最终分数：6分
关键：遇到')'时，当前stk.top()是这对括号的分数基础，对其作出相应的计算（然后弹栈）后，加到stk.top()，是给下一对括号累加分数基础

class Solution {
public:
    int scoreOfParentheses(string s) {
        stack<int>stk;
        //由于'('和')'是成对出现的，所以最后应该留一个0在栈顶记录分数，因此提前让1个0入栈
        stk.push(0);
        for(char&c:s){
            //每对括号有自己的独立的分数计算，所以遇到一个左括号，就先让1个0入栈
            if(c=='('){
                stk.push(0);
            }else{
                //遇到右括号，可能是第一个右括号，也可能是连续遇到的第n个右括号，都无所谓，因为前面计算的分数基础都在stk.top()了
                int v=stk.top();stk.pop();
                stk.top()+=max(2*v,1);
            }
        }
        return stk.top();
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [921. 使括号有效的最少添加](https://leetcode.cn/problems/minimum-add-to-make-parentheses-valid/)

```cpp
方法一：栈
括号的匹配一般都是栈，因为栈的FILO很符合括号的匹配顺序
由于左右括号至少要成对出现才能是有效的，但不能只考虑数量，还要考虑左右括号的相对位置，因为右括号必须在左括号右边才是有效的，因此用栈合适不过了，遍历过程中如果出现栈里没有左括号可以匹配，代表着要插入一个左括号了，遍历完之后，栈如果不为空，代表着还有左括号无法匹配，因此需要添加右括号了

class Solution {
public:
    int minAddToMakeValid(string s) {
        stack<char>stk;
        int res=0;
        for(char&c:s){
            if(c=='(')stk.push(c);
            else{
                //如果栈为空，没有左括号可以匹配右括号，添加一个左括号
                if(!stk.empty())stk.pop();
                else res++;
            }
        }
        //栈里如果还有剩余的左括号，需要添加右括号和其匹配
        return res+stk.size();
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [946. 验证栈序列](https://leetcode.cn/problems/validate-stack-sequences/)

```cpp
方法一：栈模拟

class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
        stack<int>stk;
        int idx=0;//popped当前需要弹栈的元素下标
        for(int&n:pushed){
            stk.push(n);
            //由于元素不会重复，因此只要栈顶元素和要被弹栈的元素相等，就要弹栈了
            while(!stk.empty()&&stk.top()==popped[idx]){
                stk.pop();
                idx++;
            }
        }
        //看栈里的元素是否能够被弹空，如果可以，序列就没问题
        //还有一个判断方式是，idx==popped.size()
        return stk.size()==0;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [1003. 检查替换后的词是否有效](https://leetcode.cn/problems/check-if-word-is-valid-after-substitutions/)

```cpp
方法一：栈
思路：
遇到'a'，压栈；遇到'b'，跟最近的'a'连接，其实就是栈顶的'a'；遇到'c'，跟最近的"ab"连接，其实就是栈顶元素。
如果遇到'b'或'c'时，栈为空，说明s肯定不是只插入"abc"得到的，因为如果是有效的，会先弹出第一个"abc"，然后后面的'b'或'c'会和前面的'a'或"ab"拼接，继续下一个"abc"的匹配

关键：
b一定跟着a，c一定跟着ab

class Solution {
public:
    bool isValid(string s) {
        stack<string>stk;
        for(char&c:s){
            if(c=='a'){//如果是'a'，就压栈
                stk.push(string(1,c));
            }else if(c=='b'){
                if(stk.empty())return false;//栈为空，说明b是第一个，无效
                stk.top().push_back(c);
                if(stk.top()!="ab")return false;//如果不等于"ab"，说明"abc"的顺序也出错了，无效
            }else{
                if(stk.empty())return false;//栈为空，说明c是第一个，无效
                stk.top().push_back(c);
                if(stk.top()!="abc")return false;//如果不等于"abc"，说明"abc"的顺序也出错了，无效
                stk.pop();//成功匹配到一个"abc"，弹栈，让下一个'a'或"ab"去匹配
            }
        }
        return stk.size()==0;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [1190. 反转每对括号间的子串](https://leetcode.cn/problems/reverse-substrings-between-each-pair-of-parentheses/)

```cpp
方法一：栈
思路：
由于左右括号肯定成对出现，所以最后栈顶肯定被弹空。因此，我们可以先提前让空字符串入栈，最后把结果添加到空字符串中，结果就是遍历完后，栈顶字符串就是我们要的结果字符串
遇到'('：压栈
遇到字母：插入栈顶字符串尾端
遇到')'：弹栈，然后反转弹出来的字符串，而后添加到栈顶字符串后面

class Solution {
public:
    //反转字符串[left,right)部分
    void reverseStr(string&str){
        int l=0;
        int r=str.length()-1;
        while(l<r){
            swap(str[l++],str[r--]);
        }
    }

    string reverseParentheses(string s) {
        stack<string>stk;
        stk.push("");
        for(char&c:s){
            if(c=='('){
                stk.push(string(1,c));
            }else if(c==')'){
                string str=stk.top();stk.pop();
                reverseStr(str);
                str.pop_back();//删掉左括号
                stk.top()+=str;
            }else{
                stk.top().push_back(c);
            }
        }
        return stk.top();
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [1209. 删除字符串中的所有相邻重复项 II](https://leetcode.cn/problems/remove-all-adjacent-duplicates-in-string-ii/)

```cpp
方法一：栈
思路：
遍历字符串，如果当前字符等于栈顶字符，就把当前字符添加到栈顶字符后面，直到栈顶字符串的长度等于k或者遇到不同字符，如果有k个相邻的相同字符，就弹栈，相当于做了一步删除操作。遍历完之后，拼接栈里的各字符串，但是需要注意的是拼接完之后要反转，因为出栈顺序和字符串顺序相反，直接反转即可，因为各字符串都是由相同字符组成的，所以反转整体字符串也不会影响各字符串的顺序

class Solution {
public:
    string removeDuplicates(string s, int k) {
        stack<string>stk;
        for(char&c:s){
            if(!stk.empty()&&c==stk.top()[0]){
                stk.top().push_back(c);
                if(stk.top().length()==k)stk.pop();
                continue;
            }
            stk.push(string(1,c));
        }

        string res;
        while(!stk.empty()){
            res+=stk.top();stk.pop();
        }
        reverse(res.begin(),res.end());
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [316. 去除重复字母](https://leetcode.cn/problems/remove-duplicate-letters/)

![image-20230908103304305](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230908103304305.png)



```cpp
方法一：贪心+单调栈
思路：
1.字典序最小：栈底到栈顶的字符严格单调递增
2.字符唯一且相对顺序不变：从左到右遍历s
关键：
字符剩余个数和字符是否已在栈中这两个条件十分重要

class Solution {
public:
    string removeDuplicateLetters(string s) {
        vector<bool>vis(26,false);//记录字母是否已入栈，防止重复入栈导致结果字符串右重复字符
        vector<int>num(26,0);//记录当前遍历到的字符后面各字符还有多少个，防止删掉某个字母处于栈中的最后一个字符
        stack<char>stk;//栈，可以保证字母的相对顺序
        
        //记录各字母数量
        for(char&c:s){
            num[c-'a']++;
        }

        for(char&c:s){
            //如果当前字符不在栈中，考虑入栈
            if(!vis[c-'a']){
                //因为要保证字典序最小，所以小字母应该尽量在前面，即栈底
                while(!stk.empty()&&c<stk.top()){
                    //如果较大字母在后面还会出现的话就可以弹栈，之后再加入
                    if(num[stk.top()-'a']>0){
                        vis[stk.top()-'a']=false;
                        stk.pop();
                    }
                    else{
                        break;
                    }
                }
                //c入栈
                vis[c-'a']=true;
                stk.push(c);
            }
            //遍历过一个字母，之后的字母数量就少一个
            num[c-'a']--;
        }

        string res;
        while(!stk.empty()){
            res.push_back(stk.top());stk.pop();
        }
        //注意出栈顺序和字符顺序
        reverse(res.begin(),res.end());
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```



### Hard

#### [84. 柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/)

```cpp
方法一：单调栈
栈底到栈顶是非严格单调递增的顺序，因为对于一个柱子，真正限制其向两边延伸的是它左右两边第一根比它低的柱子，而比它高的柱子是可以帮助它延伸且不受影响的
由于计算的时候是需要左右第一根比当前柱子矮的柱子，但是0没有左柱子，heights.size()-1没有右柱子，所以只能手动添加高度为0的柱子，这样既不影响面积的计算，也限制了左右边界，不会导致最左和最右还有中间最低的柱子没有左右边界导致空栈而无法计算面积的问题

class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        stack<int>stk;
        int maxArea=0;
        //在左右边界各添加一个高度为0的柱子，是方便原数组中最左和最右柱子矩形的计算
        heights.insert(heights.begin(),0);
        heights.push_back(0);
        for(int i=0;i<heights.size();i++){
            while(!stk.empty()&&heights[i]<heights[stk.top()]){
                int h=heights[stk.top()];stk.pop();
                int w=i-stk.top()-1;
                maxArea=max(maxArea,h*w);
            }
            stk.push(i);
        }
        return maxArea;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

#### [85. 最大矩形](https://leetcode.cn/problems/maximal-rectangle/)

```cpp
方法一：单调栈
思路：
一行一行遍历，从第一行开始，新遍历到的那一行做底，当前行有元素为1的，说明地基打好了，可以累加其上面的元素，形成一根柱子，即由各方块组成的柱子，但如果中间某个方块是0，则柱子就截断了。剩下的就和84题思路一样了

class Solution {
public:
    //不能传引用，因为会修改原heights的数据和结构
    int maxRectangle(vector<int>heights){
        stack<int>stk;
        int maxArea=0;
        heights.insert(heights.begin(),0);
        heights.push_back(0);
        for(int i=0;i<heights.size();i++){
            while(!stk.empty()&&heights[i]<heights[stk.top()]){
                int h=heights[stk.top()];stk.pop();
                int w=i-stk.top()-1;
                maxArea=max(maxArea,h*w);
            }
            stk.push(i);
        }
        return maxArea;
    }

    int maximalRectangle(vector<vector<char>>& matrix) {
        vector<int>heights(matrix[0].size(),0);
        int maxArea=0;
        for(int i=0;i<matrix.size();i++){
            for(int j=0;j<matrix[0].size();j++){
                if(matrix[i][j]=='0')heights[j]=0;
                else heights[j]+=1;
            }
            maxArea=max(maxArea,maxRectangle(heights));
        }
        return maxArea;
    }
};
时空复杂度分析:
时间复杂度：O(nm);
空间复杂度：O(m);
```



## 双指针

### Easy

#### [925. 长按键入](https://leetcode.cn/problems/long-pressed-name/)

```cpp
方法一：双指针
idx1用于遍历name
idx2用于遍历typed
typed只是name各字符由若干个分成多个了而已，因此typed不会在两组字符之间出现一个没有在对应name两字符之间出现过的字符

class Solution {
public:
    bool isLongPressedName(string name, string typed) {
        int idx1=0;
        int idx2=0;
        int m=name.size();
        int n=typed.size();

        while(idx1<m&&idx2<n){
            //name和typed每组的起始字符都应该相同，否则typed[idx2]就是中间不应该存在的字符
            if(name[idx1]!=typed[idx2])return false;
            //匹配一组字符
            while(idx1<m&&idx2<n&&name[idx1]==typed[idx2]){
                idx1++;
                idx2++;
            }
            //略过长按的字符
            while(idx2>0&&typed[idx2-1]==typed[idx2])idx2++;

            //name已经完全匹配完了，但是typed还没遍历完，说明最后面的字符不是长按name[m-1]引起的，因此不合理
            //或者typed已经遍历完了，但是name还没完全匹配完
            if((idx1==m&&idx2!=n)||(idx1!=m&&idx2==n))return false;
        }

        return true;
    }
};
时空复杂度分析:
时间复杂度：O(n+m);
空间复杂度：O(1);
```

#### [485. 最大连续 1 的个数](https://leetcode.cn/problems/max-consecutive-ones/)

```cpp
方法一：双指针

class Solution {
public:
    int findMaxConsecutiveOnes(vector<int>& nums) {
        int i=0;
        int n=nums.size();
        int maxOneNum=0;
        while(i<n){
            while(i<n&&nums[i]==0)i++;//略过0
            int start=i;//记录第一个1的下标
            while(i<n&&nums[i]==1)i++;//计算连续1的个数
            maxOneNum=max(maxOneNum,i-start);//计算最大值
        }
        return maxOneNum;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```



### Medium

#### [532. 数组中的 k-diff 数对](https://leetcode.cn/problems/k-diff-pairs-in-an-array/)

```cpp
方法一：双指针

class Solution {
public:
    int findPairs(vector<int>& nums, int k) {
        sort(nums.begin(),nums.end());
        int n=nums.size();
        int ans=0;
        for(int j=0,i=0;j<n;j++){
            if(j==0||nums[j-1]!=nums[j]){//去重
                //nums[i]也不会重复，因为nums[i]-num[j]==k的时候就退出while循环了
                //然后添加到答案之后，j就不断往后移动直到下一个不同的nums[j]，所以即使有众多nums[i]可以和当前nums[j]匹配
                //但也只有第一个nums[i]会被记录到ans中
                while(i<n&&(i<=j||nums[i]-nums[j]<k))i++;
                if(i<n&&nums[i]-nums[j]==k)ans++;
            }
        }
        return ans;
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(logn);
```

#### [56. 合并区间](https://leetcode.cn/problems/merge-intervals/)

```cpp
方法一：双指针
思路：
1.按区间起始位置从小到大排序
2.i指向当前区间，j不断遍历到intervals[i]无法覆盖完全的区间
3.向结果区间集合中，加入区间[intervals[i][0],intervals[j][1]]

class Solution {
public:
    static bool compare(const vector<int>&vec1,const vector<int>&vec2){
        if(vec1[0]<vec2[0])return true;
        else if(vec1[0]==vec2[0])return vec1[1]<vec2[1];
        else return false;
    }

    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        sort(intervals.begin(),intervals.end(),compare);
        
        int i=0;
        int j=0;
        int n=intervals.size();
        vector<vector<int>>res;
        while(i<n&&j<n){
            int left=intervals[i][0];//左边界
            int right=intervals[j][1];//右边界
            while(j<n&&intervals[j][0]<=right){
                //合并两区间后，更新右边界，注意要取当前所有右边界中最大的那个，因为能够尽可能的覆盖更多的区间，避免区间重复
                right=max(right,intervals[j++][1]);
            }
            res.push_back({left,right});
            i=j;
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(logn);
```

#### [75. 颜色分类](https://leetcode.cn/problems/sort-colors/)

```cpp
方法一：双指针

class Solution {
public:
    void sortColors(vector<int>& nums) {
        int n=nums.size();
        //p0:下一个0应该去的位置
        //p2:下一个2应该去的位置
        //0和2都处理好了，1也处理完了
        int p0=0,p2=n-1;
        for(int i=0;i<=p2;i++){
            //遇到2，就把2换到p2的位置，直到nums[i]!=2
            while(i<=p2&&nums[i]==2){
                swap(nums[i],nums[p2--]);
            }
            //如果nums[i]=0，就把0换到p0的位置
            if(nums[i]==0){
                swap(nums[i],nums[p0++]);
            }
        }
        //处理完之后，1都在中间
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [80. 删除有序数组中的重复项 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-array-ii/)

```cpp
方法一：双指针

class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int i=0,j=0;
        int n=nums.size();
        //j必然大于等于i，所以无需担心nums[i]覆盖掉还没遍历到的元素
        while(j<n){
            int cnt=1;//当前已经在nums[j]上了，所以初始化出现次数为1
            //计算nums[j]出现次数
            while(j<n-1&&nums[j]==nums[j+1]){
                cnt++;
                j++;
            }
            //至少有一个nums[j]
            nums[i++]=nums[j];
            //如果出现次数超过两次，最多只能在“新数组”里出现两次，由于前面已经添加过一次了，所以再添加一次刚好两次
            if(cnt>=2)nums[i++]=nums[j];
            j++;//指向下一个新元素
        }
        //i指向当前待添加元素的位置，所以刚好是“新数组”的大小
        return i;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [524. 通过删除字母匹配到字典里最长单词](https://leetcode.cn/problems/longest-word-in-dictionary-through-deleting/)

```cpp
方法一：双指针+排序
本题实际上就是在找dictionary中是否存在长度最长且在同样长度中字典序最小的s的子序列
按字符串长度从大到小排序，如果长度相同，就按照字典序从小到大排序，这样从前向后遍历dictionary，第一个符合要求，即是s的子序列的字符串就是结果字符串

关键：
判断删除一些元素之后，能不能和其它匹配，其实就是在问其它是否是当前的子序列

class Solution {
public:
    //字符串比大小是先看字母大小再看长度，比如"b">"ab">"a"，字典序比较
    //但是我们的目标是把长的优先，所以字符串长度优先级应该最高，再比较字典序
    static bool compare(const string&s1,const string&s2){
        if(s1.size()>s2.size())return true;
        else if(s1.size()==s2.size())return s1<s2;//保证每组长度相同的字符串中，第一个就是字典序最小的
        return false;
    }
	
    //判断word是否为s子序列
    bool isSubsequence(string s,string word){
        int i=0;
        int j=0;
        int n=s.size();
        int m=word.size();

        while(i<n){
            if(s[i]==word[j])j++;
            i++;
        }

        return j==m;
    }

    string findLongestWord(string s, vector<string>& dictionary) {
        sort(dictionary.begin(),dictionary.end(),compare);
        int n=dictionary.size();

        for(int i=0;i<n;i++){
            if(isSubsequence(s,dictionary[i])){
                return dictionary[i];
            }
        }

        return "";
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(logn);
```

#### [986. 区间列表的交集](https://leetcode.cn/problems/interval-list-intersections/)

```cpp
方法一：双指针
关键：
交集就是 大的起始点 到 小的起始点所在区间的结束点 组成的区间
然后哪个区间的结束点小，其对应的指针就先后移

卡点：
一开始想先找到起始点小的区间，然后从它开始，其实不需要这样。只要能够获取到大的起始点和小的起始点所在区间的结束点即可。
因此找交集的起始点是最大值，而结束点是最小值

class Solution {
public:
    vector<vector<int>> intervalIntersection(vector<vector<int>>& firstList, vector<vector<int>>& secondList) {
        int i=0;
        int j=0;
        int m=firstList.size();
        int n=secondList.size();
        vector<vector<int>>res;
        while(i<m&&j<n){
            //交集区间在于 大的起始点 到 小的起始点所在区间的结束点 之间
            //所以起始点取最大值，结束点取最小值
            int start=max(firstList[i][0],secondList[j][0]);
            int end=min(firstList[i][1],secondList[j][1]);
            if(start<=end){
                res.push_back({start,end});
            }
            //结束点小的区间指针先移动，因为结束点大的区间可能和另一个区间列表还有交集
            if(firstList[i][1]<secondList[j][1])i++;
            else j++;
        }
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(min(m,n));
空间复杂度：O(1);
```

#### [1004. 最大连续1的个数 III](https://leetcode.cn/problems/max-consecutive-ones-iii/)

```cpp
方法一：滑动窗口（双指针）
关键：把问题转成求连续子数组中出现最多k个0的情况下子数组的最大长度

class Solution {
public:
    int longestOnes(vector<int>& nums, int k) {
        
        int i=0,zeros=0,res=0;
        //一开始使用while循环，会出现k=0的情况下，滑动窗口不移动导致while陷入死循环的问题
        //用for循环，右边界必定会向右扩展，只需要记录0的个数不超过k，否则缩小左边界，后移i，直到删掉一个0
        for(int j=0;j<nums.size();j++){
            if(nums[j]==0)zeros++;
            while(zeros>k){
                if(nums[i++]==0)zeros--;
            }
            res=max(res,j-i+1);
        }
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [1498. 满足条件的子序列数目](https://leetcode.cn/problems/number-of-subsequences-that-satisfy-the-given-sum-condition/)

```cpp
方法一：滑动窗口（双指针）

class Solution {
public:
    int numSubseq(vector<int>& nums, int target) {
        //排序
        sort(nums.begin(),nums.end());
        int n=nums.size();
        const int mod=1000000007;

        //快速幂
        vector<int>pow(n);
        pow[0]=1;
        for(int i=1;i<n;i++){
            pow[i]=(pow[i-1]<<1)%mod;
        }

        int res=0;
        int i=0;
        int j=n-1;
        while(i<=j){
            //最大值、最小值的和过大，左移最大值指针，以减小和
            if(nums[i]+nums[j]>target){
                j--;
            }else{
                //nums[i]必须出现在子序列中，这样最小值和最大值的和才小于等于target，否则最小值变大，和可能就大于target
                //因为nums[i]+nums[j]<=target，所以nums[i]+nums[j-k]必然也小于等于target
                //所以就是nums[i]和nums[j]之间的元素(包括nums[j])的排列组合
                //nums[i+1]到nums[j]，总共有j-(i+1)+1=j-i个元素
                //j-i个元素的子集个数是2^(j-i)
                res=(res+pow[j-i])%mod;
                //以nums[i]为最小值的情况都处理完了，后移i
                i++;
            }
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(n);
```



### Hard



## 贪心算法

### Easy

#### [944. 删列造序](https://leetcode.cn/problems/delete-columns-to-make-sorted/)

```cpp
方法一：直接遍历

class Solution {
public:
    int minDeletionSize(vector<string>& strs) {
        int n=strs.size();//行数
        int m=strs[0].size();//列数
        int res=0;

        for(int j=0;j<m;j++){
            for(int i=0;i<n-1;i++){
                if(strs[i][j]>strs[i+1][j]){
                    res++;
                    break;
                }
            }
        }

        return res;
    }
};
时空复杂度分析:
时间复杂度：O(nm);
空间复杂度：O(1);
```

#### [1046. 最后一块石头的重量](https://leetcode.cn/problems/last-stone-weight/)

```cpp
方法一：大根堆
因为总是要取两块最重的石头碰撞，用最大堆保存最合适不过了

class Solution {
public:
    int lastStoneWeight(vector<int>& stones) {
        priority_queue<int,vector<int>,less<int>>pq;
        //先加入到大根堆中
        for(auto&n:stones){
            pq.push(n);
        }

        //至少要有两个石头才能碰撞
        while(pq.size()>=2){
            int x=pq.top();pq.pop();
            int y=pq.top();pq.pop();
            //x可能等于y，不用把新重量加入
            if(x-y>0){
                pq.push(x-y);
            }
        }

        //大根堆为空，代表所有石头都被粉碎
        if(pq.empty())return 0;
        //大根堆不为空，还有剩余石头，即大根堆的根
        return pq.top();
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(n);
```

#### [1217. 玩筹码](https://leetcode.cn/problems/minimum-cost-to-move-chips-to-the-same-position/)

```cpp
方法一：贪心
能走两步绝不走一步，本题实际上就是在考各筹码的位置到最后统一位置总共所需的最小代价，本质上就是能不能以偶数步从positions[i]到最后统一位置，如果可以代价就是0，否则代价就是1，因为可以先走完前面偶数步之后，最后再走一步。
问题的关键是如何确定最后统一的位置呢？其实也只和奇数偶数有关，因为所有位置要么是奇数要么是偶数，从position[i]到1或者3的代价是一样的，因为可以从1走两步到3，代价是0，所以只需要找两个奇偶数代表即可，然后计算到它们二者统一位置上的不同代价，返回最小值即可

class Solution {
public:
    int minCostToMoveChips(vector<int>& position) {
        int posOdd=1;
        int posEven=2;
        int resOdd=0;
        int resEven=0;

        for(auto&pos:position){
            //用取模2判断奇偶，最终提交时间是8ms
            //但是用位运算判断奇偶，最终提交时间是0ms
            if(abs(pos-posOdd)&1)resOdd++;
            if(abs(pos-posEven)&1)resEven++;
        }

        return min(resOdd,resEven);
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

#### [1221. 分割平衡字符串](https://leetcode.cn/problems/split-a-string-in-balanced-strings/)

```cpp
方法一：贪心
从左到右遍历s，时刻记录'L'和'R'的数量，如果二者数量一样，就代表又可以分割平衡字符串了，数量加1

class Solution {
public:
    int balancedStringSplit(string s) {
        int numL=0;
        int numR=0;
        int i=0;
        int res=0;
        while(i<s.length()){
            s[i++]=='L'?numL++:numR++;
            if(numL==numR)res++;
        }
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```



### Medium

#### [1029. 两地调度](https://leetcode.cn/problems/two-city-scheduling/)

```cpp
方法一：贪心
思路：
假设先让所有人去a市，再从其中找一半的人去b市，需要补差价bCost-aCost，这个值可正可负，为了最终费用最低，差价肯定越小越好，所以按bCost-aCost从小到大排序

class Solution {
public:
    int twoCitySchedCost(vector<vector<int>>& costs) {
        //排序
        sort(costs.begin(),costs.end(),
                [](const vector<int>&o1,const vector<int>&o2){
            return (o1[1]-o1[0]<o2[1]-o2[0]);
        });

        int res=0;//最终费用
        int n=costs.size()/2;
        //前n个人去b市，后n个人去a市
        for(int i=0;i<n;i++){
            res+=costs[i][1]+costs[i+n][0];
        }
        
        return res;
    }
};
时空复杂度分析:
时间复杂度：O(nlogn);
空间复杂度：O(1);
其中n是costs的大小
```

#### [1049. 最后一块石头的重量 II](https://leetcode.cn/problems/last-stone-weight-ii/)

```cpp
方法一：动态规划，01背包问题
思路：
把石头分成重量尽可能相等的两堆，两堆石头碰撞之后，其中一堆必然全被粉碎，剩下一堆的实际只有一个石头，重量是初始两堆石头的重量差，因此重量差越小，最后一块石头的重量就越小
因此，目标就是尽可能的凑出重量为所有石头质量之和的一半的那堆石头

class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        int sum=accumulate(stones.begin(),stones.end(),0);
        int target=sum/2;
        int n=stones.size();

        //dp[i][j]:从stones[0,i-1]里找到离j最近且小于j的石头堆的重量
        vector<vector<int>>dp(n+1,vector<int>(target+1,0));
        for(int i=1;i<=n;i++){
            for(int j=1;j<=target;j++){
                //i是dp的行，不是stones的，所以取石头应该是stones[i-1]
                //由于是01背包，stones[i-1]要么是放要么是不放，所以是找上一行的，而不是当前行的
                //因为当前行的有些可能已经更新过了，且已经放了stones[i-1]，不能重复放了
                if(j<stones[i-1])dp[i][j]=dp[i-1][j];//小于stones[i]的背包无法装下stones[i]，不更新
                else dp[i][j]=max(dp[i-1][j],dp[i-1][j-stones[i-1]]+stones[i-1]);
            }
        }
        //分成sum-dp[n][target]和dp[n][target]两堆石头，其中dp[n][target]是大的那堆
        return sum-2*dp[n][target];
    }
};
时空复杂度分析:
时间复杂度：O(n*target);
空间复杂度：O(n*target);

优化空间：二维数组优化成一维数组
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        int n=stones.size();
        int sum=accumulate(stones.begin(),stones.end(),0);

        int target=sum/2;
        vector<int>dp(target+1,0);
        for(int i=0;i<n;i++){
            //需要前面的值，所以从后面开始更新
            for(int j=target;j>=stones[i];j--){
                dp[j]=max(dp[j],dp[j-stones[i]]+stones[i]);
            }
        }

        return sum-2*dp.back();
    }
};
时空复杂度分析:
时间复杂度：O(n*target);
空间复杂度：O(target);
```



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

