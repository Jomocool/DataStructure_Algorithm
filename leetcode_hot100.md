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

### [2. 两数相加](https://leetcode.cn/problems/add-two-numbers/)

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
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int sum=0,cast=0;
        int val1=0,val2=0;
        ListNode*res_head=new ListNode(-1);
        ListNode* cur=res_head;

        while(l1||l2){
            val1=l1?l1->val:0;
            val2=l2?l2->val:0;
            sum=val1+val2+cast;
            cast=sum/10;
            ListNode* node = new ListNode(sum%10);
            cur->next=node;

            if(l1)l1=l1->next;
            if(l2)l2=l2->next;
            cur=cur->next;
        }

        if(cast)cur->next=new ListNode(1);

        return res_head->next;
    }
};
```

### [19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

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
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        if(!head->next)return nullptr;

        // 让快指针先走n个节点，然后慢指针和快指针同步移动，等快指针为空时，慢指针刚好指向倒数第n个节点
        ListNode*fast = head, *slow=head , *pre=head;
        for(int i=0;i<n&&fast;i++){
            fast=fast->next;
        }

        while(fast){
            fast=fast->next;
            pre=slow;
            slow=slow->next;
        }

        if(slow==head)return head->next;

        pre->next=pre->next->next;

        return head;
    }
};
```

### [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)

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
    ListNode* swapPairs(ListNode* head) {
        ListNode* dummy_head=new ListNode(-1);
        dummy_head->next=head;
        ListNode* cur=head;
        ListNode* pre=dummy_head; // 需要前驱节点来连接前后两对节点
        ListNode* next=nullptr;

        while(cur&&cur->next){
            next=cur->next->next;
            cur->next->next=cur;
            pre->next=cur->next;
            cur->next=next;
            pre=cur;
            cur=next;
        }

        return dummy_head->next;
    }
};
```

### [25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)

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
    ListNode* reverse(ListNode* head) {
        ListNode* dummy_head=new ListNode(-1);
        ListNode* cur=head;
        ListNode *next=nullptr;

        while(cur){
            next=cur->next;
            cur->next=dummy_head->next;
            dummy_head->next=cur;
            cur=next;
        }

        return dummy_head->next;
    }

    ListNode* reverseKGroup(ListNode* head, int k) {
        // 需要依赖每次反转的k个节点链表的头尾节点和其前驱节点，以及下一次反转链表的头节点
        ListNode* dummy_head=new ListNode(-1);
        dummy_head->next=head;
        ListNode* pre=dummy_head;
        ListNode* begin_node=head;
        ListNode* end_node=begin_node;
        ListNode* next_begin_node=nullptr;
        int tmp_k=0;

        while(begin_node){
            tmp_k=k;
            while(tmp_k>1&&end_node){
                end_node=end_node->next;
                tmp_k--;
            }
            if(!end_node){
                pre->next=begin_node;
                break;
            }
            next_begin_node=end_node->next;
            end_node->next=nullptr;
            pre->next=reverse(begin_node);
            pre=begin_node;
            begin_node=next_begin_node;
            end_node=begin_node;
        }

        return dummy_head->next;
    }
};
```

### [138. 随机链表的复制](https://leetcode.cn/problems/copy-list-with-random-pointer/)

**深拷贝和浅拷贝一般是对于指针来说的：**

- **浅拷贝：**只拷贝指针值（即某变量的地址），所以实际上二者还是共享内存，只要修改了就会互相影响

- **深拷贝：**不共享内存，而是拷贝了整个对象，修改值不会影响到对方

- ​    

```cpp
/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* next;
    Node* random;
    
    Node(int _val) {
        val = _val;
        next = NULL;
        random = NULL;
    }
};
*/

class Solution {
public:
    Node* copyRandomList(Node* head) {
        // 哈希表存储原链表节点和其对应的新链表节点
        // 这样的设计方便获取到next和random对应的新链表节点
        unordered_map<Node*,Node*>old2new_mp;

        Node* cur=head;
        while(cur){
            old2new_mp[cur]=new Node(cur->val);
            cur=cur->next;
        }

        cur=head;
        while(cur){
            old2new_mp[cur]->next=old2new_mp[cur->next];
            old2new_mp[cur]->random=old2new_mp[cur->random];
            cur=cur->next;
        }

        return old2new_mp[head];
    }
};
```

### [148. 排序链表](https://leetcode.cn/problems/sort-list/)

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
    ListNode* sortList(ListNode* head) {
        vector<int>list_vec;
        ListNode* cur=head;

        while(cur){
            list_vec.emplace_back(cur->val);
            cur=cur->next;
        }

        sort(list_vec.begin(),list_vec.end());

        cur=head;

        for(int i=0;i<list_vec.size();i++){
            cur->val=list_vec[i];
            cur=cur->next;
        }

        return head;
    }
};
```

### [23. 合并 K 个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)

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
    ListNode* mergeTwoLists(ListNode* head1,ListNode*head2){
        ListNode*dummy_head=new ListNode(-1);
        ListNode*cur_dummy=dummy_head;
        ListNode*cur1=head1;
        ListNode*cur2=head2;

        while(cur1&&cur2){
            if(cur1->val<cur2->val){
                cur_dummy->next=cur1;
                cur1=cur1->next;
            }
            else{
                cur_dummy->next=cur2;
                cur2=cur2->next;
            }

            cur_dummy=cur_dummy->next;
        }

        if(cur1)cur_dummy->next=cur1;
        else if(cur2)cur_dummy->next=cur2;

        return dummy_head->next;
    }

    ListNode* merge(vector<ListNode*>& lists,int left,int right){
        if(left==right)return lists[left];
        else if(left>right)return nullptr;
        int mid=left+((right-left)>>1);
        return mergeTwoLists(merge(lists,left,mid),merge(lists,mid+1,right));
    }

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        return merge(lists,0,lists.size()-1);
    }
};
```

