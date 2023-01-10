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

## 二、链表

### 1. LeetCode203. 移除链表元素

```c++
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        ListNode*resHead=new ListNode(-1);//虚拟头节点，防止丢失
        resHead->next=head;
        ListNode*tmp=head;//tmp去遍历所有节点
        ListNode*pre=resHead;//tmp的前驱节点
        while(tmp!=NULL){
            if(tmp->val==val){
                pre->next=tmp->next;
            }else{//pre只有当tmp的值不等于val时，才移动。因为如果当前节点要被删除，就无法成为前驱节点
                pre=tmp;
            }
            //tmp无论如何都要往下一个节点移动
            tmp=tmp->next;
        }
        return resHead->next;
    }
};
```

### 2. LeetCode707.设计链表

```c++
//注意及时return，避免函数后面的操作干扰特殊情况的结果
class MyLinkedList {
public:
    ListNode*head;
    MyLinkedList() {
        //把链表头初始化为空指针
        head=NULL;
    }
    
    int get(int index) {
        //如果链表头为空指针，直接返回-1
        if(head==NULL)return -1;
        //创建头指针副本
        ListNode*tmp=head;
        while(index--){//用后置减的原因：head的索引为0
            tmp=tmp->next;
            //如果index还没到0，但是tmp已经是空指针了，说明index越界了，返回-1
            if(tmp==NULL)return -1;
        }
        //tmp一定不是空指针，直接返回值即可
        return tmp->val;
    }
    
    void addAtHead(int val) {
        //就算链表头是空指针也无妨
        ListNode*newNode=new ListNode(val);
        newNode->next=head;
        head=newNode;
    }
    
    void addAtTail(int val) {
        //如果链表头是空指针，无法调用head->next，所以先判断一下
        ListNode*newNode=new ListNode(val);
        if(head==NULL){
            head=newNode;
            return;
        }
        ListNode*tmp=head;
        while(tmp->next!=NULL){
            tmp=tmp->next;
        }
        tmp->next=newNode;
    }
    
    void addAtIndex(int index, int val) {
        if(index<=0){//index小于0，将其加到链表头
            addAtHead(val);
            return;
        }

        ListNode*tmp=head;
        ListNode*pre=NULL;//前驱节点
        while(index>0&&tmp!=NULL){
            pre=tmp;
            tmp=tmp->next;
            index--;
        }
        if(tmp==NULL&&index>0)return;//索引大于链表长度
        if(tmp==NULL&&index==0){//索引等于链表长度，添加到链表尾，注意及时返回
            addAtTail(val);
            return;
        }

        //tmp!=NULL
        ListNode*newNode=new ListNode(val);
        pre->next=newNode;
        newNode->next=tmp;
    }
    
    void deleteAtIndex(int index) {
        //链表凹凸为空指针，没有可以删除的节点
        if(head==NULL)return;
        //删除索引若为0，意味着删除链表头，直接让链表头后移即可，注意及时return
        if(index==0){
            head=head->next;
            return;
        }

        ListNode*tmp=head;
        ListNode*pre=NULL;//前驱节点
        while(index--){
            pre=tmp;
            tmp=tmp->next;
            if(tmp==NULL)return;
        }
        pre->next=tmp->next;
    }
};
```

### 3. LeetCode206. 反转链表

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head==NULL||head->next==NULL){
            return head;
        }
        ListNode*reverseHead=new ListNode(-1);//前驱节点，防止丢失链表
        ListNode*cur=head;
        ListNode*next=cur->next;
        while(cur!=NULL){
            next=cur->next;//先记录好原链表的next节点
            cur->next=reverseHead->next;
            reverseHead->next=cur;
            cur=next;
        }
        return reverseHead->next;
    }
};
```

### 4. LeetCode24. 两两交换链表中的节点

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/1.png)

```c++
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        if(head==NULL||head->next==NULL)return head;
        ListNode*swapHead=new ListNode(-1);//创建前驱节点，防止链表丢失
        ListNode*cur=head;//当前需转换对的第一个节点
        ListNode*next=cur->next->next;//下一对的第一个节点
        ListNode*pre=swapHead;
        while(cur!=NULL&&cur->next!=NULL){//如果最后剩下节点数小于等于1，就不需要再做任何处理了
            next=cur->next->next;//因为要改变cur->next->next，所以先备份
            //由于前一对总是直接连接到后一对的第二个节点，所以会遗漏节点。因此创建一个前驱节点来前后连接
            pre->next=cur->next;
            //前驱节点即为每一对做完交换的后一个节点
            pre=cur;
            cur->next->next=cur;
            cur->next=next;
            cur=next;
        }
        return swapHead->next;
    }
};
```

### 5. LeetCode19. 删除链表的倒数第 N 个结点

```c++
双指针：快慢指针法
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode*dummyHead=new ListNode(0);
        dummyHead->next=head;
        ListNode*slow=dummyHead;
        ListNode*fast=dummyHead;

        //让fast先走n+1步，然后再让fast和slow一起移动，结束时，slow指向要删除节点的前一个
        while(n--&&fast!=NULL){
            fast=fast->next;
        }
        fast=fast->next;

        while(fast!=NULL){
            slow=slow->next;
            fast=fast->next;
        }

        slow->next=slow->next->next;
        return dummyHead->next;
    }
};
```

### 6. LeetCode面试题 02.07. 链表相交

```c++
利用A、B链表的长度差，相当于让长的链表先走长度差步
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode*A=headA;
        ListNode*B=headB;
        while(A!=NULL||B!=NULL){//A和B同时为空说明无交点
            if(A==B)return A;
            A=A==NULL?headB:A->next;
            B=B==NULL?headA:B->next;
        }
        return NULL;
    }
};
```

### 7. LeetCode142. 环形链表 II

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/2.png)

```c++
双指针：快慢指针法
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode*fast=head;
        ListNode*slow=head;
        while(fast!=NULL&&fast->next!=NULL){
            fast=fast->next->next;
            slow=slow->next;
            if(fast==slow){
                ListNode*index1=fast;
                ListNode*index2=head;
                while(index1!=index2){
                    index1=index1->next;
                    index2=index2->next;
                }
                return index1;
            }
        }
        return NULL;
    }
};

糊涂版本代码：
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        //定义快慢指针
        ListNode*slow=head;
        ListNode*fast=head;
        //由于一开始对快慢指针的定义，导致函数根本无法进入该循环
        while(fast!=slow){
            fast=fast->next->next;
            slow=slow->next;
        }
        ListNode*index1=head;
        ListNode*index2=slow;
        while(index1!=index2){
            index1=index1->next;
            index2=index2->next;
        }
        return index2;
    }
};
```

## 三、哈希表

### 1. LeetCode242. 有效的字母异位词

```c++
class Solution {
public:
    bool isAnagram(string s, string t) {
        vector<int>record(26);//26个字母
        for(int i=0;i<s.length();i++){//记录字符串s中各字母词频
            record[s[i]-'a']++;
        }

        for(int i=0;i<t.length();i++){//把相应字符词频减1
            record[t[i]-'a']--;
        }

        for(int i=0;i<record.size();i++){
            if(record[i]!=0){//如果有不为0的元素，说明字符串s、t的词频不等
                return false;
            }
        }
        
        return true;
    }
};
```

### 2. LeetCode349. 两个数组的交集

```c++
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        set<int>record(nums1.begin(),nums1.end());//记录nums1出现过的数
        set<int>recordRes;//给结果集降重
        for(int num:nums2){
            if(record.count(num)!=0){
                recordRes.insert(num);//set自带降重，因为不会添加重复元素
            }
        }
        return vector<int>(recordRes.begin(),recordRes.end());
    }
};
```

### 	3. LeetCode202. 快乐数

```c++
题目出现“无限循环”字眼，说明有重复出现的值，所以利用set记录出现过的sum，保证不会陷入死循环。
class Solution {
public:
    //计算各位平方和
    int getSum(int n){
        int sum=0;
        while(n>0){
            sum+=(n%10)*(n%10);
            n/=10;
        }
        return sum;
    }

