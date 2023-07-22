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
        int left=0;//窗口左边界
        int right=k-1;//窗口右边界
        double maxAvg=INT_MIN;//最大平均数

        double sum=0;//第一个窗口的和
        for(int i=0;i<k;i++){
            sum+=nums[i];
        }


        while(right<nums.size()){
            maxAvg=max(maxAvg,sum/k);
            //如果此时窗口右边界在最后一个元素，则直接break，否则会越界访问
            if(right==nums.size()-1)break;
            //右移窗口的同时，把窗口和计算了
            sum-=nums[left++];
            sum+=nums[++right];
        }

        return maxAvg;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```



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

eg.大致可以分为以下3种情况，隐藏条件是nums[i]>nums[i-1]
例1（i-2<0）:[4,2,5]，可以把4调小到<=2，或者把2调大到4、5
例2（nums[i]>=nums[i-2]）:[1,4,2,5]，可以把4调小到1、2，或者把2调大到4、5
例3（nums[i]<nums[i-2]）:[3,4,2,5]，只能把2调大到4、5，因为4前面是3，所以不能把4调小到<=2
总结：
当nums[i]破坏了数组递增时，即nums[i]>nums[i-1]
由于不清楚i之后的值是多少，所以尽量让i的值更小，因此修改i-1的值让其更小
例如对于[4,2,3]，如果修改了2的值为4，则遍历到3时，发现其比前面的4更小，又要修改一次值，总共两次；而如果让4修改为2，则只有调整一次值。所以修改nums[i]为更大的值，很可能会造成不必要的影响，结论就是尽可能的修改i-1的值，而不是i的值，让i值越小越好，这样递增的概率也会更大
1.i==1（例1）：nums[i-1]=nums[i]
2.nums[i]>=nums[i-2]：nums[i-1]=nums[i]
3.nums[i]<nums[i-2]：nums[i]=nums[i-1]，只能调整nums[i]了，但也尽可能小，调成和nums[i-1]一样大即可，对后面的影响尽可能小

尽量修改前面的值，即nums[i-1]和nums[i]，不要动后面的值，因为后面的始终会遍历到，会去处理的

class Solution {
public:
    bool checkPossibility(vector<int>& nums) {
        int cnt=0;
        for(int i=1;i<nums.size();i++){
            if(nums[i]<nums[i-1]){//nums[i-1]可能是前面修改过的，前面修改过的就是这样影响后面的判断
                if(i==1||nums[i]>=nums[i-2]){
                    nums[i-1]=nums[i];
                }else{
                    nums[i]=nums[i-1];
                }
                cnt++;
            }
        }

        return cnt<=1;
    }
};
时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
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