### [146. LRU 缓存](https://leetcode.cn/problems/lru-cache/)

```cpp
struct DLinkedNode {
    int key, value;
    DLinkedNode* prev;
    DLinkedNode* next;
    DLinkedNode(): key(0), value(0), prev(nullptr), next(nullptr) {}
    DLinkedNode(int _key, int _value): key(_key), value(_value), prev(nullptr), next(nullptr) {}
};

class LRUCache {
private:
    unordered_map<int, DLinkedNode*> cache;
    DLinkedNode* dummy_head;
    DLinkedNode* dummy_tail;
    int size;
    int capacity;

public:
    LRUCache(int capacity) {
        this->dummy_head=new DLinkedNode();
        this->dummy_tail=new DLinkedNode();
        this->dummy_head->next=this->dummy_tail;
        this->dummy_tail->prev=this->dummy_head;
        this->size=0;
        this->capacity=capacity;
    }
    
    int get(int key) {
        if(!cache.count(key))return -1;

        DLinkedNode* node=cache[key];
        moveToHead(node);
        return node->value;
    }
    
    void put(int key, int value) {
        if(!cache.count(key)){
            DLinkedNode* node=new DLinkedNode(key,value);
            cache[key]=node;
            addToHead(node);
            this->size++;
            
            if(this->size>this->capacity){
                DLinkedNode* removed=removeTail();
                this->cache.erase(removed->key);
                delete removed;
                size--;
            }
        }else{
            DLinkedNode* node=cache[key];
            node->value=value;
            moveToHead(node);
        }
    }

    void addToHead(DLinkedNode* node){
        node->next=this->dummy_head->next;
        node->prev=this->dummy_head;
        this->dummy_head->next->prev=node;
        this->dummy_head->next=node;
    }

    void removeNode(DLinkedNode* node){
        node->prev->next=node->next;
        node->next->prev=node->prev;
    }

    void moveToHead(DLinkedNode* node){
        removeNode(node);
        addToHead(node);
    }

    DLinkedNode* removeTail() {
        DLinkedNode* node = this->dummy_tail->prev;
        removeNode(node);
        return node;
    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
```



## 二叉树

### [94. 二叉树的中序遍历](https://leetcode.cn/problems/binary-tree-inorder-traversal/)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    void inorder(TreeNode*node, vector<int>&vec){
        if(!node)return;
        inorder(node->left,vec);
        vec.emplace_back(node->val);
        inorder(node->right,vec);
    }

    vector<int> inorderTraversal(TreeNode* root) {
        vector<int>res;
        inorder(root,res);
        return res;
    }
};
```

### [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int dfs(TreeNode* node) {
        if(!node) return 0;
        return max(dfs(node->left),dfs(node->right))+1;
    }

    int maxDepth(TreeNode* root) {
        return dfs(root);
    }
};
```

### [226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if(!root)return nullptr;
        queue<TreeNode*>que;
        que.push(root);

        while(!que.empty()){
            for(int i=que.size();i>0;i--){
                TreeNode* node=que.front();que.pop();
                swap(node->left,node->right);
                if(node->left)que.push(node->left);
                if(node->right)que.push(node->right);
            }
        }

        return root;
    }
};
```

### [101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    // 对称是两个子树要对称，而不只是左右子节点对称就行，因此需要两个子树的根节点
    bool dfs(TreeNode* left,TreeNode*right){
        if(!left&&!right)return true;
        else if(!left||!right)return false;
        else if(left->val!=right->val)return false;
        return dfs(left->left,right->right)&&dfs(left->right,right->left);
    }

    bool isSymmetric(TreeNode* root) {
        if(!root)return true;
        return dfs(root->left,root->right);
    }
};
```

### [543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int res;

    int depth(TreeNode* node){
        if(!node)return 0;
        int L=depth(node->left);
        int R=depth(node->right);
        res=max(res,L+R);
        return max(L,R)+1;
    }

    int diameterOfBinaryTree(TreeNode* root) {
        res=0;
        depth(root);
        return res;
    }
};
```

### [102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>>res;
        if(!root)return res;

        queue<TreeNode*>que;
        que.push(root);

        while(!que.empty()){
            vector<int>tmp;
            for(int i=que.size();i>0;i--){
                TreeNode*node=que.front();que.pop();
                tmp.emplace_back(node->val);
                if(node->left)que.push(node->left);
                if(node->right)que.push(node->right);
            }
            res.emplace_back(tmp);
        }

        return res;
    }
};
```

### [108. 将有序数组转换为二叉搜索树](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* dfs(vector<int>& nums,int left, int right){
        if(left<=right){
            int mid=left+((right-left)>>1);
            TreeNode*node=new TreeNode(nums[mid]);
            node->left=dfs(nums,left,mid-1);
            node->right=dfs(nums,mid+1,right);
            return node;
        }
        return nullptr;
    }

    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return dfs(nums,0,nums.size()-1);
    }
};
```