    bool isHappy(int n) {
        set<int>record;//记录sum
        while(true){
            int sum=getSum(n);
            if(sum==1)return true;
            if(record.count(sum)!=0){//说明曾经出现过
                return false;
            }else{
                record.insert(sum);
            }
            n=sum;
        }
    }
};
```

### 4. LeetCode1. 两数之和

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        //key：target-nums[value]
        //value：索引
        map<int,int>record;
        vector<int>res;
        for(int i=0;i<nums.size();i++){
            if(record.count(nums[i])!=0){
                res={i,record[nums[i]]};
                break;
            }else{
                record[target-nums[i]]=i;
            }
        }
        return res;
    }
};
```

### 5. LeetCode454.四数相加II

```c++
class Solution {
public:
    int fourSumCount(vector<int>& nums1, vector<int>& nums2, vector<int>& nums3, vector<int>& nums4) {
        //key：num1+num2
        //value：key的次数
        map<int,int>record;
        int count=0;

        //记录num1+num2的次数
        for(int num1:nums1){
            for(int num2:nums2){
                record[num1+num2]++;
            }
        }

        //判断num3+num4是否等于-(num1+num2)，如果相等，四元组个数增加
        for(int num3:nums3){
            for(int num4:nums4){
                    count+=record[-(num3+num4)];
            }
        }

        return count;
    }
};
```

### 6. LeetCode383. 赎金信

```c++
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        //记录magazine各字符的频率，只有26个字母
        vector<int>record(26);
        for(int i=0;i<magazine.size();i++){
            record[magazine[i]-'a']++;
        }

        for(int i=0;i<ransomNote.size();i++){
            if((--record[ransomNote[i]-'a'])<0){//magazine的字符不够用了
                return false;
            }
        }
        
        return true;
    }
};
```

### 7. LeetCode15.三数之和

```c++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        //因为最终是返回元素值而不是下标，所以可以排序
        sort(nums.begin(),nums.end());
        vector<vector<int>>res;//结果集

        //找到a+b+c=0
        //找到nums[i...]组成的三元组
        for(int i=0;i<nums.size();i++){
            if(nums[i]>0){//由于是升序数组，当前值大于0，后面的值肯定都大于0
                break;
            }

            //第一元素去重
            //1.一定是nums[i]和nums[i-1]比较
            //2.而不是nums[i+1]和nums[i]比较
            //3.因为如果相等，nums[i-1]一定比nums[i]先进入结果集，必定重复
            //4.nums[i+1]只是重复元素，并不一定构成重复结果集

            //1.i>0一定优先判断，不然会取到nums[-1]然后进行判断
            if(i>0&&nums[i]==nums[i-1]){
                continue;
            }

            //定义左右边界，在[left,right]间寻找能和nums[i]匹配的两个数
            int left=i+1;
            int right=nums.size()-1;

            while(left<right){
                if(nums[i]+nums[left]+nums[right]>0){//偏大
                    right--;
                }else if(nums[i]+nums[left]+nums[right]<0){//偏小
                    left++;
                }else{
                    res.push_back({nums[i],nums[left],nums[right]});
                    while(left<right&&nums[left]==nums[left+1])left++;//第二元素去重
                    while(right>left&&nums[right]==nums[right-1])right--;//第三元素去重
                    //出循环后，仍然是相同元素，再向内收缩
                    left++;
                    right--;
                }
            }
        }
        return res;
    }
};
```

### 8. LeetCode18.四数之和

```c++
思路：
整体思路和三数之和大同小异，关键在于以下三点：
1.剪枝：最小的值都一直大于target，但必须保证nums[i]>=0
2.去重：当前元素不能和上一个结果集的同位置元素相同
3.防溢出：四数相加后可能会溢出，所以提前把一个数转为long，然后再和其他数相加，这样就都为long了
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        vector<vector<int>>res;
        sort(nums.begin(),nums.end());
        for(int i=0;i<nums.size()-1;i++){//[i,j]、[left,right]各两个元素
            //剪枝
            //注意两个负数相加和会更小
            if(nums[i]>target&&nums[i]>=0){
                break;
            }
            //对nums[i]去重
            if(i>0&&nums[i]==nums[i-1]){
                continue;
            }
            for(int j=i+1;j<nums.size();j++){
                //剪枝
                if(nums[i]+nums[j]>target&&nums[i]+nums[j]>=0){
                    break;
                }
                //对nums[j]去重
                if(j>i+1&&nums[j]==nums[j-1]){
                    continue;
                }
                int left=j+1;
                int right=nums.size()-1;
                while(left<right){//相加后可能会溢出，先转为long再相加，相加后再转中途会溢出
                    if((long)nums[i]+nums[j]+nums[left]+nums[right]>target){
                        right--;
                    }else if((long)nums[i]+nums[j]+nums[left]+nums[right]<target){
                        left++;
                    }else{
                        res.push_back({nums[i],nums[j],nums[left],nums[right]});
                        //去重
                        while(left<right&&nums[left]==nums[left+1])left++;
                        while(right>left&&nums[right]==nums[right-1])right--;
                        left++;
                        right--;
                    }
                }
            }
        }
        return res;
    }
};
```

## 四、链表

### 1. LeetCode344. 反转字符串

```c++
class Solution {
public:
    void reverseString(vector<char>& s) {
        if(s.size()==1){
            return;
        }

        for(int i=0,j=s.size()-1;i<j;i++,j--){
            swap(s[i],s[j]);
        }
    }
};
```

### 2. LeetCode541. 反转字符串 II

```c++
class Solution {
public:
    string reverseStr(string s, int k) {
        if(s.length()==1){
            return s;
        }


        for(int i=0;i<s.length();i+=k*2){//每2k个字符进行处理，i每次往后移动2k
            if(i>(int)s.length()-k){//由于s.length()为64位无符号整数，和k做运算时要先转为int
                reverse(s.begin()+i,s.end());
                break;
            }else{
                reverse(s.begin()+i,s.begin()+i+k);
            }
        }
        return s;
    }
};
```

### 3. LeetCode剑指 Offer 05. 替换空格

```c++
双指针：前后指针法
1.不用申请新字符串，节省额外空间
2.从后向前扩充字符串，不用每次填写后向后移动后面所有元素
class Solution {
public:
    string replaceSpace(string s) {
        if(s.length()==0)return s;

        int sOldSize=s.size();//原长度
        
        int countSpace=0;//记录空格数
        for(int i=0;i<s.length();i++){
            if(s[i]==' ')countSpace++;
        }

        s.resize(s.length()+2*countSpace);//在原字符串基础上进行扩充
        int sNewSize=s.size();
        for(int i=sOldSize-1,j=sNewSize-1;i<j;i--,j--){//当i==j时，说明前面的字符串没有空格了
            if(s[i]!=' '){
                s[j]=s[i];
            }else{
                s[j]='0';
                s[j-1]='2';
                s[j-2]='%';
                j-=2;
            }
        }
        return s;
    }
};
```

### 4. LeetCode151. 反转字符串中的单词

