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

### [76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)

```cpp
class Solution {
public:
    unordered_map <char,int>s_mp,t_mp;

    bool check() {
        for (const auto&pair:t_mp){
            if(s_mp[pair.first]<pair.second)return false;
        }

        return true;
    }

    string minWindow(string s, string t) {
        for(const auto&c:t){
            ++t_mp[c];
        }

        int l=0,r=-1;
        int minLen=INT_MAX,ansL=-1,ansR=-1;

        while(r<int(s.size())){
            s_mp[s[++r]]++;
            while(check()&&l<=r){
                if (r-l+1<minLen){
                    minLen=r-l+1;
                    ansL=l;
                    ansR=r;
                }
                s_mp[s[l++]]--;
            }
        }

        return ansL==-1?string():s.substr(ansL,minLen);
    }
};
```

## 普通数组

### [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int cnt_sum=0;
        int max_sum=INT_MIN;

        for(const auto&n:nums){
            cnt_sum = cnt_sum>0?cnt_sum+n:n;
            max_sum=max(max_sum,cnt_sum);
        }

        return max_sum;
    }
};
```

### [56. 合并区间](https://leetcode.cn/problems/merge-intervals/)

```cpp
class Solution {
public:
    static bool cmp(const vector<int>&v1,const vector<int>&v2){
        if (v1[0]<v2[0])return true;
        else if (v1[0]==v2[0])return v1[1]<v2[1];
        return false;
    }

    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        sort(intervals.begin(),intervals.end(),cmp);

        vector<vector<int>>res;
        int start=0,end=0;
        int cnt=0;
        int n=intervals.size();

        while(cnt<n){
            if(cnt == 0){
                start=intervals[cnt][0];
                end=intervals[cnt][1];
            }

            while(cnt<n&&end>=intervals[cnt][0]){
                end=max(end,intervals[cnt][1]);
                cnt++;
            }
            
            res.push_back({start,end});
            if(cnt<n){
                start=intervals[cnt][0];
                end=intervals[cnt][1];
            }
        }

        return res;
    }
};
```

### [189. 轮转数组](https://leetcode.cn/problems/rotate-array/)

```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n=nums.size();
        k=k%n;

        // eg.[1,2,3,4,5,6,7] 3
        // 反转所有元素 => [7,6,5,4,3,2,1]
        reverse(nums.begin(),nums.end());
        // 反转前k个元素 => [5,6,7,4,3,2,1]
        reverse(nums.begin(),nums.begin()+k);
        // 反转后k个元素 => [5,6,7,1,2,3,4]
        reverse(nums.begin()+k,nums.end());
    }
};
```

### [238. 除自身以外数组的乘积](https://leetcode.cn/problems/product-of-array-except-self/)

```cpp
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        int n=nums.size();
        vector<int>res(n,1);
        int tmp=1;

        // 左到右遍历获取左边乘积
        for(int i=1;i<n;i++){
            res[i]=tmp*nums[i-1];
            tmp*=nums[i-1];
        }

        tmp=1;
        // 右到左遍历获取右边乘积
        for(int i=n-2;i>=0;i--){
            res[i]*=tmp*nums[i+1];
            tmp*=nums[i+1];
        }

        return res;
    }
};
```

### [41. 缺失的第一个正数](https://leetcode.cn/problems/first-missing-positive/)

```cpp
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        // 让nums[i]=i+1;
        int n=nums.size();
        
        for(int i=0;i<n;i++){
            while(nums[i]>=1&&nums[i]<=n&&nums[i]!=nums[nums[i]-1]){
                swap(nums[i],nums[nums[i]-1]);
            }
        }

        for(int i=0;i<n;i++){
            if(nums[i]!=i+1)return i+1;
        }

        return n+1;
    }
};
```



## 矩阵

### [73. 矩阵置零](https://leetcode.cn/problems/set-matrix-zeroes/)

```cpp
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int n=matrix.size();
        int m=matrix[0].size();
        unordered_set<int>zero_row;
        unordered_set<int>zero_col;

        for(int i=0;i<n;i++){
            for(int j=0;j<m;j++){
                if(matrix[i][j]==0){
                    zero_row.insert(i);
                    zero_col.insert(j);
                }
            }
        }

        for (const auto&i:zero_row){
            fill(matrix[i].begin(),matrix[i].end(),0);
        }

        for (const auto&j:zero_col){
            for(int i=0;i<n;i++)matrix[i][j]=0;
        }
    }
};
```

### [54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        vector<int>res;
        int n=matrix.size();
        int m=matrix[0].size();

        pair<int,int>left_up(0,0);
        pair<int,int>right_down(n-1,m-1);

        while(left_up.first<=right_down.first&&left_up.second<=right_down.second){
            int up=left_up.first;
            int left=left_up.second;
            int down=right_down.first;
            int right=right_down.second;

            // 只剩一行
            if (up==down){
                for(int j=left;j<=right;j++){
                    res.emplace_back(matrix[up][j]);
                }
                break;
            }

            // 只剩一列
            if (left==right){
                for(int i=up;i<=down;i++){
                    res.emplace_back(matrix[i][left]);
                }
                break;
            }

            for(int j=left;j<right;j++){
                res.emplace_back(matrix[up][j]);
            }
            for(int i=up;i<down;i++){
                res.emplace_back(matrix[i][right]);
            }
            for(int j=right;j>left;j--){
                res.emplace_back(matrix[down][j]);
            }
            for(int i=down;i>up;i--){
                res.emplace_back(matrix[i][left]);
            }

            left_up.first+=1;
            left_up.second+=1;
            right_down.first-=1;
            right_down.second-=1;
        }

        return res;
    }
};
```

