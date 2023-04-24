# LeetCode

## [1. 两数之和](https://leetcode.cn/problems/two-sum/)

```cpp
思路:
1.定义哈希表record，遍历数组nums，假设当前值为num，(key:target-num,value:i)
2.如果num出现在哈希表中，说明之间有一个值加上当前值等于target，返回{record[num],i}

代码:
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int,int>record;//(key:target-num,value:i)
        //index1,index2记录两数的索引
        int index1=0;
        int index2=0;
        for(int i=0;i<nums.size();i++){
            if(record.find(nums[i])==record.end()){//未存放在record中
                record[target-nums[i]]=i;
            }else{
                index1=record[nums[i]];
                index2=i;
            }
        }
        return {index1,index2};
    }
};

时空复杂度分析:
时间复杂度：O(n)  遍历整个数组;
空间复杂度：O(n)  哈希表record记录索引;
```

## [2. 两数相加](https://leetcode.cn/problems/add-two-numbers/description/)

```cpp
思路：
1.链表已经是逆序，所以遍历链表的顺序刚好和做加法的顺序一样，从低位开始，我们就不需要反转链表了
2.其实只需要考虑保存进位传递给高位，其中有关键的两点
  2.1两个链表的长度不同，即位数不同，只需要让长链表多出来的部分加0即可
  2.2最高位相加如果有进位的话，需要新增节点在链表尾
3.为了节省空间，我们可以在长链表原地操作。但计算长度需要花费额外时间，时间比空间更重要，所以优先开率时间开销

算法过程：
1.声明需要用到的变量，结果链表、遍历结果链表的辅助指针、记录l1、l2的值、相加和、进位
2.遍历两链表，确定循环条件(有一个不为空就继续循环)，空指针的节点值就当作0，不影响相加和，移动链表指针
3.判断最高位是否要补1
4.返回的是结果链表头的next指针，因为一开始就直接将相加和的节点置为cur->next

代码：
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
        //1.声明相关变量
        ListNode*sumHead=new ListNode(-1);//结果和的链表头
        ListNode*cur=sumHead;//用于补充链表
        int sum=0;//当前节点的相加和
        int carry=0;//当前节点的传给高位的进位
        int val1=0;//l1当前节点值
        int val2=0;//l2当前节点值

        //2.遍历链表，即相加
        while(l1||l2){//只要l1,l2有一个不为空就继续循环
            val1=l1?l1->val:0;
            val2=l2?l2->val:0;
            
            sum=val1+val2+carry;
            ListNode *newNode=new ListNode(sum%10);
            carry=sum/10;

            //向后移动
            cur->next=newNode;
            cur=cur->next;
            //l1、l2只有不为空指针才能移动
            if(l1)l1=l1->next;
            if(l2)l2=l2->next;
        }
		
        //3.最高位补1
        if(carry){//退出循环，如果进位是1，说明最高位的更高一位要补1,
            cur->next=new ListNode(1);
        }
		
        return sumHead->next;
    }
};

时空复杂度分析:
时间复杂度：O(n)  遍历最长链表;
空间复杂度：O(n)  结果链表的长度;
```

## [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

```cpp
子串：连续的
子序列：不连续，但相对顺序一致

思路：滑动窗口
1.窗口意味着有边界，定义left为左边界，right为右边界，哈希表record，key(字符)，value(字符在窗口中出现的次数)
2.right不断右扫加入新的字符到窗口中，首先需要判断是否有重复字符在窗口中，如果有就不断让left右移，一个一个删去字符直到没有重复字符为止。如果没有重复字符，加入新字符后，判断当前窗口大小是否大于已知最大大小，如果大于就更新

卡点：
一开始我一直在想如果要加入的新字符和窗口中间的字符重复的话，岂不是要遍历整个窗口去查到底是否有重复字符。但是其实可以用哈希表记录当前要加入的字符是否已经在窗口中，然后从left向右不断删去就好了。因为是子串，所以必须是连续的，中间有重复字符要从中间删去的话，那左边的必然也被切割了，所以不如直接从左边开始一个一个删直到没有重复字符即可。

算法：
1.定义left，right，record，maxLen
2.滑动窗口遍历字符串，注意细节的判断
3.返回最大长度

代码：
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        //窗口[left,right)
        int left=0;
        int right=0;
        int maxLen=0;
        unordered_map<char,int>record;
        while(right<s.length()){
            while(right<s.length()&&record[s[right]]==0){//没有重复字符时，才可以加入
                record[s[right++]]++;
                maxLen=max(maxLen,right-left);
            }
            while(right<s.length()&&record[s[right]]==1){//有重复字符时，就从左边开始一直删直到没有重复字符
                record[s[left++]]--;
            }
        }
        return maxLen;
    }
};

时空复杂度分析:
时间复杂度：O(n)  滑动窗口移动，由于right每次循环时都是加，没有往回走，所以即使有两个while循环，但仍然是n;
空间复杂度：O(n)  哈希表的最坏情况是把整个字符串都记录下来了;

小结：之前写过两次这道题，但这一次还是被如何判断重复字符卡住了，因为总想着中间出现重复字符怎么办，最后意识到因为是子串，所以从左向右删除即可。还有就是虽然看的是别人的思路，但最好还是按着自己的理解手写一遍代码，因为这才是属于自己的代码
```