```c++
思路：
1.移除多余空格
2.反转整个字符串
3.反转每个单词
class Solution {
public:
    string reverseWords(string s) {
        //1.移除多余空格
        removeExtraSpaces(s);

        //2.反转整个字符串
        reverse(s,0,s.length()-1);

        //3.反转每个单词
        int start=0;
        for(int i=0;i<=s.length();i++){
            if(i==s.length()||s[i]==' '){//最后一个单词后没有空格
                reverse(s,start,i-1);//s[i-1]是前一个单词的最后一个字符
                start=i+1;//开始位更新为下一个单词的开头
            }
        }
        return s;
    }

    //移除字符串s多余空格
    void removeExtraSpaces(string&s){
        int slow=0;
        for(int fast=0;fast<s.length();fast++){//将快指针不为空格的字符填到慢指针
            if(s[fast]!=' '){
                if(slow!=0)s[slow++]=' ';
                while(fast<s.length()&&s[fast]!=' '){
                    s[slow++]=s[fast++];
                }
            }
        }
        s.resize(slow);//slow的大小和移除多余空格后的字符串长度相等
    }

    void reverse(string&s,int start,int end){
        for(int i=start,j=end;i<j;i++,j--){
            swap(s[i],s[j]);
        }
    }
};
```

### 5. LeetCode剑指 Offer 58 - II. 左旋转字符串

```c++
class Solution {
public:
    string reverseLeftWords(string s, int n) {
        int sOldSize=s.length();
        s+=s;//所有左旋转子串都是拼接字符串的子串，且第一位和n相等
        return s.substr(n,sOldSize);
    }
};
```

### 6. LeetCode28. 找出字符串中第一个匹配项的下标

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/3.png)

```c++
前缀：指不包含最后一个字符的所有以第一个字符开头的连续子串。
后缀：指不包含第一个字符的所有以最后一个字符结尾的连续子串。
class Solution {
public:
    int strStr(string haystack, string needle) {
        int i1=0;//haystack中比对的下标
        int i2=0;//needle中比对的下标
        vector<int>next=getNext(needle);

        while(i1<haystack.length()&&i2<needle.length()){
            if(haystack[i1]==needle[i2]){
                i1++;
                i2++;
            }else if(i2==0){//没有前缀了就让i1后移
                i1++;
            }else{
                //换作前缀部分去和当前大的haystack[i1]比较
                //因为next数组记录的是前缀长度的特性，所以一定是i2往前跳而不是i1往后跳
                //相当于把needle相应前缀拉到当前取比对
                i2=next[i2];
            }
        }
        //结束循环后，除非i2比对到最后一个字符且相等
        //否则就是i1都比对到最后字符也没有找到相应子串
        return i2==needle.length()?i1-i2:-1;
    }

    //获取字符串str的next数组
    vector<int>getNext(string str){
        if(str.length()==1)return{-1,0};

        vector<int>next(str.length());//next[i]：str(0,i)最长公共前后缀长度，不包含str[i]
        //默认值
        next[0]=-1;
        next[1]=0;
        int i=2;//0、1已确定，从2开始
        int cn=0;//cn是next[2-1]的值，也恰好是要对比的索引
        while(i<str.length()){
            if(str[i-1]==str[cn]){
                next[i++]=++cn;
            }else if(cn>0){//回到上一个可能是最长公共前缀的最后一位
                cn=next[cn];
            }else{
                next[i++]=0;
            }
        }
        return next;
    }
};
```

### 7. LeetCode459. 重复的子字符串

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/4.png)

```c++
移动匹配：
任何可由重复子串组成的字符串都可以分成相同的两半，所以把该字符串拼接起来后，必定会包含原字符串
class Solution {
public:
    bool repeatedSubstringPattern(string s) {
        string ss=s+s;
        //掐头去尾，不然ss里必定包含s
        ss.erase(ss.begin());
        ss.erase(ss.end()-1);
        //如果ss里有s，说明s可以被分成相同的两半，也就可以由重复子串组成了
        if(ss.find(s)==std::string::npos)return false;
        return true;
    }
};

kmp：
能由重复子串组成的字符串的最小组成单位是最长公共前后缀不包含的那一部分
class Solution {
public:
    bool repeatedSubstringPattern(string s) {
        if(s.size()==1){
            return false;
        }
        //next[i]：s(0,i)最长公共前后缀长度-1，因为j从-1开始
        vector<int>next=getNext(s);
        int maxPreLen=next[next.size()-1];
        int len=s.length()-maxPreLen-1;
        if(maxPreLen!=-1&&s.length()%len==0){
            return true;
        }
        return false;
    }

    vector<int>getNext(string s){
        vector<int>next(s.length());
        next[0]=-1;
        int j=-1;
        for(int i=1;i<next.size();i++){
            while(j>=0&&s[i]!=s[j+1]){
                j=next[j];
            }
            if(s[i]==s[j+1]){
                j++;
            }
            next[i]=j;
        }
        return next;
    }
};
```



## 五、栈与队列

### 1. LeetCode232. 用栈实现队列

```c++
思路：
1.准备两个栈pushStk、popStk
2.队列push时：直接push到pushStk
3.队列pop时：先判断popStk是否为空
  3.1不空：记录popStk栈顶元素、popStk做pop操作、返回栈顶元素
  3.2空：把pushStk栈顶元素不断弹出加入到popStk中直至空(pushStk栈底元素是最先加入的，到popStk中就是栈顶元素了)
class MyQueue {
    stack<int>pushStk;
    stack<int>popStk;
public:
    MyQueue() {}
    
    void push(int x) {
        pushStk.push(x);
    }
    
    int pop() {
        if(popStk.empty()){
            while(!pushStk.empty()){
                popStk.push(pushStk.top());
                pushStk.pop();
            }
        }

        int res=popStk.top();
        popStk.pop();
        return res;
    }
    
    int peek() {
        if(popStk.empty()){
            while(!pushStk.empty()){
                popStk.push(pushStk.top());
                pushStk.pop();
            }
        }

        return popStk.top();
    }
    
    bool empty() {
        //两个栈都存放着元素，只有两个栈同时为空才能说明队列为空
        return pushStk.empty()&&popStk.empty();
    }
};
```

### 2. LeetCode225. 用队列实现栈

```c++
class MyStack {
public:
    queue<int>que;
    MyStack() {}
    
    void push(int x) {
        que.push(x);
    }
    
    int pop() {
        //找到队尾元素并弹出即可
        int size=que.size();
        while((--size)>0){//将队头不断加入队尾，就可以让初始队尾到队头
            que.push(que.front());
            que.pop();            
        }
        int res=que.front();
        que.pop();
        return res;
    }
    
    int top() {
        //队尾元素是最后进来的，即栈顶元素
        return que.back();
    }
    
    bool empty() {
        return que.empty();
    }
};
```

### 3. LeetCode20. 有效的括号

```c++
class Solution {
public:
    bool isValid(string s) {
        //key：右括号
        //value：与key对应的左括号
        map<char,char>m;
        m[')']='(';
        m[']']='[';
        m['}']='{';
        stack<char>record;//放入当前遍历过的左括号
        for(int i=0;i<s.length();i++){
            if(s[i]=='('||s[i]=='['||s[i]=='{'){
                record.push(s[i]);
            }else{
                //当前遇到的左括号必定多于右括号，所以record中途不可能为空，否则就是无效的
                //或者当前遇到的右括号和最新的左括号无法匹配，也就无法闭合，同样也是无效的
                if(record.empty()||record.top()!=m[s[i]]){
                    return false;
                }else{
                    //可以匹配，消去
                    record.pop();
                }
            }
        }
        //如果record不为空，说明有多余左括号无法匹配
        return record.empty();
    }
};
```