### [48. 旋转图像](https://leetcode.cn/problems/rotate-image/)

```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n=matrix.size();
        int m=matrix[0].size();

        pair<int,int>left_up{0,0};
        pair<int,int>right_down{n-1,m-1};

        while(left_up.first<=right_down.first&&left_up.second<=right_down.second){
            int left = left_up.first;
            int up = left_up.second;
            int right= right_down.first;
            int down = right_down.second;

            int tmp=0;
            for(int i=0;i<right-left;i++){
                int tmp=matrix[up][left+i];
                matrix[up][left+i]=matrix[down-i][left];
                matrix[down-i][left]=matrix[down][right-i];
                matrix[down][right-i]=matrix[up+i][right];
                matrix[up+i][right]=tmp;
            }

            left_up.first+=1;
            left_up.second+=1;
            right_down.first-=1;
            right_down.second-=1;
        }
    }
};
```

### [240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/)

![Picture1.png](https://pic.leetcode-cn.com/6584ea93812d27112043d203ea90e4b0950117d45e0452d0c630fcb247fbc4af-Picture1.png)

```cpp
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int n=matrix.size();
        int m=matrix[0].size();
        int i=0;
        int j=m-1;

        while(i<n&&j>=0){
            if (target>matrix[i][j])i++;
            else if(target<matrix[i][j])j--;
            else return true;
        }

        return false;
    }
};
```



## 链表

### [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode* tmpA=headA;
        ListNode* tmpB=headB;

        while(tmpA!=tmpB){
            tmpA=tmpA?tmpA->next:headB;
            tmpB=tmpB?tmpB->next:headA;
        }

        return tmpA;
    }
};
```

### [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(!head)return nullptr;

        ListNode* dummy_head=new ListNode(-1);
        ListNode* cur=head;
        ListNode*next=cur->next;

        while(cur){
            next=cur->next;
            cur->next=dummy_head->next;
            dummy_head->next=cur;
            cur=next;
        }

        return dummy_head->next;
    }
};
```

### [234. 回文链表](https://leetcode.cn/problems/palindrome-linked-list/)

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* reverse(ListNode* node){
        if(!node)return nullptr;

        ListNode* dummy_head=new ListNode(-1);
        ListNode* cur=node;
        ListNode* next=cur->next;

        while(cur){
            next=cur->next;
            cur->next=dummy_head->next;
            dummy_head->next=cur;
            cur=next;
        }

        return dummy_head->next;
    }

    bool isPalindrome(ListNode* head) {
        ListNode* slow=head;
        ListNode* fast=head;
        ListNode* pre= slow;

        while(fast&&fast->next){
            pre=slow;
            slow=slow->next;
            fast=fast->next->next;
        }

        pre->next=nullptr;
        slow=reverse(slow);

        fast=head;
        while(fast&&slow){
            if(fast->val!=slow->val)return false;
            slow=slow->next;
            fast=fast->next;
        }

        return true;
    }
};
```

### [141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    bool hasCycle(ListNode *head) {
        if(!head||!head->next)return false;
        ListNode* slow=head;
        ListNode* fast=head;

        while(fast&&fast->next){
            slow=slow->next;
            fast=fast->next->next;
            if(slow==fast)return true;
        }

        return false;
    }
};
```

### [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode*fast=head;
        ListNode*slow=head;

        while(fast&&fast->next){
            slow=slow->next;
            fast=fast->next->next;
            if(fast==slow)break;
        }
        if(!fast||!fast->next)return nullptr;

        fast=head;
        while(fast!=slow){
            fast=fast->next;
            slow=slow->next;
        }

        return fast;
    }
};
```

### [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode* cur1=list1;
        ListNode* cur2=list2;
        ListNode* new_head=new ListNode(-1);
        ListNode* cur_new_head=new_head;

        while(cur1||cur2){
            int val1=cur1?cur1->val:INT_MAX;
            int val2=cur2?cur2->val:INT_MAX;

            if(val1<val2){
                cur_new_head->next=new ListNode(val1);
                cur1=cur1->next;
            }
            else{
                 cur_new_head->next=new ListNode(val2);
                 cur2=cur2->next;
            }
            
            cur_new_head=cur_new_head->next;
        }

        return new_head->next;
    }
};
```

