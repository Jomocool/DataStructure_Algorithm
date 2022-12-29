# LeetCode

## 一、数组

### 1. LeetCode704. 二分查找

```c++
class Solution {
public:
    int search(vector<int>& nums, int target) {
        return binarySearch(nums,0,nums.size()-1,target);
    }
    
    int binarySearch(vector<int>&nums,int L,int R,int target){
        while(L<=R){
            int mid=L+((R-L)>>1);
            if(nums[mid]>target){
                R=mid-1;
            }else if(nums[mid]<target){
                L=mid+1;
            }else{
                return mid;
            }
        }
        return -1;
    }
};
```

### 2. LeetCode27. 移除元素

```c++
双指针：快慢指针
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int slowIndex=0;
        for(int fastIndex=0;fastIndex<nums.size();fastIndex++){
            if(nums[fastIndex]!=val){//数组元素不能被删除只能被覆盖
                nums[slowIndex++]=nums[fastIndex];
            }
        }
        return slowIndex;
    }
};
```

### 3. LeetCode977. 有序数组的平方

```c++
双指针法：
数组其实是有序的，元素平方和从两边向中间递减，因此最大的平方和一定是在两边，双指针谁大谁先动，找到相遇。
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        int i=0;
        int j=nums.size()-1;
        int k=nums.size()-1;
        vector<int>res(nums.size());
        while(i<=j){
            if(nums[i]*nums[i]<nums[j]*nums[j]){
                res[k--]=nums[j]*nums[j];
                j--;
            }else{
                res[k--]=nums[i]*nums[i];
                i++;
            }
        }
        return res;
    }
};
```

### 4. LeetCode209. 长度最小的子数组

```c++
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int left=0;
        int right=0;
        int sum=0;
        int minLen=INT_MAX;
        
        //该结构很好地考虑到了当right==nums.size()，但还没记录最后一次最小长度时就退出大while循环了
        //因为退出循环前一定会记录一次最小长度
        while(right<nums.size()){
            while(right<nums.size()&&sum<target)sum+=nums[right++];
            //这里一定是大于等于，不然sum==target时会陷入死循环
            while(left<=right&&sum>=target)sum-=nums[left++];
            minLen=min(minLen,right-left+1);
        }
        return minLen==nums.size()+1?0:minLen;//考虑数组总和<target
    }
};
```

### 5. LeetCode59. 螺旋矩阵 II

```c++
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>>res(n,vector<int>(n));
        //左上角
        int left=0;
        int up=0;
        //右下角
        int right=n-1;
        int down=n-1;
        
        int path=1;
        while(left<=right&&up<=down){
            int curLeft=left;
            int curUp=up;
            int curRight=right;
            int curDown=down;
            while(curLeft<right){
                res[up][curLeft++]=path++;
            }
            while(curUp<down){
                res[curUp++][right]=path++;
            }
            while(curRight>left){
                res[down][curRight--]=path++;
            }
            while(curDown>up){
                res[curDown--][left]=path++;
            }
            left++;
            up++;
            right--;
            down--;
        }
        
        //如果n为奇数，要另外在中间块赋值
        int mid=n/2;
        if(n%2)res[mid][mid]=path;
        return res;
    }
};
```