### 4. LeetCode1047. 删除字符串中的所有相邻重复项

```c++
class Solution {
public:
    string removeDuplicates(string s) {
        if(s.length()==1){
            return s;
        }

        //record用于记录前一个无重复项的字母
        stack<char>record;
        for(int i=0;i<s.length();i++){
            if(record.empty()||record.top()!=s[i]){
                record.push(s[i]);
            }else{
                //有重复项，删除前一个字母，并且当前字母不入栈
                record.pop();
            }
        }

        //最终留在栈里的拼接成字符串
        string res;
        while(!record.empty()){
            res.push_back(record.top());
            record.pop();
        }
        //反转字符串
        reverse(res.begin(),res.end());
        return res;
    }
};
```

### 5. LeetCode150. 逆波兰表达式求值

```c++
class Solution {
public:
    int evalRPN(vector<string>& tokens) {
        stack<int>record;//符号不会入栈，所以用int类型的栈
        for(int i=0;i<tokens.size();i++){
            if(tokens[i]=="+"||tokens[i]=="-"||tokens[i]=="*"||tokens[i]=="/"){
                //取完元素后记得弹出
                int num1=record.top();
                record.pop();
                int num2=record.top();
                record.pop();
                int newNum=0;
                //注意是先加入栈的数对后加入栈的数作相应运算
                if(tokens[i]=="+"){
                    newNum=num2+num1;
                }else if(tokens[i]=="-"){
                    newNum=num2-num1;
                }else if(tokens[i]=="*"){
                    newNum=num2*num1;
                }else{
                    newNum=num2/num1;
                }
                //把结果入栈
                record.push(newNum);
            }else{
                record.push(stoi(tokens[i]));
            }
        }
        return record.top();
    }
};
```

### 6. LeetCode239. 滑动窗口最大值

```c++
代码整体结构：
1.保证双端队列从队头到队尾所对应元素大小严格降序
2.判断队头元素是否过期
3.将最大值加入到结果集中，且只有当窗口形成时才能加入
    
队列没有必要维护窗口里的所有元素，只需要维护有可能成为窗口里最大值的元素就可以了，同时保证队列里的元素数值是由大到小的。
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        //记录可能成为最大值元素的索引
        //因为利用所以既可以知道相对位置，也可知道元素大小
        //先进的大值必定也先出去，和队列的特性不谋而合，所以用队列
        deque<int>dqu;
        vector<int>res;//结果集
        for(int i=0;i<nums.size();i++){//以i为结尾的窗口最大值
            //队头到队尾对应元素大小严格降序，队头永远放着以i结尾的窗口的最大值的索引
            //如果当前元素比队列中某些元素大，就删掉队列中不可能成为最大值的元素
            while(!dqu.empty()&&nums[dqu.back()]<=nums[i]){
                dqu.pop_back();
            }
            dqu.push_back(i);
            if(dqu.back()-dqu.front()>=k){//窗口移动
                dqu.pop_front();
            }
            if(i>=k-1){
                res.push_back(nums[dqu.front()]);
            }
        }
        return res;
    }
};
```

### 7. LeetCode347. 前 K 个高频元素

```c++
class Solution {
public:
    class myComparison{//比较器
    public:
        bool operator()(const pair<int,int>&lhs,const pair<int,int>&rhs){
            return lhs.second>rhs.second;
        }
    };
    
    vector<int> topKFrequent(vector<int>& nums, int k) {
        //key：元素
        //value：词频
        map<int,int>record;
        for(int num:nums){
            record[num]++;
        }

        //小顶堆
        priority_queue<pair<int,int>,vector<pair<int,int>>,myComparison>pri_que;
        for(map<int,int>::iterator it=record.begin();it!=record.end();it++){
            pri_que.push(*it);
            if(pri_que.size()>k){
                pri_que.pop();//把最小的弹出
            }
        }
        vector<int>res(k);
        for(int i=k-1;i>=0;i--){
            res[i]=pri_que.top().first;
            pri_que.pop();
        }
        return res;
    }
};
```

## 六、二叉树

### 1. LeetCode144. 二叉树的前序遍历

```c++
迭代法：
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        if(root==NULL)return {};
        stack<TreeNode*>stk;
        vector<int>res;
        stk.push(root);
        while(!stk.empty()){
            TreeNode*node=stk.top();
            stk.pop();
            res.push_back(node->val);
            //要让左节点先被弹出，所以后入栈
            if(node->right)stk.push(node->right);
            if(node->left)stk.push(node->left);
        }
        return res;
    }
};
```

### 2. LeetCode94. 二叉树的中序遍历

```c++
迭代法：
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        stack<TreeNode*>stk;
        vector<int>res;
        TreeNode*cur=root;
        while(cur!=NULL||!stk.empty()){
            if(cur!=NULL){//指针访问节点，先到最底层
                stk.push(cur);//将访问到的节点入栈
                cur=cur->left;//左
            }else{
                cur=stk.top();//从栈里弹出的就是要处理的数据
                stk.pop();
                res.push_back(cur->val);//中
                cur=cur->right;//右
            }
        }
        return res;
    }
};
```

### 3. LeetCode145. 二叉树的后序遍历

```c++
思路：
前序遍历数组顺序（中左右），后序遍历数组顺序（左右中），让前序遍历数组顺序变为（中右左），再反转数组
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        if(root==NULL)return {};
        stack<TreeNode*>stk;
        vector<int>res;
        stk.push(root);
        while(!stk.empty()){
            TreeNode*node=stk.top();
            stk.pop();
            res.push_back(node->val);
            if(node->left)stk.push(node->left);
            if(node->right)stk.push(node->right);
        }
        reverse(res.begin(),res.end());
        return res;
    }
};
```

### 4. LeetCode102. 二叉树的层序遍历

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        if(root==NULL){
            return {};
        }
        queue<TreeNode*>qu;
        qu.push(root);
        vector<vector<int>>res;
        while(!qu.empty()){
            vector<int>vec;//记录每层节点值
            for(int i=qu.size();i>0;i--){
                TreeNode*node=qu.front();
                qu.pop();//注意qu的size是不断变化的
                vec.push_back(node->val);
                if(node->left)qu.push(node->left);
                if(node->right)qu.push(node->right);
            }
            res.push_back(vec);
        }
        return res;
    }
};
```

### 5.LeetCode107. 二叉树的层序遍历 II

```c++
思路：把从上到下的层序遍历数组反转一下即可
class Solution {
public:
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        if(root==NULL){
            return {};
        }
        queue<TreeNode*>qu;
        qu.push(root);
        vector<vector<int>>res;
        while(!qu.empty()){
            vector<int>vec;//记录每层节点值
            for(int i=qu.size();i>0;i--){
                TreeNode*node=qu.front();
                qu.pop();//注意qu的size是不断变化的
                vec.push_back(node->val);
                if(node->left)qu.push(node->left);
                if(node->right)qu.push(node->right);
            }
            res.push_back(vec);
        }
        reverse(res.begin(),res.end());
        return res;
    }
};
```

### 6. LeetCode199. 二叉树的右视图

```c++
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        vector<int>res;
        if(root==NULL)return res;
        queue<TreeNode*>qu;
        qu.push(root);
        while(!qu.empty()){
            for(int i=qu.size();i>0;i--){
                TreeNode*node=qu.front();
                qu.pop();
                if(node->left)qu.push(node->left);
                if(node->right)qu.push(node->right);
                if(i==1){
                    res.push_back(node->val);//每层最后一个节点即最右节点
                }
            }
        }
        return res;
    }
};
```

### 7. LeetCode637. 二叉树的层平均值

```c++
class Solution {
public:
    vector<double> averageOfLevels(TreeNode* root) {
        vector<double>res;
        if(root==NULL)return res;
        queue<TreeNode*>qu;
        qu.push(root);
        while(!qu.empty()){
            double sum=0;
            int size=qu.size();
            for(int i=0;i<size;i++){
                TreeNode*node=qu.front();
                qu.pop();
                sum+=node->val;
                if(node->left)qu.push(node->left);
                if(node->right)qu.push(node->right);
            }
            res.push_back(sum/size);
        }
        return res;
    }
};
```

### 8. LeetCode429. N 叉树的层序遍历

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(Node* root) {
        vector<vector<int>>res;
        if(root==NULL)return res;
        queue<Node*>qu;
        qu.push(root);
        while(!qu.empty()){
            vector<int>vec;
            for(int i=qu.size();i>0;i--){
                Node*node=qu.front();
                qu.pop();
                vec.push_back(node->val);
                for(int j=0;j<node->children.size();j++){
                    qu.push(node->children[j]);
                }
            }
            res.push_back(vec);
        }
        return res;
    }
};
```

