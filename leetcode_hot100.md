## 哈希表

### [1. 两数之和](https://leetcode.cn/problems/two-sum/)

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        // key:nums[i]
        // value:i
        unordered_map<int,int>mp;

        for(int i=0;i<nums.size();i++){
            if(mp.count(target-nums[i]))return {mp[target-nums[i]],i};
            mp[nums[i]]=i;
        }

        return {};
    }
};
```

### [49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        // 字母异位词排序后的结果是唯一的，可以以此做哈希表的key
        vector<vector<string>>res;
        unordered_map<string,vector<string>>mp;

        for(auto&str:strs){
            string key=str;
            sort(key.begin(),key.end());
            mp[key].push_back(str);
        }

        for(auto&pair:mp){
            res.push_back(pair.second);
        }

        return res;
    }
};
```

### [128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int>st;
        for(auto&num:nums){
            st.insert(num);
        }
        
        int maxLen=0;
        for(auto&num:st){
            // 如果有n-1，那么从n-1开始的序列必然是长于从n开始的序列的，因此不作无用的遍历
            if(!st.count(num-1)){
                int end=num;
                while(st.count(++end));
                maxLen=max(maxLen,end-num);
            }
        }

        return maxLen;
    }
};
```



## 双指针

### [283. 移动零](https://leetcode.cn/problems/move-zeroes/)

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int n=nums.size();
        int nonzero_idx=0;

        for(int i=0;i<n;i++){
            // 非0元素按原相对顺序移到数组前面，后面直接补0即可
            // nonzero_idx<=i：因为nums[i]还可能等于0，此时nonzero_idx不移动，因此肯定不会重覆盖还没遍历到的值
            if(nums[i]!=0){
                nums[nonzero_idx++]=nums[i];
            }
        }

        for(int i=nonzero_idx;i<n;i++){
            nums[i]=0;
        }
    }
};
```

### [11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int n=height.size();
        int i=0;
        int j=n-1;

        int maxArea=0;
        while(i<j){
            // 移动短板
            int w=j-i;
            int h=height[i]<height[j]?height[i++]:height[j--];
            maxArea=max(maxArea,h*w);
        }

        return maxArea;
    }
};
```

### [15. 三数之和](https://leetcode.cn/problems/3sum/)

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        int n=nums.size();
        if(n<3)return {};

        vector<vector<int>>res;
        // 排序nums数组，让相同的元素聚集在一起，方便去重
        sort(nums.begin(),nums.end());

        for(int i=0;i<n-2;i++){
            if(i>0&&nums[i]==nums[i-1])continue; //去重 
            int left=i+1;
            int right=n-1;
            while(left<right){
                if(nums[i]+nums[left]+nums[right]==0){
                    res.push_back({nums[i],nums[left],nums[right]});
                    // 去重
                    while(left<right&&nums[left]==nums[left+1])left++;
                    while(left<right&&nums[right]==nums[right-1])right--;
                    // 此时left和right都指向相同元素的最后一个，让它们再位移一次，就是新元素的开头
                    left++;
                    right--;
                }else if(nums[i]+nums[left]+nums[right]>0){// 偏大
                    right--;
                }else{// 偏小
                    left++;
                }
            }
        }
        
        return res;
    }
};
```

### [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/)

```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        // 下标i能接的雨水量是由左、右最高柱子中短的那根决定的
        int left=0;
        int right=height.size()-1;
        int leftMax=0; // left左边最高柱子高度
        int rightMax=0; // right右边最高柱子高度
        int ans=0;

        while(left<=right){
            leftMax=max(height[left],leftMax);
            rightMax=max(height[right],rightMax);
            // 由于是先赋值leftMax和rightMax，因此可以把height[left]、height[right]类比成leftMax、rightMax
            if(height[left]<height[right]){// leftMax<rightMax
                ans+=leftMax-height[left++];
            }else{// leftMax>=rightMax
                ans+=rightMax-height[right--];
            }
        }

        return ans;
    }
};
```



## 滑动窗口

### [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int left=0;// 滑动窗口左边界
        int right=0;// 滑动窗口右边界
        int n=s.size();
        unordered_map<char,int>mp;//滑动窗口内部字符串的各字符出现频率
        int maxLen=0;

        while(right<n){
            // 向右扩展直至出现重复字符
            while(right<n&&mp[s[right]]==0){
                mp[s[right]]=1;
                right++;
            }
            maxLen=max(maxLen,right-left);
            while(mp[s[right]]>0)mp[s[left++]]--;// 回收左边字符直至重复字符被筛除
        }

        return maxLen;
    }
};
```

### [438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        vector<int>res;
        int sLen=s.size();
        int pLen=p.size();

        if(sLen<pLen)return res;

        vector<int>s_count(26,0);
        vector<int>p_count(26,0);

        for(int i=0;i<pLen;i++){
            s_count[s[i]-'a']++;
            p_count[p[i]-'a']++;
        }

        // 维护一个长度为pLen的窗口
        for(int i=0;i<sLen-pLen;i++){
            if(s_count==p_count)res.emplace_back(i);
            // 窗口整体右移
            s_count[s[i]-'a']--;
            s_count[s[i+pLen]-'a']++;
        }

        // for循环在最后一次窗口右移后没有处理异位词的判断就退出循环了，因此要最后多判断一次，避免漏掉
        if(s_count==p_count)res.emplace_back(sLen-pLen);

        return res;
    }
};
```



## 子数组

### [560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        // 逆向思考：sum[i~j]=k=sum[0~j]-sum[0,i-1]
        unordered_map<int,int>mp; // 记录前缀和出现次数
        mp[0]=1; // 用于处理pre-k=0的情况，此时子数组是从下标0开始，也属于一种情况
        int res=0;

        int pre=0;// 前缀和
        for(auto&x:nums){
            pre+=x;
            if(mp.find(pre-k)!=mp.end()){
                res+=mp[pre-k];
            }
            mp[pre]++;
        }

        return res;
    }
};
```

### [239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)

```cpp
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        // 严格单调递减(保证最大值在最前面)双端队列，但是存下标，因为这样既可以得到值，也可以得到下标用于计算窗口大小
        deque<int>dqe;
        vector<int>res;

        for(int i=0;i<nums.size();i++){
            while(!dqe.empty()&&nums[dqe.back()]<=nums[i])dqe.pop_back();
            dqe.push_back(i);
            // 窗口过大
            if(dqe.back()-dqe.front()+1>k)dqe.pop_front(); // 舍弃最前面的
            if(i>=k-1)res.push_back(nums[dqe.front()]);
        }

        return res;
    }
};
```

