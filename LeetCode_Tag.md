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