### 9. LeetCode515. 在每个树行中找最大值

```c++
class Solution {
public:
    vector<int> largestValues(TreeNode* root) {
        vector<int>res;
        if(root==NULL)return res;
        queue<TreeNode*>qu;
        qu.push(root);
        while(!qu.empty()){
            int maxVal=INT_MIN;
            for(int i=qu.size();i>0;i--){
                TreeNode*node=qu.front();
                qu.pop();
                maxVal=max(maxVal,node->val);
                if(node->left)qu.push(node->left);
                if(node->right)qu.push(node->right);
            }
            res.push_back(maxVal);
        }
        return res;
    }
};
```

### 10. LeetCode116. 填充每个节点的下一个右侧节点指针

```c++
class Solution {
public:
    Node* connect(Node* root) {
        if(root==NULL)return root;
        queue<Node*>qu;
        qu.push(root);
        while(!qu.empty()){
            int size=qu.size();
            Node*pre=NULL;
            Node*cur=NULL;
            for(int i=0;i<size;i++){
                if(i==0){
                    cur=qu.front();
                    qu.pop();
                    pre=cur;
                }else{
                    cur=qu.front();
                    qu.pop();
                    pre->next=cur;
                    pre=pre->next;
                }
                if(cur->left)qu.push(cur->left);
                if(cur->right)qu.push(cur->right);
            }
            cur->next=NULL;
        }
        return root;
    }
};
```

### 11. LeetCode117. 填充每个节点的下一个右侧节点指针 II

```c++
class Solution {
public:
    Node* connect(Node* root) {
         if(root==NULL)return root;
        queue<Node*>qu;
        qu.push(root);
        while(!qu.empty()){
            int size=qu.size();
            Node*pre=NULL;
            Node*cur=NULL;
            for(int i=0;i<size;i++){
                if(i==0){
                    cur=qu.front();
                    qu.pop();
                    pre=cur;
                }else{
                    cur=qu.front();
                    qu.pop();
                    pre->next=cur;
                    pre=pre->next;
                }
                if(cur->left)qu.push(cur->left);
                if(cur->right)qu.push(cur->right);
            }
            cur->next=NULL;
        }
        return root;
    }
};
```

### 12. LeetCode104. 二叉树的最大深度

```c++
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(root==NULL){
            return 0;
        }
        int depth=max(maxDepth(root->left),maxDepth(root->right))+1;
        return depth;
    }
};
```

### 13. LeetCode111. 二叉树的最小深度

```c++
注意：只有当遍历到叶子节点时，即左右子节点都为空，才能认为最小深度是1
要明白每个节点可能遇到的情况，从而分析得出相应的策略。
class Solution {
public:
    int minDepth(TreeNode* root) {
        if(root==NULL)return 0;
        int leftDepth=minDepth(root->left);
        int rightDepth=minDepth(root->right);
       
        return root->left==NULL||root->right==NULL?
        leftDepth+rightDepth+1:min(leftDepth,rightDepth)+1;
    }
};

class Solution {
    class Info{
    public:
        int minDepth;
        Info(int minDepth){
            this->minDepth=minDepth;
        }
    };

public:
    int minDepth(TreeNode* root) {
        Info data=process(root);
        return data.minDepth;
    }

    Info process(TreeNode*node){
        if(node==NULL)return Info(0);
        if(node->left==NULL&&node->right==NULL)return Info(1);
        Info leftData=process(node->left);//向左子树要信息
        Info rightData=process(node->right);//向右子树要信息
        /*
        处理当前信息，分析当前节点左右子树情况
        1.左右子节点都为空，说明是叶子节点，0+0+1=1
        2.左右子节点有一个为空，那就是不为空的子树高度+1，其中空的深度是0，不影响
        3.都不为空，左右最小高度+1
        */
        if(node->left==NULL||node->right==NULL)
            return Info(leftData.minDepth+rightData.minDepth+1);
        
        return Info(min(leftData.minDepth,rightData.minDepth)+1);
    }
};
```

### 14. LeetCode226. 翻转二叉树

```c++
思路：
只要能遍历二叉树每个节点并交换其左右子节点即可，但是中序遍历例外。
中序遍历顺序：左中右
1.左子节点先交换自己的左右孩子
2.关键：父节点交换自己的左右孩子
3.此时的右子节点是原先的左子节点，其左右孩子被交换两次，而原先左子节点未交换其左右孩子
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if(root==NULL)return root;
        queue<TreeNode*>qu;
        qu.push(root);
        while(!qu.empty()){
            for(int i=qu.size();i>0;i--){
                TreeNode*node=qu.front();
                qu.pop();
                swap(node->left,node->right);
                if(node->left)qu.push(node->left);
                if(node->right)qu.push(node->right);
            }
        }
        return root;
    }
};
```

### 15. LeetCode589. N 叉树的前序遍历

```c++
和二叉树的前序遍历类似，都是先处理当前节点，再按序处理子节点
递归：
class Solution {
public:
    vector<int>res;
    void helper(Node*node){
        if(node==NULL)return;
        res.push_back(node->val);
        for(int i=0;i<node->children.size();i++){
            helper(node->children[i]);
        }
    }

    vector<int> preorder(Node* root) {
        helper(root);
        return res;
    }
};

迭代：
class Solution {
public:
    vector<int> preorder(Node* root) {
        vector<int>res;
        if(root==NULL)return res;
        stack<Node*>stk;
        stk.push(root);
        while(!stk.empty()){
            Node*node=stk.top();
            stk.pop();
            res.push_back(node->val);
            for(int i=node->children.size()-1;i>=0;i--){
                stk.push(node->children[i]);
            }
        }
        return res;
    }
};
```

### 16. LeetCode590. N 叉树的后序遍历

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/5.png)

```c++
class Solution {
public:
    vector<int>res;
    void helper(Node*node){
        if(node==NULL){
            return;
        }
        for(int i=0;i<node->children.size();i++){
            helper(node->children[i]);
        }
        res.push_back(node->val);
    }

    vector<int> postorder(Node* root) {
        helper(root);
        return res;
    }
};

迭代：
和二叉树后序遍历相似，在前序遍历的基础上，改变子节点入栈顺序，最后反转数组。
class Solution {
public:
    vector<int> postorder(Node* root) {
        vector<int>res;
        if(root==NULL)return res;
        stack<Node*>stk;
        stk.push(root);
        while(!stk.empty()){
            Node*node=stk.top();
            stk.pop();
            res.push_back(node->val);
            for(int i=0;i<node->children.size();i++){
                stk.push(node->children[i]);
            }
        }
        reverse(res.begin(),res.end());
        return res;
    }
};
```

