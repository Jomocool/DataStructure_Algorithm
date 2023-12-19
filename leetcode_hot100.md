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