### 17. LeetCode101. 对称二叉树

```c++
思路：
1.本质上就是在判断每个节点的左右子树翻转后是否和原先一样
2.以对称轴为中心轴，内侧和内侧的比较、外侧和外侧的比较
3.确定遍历顺序：only后序(左右中)，因为我们需要左右子树的信息，才能判断是否能够翻转

递归：并非传统的向左右子节点要信息，而是向两个应当匹配的节点索要信息
class Solution {
public:
    bool compare(TreeNode*compareLeft,TreeNode*compareRight){
        //base case(确保非空后才可判断值)
        //前三种情况是判断结构，最后一种情况是判断值
        if(compareLeft!=NULL&&compareRight==NULL)return false;
        else if(compareLeft==NULL&&compareRight!=NULL)return false;
        else if(compareLeft==NULL&&compareRight==NULL)return true;
        else if(compareLeft->val!=compareRight->val)return false;

        //索要内侧和外侧的信息
        bool inside=compare(compareLeft->right,compareRight->left);
        bool outside=compare(compareLeft->left,compareRight->right);

        //只有当内侧和外侧的节点都符合要求时，整体才对称
        return inside&&outside;
    }

    bool isSymmetric(TreeNode* root) {
        return compare(root->left,root->right);
    }
};

迭代：
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        queue<TreeNode*>que;
        que.push(root->left);
        que.push(root->right);
        while(!que.empty()){
            //取出的两个节点是将要匹配的
            TreeNode*compareLeft=que.front();que.pop();
            TreeNode*compareRight=que.front();que.pop();
            if(!compareLeft&&!compareRight)continue;
            else if(!compareLeft||!compareRight||compareLeft->val!=compareRight->val)return false;
            //由于是按序取，所以也要按序存
            que.push(compareLeft->right);
            que.push(compareRight->left);
            que.push(compareLeft->left);
            que.push(compareRight->right);
        }
        return true;
    }
};
```

### 18. LeetCode222. 完全二叉树的节点个数

```c++
1.当作普通二叉树：
class Solution {
public:
    int countNodes(TreeNode* root) {
        if(root==NULL)return 0;
        int leftNodes=countNodes(root->left);
        int rightNodes=countNodes(root->right);
        return leftNodes+rightNodes+1;
    }
};

2.利用完全二叉树的特性：
2.1如果子树是完全满二叉树，利用公式(2^h-1)直接得出节点个数
2.2判断完全满二叉树：在完全二叉树的基础上，一直向左遍历的深度等于一直向右遍历的深度，就说明是完全满二叉树
class Solution {
public:
    int countNodes(TreeNode* root) {
        //base case
        if(root==NULL)return 0;
        
        //和普通二叉树唯一的区别：多了一步判断完全满二叉树，可以少遍历中间节点
        TreeNode*left=root->left;
        TreeNode*right=root->right;
        int leftDepth=0;
        int rightDepth=0;
        while(left){
            left=left->left;
            leftDepth++;
        }
        while(right){
            right=right->right;
            rightDepth++;
        }
        if(leftDepth==rightDepth){
            return (2<<leftDepth)-1;
        }

        //当前节点逻辑
        int leftNodes=countNodes(root->left);
        int rightNodes=countNodes(root->right);
        return leftNodes+rightNodes+1;
    }
};
```

### 19. LeetCode110. 平衡二叉树

```c++
class Solution {
public:
    class Info{
    public:
        bool isBalanced;
        int height;
        
        Info(bool isBalanced,int height){
            this->isBalanced=isBalanced;
            this->height=height;
        }
    };

    Info process(TreeNode*node){
        if(node==NULL)return Info(true,0);

        //向左右子树索要信息
        Info leftData=process(node->left);
        Info rightData=process(node->right);

        //处理当前节点信息
        bool isBalanced=(abs(leftData.height-rightData.height)<=1
        &&leftData.isBalanced
        &&rightData.isBalanced);

        int height=max(leftData.height,rightData.height)+1;

        return Info(isBalanced,height);
    }

    bool isBalanced(TreeNode* root) {
        Info data=process(root);
        return data.isBalanced;
    }
};
```

### 20. LeetCode257. 二叉树的所有路径

```c++
递归：
class Solution {
public:
    vector<string>res;

    void process(TreeNode*node,string path){
        if(node==NULL){
            return;
        }
        if(node->left==NULL&&node->right==NULL){//遍历到叶子节点才停止
            res.push_back(path+to_string(node->val));
            return;
        }
        process(node->left,path+to_string(node->val)+"->");
        process(node->right,path+to_string(node->val)+"->");
    }

    vector<string> binaryTreePaths(TreeNode* root) {
        process(root,"");
        return res;
    }
};

迭代：
class Solution {
public:
    vector<string> binaryTreePaths(TreeNode* root) {
        stack<TreeNode*>treeStk;//保存树的节点
        stack<string>pathStk;//保留路径
        vector<string>res;//最终路径集合
        treeStk.push(root);
        pathStk.push(to_string(root->val));
        while(!treeStk.empty()){
            TreeNode*node=treeStk.top();treeStk.pop();//取出该节点
            string path=pathStk.top();pathStk.pop();//取出该节点对应路径
            if(node->left==NULL&&node->right==NULL){//叶子节点
                res.push_back(path);
            }
            if(node->right){//栈：右先
                treeStk.push(node->right);
                pathStk.push(path+"->"+to_string(node->right->val));
            }
            if(node->left){
                treeStk.push(node->left);
                pathStk.push(path+"->"+to_string(node->left->val));
            }
        }
        return res;
    }
};
```

### 21. LeetCode404. 左叶子之和

```c++
思路：
1.左叶子：是叶子节点且是其父节点的左子节点
2.由于我们需要知道父子关系，所以不能深入到叶子节点再做判断，而是在叶子节点的上一层就需要处理了
3.后序遍历

递归：
class Solution {
public:
    int sumOfLeftLeaves(TreeNode* root) {
        //base case
        if(root==NULL)return 0;
        if(root->left==NULL&&root->right==NULL)return 0;

        //向左右子树索要信息
        int leftSum=sumOfLeftLeaves(root->left);
        int rightSum=sumOfLeftLeaves(root->right);
        //处理当前节点
        if(root->left!=NULL&&root->left->left==NULL&&root->left->right==NULL){//中
            leftSum=root->left->val;//左子节点是左叶子
        }
        return leftSum+rightSum;
    }
};

迭代：关键点在于判断左叶子
class Solution {
public:
    int sumOfLeftLeaves(TreeNode* root) {
        stack<TreeNode*>stk;
        int res=0;
        stk.push(root);
        while(!stk.empty()){
            TreeNode*node=stk.top();
            stk.pop();
            if(node->left!=NULL&&node->left->left==NULL&&node->left->right==NULL){
                res+=node->left->val;
            }
            if(node->right)stk.push(node->right);
            if(node->left)stk.push(node->left);
        }
        return res;
    }
};
```

### 22.LeetCode513. 找树左下角的值

```c++
递归：由于是找左下角值，所以我们应当优先处理左子树
很多人可能会疑惑为什么取值不是最后一层其他节点的呢？
其实最后一层最先被遍历到的节点必定是最左边的节点，因为我们是优先处理左子树的。当遍历到最后一层其他节点时，深度depth等于maxDepth，不满足修改值的条件。
class Solution {
public:
    int maxDepth=INT_MIN;
    int res=0;

    void traversal(TreeNode*node,int depth){
        //结束条件，叶子节点
        if(node->left==NULL&&node->right==NULL){
            if(depth>maxDepth){
                res=node->val;
                maxDepth=depth;
                return;
            }
        }
        if(node->left){
            depth++;
            traversal(node->left,depth);
            depth--;//回溯
        }
        if(node->right){
            depth++;
            traversal(node->right,depth);
            depth--;//回溯
        }
    }

    int findBottomLeftValue(TreeNode* root) {
        traversal(root,1);
        return res;
    }
};

迭代：层序遍历
class Solution {
public:
    int findBottomLeftValue(TreeNode* root) {
        queue<TreeNode*>que;
        int res=0;
        que.push(root);
        while(!que.empty()){
            res=que.front()->val;
            for(int i=que.size();i>0;i--){
                TreeNode*node=que.front();
                que.pop();
                if(node->left)que.push(node->left);
                if(node->right)que.push(node->right);
            }
        }
        return res;
    }
};
```

### 23.  LeetCode112. 路径总和

```c++
递归：
class Solution {
public:
    bool process(TreeNode*node,int sum){
        //base case
        if(node->left==NULL&&node->right==NULL&&sum-node->val==0)return true;
        if(node->left==NULL&&node->right==NULL&&sum-node->val!=0)return false;

        if(node->left){
            if(process(node->left,sum-node->val))return true;
        }
        if(node->right){
            if(process(node->right,sum-node->val))return true;
        }
        return false;
    }

    bool hasPathSum(TreeNode* root, int targetSum) {
        if(root==NULL)return false;
        return process(root,targetSum);
    }
};

迭代：
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if(root==NULL)return false;
        //<节点，路径和>
        stack<pair<TreeNode*,int>>stk;
        stk.push(pair<TreeNode*,int>(root,root->val));
        while(!stk.empty()){
            pair<TreeNode*,int>node=stk.top();
            stk.pop();
            if(!node.first->left&&!node.first->right&&node.second==targetSum)return true;
            if(node.first->right){
                stk.push(pair<TreeNode*,int>
                         (node.first->right,node.second+node.first->right->val));
            }
            if(node.first->left){
                stk.push(pair<TreeNode*,int>
                         (node.first->left,node.second+node.first->left->val));
            }
        }
        return false;
    }
};
```

### 24. LeetCode113. 路径总和 II

```c++
class Solution {
public:
    vector<vector<int>>res;

    //vec必须是赋值拷贝，不能是引用，否则会把所有路径值都加入到同一个集合中
    void traversal(TreeNode*node,vector<int>vec,int sum){
        if(node->left==NULL&&node->right==NULL&&sum-node->val==0){
            vec.push_back(node->val);
            res.push_back(vec);
            return;
        }
        if(node->left==NULL&&node->right==NULL&&sum-node->val!=0){
            return;
        }
        if(node->left){
            vec.push_back(node->val);
            traversal(node->left,vec,sum-node->val);
            vec.pop_back();//回溯
        }
        if(node->right){
            vec.push_back(node->val);
            traversal(node->right,vec,sum-node->val);
            vec.pop_back();//回溯
        }
    }

    vector<vector<int>> pathSum(TreeNode* root, int targetSum) {
        if(root==NULL)return {};
        traversal(root,{},targetSum);
        return res;
    }
};
```

### 25. LeetCode106. 从中序与后序遍历序列构造二叉树

```c++
思路：
中序：左中右
后序：左右中
1.后续数组为0，空节点
2.后续数组的最后一个元素为节点元素
3.寻找中序数组位置作为切割点
4.切中序数组
5.切后序数组
6.递归处理左区间后区间
class Solution {
public:
    map<int,int>dic;//用于映射中序遍历元素对应的索引

    TreeNode*traversal(vector<int>&inorder,vector<int>&postorder,int l1,int r1,int l2,int r2){
        if(l1>r1&&l2>r2)return NULL;
        TreeNode*root=new TreeNode(postorder[r2]);
        int index=dic[root->val];
        root->left=traversal(inorder,postorder,l1,index-1,l2,l2+index-l1-1);
        root->right=traversal(inorder,postorder,index+1,r1,l2+index-l1,r2-1);
        return root;
    }

    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        if(inorder.size()==0)return NULL;
        for(int i=0;i<inorder.size();i++){
            dic[inorder[i]]=i;
        }
        return traversal(inorder,postorder,0,inorder.size()-1,0,postorder.size()-1);
    }
};
```

### 26. LeetCode105. 从前序与中序遍历序列构造二叉树

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/6.png)

```c++
前序和后序无法确定一棵树，因为无法分割左右子树
class Solution {
public:
    map<int,int>dic;//映射中序数组索引

    TreeNode*traversal(vector<int>&preorder,vector<int>&inorder,int l1,int r1,int l2,int r2){
        if(l1>r1&&l2>r2){
            return NULL;
        }
        TreeNode*root=new TreeNode(preorder[l1]);
        int index=dic[root->val];
        root->left=traversal(preorder,inorder,l1+1,l1+1+index-1-l2,l2,index-1);
        root->right=traversal(preorder,inorder,l1+1+index-1-l2+1,r1,index+1,r2);
        return root;
    }

    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        if(preorder.size()==0){
            return NULL;
        }
        for(int i=0;i<inorder.size();i++){
            dic[inorder[i]]=i;
        }
        return traversal(preorder,inorder,0,preorder.size()-1,0,inorder.size()-1);
    }
};
```

### 27. LeetCode654. 最大二叉树

```c++
本题和前两题构造二叉树有异曲同工之处，都是先创建父节点，然后分割数组进行递归
class Solution {
public:
    //找到nums在l和r之间最大值的索引
    int indexOfMax(vector<int>&nums,int l,int r){
        int res=l;
        while(l<=r){
            res=nums[l]>nums[res]?l:res;
            l++;
        }
        return res;
    }

    TreeNode*traversal(vector<int>&nums,int l,int r){
        if(l>r)return NULL;
        int index=indexOfMax(nums,l,r);
        //以最大值为父节点值
        TreeNode*root=new TreeNode(nums[index]);
        root->left=traversal(nums,l,index-1);//前缀数组是左子树
        root->right=traversal(nums,index+1,r);//后缀数组是右子树
        return root;
    }

    TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
        return traversal(nums,0,nums.size()-1);
    }
};
```

### 28. LeetCode617. 合并二叉树

```c++
递归：
class Solution {
public:
    TreeNode*traversal(TreeNode*node1,TreeNode*node2){
        //某一节点为空，直接把不为空的子树接到总树即可
        if(node1==NULL||node2==NULL){
            return node1?node1:node2;
        }
        TreeNode*root=new TreeNode(node1->val+node2->val);
        root->left=traversal(node1->left,node2->left);
        root->right=traversal(node1->right,node2->right);
        return root;
    }

    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if(root1==NULL||root2==NULL){//如果有空树，返回不为空的
            return root1?root1:root2;
        }
        return traversal(root1,root2);
    }
};

迭代：队列层序遍历
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if(root1==NULL||root2==NULL){
            return root1?root1:root2;
        }
        queue<TreeNode*>que;
        que.push(root1);
        que.push(root2);
        while(!que.empty()){
            TreeNode*node1=que.front();que.pop();//取root1节点
            TreeNode*node2=que.front();que.pop();//取root2节点
            
            node1->val+=node2->val;//node1和node2必不为空

            //加入子树
            if(node1->left&&node2->left){
                que.push(node1->left);
                que.push(node2->left);
            }
            if(node1->right&&node2->right){
                que.push(node1->right);
                que.push(node2->right);
            }
            
            //有一个为空，接入不为空的
            if(node1->left==NULL&&node2->left!=NULL){
                node1->left=node2->left;
            }
            if(node1->right==NULL&&node2->right!=NULL){
                node1->right=node2->right;
            }
        }
        return root1;
    }
};
```

### 29. LeetCode700. 二叉搜索树中的搜索

```c++
递归：
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        if(root==NULL)return NULL;
        if(val==root->val)return root;
        else if(val>root->val)return searchBST(root->right,val);
        else return searchBST(root->left,val);
    }
};

迭代：
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        while(root!=NULL&&root->val!=val){
            if(val>root->val)root=root->right;
            else if(val<root->val)root=root->left;
        }
        return root;
    }
};
```

### 30. LeetCode98. 验证二叉搜索树

```c++
递归：
1.有序数组：
class Solution {
public:
    vector<int>vec;
    void traversal(TreeNode*node){
        if(node==NULL)return;
        traversal(node->left);
        vec.push_back(node->val);
        traversal(node->right);
    }
    bool isValidBST(TreeNode* root) {
        traversal(root);
        for(int i=0;i<vec.size()-1;i++){
            if(vec[i]>=vec[i+1])return false;
        }
        return true;
    }
};
2.双指针
class Solution {
public:
    TreeNode*pre=NULL;
    bool isValidBST(TreeNode* root) {
        if(root==NULL)return true;
        bool left=isValidBST(root->left);
        //由于中序遍历二叉搜索树，所以前一个节点的值必定小于当前节点的值
        if(pre!=NULL&&root->val<=pre->val)return false;
        pre=root;
        bool right=isValidBST(root->right);
        return left&&right;
    }
};

迭代：
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        stack<TreeNode*>stk;
        TreeNode*cur=root;
        TreeNode*pre=NULL;
        while(cur!=NULL||!stk.empty()){//未遍历完或者未处理完都不可退出循环
            if(cur!=NULL){
                stk.push(cur);
                cur=cur->left;//左
            }else{
                cur=stk.top();//中
                stk.pop();
                if(pre!=NULL&&pre->val>=cur->val)return false;
                pre=cur;
                cur=cur->right;//右
            }
        }
        return true;
    }
};
```

### 31. LeetCode530. 二叉搜索树的最小绝对差

```c++
迭代：
有序数组
class Solution {
public:
    vector<int>vec;
    void traversal(TreeNode*node){
        if(node==NULL)return;
        traversal(node->left);
        vec.push_back(node->val);
        traversal(node->right);
    }
    int getMinimumDifference(TreeNode* root) {
        int minVal=INT_MAX;
        traversal(root);
        for(int i=0;i<vec.size()-1;i++){
            minVal=min(minVal,vec[i+1]-vec[i]);
        }
        return minVal;
    }
};

双指针：
class Solution {
public:
    int minVal=INT_MAX;
    TreeNode*pre=NULL;
    void traversal(TreeNode*node){
        if(node==NULL)return;
        traversal(node->left);
        if(pre!=NULL)minVal=min(minVal,node->val-pre->val);//差值最小必定相邻
        pre=node;
        traversal(node->right);
    }
    int getMinimumDifference(TreeNode* root) {
        traversal(root);
        return minVal;
    }
};

迭代：
class Solution {
public:
    int getMinimumDifference(TreeNode* root) {
        stack<TreeNode*>stk;
        TreeNode*cur=root;
        TreeNode*pre=NULL;
        int minVal=INT_MAX;
        while(cur!=NULL||!stk.empty()){
            if(cur!=NULL){
                stk.push(cur);
                cur=cur->left;
            }else{
                cur=stk.top();
                stk.pop();
                if(pre!=NULL)minVal=min(minVal,cur->val-pre->val);
                pre=cur;
                cur=cur->right;
            }
        }
        return minVal;
    }
};
```

### 32. LeetCode501. 二叉搜索树中的众数

```c++
递归：
普通二叉树：
class Solution {
public:
    map<int,int>record;

    //比较器且必须是全局函数
    bool static cmp(const pair<int,int>&p1,const pair<int,int>&p2){
        return p1.second>p2.second;
    }

    //统计词频
    void traversal(TreeNode*node){
        if(node==NULL)return;
        record[node->val]++;
        traversal(node->left);
        traversal(node->right);
    }

    vector<int> findMode(TreeNode* root) {
        traversal(root);
        vector<pair<int,int>>vec(record.begin(),record.end());//录入数组
        sort(vec.begin(),vec.end(),cmp);
        vector<int>res;
        for(int i=0;i<vec.size();i++){
            if(vec[i].second==vec[0].second)res.push_back(vec[i].first);
            else break;
        }
        return res;
    }
};

搜索二叉树(双指针)：
class Solution {
public:
    vector<int>res;
    TreeNode*pre=NULL;
    int maxCount=0;
    int count=0;
    void traversal(TreeNode*node){
        if(node==NULL)return;
        traversal(node->left);//左
        if(pre==NULL)count=1;//第一个节点
        else if(pre->val==node->val)count++;
        else count=1;
        //中
        if(count==maxCount)res.push_back(node->val);
        else if(count>maxCount){
            maxCount=count;
            res.clear();//之前加入结果集的众数无效，清空
            res.push_back(node->val);
        }
        pre=node;
        //右
        traversal(node->right);
    }
    vector<int> findMode(TreeNode* root) {
        traversal(root);
        return res;
    }
};

迭代：
class Solution {
public:
    vector<int> findMode(TreeNode* root) {
        stack<TreeNode*>stk;
        TreeNode*cur=root;
        TreeNode*pre=cur;
        int maxCount=0;
        int count=0;
        vector<int>res;
        while(cur!=NULL||!stk.empty()){
            if(cur!=NULL){
                stk.push(cur);
                cur=cur->left;
            }else{
                cur=stk.top();
                stk.pop();
                if(pre==NULL)count=1;
                else if(pre->val==cur->val)count++;
                else count=1;

                if(count==maxCount)res.push_back(cur->val);
                else if(count>maxCount){
                    maxCount=count;
                    res.clear();
                    res.push_back(cur->val);
                }
                
                pre=cur;
                cur=cur->right;
            }
        }
        return res;
    }
};
```

### 33. LeetCode236. 二叉树的最近公共祖先

**优化版图解：**

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/7.png)

```c++
普通版：
class Solution {
public:
    map<TreeNode*,TreeNode*>fatherMap;

    //完善父节点表
    void traversal(TreeNode*node){
        if(node==NULL)return;
        fatherMap[node->left]=node;
        fatherMap[node->right]=node;
        traversal(node->left);
        traversal(node->right);
    }

    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        fatherMap[root]=root;
        traversal(root);
        set<TreeNode*>pAncestor;//记录p的所有祖先
        while(fatherMap[p]!=p){//先加入自己，因为p有可能是q的祖先
            pAncestor.insert(p);
            p=fatherMap[p];
        }
        while(fatherMap[q]!=q){
            if(pAncestor.count(q)!=0)return q;
            q=fatherMap[q];
        }
        return q;
    }
};

优化版：后序遍历最合适
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root==p||root==q||root==NULL)return root;
        TreeNode*left=lowestCommonAncestor(root->left,p,q);//左
        TreeNode*right=lowestCommonAncestor(root->right,p,q);//右
        if(left!=NULL&&right!=NULL)return root;//中
        return left?left:right;
    }
};
```

