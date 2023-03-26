# LeetCode经典题目

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

### 34. LeetCode235. 二叉搜索树的最近公共祖先

```c++
遍历到第一个值处于min(p->val,q->val)和max(p->val,q->val)之间的节点，必然就是p、q的最近公共祖先，且只有一个满足该条件的节点
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root->val>p->val&&root->val>q->val){//root值偏大
            return lowestCommonAncestor(root->left,p,q);
        }else if(root->val<p->val&&root->val<q->val){//root值偏小
            return lowestCommonAncestor(root->right,p,q);
        }else{
            return root;
        }
    }
};
```

### 35. LeetCode701. 二叉搜索树中的插入操作

```c++
class Solution {
public:
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        if(root==NULL)return new TreeNode(val);
        if(val<root->val)root->left=insertIntoBST(root->left,val);//val小，往左
        else root->right=insertIntoBST(root->right,val);//val大，往右
        return root;
    }
};
```

### 36. LeetCode450. 删除二叉搜索树中的节点

```c++
思路：
1.无对应节点
2.叶子节点
3.左不空，右空
4.左空，右不空
5.左不空，右不空
关键：把root的左子树接到root右子树最小值节点的左下方
class Solution {
public:
    TreeNode* deleteNode(TreeNode* root, int key) {
        //1.无对应节点
        if(root==NULL)return NULL;
        if(root->val==key){
            //2.叶子节点
            if(root->left==NULL&&root->right==NULL){
                //释放内存
                delete root;
                return NULL;
            }
            //3.左不空，右空
            if(root->right==NULL){
                //直接让左子树接过来
                TreeNode*retNode=root->left;
                //释放内存
                delete root;
                return retNode;
            }
            //4.左空，右不空
            if(root->left==NULL){
                //直接让右子树接过来
                TreeNode*retNode=root->right;
                //释放内存
                delete root;
                return retNode;
            }
            //5.左不空，右不空
            else{
                TreeNode*cur=root->right;
                while(cur->left!=NULL){
                    cur=cur->left;
                }
                //把root的左子树接到cur左子树
                cur->left=root->left;
                TreeNode*retNode=root->right;
                delete root;
                return retNode; 
            }
        }
        if(key<root->val)root->left=deleteNode(root->left,key);
        if(key>root->val)root->right=deleteNode(root->right,key);
        return root;
    }
};
```

### 37. LeetCode669. 修剪二叉搜索树

```c++
如果有越界的节点，第一时间不是直接删除该节点(return NULL)，而是处理其有效的子树。在回溯过程中，二叉搜索树就慢慢被修剪了
class Solution {
public:
    TreeNode* trimBST(TreeNode* root, int low, int high) {
        if(root==NULL)return NULL;
        if(root->val<low)return trimBST(root->right,low,high);
        if(root->val>high)return trimBST(root->left,low,high);
        root->left=trimBST(root->left,low,high);
        root->right=trimBST(root->right,low,high);
        return root;
    }
};
```

### 38. LeetCode108. 将有序数组转换为二叉搜索树

```c++
二分法：
保证左右子树节点个数相同，就可以平衡。
class Solution {
public:
    TreeNode*traversal(vector<int>&nums,int l,int r){
        if(l>r)return NULL;
        int mid=l+((r-l)>>1);
        TreeNode*root=new TreeNode(nums[mid]);
        root->left=traversal(nums,l,mid-1);
        root->right=traversal(nums,mid+1,r);
        return root;
    }

    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return traversal(nums,0,nums.size()-1);
    }
};
```

### 39. LeetCode538. 把二叉搜索树转换为累加树

```c++
题意：把每个节点的值更新成原树中所有大于等于该值的节点值之和(包括自身)
思路：
后序逆序遍历：右中左（大->小），双指针遍历，并记录前面累加和

递归：
class Solution {
public:
    int pre=0;//记录累加和
    void traversal(TreeNode*node){
        if(node==NULL)return;
        //右
        traversal(node->right);
        //中
        node->val+=pre;//更新节点值
        pre=node->val;//更新pre，注意不是加等于，因为node->val是已被更新过的
        //左
        traversal(node->left);
    }
    TreeNode* convertBST(TreeNode* root) {
        traversal(root);
        return root;
    }
};

迭代：
class Solution {
public:
    TreeNode* convertBST(TreeNode* root) {
        if(root==NULL)return NULL;
        int pre=0;//记录累加和
        stack<TreeNode*>stk;//记录节点
        TreeNode*cur=root;
        while(cur!=NULL||!stk.empty()){
            if(cur!=NULL){
                stk.push(cur);
                cur=cur->right;//右
            }else{
                //中
                cur=stk.top();
                stk.pop();
                cur->val+=pre;//更新cur节点值
                pre=cur->val;//更新pre累加和
                cur=cur->left;//左
            }
        }
        return root;
    }
};
```

## 七、回溯算法

**回溯算法模板：**base case(终止条件)+for循环(纵向递归+撤销操作)

### 1. LeetCode77. 组合

```c++
class Solution {
public:
    vector<vector<int>>res;
    vector<int>path;

    void backTracking(int start,int n,int k){
        if(path.size()==k){
            res.push_back(path);
        }
        for(int i=start;i<=n-(k-path.size())+1;i++){//减枝，因为path的大小有限制
            path.push_back(i);
            backTracking(i+1,n,k);
            path.pop_back();//回溯
        }
    }

    vector<vector<int>> combine(int n, int k) {
        backTracking(1,n,k);
        return res;
    }
};
```

### 2. LeetCode216. 组合总和 III

```c++
class Solution {
public:
    vector<vector<int>>res;//结果集
    vector<int>path;//路径
    int pathSum;//路径和

    void backTracking(int startIndex,int k,int n){
        if(path.size()>k||pathSum>n){//剪枝
            return;
        }
        if(path.size()==k&&pathSum==n){
            res.push_back(path);
            return;
        }
        for(int i=startIndex;i<=9-(k-path.size())+1;i++){//剪枝
            path.push_back(i);
            pathSum+=i;
            backTracking(i+1,k,n);
            //回溯
            path.pop_back();
            pathSum-=i;
        }
    }

    vector<vector<int>> combinationSum3(int k, int n) {
        backTracking(1,k,n);
        return res;
    }
};
```

### 3. LeetCode17. 电话号码的字母组合

```c++
思路：
1.创建映射表vector<string>
2.确定递归深度(终止条件)
3.确定递归宽度(for循环)
class Solution {
public:
    vector<string>res;//结果集
    string path;//路径
    vector<string>letterMap;//映射表

    void backTracking(string digits,int startIndex){
        if(startIndex==digits.size()){
            res.push_back(path);
            return;
        }
        int digit=digits[startIndex]-'0';
        string letter=letterMap[digit];
        for(int i=0;i<letter.size();i++){
            path.push_back(letter[i]);
            backTracking(digits,startIndex+1);
            path.pop_back();
        }
    }

    vector<string> letterCombinations(string digits) {
        //初始化path
        path.clear();
        res.clear();
        if(digits.size()==0)return res;
        //完善映射表
        letterMap={"","","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
        backTracking(digits,0);
        return res;
    }
};
```

### 4. LeetCode39. 组合总和

```c++
class Solution {
public:
    vector<vector<int>>res;//结果集
    vector<int>path;//路径
    int pathSum=0;//路径和

    void backTracking(vector<int>&candidates,int target,int startIndex){
        if(pathSum>target){//剪枝
            return;
        }
        if(pathSum==target){//收集
            res.push_back(path);
            return;
        }
        //注意count必定从1开始，如果从0开始，意味着不取当前值，但是我们有startIndex，意味着不取startIndex之前的元素，二者含义相同，会导致结果集有重复元素
        for(int i=startIndex;i<candidates.size();i++){
            for(int count=1;candidates[i]*count<=target;count++){//剪枝
                for(int j=0;j<count;j++){//加入对于数量的元素
                    path.push_back(candidates[i]);
                    pathSum+=candidates[i];
                }
                backTracking(candidates,target,i+1);
                //回溯
                for(int j=0;j<count;j++){
                    path.pop_back();
                    pathSum-=candidates[i];
                }
            }
        }
    }

    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        path.clear();
        res.clear();
        backTracking(candidates,target,0);
        return res;
    }
};
```

### 5. LeetCode40. 组合总和 II

```c++
1.深度去重：保证同一个元素不被重复使用
2.广度去重：保证结果集没有重复的路径集合，是对回溯之后相同的元素进行去重
    
误区：candidates[1,2,1,6,1]，target=8；不考虑广度去重的话，结果集会有重复路径集合([1,1,6],[1,6,1],[1,6,1])，因为第一个1遇到的情况包含了后面两个1所有的情况，因此会重复。但如果只考虑广度去重而不和深度去重做区分，则会漏掉[1,1,6]，因为[1,1]是深度上值相同的元素，是还没开始回溯的状态，是有效的。

class Solution {
public:
    vector<vector<int>>res;//结果集
    vector<int>path;//路径
    vector<bool>used;//记录已被使用过的元素

    void backTracking(vector<int>&candidates,int target,int sum,int startIndex){
        if(sum>target){//剪枝
            return;
        }
        if(sum==target){
            res.push_back(path);
            return;
        }
        for(int i=startIndex;i<candidates.size();i++){
            //!used[i-1]表示如果前一个元素如果没被使用过，但和当前元素相同，说明要广度去重
            if(i>0&&candidates[i]==candidates[i-1]&&!used[i-1]){
                continue;
            }
            path.push_back(candidates[i]);
            used[i]=true;
            backTracking(candidates,target,sum+candidates[i],i+1);
            path.pop_back();//回溯
            used[i]=false;
        }
    }

    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        res.clear();
        path.clear();
        used.clear();
        used=vector<bool>(candidates.size());//默认都未被使用过false
        sort(candidates.begin(),candidates.end());//排序很关键
        backTracking(candidates,target,0,0);
        return res;
    }
};
```

### 6. LeetCode131. 分割回文串

```c++
关键：和数组组合思路类似，搞清楚分割线startIndex即可
class Solution {
public:
    vector<vector<string>>res;//结果集
    vector<string>path;//路径
    vector<vector<bool>>isPalidrome;//动态规划表
	
    //完善
    void computePalidrome(string&s){
        //isPalidrome[i,j]：s(i-1,j+1)是否回文
        isPalidrome.resize(s.size(),vector<bool>(s.size()));
        for(int i=0;i<s.length();i++){
            isPalidrome[i][i]=true;
            if(i<s.length()-1&&s[i]==s[i+1])isPalidrome[i][i+1]=true;
        }

        for(int i=s.length()-3;i>=0;i--){
            for(int j=i+2;j<s.length();j++){
                isPalidrome[i][j]=s[i]==s[j]&&isPalidrome[i+1][j-1];
            }
        }
    }

    void backTracking(string&s,int startIndex){//切割线位于s[startIndex]和s[startIndex+1]之间
        //如果startIndex==s.length()-1，还有最后一个字符要加进来
        if(startIndex>=s.length()){
            res.push_back(path);
            return;
        }
        for(int i=startIndex;i<s.length();i++){
            if(isPalidrome[startIndex][i]){//是回文子串才往下继续递归
                string str=s.substr(startIndex,i-startIndex+1);
                path.push_back(str);
                backTracking(s,i+1);
                path.pop_back();//回溯
            }
        }
    }

    vector<vector<string>> partition(string s) {
        res.clear();
        path.clear();
        isPalidrome.clear();
        computePalidrome(s);
        backTracking(s,0);
        return res;
    }
};
```

### 7. LeetCode93. 复原 IP 地址

```c++
class Solution {
public:
    vector<string>res;

    //判断s(start,end)是否有效
    bool isValid(string&s,int start,int end){
        if(start>end){
            return false;
        }
        if(s[start]=='0'&&start!=end){//0开头且后面还有数
            return false;
        }
        int num=0;
        for(int i=start;i<=end;i++){
            if(s[i]<'0'||s[i]>'9'){//特殊字符
                return false;
            }
            num=num*10+(s[i]-'0');
            if(num>255){
                return false;
            }
        }
        return true;
    }

    void backTracking(string&s,int startIndex,int pointNum){
        if(pointNum==3&&isValid(s,startIndex,s.size()-1)){
            res.push_back(s);
            return;
        }
        for(int i=startIndex;i<s.size();i++){
            if(isValid(s,startIndex,i)){
                s.insert(s.begin()+i+1,'.');//在相应位置加'.'
                backTracking(s,i+2,pointNum+1);//跳过点
                s.erase(s.begin()+i+1);//回溯
            }else{
                break;//不合法，那么后面的也都不合法了，直接结束本层循环
            }
        }
    }

    vector<string> restoreIpAddresses(string s) {
        res.clear();
        backTracking(s,0,0);
        return res;
    }
};
```

### 8. LeetCode78. 子集

```c++
所有子集实际上就是回溯树上的所有节点加空集，递归层数就是子集大小，且有startIndex，所以不会重复
class Solution {
public:
    vector<vector<int>>res;//结果集
    vector<int>path;//路径

    void backTracking(vector<int>&nums,int startIndex){
        res.push_back(path);//先加再递归，不然会漏
        for(int i=startIndex;i<nums.size();i++){
            path.push_back(nums[i]);
            backTracking(nums,i+1);
            path.pop_back();
        }
    }

    vector<vector<int>> subsets(vector<int>& nums) {
        res.clear();
        path.clear();
        backTracking(nums,0);
        return res;
    }
};
```

### 9. LeetCode90. 子集 II

```c++
class Solution {
public:
    vector<vector<int>>res;//结果集
    vector<int>path;//路径

    void backTracking(vector<int>nums,int startIndex){
        res.push_back(path);//把每个节点都加入结果集
        for(int i=startIndex;i<nums.size();i++){
            if(i>startIndex&&nums[i]==nums[i-1]){//广度去重，去除重复子集
                continue;
            }
            path.push_back(nums[i]);
            used[i]=true;
            backTracking(nums,i+1);
            used[i]=false;
            path.pop_back();
        }
    }

    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        res.clear();
        path.clear();
        used.clear();
        used.resize(nums.size());
        sort(nums.begin(),nums.end());
        backTracking(nums,0);
        return res;
    }
};
```

### 10. LeetCode491. 递增子序列

```c++
关键：脑海里要有一棵回溯树
[4,6,7,7]：{4,7(2)}和{4,7(3)}重复，所以要对当前层去重，要记录已经有过7了，则第二个7直接跳过，防止重复
class Solution {
public:
    vector<vector<int>>res;//结果集
    vector<int>path;//路径

    void backTracking(vector<int>&nums,int startIndex){
        if(path.size()>=2){
            res.push_back(path);
        }
        
        //记录本层已被用过的元素
        //要记录用过的值而不是索引，因为原数组无法先排好序
        vector<bool>used(201);//nums元素范围[-100,100]
        for(int i=startIndex;i<nums.size();i++){
            if(!path.empty()&&nums[i]<path.back()||used[nums[i]+100]){
                //不符合递增要求或者当前元素在本层已被用过
                continue;
            }
            path.push_back(nums[i]);
            used[nums[i]+100]=true;
            backTracking(nums,i+1);
            path.pop_back();
        }
    }

    vector<vector<int>> findSubsequences(vector<int>& nums) {
        res.clear();
        path.clear();
        backTracking(nums,0);
        return res;
    }
};
```

### 11. LeetCode46. 全排列

```c++
注意每次循环要从0开始，因为是全排列，所以要考虑所有元素，以防遗漏。并且要记录已经加入path的元素，防止重复。
class Solution {
public:
    vector<vector<int>>res;
    vector<int>path;
    vector<bool>used;

    void backTracking(vector<int>&nums){
        if(path.size()==nums.size()){
            res.push_back(path);
            return;
        }
        for(int i=0;i<nums.size();i++){
            if(used[i])continue;
            path.push_back(nums[i]);
            used[i]=true;
            backTracking(nums);
            used[i]=false;
            path.pop_back();
        }
    }

    vector<vector<int>> permute(vector<int>& nums) {
        res.clear();
        path.clear();
        used.clear();
        used.resize(nums.size());
        backTracking(nums);
        return res;
    }
};
```

### 12. LeetCode47. 全排列 II

```c++
由于有重复元素，所以不仅要注意深度去重(path不许有重复使用过的元素(注意元素值相同不一定就是被重复使用了，必须是同一个索引位上的元素))，还要注意广度去重(res不能用重复的子集，以同一个元素值开头的子集会重复)
class Solution {
public:
    vector<vector<int>>res;
    vector<int>path;
    vector<int>used;

    void backTracking(vector<int>&nums){
        if(path.size()==nums.size()){//收割叶子节点
            res.push_back(path);
            return;
        }
        for(int i=0;i<nums.size();i++){
            //深度去重+广度去重
            if(used[i]||(i>0&&nums[i]==nums[i-1]&&!used[i-1]))continue;
            path.push_back(nums[i]);
            used[i]=true;
            backTracking(nums);
            used[i]=false;
            path.pop_back();
        }
    }

    vector<vector<int>> permuteUnique(vector<int>& nums) {
        res.clear();
        path.clear();
        used.clear();
        used.resize(nums.size());
        sort(nums.begin(),nums.end());
        backTracking(nums);
        return res;
    }
};
```

### 13. LeetCode332. 重新安排行程

```c++
难点：
1.可能会出现死循环
2.存在多种解法，但结果要求字典序最小
3.使用回溯法(深度优先探索)，终止条件是什么
4.探索的过程中如何遍历一个出发机场对应的目的机场

思路：
1.创建unordered_map<string,map<string,int>>targets，map可以删除
key1:出发机场;value1:目的机场及航班次数
key2:目的机场;value2:航班次数
2.理解题意后，其实我们只需要返回一个叶子节点即可。

关键：利用map存储映射关系和航班次数十分重要，前者解决了字典序问题，后者解决了死循环问题，缺一不可。并且还省去了哈希表erase操作，大大降低了时间复杂度。
class Solution {
public:
    vector<string>res;
    unordered_map<string,map<string,int>>targets;
    int ticketsNum;

    bool backTracking(){
        if(res.size()==ticketsNum+1){
            return true;
        }
        //遍历当前已到达机场的目的机场
        for(pair<const string,int>&target:targets[res[res.size()-1]]){
            if(target.second>0){//还有航班可飞
                res.push_back(target.first);
                target.second--;
                //但凡有一个叶子节点满足，直接一路返回，不用再管其他节点了
                if(backTracking())return true;
                res.pop_back();
                target.second++;
            }
        }
        return false;
    }

    vector<string> findItinerary(vector<vector<string>>& tickets) {
        res.clear();
        targets.clear();
        //tickets.size()：有几条路线，路线+1就是机场数
        ticketsNum=tickets.size();
        for(vector<string>&vec:tickets){
            //出发机场对应的目的机场的可到达航班次数+1
            targets[vec[0]][vec[1]]++;
        }
        //从"JFK"开始
        res.push_back("JFK");
        backTracking();
        return res;
    }
};
```

### 14. LeetCode51. N 皇后

```c++
总体就是在每一行的每一列中找到一个合适的位置放置皇后，然后继续向下递归。
class Solution {
public:
    vector<vector<string>>res;
    vector<string>chessboard;

    bool isValid(int row,int col,int n){
        //同一行肯定不会有皇后，所以不判断

        //判断同一列
        for(int i=0;i<n;i++){
            if(chessboard[i][col]=='Q')return false;
        }
        //因为只要上面几行有皇后，所以只需向上遍历
        //45度角
        for(int i=row-1,j=col+1;i>=0&&j<n;i--,j++){
            if(chessboard[i][j]=='Q')return false;
        }
        //135度角
        for(int i=row-1,j=col-1;i>=0&&j>=0;i--,j--){
            if(chessboard[i][j]=='Q')return false;
        }
        return true;
    }

    void backTracking(int row,int n){
        if(row==n){
            res.push_back(chessboard);
            return;
        }
        for(int col=0;col<n;col++){
            if(isValid(row,col,n)){//在[row][col]的位置上如果能放皇后
                chessboard[row][col]='Q';
                backTracking(row+1,n);
                chessboard[row][col]='.';
            }
        }
    }

    vector<vector<string>> solveNQueens(int n) {
        res.clear();
        chessboard.clear();
        chessboard.resize(n,string(n,'.'));
        backTracking(0,n);
        return res;
    }
};
```

### 15. LeetCode37. 解数独

```c++
class Solution {
public:
    bool isValid(int row, int col,char val,vector<vector<char>>&board){
        //同行
        for(int j=0;j<board[0].size();j++){
            if(board[row][j]==val)return false;
        }

        //同列
        for(int i=0;i<board.size();i++){
            if(board[i][col]==val)return false;
        }

        //九宫格
        int startRow=(row/3)*3;
        int startCol=(col/3)*3;
        for(int i=startRow;i<startRow+3;i++){
            for(int j=startCol;j<startCol+3;j++){
                if(board[i][j]==val)return false;
            }
        }
        return true;
    }

    bool backTracking(vector<vector<char>>&board){
        //需要填满所有格子，因此二维遍历
        for(int i=0;i<board.size();i++){
            for(int j=0;j<board[0].size();j++){
                if(board[i][j]!='.')continue;
                for(char k='1';k<='9';k++){
                    if(isValid(i,j,k,board)){
                        board[i][j]=k;
                        if(backTracking(board))return true;
                        board[i][j]='.';
                    }
                }
                //‘1’到'9'都无效，说明前面填得不对
                return false;
            }
        }
        //遍历完且没有返回false，说明该位置合适
        //最后一个格子不用填数字，且前面填完的都合法，所以就在最后返回true，以防漏掉
        return true;
    }

    void solveSudoku(vector<vector<char>>& board) {
        backTracking(board);
    }
};
```

## 八、贪心算法

### 1. LeetCode455. 分发饼干

```c++
局部最优：把最大的饼干喂给能满足且胃口最大的孩子
class Solution {
public:
    int findContentChildren(vector<int>& g, vector<int>& s) {
        sort(g.begin(),g.end());
        sort(s.begin(),s.end());
        int i=g.size()-1;
        int j=s.size()-1;
        int res=0;
        while(i>=0&&j>=0){
            if(s[j]>=g[i]){//能满足当前的孩子
                res++;
                i--;
                j--;
            }else{//当前孩子胃口太大，即使是最大的饼干也无法满足胃口，只能换下一个孩子
                i--;
            }
        }
        //退出循环
        //1.i<0，最大的饼干已经无法满足剩余所有孩子的胃口
        //2.j<0，所有饼干都已经给出去了
        return res;
    }
};
```

### 2. LeetCode376. 摆动序列

```c++
贪心算法：从第一个元素开始，把所有元素都考虑到了，必然是最大长度。
class Solution {
public:
    int wiggleMaxLength(vector<int>& nums) {
        if(nums.size()<=1)return nums.size();
        int curDif=0;//当前一对差值
        int preDif=0;//前一对差值
        int res=1;//默认序列最右有一个峰值
        for(int i=0;i<nums.size()-1;i++){
            curDif=nums[i+1]-nums[i];
            //等于0，是因为初始preDif和curDif都是0。
            if((preDif<=0&&curDif>0)||(preDif>=0&&curDif<0)){
                res++;
                preDif=curDif;
            }
        }
        return res;
    }
};
```

### 3. LeetCode53. 最大子数组和

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int cur=0;
        int res=INT_MIN;
        for(int i=0;i<nums.size();i++){
            //如果前面的连续子数组和<=0，则抛弃，因为拖后腿
            cur=cur<=0?nums[i]:cur+=nums[i];
            res=max(res,cur);
        }
        return res;
    }
};
```

### 4. LeetCode122. 买卖股票的最佳时机 II

```c++
思路：
假设在第一天买入，第四天卖出，发现可以拆分成以每两天为单位的，从而找到局部最优达到整体最优
prices[3]-prices[0]=(prices[3]-prices[2])+(prices[2]-prices[1])+(prices[1]-prices[0]);
以每两天为一个利润单位，而不是0~3天整体，因为中间可能有负值。
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int res=0;
        for(int i=0;i<prices.size()-1;i++){
            //只选正的
            res+=max(prices[i+1]-prices[i],0);
        }
        return res;
    }
};
```

### 5. LeetCode55. 跳跃游戏

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/8.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/9.png)

```c++
思路：
每一个值代表在当前位置能够跳跃的最大长度，所以只需要判断每次都跳跃最大长度（局部最优）能否超过最后一个下标（整体最优）即可。如果在既定覆盖范围内都无法跳出覆盖范围，就代表怎么样都无法到达最后一个下标，就退出循环。
class Solution {
public:
    bool canJump(vector<int>& nums) {
        if(nums.size()==1)return true;
        int cover=0;
        for(int i=0;i<=cover;i++){//走一步看一步
            cover=max(i+nums[i],cover);
            //如果覆盖范围超过了最后一个下标，则不用看后面了
            if(cover>=nums.size()-1)return true;
        }
        //走完当前所有覆盖范围都无法到达最后一个下标
        return false;
    }
};
```

### 6. LeetCode45. 跳跃游戏 II

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/10.png)

```c++
思路：
1.需要两个信息：当前的覆盖范围，和下一步的覆盖范围。尽可能地让每一步的覆盖范围最大(局部最优)
class Solution {
public:
    int jump(vector<int>& nums) {
        int res=0;//结果
        int start=0;//开始位置
        int end=0;//结束位置同时也是每一步的目的地
        //第一轮循环是初始化start和end的过程，同时让res多补上1，因为我们的判断条件是end<nums.size()-1，而不是start
        while(end<nums.size()-1){
            int maxPos=0;
            for(int i=start;i<=end;i++){
                maxPos=max(maxPos,i+nums[i]);
            }
            start=end+1;//跳完一步后，要从当前位置的下一个开始
            end=maxPos;
            res++;
        }
        return res;
    }
};
```

### 7. LeetCode1005. K 次取反后最大化的数组和

```c++
class Solution {
public:
    static bool cmp(int a,int b){
        return abs(a)>abs(b);
    }
    int largestSumAfterKNegations(vector<int>& nums, int k) {
        sort(nums.begin(),nums.end(),cmp);//按绝对值排序
        for(int i=0;i<nums.size();i++){
            if(nums[i]<0&&k>0){
                nums[i]*=-1;
                k--;
            }
        }
        //退出循环有两张情况
        //1.还有负数但是k==0，这种情况不需要额外处理
        //2.k>0，但是全为正数，那就让绝对值最小的取反
        if(k%2==1)nums[nums.size()-1]*=-1;
        int res=0;
        for(int num:nums)res+=num;
        return res;
    }
};
```

### 8. LeetCode134. 加油站

```c++
贪心策略：尽量先把剩余油量为正的站点先走了，储存油量
rest<0，则[0,i]区间都无法作为起始点
证明：start从0开始
1. gas[start]-cost[start]<0：那么一开始的rest就小于0，重置起始点
2. gas[start]-cost[start]>=0：当前rest<0，说明gas[i]-cost[i]必定小于0，如果以[1.i]区间上的某一个点为起始点，意味着要删去gas[start]-cost[start]，一个小于0的数减去一个正数必然还是小于0的
综上：当rest<0时，[0,i]区间都无法作为起始点，必须要更新起始点为i+1。

class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int rest=0;//不同起始点剩余油量累加和
        int totalSum=0;//整体剩余油量累加和
        int start=0;//起始点
        for(int i=0;i<gas.size();i++){
            rest+=gas[i]-cost[i];
            totalSum+=gas[i]-cost[i];
            if(rest<0){//说明当前起始点无法满足绕环
                start=i+1;
                rest=0;//重置剩余油量
            }
        }
        //start还不一定就是有效的起始点，还需要再判断整体剩余油量是否为正
        if(totalSum<0){//总剩余油量小于0，说明无论如何都无法绕环
            return -1;
        }
        return start;
    }
};
```

### 9. LeetCode135. 分发糖果

```c++
思路：
相邻有左邻居和右邻居，所以要判断两边，但不能同时判断。所以我们定义左规则和右规则
左规则：从左向右遍历，分数高的孩子多一块糖
右规则：从右向左遍历，分数高的孩子多一块糖
class Solution {
public:
    int candy(vector<int>& ratings) {
        //左规则数组，先给每人一块糖
        vector<int>left(ratings.size(),1);
        for(int i=0;i<left.size()-1;i++){
            if(ratings[i+1]>ratings[i])left[i+1]=left[i]+1;
        }
        //右规则数组同理
        vector<int>right(ratings.size(),1);
        for(int i=right.size()-1;i>0;i--){
            if(ratings[i-1]>ratings[i])right[i-1]=right[i]+1;
        }

        int res=0;//最少糖果数
        for(int i=0;i<ratings.size();i++){
            //因为相邻两个孩子，高分必然获得更多糖果，所以既要满足左规则也要满足右规则，因此取max
            res+=max(left[i],right[i]);
        }
        return res;
    }
};
```

### 10. 860. 柠檬水找零

```c++
class Solution {
public:
    bool lemonadeChange(vector<int>& bills) {
        int five=0,ten=0,twenty=0;//记录5/10/20数量
        for(int bill:bills){
            if(bill==5){//不需找零
                five++;
            }
            else if(bill==10){//找5
                if(five==0){
                    return false;
                }else{
                    five--;
                    ten++;
                }
            }
            else{//找15（10/5）（5/5/5）
                if(ten>0&&five>0){
                    ten--;
                    five--;
                }else if(five>=3){
                    five-=3;
                }else{
                    return false;
                }
            }
        }
        return true;
    }
};
```

### 11. LeetCode406. 根据身高重建队列

```c++
class Solution {
public:
    static bool cmp(const vector<int>&vec1,const vector<int>&vec2){
        return vec1[0]>vec2[0];
    }

    vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
        //根据身高排序，非常关键
        sort(people.begin(),people.end(),cmp);
        list<vector<int>>que;//效率高于vector
        for(int i=0;i<people.size();i++){
            int position=people[i][1];
            list<vector<int>>::iterator it=que.begin();
            while(position--){
                it++;
            }
            //先插入身高高的，因为身高高的前面人数必然更少，从少到多
            que.insert(it,people[i]);
        }
        return vector<vector<int>>(que.begin(),que.end());
    }
};
```

### 12. LeetCode452. 用最少数量的箭引爆气球

```c++
思路：
1.排序数组，让起始点从小到大
2.分析重叠的气球，看当前所覆盖的气球中的最小结束点
class Solution {
public:
    static bool cmp(const vector<int>&vec1,const vector<int>&vec2){
        return vec1[0]<vec2[0];
    }

    int findMinArrowShots(vector<vector<int>>& points) {
        //排序二维数组，起始点从小到大
        sort(points.begin(),points.end(),cmp);
        int res=0;
        for(int i=0;i<points.size();){
            int minEnd=points[i][1];//被涵盖气球的最小结束点，初始时当前气球的的结束点
            int count=0;//最小结束点所能涵盖气球的数量
            while(i+count<points.size()&&points[i+count][0]<=minEnd){//注意是'<='
                //更新最小结束点，因为前一个气球能覆盖到的，后一个气球不一定可以
                minEnd=min(minEnd,points[i+count][1]);
                //被打爆的气球数加1
                count++;
                //先更新minEnd，在count++，不然堆溢出
            }
            res++;
            i+=count;
        }
        return res;
    }
};
```

### 13. LeetCode435. 无重叠区间

```c++
思路：
1.按照右边界排序，从左向右遍历，因为右边界越小，留给下一个区间的空间就越大。右边界越大，从左向右遍历的过程中，和其他区间发生重叠的概率也越大，要删除的区间也就越多。
2.如果按照左边界排序，左边界只是区间的起始点，后面覆盖的范围无法确定，因此在从左向右遍历的过程中无法做到局部最优。
class Solution {
public:
    static bool cmp(const vector<int>&vec1,const vector<int>&vec2){
        return vec1[1]<vec2[1];
    }

    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        //按照右边界排序
        sort(intervals.begin(),intervals.end(),cmp);

        int count=0;//不重叠区间数
        int maxEnd=INT_MIN;//当前不重叠区间中的最大右边界
        for(int i=0;i<intervals.size();i++){
            if(maxEnd<=intervals[i][0]){
                count++;
                maxEnd=intervals[i][1];//更新end
            }
        }
        //总数减去不重叠区间数就是要删去的区间数
        return intervals.size()-count;
    }
};
```

### 14. LeetCode763. 划分字母区间

```c++
思路：
1.统计每一个字符最后出现的位置
2.当前下标如果和已遍历字符的最后一个下标相同的话，说明到达了分界点。
class Solution {
public:
    vector<int> partitionLabels(string s) {
        int record[26]={0};
        for(int i=0;i<s.length();i++){
            record[s[i]-'a']=i;//统计字符最大小标
        }

        vector<int>res;
        int left=0;//记录片段左边界
        int right=0;//当前加入字符的最远右边界
        for(int i=0;i<s.length();i++){
            right=max(right,record[s[i]-'a']);
            if(i==right){
                res.push_back(right-left+1);
                left=i+1;
            }
        }
        return res;
    }
};
```

### 15. LeetCode56. 合并区间

```c++
class Solution {
public:
    static bool cmp(const vector<int>&vec1,const vector<int>&vec2){
        return vec1[0]<vec2[0];
    }

    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        sort(intervals.begin(),intervals.end(),cmp);
        vector<vector<int>>res;
        int start=intervals[0][0];
        int end=intervals[0][1];
        for(int i=1;i<intervals.size();i++){
            if(end>=intervals[i][0]){//可覆盖当前区间，更新end
                //因为是覆盖，所以取最大右边界（贪心策略）
                end=max(end,intervals[i][1]);
            }else{
                //无法覆盖，重置start和end
                res.push_back({start,end});
                start=intervals[i][0];
                end=intervals[i][1];
            }
        }
        //由于判断完最后一个区间后就退出循环，还未来得及加入结果集，因此循环结束后单独加入
        res.push_back({start,end});
        return res;
    }
};
```

16. LeetCode738.单调递增的数字

```cpp
思路：
1.当出现strNum[i-1]>strNum[i]时，让strNum[i-1]--，然后strNum[i]=9，即局部最优。
2.遍历顺序：从右向左（如果是从左向右遍历，无法保证strNum[i]--后，strNum[i]>str[i-1])）
即无法保证局部最优传递下来。
3.因为只要有一个strNum[i-1]>strNum[i]，strNum[i]就要置为9，为了保证单调递增，后面
数位也必须置为9。因此，我们需要记录最高位的i(strNum[i-1]>strNum[i])
class Solution {
public:
    int monotoneIncreasingDigits(int n) {
        //转为字符串
        string strNum=to_string(n);
        //记录最高位i，让其后面数位都置为9
        int flag=strNum.size();
        //找最高数位i
        for(int i=strNum.size()-1;i>0;i--){
            if(strNum[i-1]>strNum[i]){
                flag=i;//更新flag
                strNum[i-1]--;//必须减1，不然就比原来的数大
            }
        }
        for(int i=flag;i<strNum.size();i++){
            strNum[i]='9';
        }
        return stoi(strNum);
    }
};
```

### 17. LeetCode714.买卖股票的最佳时机含手续费

```cpp
思路：
每天有3种情况：
buy：买入价格
1.prices[i]+fee<buy：更新buy为最小买入价格
2.prices[i]>buy：说明有利润可得。但是我们又无法确定当前是否为最大卖出价格，因此为了能
够后悔，把buy更新为prices[i]，不加手续费，因为防止重复扣除手续费，后续遇到更高卖出价格
时直接补上差价即可。
3.prices[i]==buy：不做任何处理


卡点：buy=prices[i]
利润=prices[i]-buy
假卖一次后，buy更新为prices[i]，下一次循环时和prices[i+1]+fee比较，下一次获取利润有
两种情况。
假设第j天最高卖出价格
1.第i天真卖：下一次 利润=prices[j]-(prices[i+1]+fee)
2.第i天假卖: 下一次 利润=prices[j]-prices[i]
为了获取最大利润，肯定让buy=min(prices[i],prices[i+1]+fee)

本质：
当碰到一个高点时，真卖或假卖取决于后一天的prices[i+1]+fee是否足够小，能够比我假卖时的
成本还要低
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int res=0;
        int buy=prices[0]+fee;//最小买入价格
        for(int i=1;i<prices.size();i++){
            if(prices[i]+fee<buy){
                //更新最低买入价格
                buy=prices[i]+fee;
            }
            else if(prices[i]>buy){
                res+=prices[i]-buy;//先记上一部分利润
                buy=prices[i];//假卖
            }
        }
        return res;
    }
};
```

### 18. LeetCode968.监控二叉树

```cpp
思路：
叶子节点不用放摄像头，从叶子节点开始看，局部最优推出全局最优。如果从根节点开始，只能省
一个摄像头，从叶子节点开始可以省2^h个
从底向上：后序遍历


每个节点有3种状态：
1.有覆盖(0)
2.无覆盖(1)
3.有摄像头(2)
无摄像头拆分为0和1

class Solution {
public:
    int res;

    //0：有覆盖
    //1：无覆盖
    //2：有摄像头
    //无摄像头拆分为0和1
    int traversal(TreeNode*node){
        //空节点是已覆盖状态
        if(node==NULL){
            return 0;
        }

        int left=traversal(node->left);//左孩子
        int right=traversal(node->right);//右孩子

        //左右有一个未被覆盖的，当前节点就要安装摄像头
        if(left==1||right==1){
            res++;
            return 2;
        }
        //左右有一个摄像头，当前节点就是已被覆盖状态
        if(left==2||right==2){
            return 0;
        }
        //上述条件都不满足，left和right肯定都为0
        //左右都是已被覆盖状态，说明都没有摄像头，当前节点未被覆盖
        return 1;
    }

    int minCameraCover(TreeNode* root) {
        res=0;
        if(traversal(root)==1){//如果根节点是未被覆盖的状态，只能自己加一个摄像头了
            res++;
        }
        return res;
    }
};
```

九、动态规划

### 1. LeetCode509. 斐波那契数

```cpp
1.动态规划
class Solution {
public:
    int fib(int n) {
        //边界情况
        if(n<0){
            return -1;
        }
        if(n==0){
            return 0;
        }

        //dp[i]=F(i)
        vector<int>dp(n+1);//0~n，n+1个数

        //转移方程：F(n)=F(n-1)+F(n-2)
        //所以先初始化F(1),F(0)
        dp[0]=0;
        dp[1]=1;

        //完善动态规划表
        for(int i=2;i<=n;i++){
            dp[i]=dp[i-1]+dp[i-2];
        }
        return dp[n];
    }
};
2.动态规划
完善动态规划表的时候发现，只需要前两个数即可，所以我们只需要三个变量即可。
空间复杂度从O(n)->O(1)
class Solution {
public:
    int fib(int n) {
        if(n<0){
            return -1;
        }

        int a=0;//F(0)
        int b=1;//F(1)
        int c=a+b;//F(2)
        while(n--){
            c=a+b;
            a=b;
            b=c;
        }
        //在while循环中，相当于a/b/c都往后移了n个数
        //所以最终a：F(n)，b：F(n+1)，c：F(n+2)
        return a;
    }
};
可能很多人不理解为什么返回a而不是c，其实只要代入一个案例进去算就明白了
```

### 2. LeetCode70. 爬楼梯

```cpp
1.动态规划表：
class Solution {
public:
    int climbStairs(int n) {
        //边界情况
        if(n<1){
            return 0;
        }
        if(n==1){
            return 1;
        }
        
        //dp[i]：到达i阶楼梯的方法数
        vector<int>dp(n+1);//有dp[n]
        dp[1]=1;
        dp[2]=2;
        //完善动态规划表
        for(int i=3;i<=n;i++){
            dp[i]=dp[i-1]+dp[i-2];
        }
        return dp[n];
    }
};
2.动态规划：我们仍只需维护三个变量即可
class Solution {
public:
    int climbStairs(int n) {
        //边界情况
        if(n<1){
            return 0;
        }
        if(n==1){
            return 1;
        }
        //注意要用long，不然c可能会溢出
        long a=1;//F(1)
        long b=2;//F(2)
        long c=a+b;//F(3)
        n-=1;//a往后(n-1)个数才是F(n)，所以n变成n-1
        while(n--){
            c=a+b;
            a=b;
            b=c;
        }
        return a;
    }
};
```

### 3. LeetCode746. 使用最小花费爬楼梯

```cpp
1.动态规划表
class Solution {
public:
    int minCostClimbingStairs(vector<int>& cost) {
        int n=cost.size();
        //dp[i]：爬'到'第i阶楼梯所需最低费用
        vector<int>dp(n+1);

        //初始化dp，可以从下标为0或1的台阶开始爬楼梯
        dp[0]=0;//从下标为0开始
        dp[1]=0;//从下标为1开始
        
        //完善dp
        for(int i=2;i<=n;i++){
            dp[i]=min(dp[i-1]+cost[i-1],dp[i-2]+cost[i-2]);
        }
        return dp[n];
    }
};
2.动态规划：只需维护三个变量
class Solution {
public:
    int minCostClimbingStairs(vector<int>& cost) {
        int a=0;
        int b=0;
        int c=0;
        for(int i=2;i<=cost.size();i++){
            //a是台阶更低的所需费用，所以和cost[i-2]搭配
            c=min(a+cost[i-2],b+cost[i-1]);
            a=b;
            b=c;
        }
        //i向后移n-2位，c初始为F(2)，结束循环后刚好是F(n)
        return c;
    }
};
```

### 4. LeetCode62. 不同路径

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/11.png)

```cpp
1.动态规划二维表
class Solution {
public:
    int uniquePaths(int m, int n) {
        //特殊情况
        if(m<0||n<0){
            return -1;
        }
        if(m==0||n==0){
            return 1;
        }

        //dp[i][j]：机器人到达[i][j]有几种方法
        vector<vector<int>>dp(m,vector<int>(n));

        //初始化dp(注意边界情况：第0行和第0列)
        for(int i=0;i<m;i++){
            dp[i][0]=1;
        }
        for(int j=0;j<n;j++){
            dp[0][j]=1;
        }

        //由于机器人只能向下或向右移动一格
        //因此每一个格子只能从其上或左抵达
        //转移方程：dp[i][j]=dp[i-1][j]+dp[i][j-1]
        //完善dp
        for(int i=1;i<m;i++){
            for(int j=1;j<n;j++){
                dp[i][j]=dp[i-1][j]+dp[i][j-1];
            }
        }
        return dp[m-1][n-1];
    }
}
时间复杂度：O(m*n)
空间复杂度：O(m*n)

2.动态规划一维表(滚动数组)：我们只需要用到前一行和当前行前一列的变量，所以我们可以利用一张一维表不断更新迭代即可。前面有图片讲解。
class Solution {
public:
    int uniquePaths(int m, int n) {
        if(m<0||n<0){
            return -1;
        }
        if(m==0||n==0){
            return 1;
        }
        //初始化dp
        vector<int>dp(n,1);
        for(int i=1;i<m;i++){
            for(int j=1;j<n;j++){
                dp[j]=dp[j-1]+dp[j];
            }
        }
        return dp[n-1];
    }
};
时间复杂度：O(m*n)
空间复杂度：O(n)
    
3.数论：排列组合(在总步数m-n-2中选m-1步来向下，其余往右走)
方法数=C(m-n-2)(m-1)
class Solution {
public:
    int uniquePaths(int m, int n) {
        //特殊情况
        if(m<0||n<0){
            return -1;
        }
        if(m==0||n==0){
            return 1;
        }

        //分子可能会溢出，所以用long long类型
        long long numerator=1;
        //分母是(m-1)的阶乘，但先初始化为m-1，阶乘太大容易溢出，所以在计算过程中看能否约掉
        int denominator=m-1;
        int count=m-1;
        int t=m+n-2;
        while(count--){
            numerator*=(t--);
            //判断是否能约,分母不能等于0
            while(denominator!=0&&numerator%denominator==0){
                numerator/=denominator;
                denominator--;//因为是阶乘，所以m-1的下一个除数是m-2
            }
        }
        return numerator;
    }
};
时间复杂度：O(m)
空间复杂度：O(1)
    
4.深度优先搜索会超时，二叉树深度为（m+n-1）（深度从1开始），时间复杂度即结点个数为2^(m+n-1)-1，已经是指数级别了。
分析：每个二叉树结点有两个选择：向下或向右。先一直往同一个方向走，到达尽头后，再往另一个方向一直走到尽头，那么深度就是m+n-1了。
```

### 5. LeetCode63. 不同路径 II

```cpp
1.动态规划二维表
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        //dp[i][j]：到达[i][j]的方法数
        int m=obstacleGrid.size();
        int n=obstacleGrid[0].size();
        vector<vector<int>>dp(m,vector<int>(n));
        //初始化dp，如果第0行和第0列有障碍物，那么其右边或下边的格子无法抵达
        for(int i=0;i<m;i++){
            if(obstacleGrid[i][0]==1)break;
            dp[i][0]=1;
        }
        for(int j=0;j<n;j++){
            if(obstacleGrid[0][j]==1)break;
            dp[0][j]=1;
        }

        //完善dp
        for(int i=1;i<m;i++){
            for(int j=1;j<n;j++){
                if(obstacleGrid[i][j]==1){//当前位置是障碍物，走不了，保留0
                    continue;
                }
                //即使左边和上边是障碍物也不影响，因为方法数是0
                dp[i][j]=dp[i-1][j]+dp[i][j-1];
            }
        }
        return dp[m-1][n-1];
    }
};

2.滚动数组：优化空间效率
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int m=obstacleGrid.size();
        int n=obstacleGrid[0].size();

        if(m==1){
            for(int j=0;j<n;j++){
                if(obstacleGrid[0][j]==1)return 0;
            }
        }else if(n==1){
            for(int i=0;i<m;i++){
                if(obstacleGrid[i][0]==1)return 0;
            }
        }

        vector<int>dp(n);//滚动数组
        //初始化dp
        for(int j=0;j<n;j++){
            if(obstacleGrid[0][j]==1)break;
            dp[j]=1;
        }

        for(int i=1;i<m;i++){
            for(int j=0;j<n;j++){//第一列也可能有障碍物
                if(obstacleGrid[i][j]==1){
                    dp[j]=0;//障碍物，需要置为0
                    continue;
                }
                if(j>0)dp[j]=dp[j-1]+dp[j];
            }
        }
        return dp[n-1];
    }
};
```

### 6. LeetCode343. 整数拆分

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/12.png)

```cpp
1.动态规划
class Solution {
public:
    int integerBreak(int n) {
        //特殊情况
        if(n<2){
            return 0;
        }

        //dp[i]：拆分数字i的最大乘积
        vector<int>dp(n+1);
        //初始化dp
        dp[2]=1;
        for(int i=3;i<=n;i++){
            for(int j=1;j<=i/2;j++){//3拆分成1和2，所以从1开始
                //把i拆分成i-j和j两个数
                //或者拆分成j和其他数(数量>1)，只需要直到被拆分数的最大乘积即可
                dp[i]=max(dp[i],max((i-j)*j,j*dp[i-j]));
            }
        }
        return dp[n];
    }
};
时间复杂度：O(n^2)
空间复杂度：O(n)

2.数论
class Solution {
public:
    int integerBreak(int n) {
        //特殊情况
        if(n==2)return 1;
        if(n==3)return 2;
        if(n==4)return 4;//4=(2+2)=>2*2>3*1

        int res=1;
        while(n>4){
            res*=3;
            n-=3;
        }
        res*=n;
        return res;
    }
};
时间复杂度：O(n)
空间复杂度：O(1)
```

### 7. LeetCode96. 不同的二叉搜索树

```cpp
本题我们只用管有i个节点时的结构数，不用管值。因为值都是不同的，所以可以把结构安排好后再把值填入即可。
    
class Solution {
public:
    int numTrees(int n) {
        //dp[i]：节点个数为i的二叉搜索树结构数
        vector<int>dp(n+1);
        
        //初始化dp
        dp[0]=1;
        dp[1]=1;
        
        for(int i=2;i<=n;i++){//整棵树节点个数
            for(int j=0;j<=i-1;j++){//左子树节点个数
                dp[i]+=dp[j]*dp[i-j-1];
            }
        }
        return dp[n];
    }
};
```

### 8. 0-1背包

```cpp
0-1背包：
有n件物品和一个最多能背重量为w 的背包。第i件物品的重量是weight[i]，得到的价值是value[i] 。每件物品只能用一次，求解将哪些物品装入背包里物品价值总和最大。

思路：
1.动态规划二维表：
dp[i][j]：任意取0~i的物品，放假容量为j的背包里，价值总和最大值。
2.转移方程：
对于物品i：
  2.1取物品i：dp[i][j]=dp[i-1][j](其实是背包当前容量无法再放入物品i)
  2.2不取物品i：dp[i][j]=dp[i-1][j-weight[i]]+value[i](j-weight[i]是为物品i腾出空间)
综上：dp[i][j]=max(dp[i-1][j],dp[i-1][j-weight[i]]+value[i])
3.初始化dp：
由转移方程可知，新dp值由左上角推得，所以先初始化第0行和第0列

完整代码：
int maxValueInBag(vector<int>weight,vector<int>value,int capacity){
    //dp[i][j]：任意取0~i的物品，放假容量为j的背包里，价值总和最大值。
    vector<vector<int>>dp(weight.size(),vector<int>(capacity+1));
    
    //初始化dp
    for(int i=0;i<weight.size();i++){
        //容量为0的情况下，什么都放不了，价值是0
        dp[i][0]=0;
    }
    for(int j=weight[0];j<=capacity;j++){
        dp[0][j]=value[0];
    }
    
    //完善dp
    for(int i=1;i<weight.size();i++){
        for(int j=1;j<=capacity;j++){
            if(j<weight[i])dp[i][j]=dp[i-1][j];
            else dp[i][j]=max(dp[i-1][j],dp[i-1][j-weight[i]]+value[i]);
            //腾出空间放入新物品的总价值不一定比不放入新物品的总价值大
        }
    }
    return dp[weight.size()-1][capacity];
}

优化空间效率：滚动数组
int maxValueInBag(vector<int>weight,vector<int>value,int capacity){
    vector<int>dp(capacity+1);
    //初始化dp
    for(int j=weight[0];j<=capacity;j++){
        dp[j]=value[0];
    }
    for(int i=1;i<weight.size();i++){
        for(int j=capacity;j>=weight[i];j--){//因为依赖左上角值，所以从右往左遍历
            //如果当前j<weight[i]，j--后必然也小于weight[i]，直接结束内层循环即可
            dp[j]=max(dp[j],dp[j-weight[i]]+value[i]);
        }
    }
    return dp[capacity];
}
```

### 9. LeetCode416. 分割等和子集

```cpp
思路：
转换成0-1背包问题，只要dp中有值等于sum/2的，就return true

1. dp[i][j]：从0~i下标中组合起来的值不超过j的最大值
2. dp[i][j]=max(dp[i-1][j],dp[i-1][j-nums[i]]+nums[i])
3. 初始化dp
    
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum=0;//nums总和
        for(int i=0;i<nums.size();i++){
            sum+=nums[i];
        }
        if(sum%2==1)return false;//奇数无法二分

        //dp[i][j]：从0~i下标中组合起来的值不超过j的最大值
        vector<vector<int>>dp(nums.size(),vector<int>((sum/2)+1));
        
        //初始化dp
        for(int i=0;i<nums.size();i++){
            dp[i][0]=0;
        }
        for(int j=nums[0];j<=sum/2;j++){
            dp[0][j]=nums[0];
        }

        //完善dp
        for(int i=1;i<nums.size();i++){
            for(int j=1;j<=sum/2;j++){
                if(j<nums[i])dp[i][j]=dp[i-1][j];
                else dp[i][j]=max(dp[i-1][j],dp[i-1][j-nums[i]]+nums[i]);
            }
        }
        //在整个nums数组(下标0~nums.size()-1)中，是否有能组成sum/2的集合
        return dp[nums.size()-1][sum/2]==sum/2;
    }
};

优化空间效率：滚动数组
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum=0;//nums总和
        for(int i=0;i<nums.size();i++){
            sum+=nums[i];
        }

        if(sum%2==1)return false;

        vector<int>dp((sum/2)+1);
        for(int j=nums[0];j<=sum/2;j++){
            dp[j]=nums[0];
        }
        for(int i=1;i<nums.size();i++){
            for(int j=sum/2;j>=nums[i];j--){
                dp[j]=max(dp[j],dp[j-nums[i]]+nums[i]);
            }
        }
        return dp[sum/2]==sum/2;
    }
};
```

### 10. LeetCode1049. 最后一块石头的重量 II

```cpp
思路：
可把问题转变成将石头分成两堆且质量和尽可能相同，即靠近sum/2，这样两堆石头互相磨损完后，剩余石头质量最小。

1. dp[i][j]：从0~i的石头中，取最靠近质量j的石头组合的质量和
2. dp[i][j]=max(dp[i-1][j],dp[i-1][j-stones[i]]+stones[i])
3. 初始化dp

class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        int sum=0;
        for(int i=0;i<stones.size();i++){
            sum+=stones[i];
        }
        int target=(sum+1)/2;//取较大的中间值，因为大的包含小的

        //dp[i][j]：从0~i的石头中，取最靠近质量j的石头组合的质量和
        vector<vector<int>>dp(stones.size(),vector<int>(target+1));

        //初始化dp
        for(int i=0;i<stones.size();i++){
            dp[i][0]=0;
        }
        for(int j=stones[0];j<=target;j++){
            dp[0][j]=stones[0];
        }

        //完善dp
        for(int i=1;i<stones.size();i++){
            for(int j=1;j<=target;j++){
                if(j<stones[i])dp[i][j]=dp[i-1][j];
                else dp[i][j]=max(dp[i-1][j],dp[i-1][j-stones[i]]+stones[i]);
            }
        }
        return abs((sum-dp[stones.size()-1][target])-dp[stones.size()-1][target]);
    }
};

优化空间效率：滚动数组
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        int sum=0;
        for(int i=0;i<stones.size();i++){
            sum+=stones[i];
        }
        int target=(sum+1)/2;

        vector<int>dp(target+1);
        for(int j=stones[0];j<=target;j++){
            dp[j]=stones[0];
        }

        for(int i=1;i<stones.size();i++){
            for(int j=target;j>=stones[i];j--){
                dp[j]=max(dp[j],dp[j-stones[i]]+stones[i]);
            }
        }
        return abs((sum-dp[target])-dp[target]);
    }
};
```

### 11. LeetCode494. 目标和

```cpp
思路：
本题可分成两个组合，plus和minus，一个加正号，另一个加负号。
plus-minus=target,plus+minus=sum=>plus=(target+sum)/2
因此问题转变成在集合nums中找出和为plus的组合，即装满容量为(target+sum)/2的背包。把二维标准（+和-）转换成唯一标准（+），让集合nums里的元素组成我们的newTarget

1.注意特殊情况：
  1.1如果target+sum是奇数，(target+sum)/2无法精准取到，所以无解，return 0
  1.2如果target的绝对值大于sum，无论如何都无法组合成target，也是无解
2.因为每个物品（数字）只能用一次，所以是0-1背包问题，之前是装满的最大价值，但是现在求的是有几种方法装满背包。
3.dp[i][j]：使用下标为[0,i]的数字装满容量为j的背包的方法数
4.dp[i][j]=dp[i-1][j]+dp[i-1][j-nums[i]]：（j-nums[i]+nums[i]=j）
5.初始化dp：dp[0][0]=1，因为dp[0][0]是一切递推结果的起源

滚动数组：（如果用二维表需要考虑太多没必要的情况，防止堆溢出）
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        //分成plus和minus
        //plus-minus=target,plus+minus=sum=>new target=plus=(sum+target)/2
        
        int sum=0;//nums总和
        for(int i=0;i<nums.size();i++){
            sum+=nums[i];
        }

        //特殊情况
        if((sum+target)%2==1||abs(target)>sum)return 0;
        int newTarget=(sum+target)/2;

        //dp[j]：装满容量为j的背包的方法数
        vector<int>dp(newTarget+1);
        //初始化dp
        dp[0]=1;
        //完善dp
        for(int i=0;i<nums.size();i++){
            for(int j=newTarget;j>=nums[i];j--){
                dp[j]=dp[j]+dp[j-nums[i]];//取或不取的方法总数
            }
        }
        return dp[newTarget];
    }
};
```

### 12. LeetCode474. 一和零

```cpp
1.dp[i][j]：最多有i个0和j个1的最大子集大小
2.dp[i][j]=max(dp[i][j],dp[i-count0][j-count1]+1)
3.初始化dp
4.遍历顺序：外层遍历物品(strs)，内层遍历背包容量(count0,count1)

滚动二维表：
class Solution {
public:
    pair<int,int>getZeroOne(string& str){
        int count0=0;
        int count1=0;
        for(int i=0;i<str.length();i++){
            if(str[i]=='0')count0++;
            else count1++;
        }
        return {count0,count1};
    }

    int findMaxForm(vector<string>& strs, int m, int n) {
        //dp[i][j]：最多有i个0和j个1的最大子集大小
        vector<vector<int>>dp(m+1,vector<int>(n+1));
        //初始化dp，初始全为0即可
        //完善dp
        for(string&str:strs){//遍历物品
            pair<int,int>data=getZeroOne(str);
            for(int i=m;i>=data.first;i--){
                for(int j=n;j>=data.second;j--){//从后向前遍历背包容量
                    dp[i][j]=max(dp[i][j],dp[i-data.first][j-data.second]+1);
                }
            }
        }
        return dp[m][n];
    }
};
```

### 13. 完全背包

```cpp
完全背包：有N件物品和一个最多能背重量为W的背包。第i件物品的重量是weight[i]，得到的价值是value[i] 。每件物品都有无限个（也就是可以放入背包多次），求解将哪些物品装入背包里物品价值总和最大。

完全背包与0-1背包唯一不同的是：完全背包每件物品有无限件。
```

### 14. LeetCode518. 零钱兑换 II

```cpp
1.dp[j]：组成j的组合数
2.dp[j]+=dp[j-coins[i]]
3.初始化dp：dp[0]=1;//只有不取coins[0]，才能组成0
4.遍历顺序：先遍历物品，在遍历容量

滚动数组：
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        //dp[j]：拼成j的组合数
        vector<int>dp(amount+1);
        //初始化dp
        dp[0]=1;
        //完善dp
        for(int i=0;i<coins.size();i++){//遍历物品
            for(int j=coins[i];j<=amount;j++){//遍历背包
                //假设在二维表中，取n个coins[i]组成j的方法数由n-1个coins[i]递推而得
                //dp[i][j]需要用到同一行的dp值，因此在滚动数组中，先更新前面再由前面推得后面，使后面的dp值更新
		//组成j：既可以不取coins[i](dp[j]),也可以比前一次多取一个coins[i](dp[j-coins[i]])
                dp[j]+=dp[j-coins[i]];//因为是求方法数，所以要累加起来
        }
        return dp[amount];
    }
};
```

### 15. LeetCode377. 组合总和 Ⅳ

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/13.png)

```cpp
思路：本题和零钱兑换II是同一类型的题，换汤不换药
注意：顺序不同的序列视作不同的组合，因此是求排列数（元素是有序的）而不是组合数

1.dp[i]：组成i的排列个数
2.dp[i]+=dp[i-nums[j]]：只要能够获取到nums[j],dp[i-nums[j]]是dp[i]的一部分
3.初始化dp：dp[0]=1，这样递推其他数值时才有基础，其他数值初始化为0，这样才不会影响累加值
4.遍历顺序：本质上就是对于每个容量i，用nums里所有元素去组合，从最基础的开始。比如说1个1组成1，2个1组成2,1个2组成2……
  4.1组合数：外层物品，内层容量
  4.2排列数：外层容量，内层物品（因为物品有顺序，顺序不同组合不同，所以我们在计算每个容量的方法数时，要把每个元素每种顺序都考虑到）
  4.3 eg.如果是外层遍历物品内层遍历容量，在计算dp[4]时，就会出现只有组合{1,3}而没有{3,1}，因为3只会出现在1后面
class Solution {
public:
    int combinationSum4(vector<int>& nums, int target) {
        //dp[i]：组成i的排列个数
        vector<int>dp(target+1);
        //初始化dp
        dp[0]=1;
        //完善dp
        for(int i=0;i<=target;i++){
            for(int j=0;j<nums.size();j++){
                //防止溢出
                if(i-nums[j]>=0&&dp[i]<=INT_MAX-dp[i-nums[j]]){
                    //在组成i-nums[j]的基础上，用nums[j]填补，从而累加相应的方法数
                    //在组成dp[4]时：
                    //用1填补dp[3]：{1}+{1,2},{1}+{2,1},{1}+{1,1,1},{1}+{3}
                    //用2填补dp[2]：{2}+{1,1},{2}+{2}
                    //用3填补dp[1]：{3}+{1}
                    //不用去管dp[1],dp[2],dp[3]是怎么组成的，但是可以确定1可以出现在3后面了
                    //组成不同的i时，我们只遍历了一次nums里的所有元素，因此不会有重复排列出现
                    dp[i]+=dp[i-nums[j]];
                }
            }
        }
        return dp[target];
    }
};
```

### 16. 爬楼梯(进阶班版)

```cpp
改为：一步1个台阶、2个台阶、3个台阶、4个台阶……直到m个台阶(每一阶可以重复使用)，求到达楼顶的总方法数。
理解：台阶就是物品，楼顶就是背包，本题被改编成了完全背包问题
1.dp[i]：爬到第i个台阶的方法数
2.dp[i]+=dp[i-j]：加入当前要走j个台阶，那么dp[i-j]就是dp[i]的一部分（从dp[i-1]累加到dp[i-j]），因为一步可以跨1~j个台阶
3.初始化dp：dp[0]是基础数值，由其推得其他dp值，所以dp[0]=1
4.遍历顺序：外背包，内物品。比如{1,2,1}和{1,1,2}实际上是不同的方法，其实就是求排列个数
int climbStairs(int m,int n){
    //dp[i]：爬到第i个台阶的方法数
    vector<int>dp(n+1);
    //初始化dp
    dp[0]=1;
    //完善dp
    for(int i=0;i<=n;i++){
        for(int j=1;j<=i;j++){//从1个台阶开始累加
            dp[i]+=dp[i-j];
        }
    }
    return dp[n];
}
```

### 17. LeetCode322. 零钱兑换

```cpp
思路：
1.dp[j]：组成j的最少硬币数
2.dp[j]=min(dp[j],dp[j-coins[i]]+1)：组成j-coins[i]最少的硬币数+coins[i]这枚硬币
3.初始化dp：dp[0]=0
4.遍历顺序：外物品内背包，外背包内物品都可以。本题只是求硬币个数，所以无论硬币面额有序还是无序都不影响。

外物品内背包：
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        //dp[j]：组成j的最少硬币数
        vector<int>dp(amount+1,INT_MAX-1);//不影响最小值判断
        //初始化dp
        dp[0]=0;
        //完善dp
        for(int i=0;i<coins.size();i++){
            for(int j=coins[i];j<=amount;j++){
                dp[j]=min(dp[j],dp[j-coins[i]]+1);
            }
        }
        return dp[amount]==INT_MAX-1?-1:dp[amount];
    }
};

外背包内物品：
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int>dp(amount+1,INT_MAX-1);
        dp[0]=0;
        for(int i=0;i<=amount;i++){
            for(int j=0;j<coins.size();j++){
                if(i-coins[j]>=0)dp[i]=min(dp[i],dp[i-coins[j]]+1);
            }
        }
        return dp[amount]==INT_MAX-1?-1:dp[amount];
    }
};
```

### 18. LeetCode279. 完全平方数

```cpp
思路：
1.dp[i]：和为i的完全平方数的最少数量
2.dp[i]=min(dp[i],dp[i-j*j]+1)
3.初始化dp：vector<int>dp(n+1,INT_MAX-1)，dp[0]=0
4.遍历顺序：由于是求个数，所以无论哪种顺序都可以
class Solution {
public:
    int numSquares(int n) {
        //dp[i]：和为i的完全平方数的最少数量
        vector<int>dp(n+1,INT_MAX-1);
        //初始化dp
        dp[0]=0;
        //完善dp
        for(int i=1;i<=n;i++){
            for(int j=1;j*j<=i;j++){
                dp[i]=min(dp[i],dp[i-j*j]+1);
            }
        }
        return dp[n]==INT_MAX?-1:dp[n];
    }
};
j^2不是j的平方，而是j异或2
```

**总结：**题目要我们返回什么结果，动态规划表就表达什么意思。注意遍历顺序：如果是求组合数，就外物品内背包；如果是求组合数，就外背包内物品

### 19. LeetCode139. 单词拆分

```cpp
思路：
wordDict是物品，s是背包，本题问wordDict里的单词能否组成s，意思就是物品word能否装满背包s，单词可以重复使用，所以就是完全背包。
1.dp[i]：s.substr(0,i)可以被wordDict中的单词组成
2.
dp[i]=dp[i]||(dp[i-wordDict[j].length()]&&s.substr(i-wordDict[j].length()+1,wordDic[j].length())==wordDict[j])
3.初始化dp：dp[0]=true，因为dp[0]是基础值，如果为false后面所有都会推得为false
4.遍历顺序：外背包，内物品：由于{leet，code}和{code，leet}拼成单词不同，所以物品是有顺序的，因此求得是排列个数。
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        //dp[i]：s.substr(0,i)可以被wordDict中的单词组成
        //因为转移方程减去的是单词的长度，所以i也表示长度才不容易混淆
        vector<bool>dp(s.length()+1,false);
        //初始化dp
        dp[0]=true;
        //完善dp
        for(int i=1;i<=s.length();i++){
            for(int j=0;j<wordDict.size();j++){
                //字符串长度类型是unsigned int，
                //int类型和unsigned int类型做运算时结果是unsigned int，永大于等于0
                //所以要先转成int
                if(i-(int)wordDict[j].length()>=0){
                    //substr(起始下标，截取长度)
                    string tmp=s.substr(i-wordDict[j].length(),wordDict[j].length());
                    dp[i]=dp[i]||(dp[i-wordDict[j].length()]&&tmp==wordDict[j]);
                }
            }
        }
        return dp[s.length()];
    }
};
```

### 20. 多重背包

```cpp
多重背包：有N种物品和一个容量为V 的背包。第i种物品最多有Mi件可用，每件耗费的空间是Ci ，价值是Wi 。求解将哪些物品装入背包可使这些物品的耗费的空间 总和不超过背包容量，且价值总和最大。

多重背包类似于0-1背包，物品都是有限个的，只要把物品摊开就是0-1背包了，只不过有多件相同的物品。

int multiBagValue(vector<int>&weight,vector<int>&value,vector<int>&nums,int bagWeight){
    //dp[j]：装满容量为j的背包的最大价值
    vector<int>dp(bagWeight+1);
    //初始化dp
    //完善dp，求放入物品总价值，组合数和排列数没有区别
    for(int i=0;i<weight.size();i++){
        for(int j=bagWeight;j>=weight[i];j--){
            //从放入1个开始算，因为dp[j]初始值就是放入0个物品i的值
            for(int k=1;k<=nums[i]&&j-k*weight[i]>=0;k++){
                dp[j]=max(dp[j],dp[j-k*weight[i]]+k*value[i]);
            }
        }
    }
    return dp[bagWeight];
}
```

### 21.LeetCode198. 打家劫舍

```cpp
思路：
1.dp[i]：经过第i家时，偷或不偷（二选一）的最大累积收获
2.dp[i]=max(dp[i-1],dp[i-2]+nums[i])
3.初始化dp：dp[0]=nums[0]，dp[1]=max(dp[0]，nums[1])
class Solution {
public:
    int rob(vector<int>& nums) {
        //特殊情况
        if(nums.size()==1){
            return nums[0];
        }
        //dp[i]：经过第i家时，偷或不偷（二选一）的最大累积收获
        vector<int>dp(nums.size());
        //初始化dp
        dp[0]=nums[0];//在第0家肯定是偷有最大收获
        dp[1]=max(dp[0],nums[1]);//在第1家要判断在第0家偷收获大，还是在当前偷收获更大
        //完善dp
        for(int i=2;i<nums.size();i++){
            dp[i]=max(dp[i-1],dp[i-2]+nums[i]);
        }
        return dp[nums.size()-1];
    }
};
```

### 22. LeetCode213. 打家劫舍 II

```cpp
思路：
1.情况一：考虑不包含首尾元素(只是考虑包不包含，还不确定，下面同理)(1,nums.size()-2)
2.情况二：考虑包含首元素，不包含尾元素(0,nums.size()-2)
3.情况三：考虑不包含首元素，包含尾元素(1,nums.size()-1)
最终决定偷或不偷的属于完善dp表的工作
情况二和情况三都包含了情况一，所以只需考虑情况二和情况三：
(1,nums.size()-2)∈(0,nums.size()-2)&&(1,nums.size()-2)∈(1,nums.size()-1)

class Solution {
public:
    int robRotate(vector<int>&nums,int start,int end){
        //dp[i]：经过下标为i的屋舍，偷或不偷的最大累积收入
        vector<int>dp(end+1);
        //初始化dp
        dp[start]=nums[start];
        dp[start+1]=max(dp[start],nums[start+1]);
        for(int i=start+2;i<=end;i++){//注意从start+2开始，因为基础值为dp[start]和dp[start+1]
            dp[i]=max(dp[i-1],dp[i-2]+nums[i]);
        }
        return dp[end];
    }

    int rob(vector<int>& nums) {
        if(nums.size()==1){
            return nums[0];
        }
        if(nums.size()==2){
            return max(nums[0],nums[1]);
        }
        int rob1=robRotate(nums,0,nums.size()-2);//情况二
        int rob2=robRotate(nums,1,nums.size()-1);//情况三
        return max(rob1,rob2);
    }
};
```

### 23. LeetCode337. 打家劫舍 III

```cpp
1.暴力递归：
class Solution {
public:
    int rob(TreeNode* root) {
        if(root==NULL)return 0;
        //叶子节点没有左右孩子且没有被跳过，所以肯定会被偷
        if(root->left==NULL&&root->right==NULL)return root->val;

        //偷父节点
        int val1=root->val;
        if(root->left)val1+=rob(root->left->left)+rob(root->left->right);//跳过左孩子
        if(root->right)val1+=rob(root->right->left)+rob(root->right->right);//跳过右孩子

        //不偷父节点
        int val2=0;
        val2+=rob(root->left)+rob(root->right);

        return max(val1,val2);
    }
};
代码超时了，因为有重复计算（当前节点已经计算过孩子节点的孩子节点，但是当前节点的孩子节点会再算一遍其孩子节点）

2.记忆化递推：
class Solution {
public:
    map<TreeNode*,int>recorded;//记录已经计算过的节点
    int rob(TreeNode* root) {
        if(root==NULL)return 0;
        if(root->left==NULL&&root->right==NULL)return root->val;
        if(recorded[root])return recorded[root];

        //偷父节点
        int val1=root->val;
        if(root->left)val1+=rob(root->left->left)+rob(root->left->right);//跳过左孩子
        if(root->right)val1+=rob(root->right->left)+rob(root->right->right);//跳过右孩子

        //不偷父节点
        int val2=0;
        val2+=rob(root->left)+rob(root->right);
        
        recorded[root]=max(val1,val2);
        return max(val1,val2);
    }
};

3.动态规划：
(1)确定递归函数的参数和返回类型
用大小为2的数组记录当前节点偷与不偷的最大金额
dp[0]：不偷该节点的最大金额
dp[1]：偷该节点的最大金额
(2)终止条件
if(node==NULL)return {0,0};
(3)遍历顺序
后序遍历
(4)单层递归逻辑
偷当前节点：左右孩子就不能偷
不偷当前节点：左右孩子就可以偷（当不一定就必须要偷，选一个最大值）

class Solution {
public:
    vector<int>process(TreeNode*node){
        if(node==NULL)return {0,0};
        vector<int>left=process(node->left);//左子树信息
        vector<int>right=process(node->right);//右子树信息

        //当前层逻辑
        int val1=node->val+left[0]+right[0];//偷当前节点
        int val2=max(left[0],left[1])+max(right[0],right[1]);//不偷当前节点

        return {val2,val1};
    }

    int rob(TreeNode* root) {
        vector<int>data=process(root);
        return max(data[0],data[1]);
    }
};

2和3的时间复杂度都是O(n)：每个节点都只遍历的一次
```

### 24. LeetCode121. 买卖股票的最佳时机

```cpp
1.贪心算法：
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int minCost=INT_MAX;
        int res=INT_MIN;
        for(int i=0;i<prices.size();i++){
            minCost=min(minCost,prices[i]);
            res=max(res,prices[i]-minCost);//必须在卖出前买入
        }
        return res;
    }
};

2.动态规划：每天只可能有两种状态（持有或不持有股票）
(1)dp[i][0]：第i天持有股票的所得最多现金（持有不是买入，可能前一天就持有了）- 历史最低值
   dp[i][1]：第i天不持有股票所得最多现金（可能今天卖出。也可能前一天就不持有股票）- 历史最高利润 
(2)dp[i][0]=max(dp[i-1][0],-prices[i])
   dp[i][1]=max(dp[i-1][1],prices[i]+dp[i-1][0]);
(3)初始化dp

class Solution {
public:
    int maxProfit(vector<int>& prices) {
        //dp[i][0]：第i天持有股票的所得最多现金（持有不是买入，可能前一天就持有了）- 历史最低值
        //dp[i][1]：第i天不持有股票所得最多现金（可能今天卖出或前一天就不持有股票）- 历史最高利润
        vector<vector<int>>dp(prices.size(),vector<int>(2));
        //初始化dp,dpp0][1]初始值就是0，因为没有前一天，没有买入就一定没有卖出，所以是0
        dp[0][0]=-prices[0];//没有前一天，所以肯定是当天买入股票
        //完善dp
        for(int i=1;i<prices.size();i++){
            dp[i][0]=max(dp[i-1][0],-prices[i]);
            dp[i][1]=max(dp[i-1][1],prices[i]+dp[i-1][0]);
        }
        return dp[prices.size()-1][1];
    }
};

滚动数组：
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        //dp[i][0]：第i天持有股票所得现金
        //dp[i][1]：第i天不持有股票所得最多现金
        vector<int>dp(2);

        //初始化
        dp[0]=-prices[0];
        dp[1]=0;

        //完善dp
        for(int i=1;i<prices.size();i++){
            dp[1]=max(dp[1],dp[0]+prices[i]);
            dp[0]=max(dp[0],dp[1]-prices[i]);
        }

        return dp[1];  
    }
};
```

### 25. LeetCode122. 买卖股票的最佳时机 II

```cpp
思路：
每天只可能有两种状态：持有或不持有股票
持有股票：今天之前就已经买入，或者今天买入，选能够让现金最多的情况，因为dp[i][0]越大，说明买入价格越低
不持有股票：今天之前就不持有，或者今天卖出，选能够让现金最多的情况，因为dp[i][1]越大，说明利润越高

1.dp[i][0]：第i天持有股票所得现金
  dp[i][1]：第i天不持有股票所得最多现金
2.dp[i][0]=max(dp[i-1][0],dp[i-1][1]-prices[i])
    由于本题的股票可以买卖多次，所以第i天买入股票时，所持有的现金可能包含之前买卖过的利润，则dp[i][0]=dp[i-1][1]-prices[i]     dp[i][1]=max(dp[i-1][1],dp[i-1][0]+prices[i])
    第i-1天就不持有股票，或者第i天卖出股票，dp[i-1][0]已经减去过买入股票的价格，加上prices[i]刚好就是买卖一次股票的利润
3.初始化dp：
dp[0][0]=-prices[0]：第0天之前不可能买入股票，所以第0天必须买入股票
dp[0][1]=0：第0天不持有股票的唯一选择就是不买入

class Solution {
public:
    int maxProfit(vector<int>& prices) {
        //dp[i][0]：第i天持有股票所得现金
        //dp[i][1]：第i天不持有股票所得最多现金
        vector<vector<int>>dp(prices.size(),vector<int>(2));

        //初始化
        dp[0][0]=-prices[0];
        dp[0][1]=0;

        //完善dp
        for(int i=1;i<prices.size();i++){
            dp[i][0]=max(dp[i-1][0],dp[i-1][1]-prices[i]);
            dp[i][1]=max(dp[i-1][1],dp[i-1][0]+prices[i]);//有利润就卖出，保证在最短的时间内完成交易，且每次交易利润为正
        }

        return dp[prices.size()-1][1];  
    }
};

滚动数组：
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        //dp[i][0]：第i天持有股票所得现金
        //dp[i][1]：第i天不持有股票所得最多现金
        vector<int>dp(2);

        //初始化
        dp[0]=-prices[0];
        dp[1]=0;

        //完善dp
        for(int i=1;i<prices.size();i++){
            int pre=dp[0];//记录前一天的dp[0][0]，用于计算dp[0][1]
            dp[0]=max(dp[0],dp[1]-prices[i]);
            dp[1]=max(dp[1],dp[0]+prices[i]);
        }

        return dp[1];  
    }
};
```

### 26. LeetCode123. 买卖股票的最佳时机 III

```  cpp
思路：
每天可能有四个状态
持有股票：第一次持有或第二次持有
不持有股票：第一次不持有或第二次不持有
0.第一次持有股票
1.第一次不持有股票
2.第二次持有股票
3.第二次不持有股票

定义dp
dp[i][0]：第一次持有股票最多现金
dp[i][1]：第一次不持有股票所得最多现金
dp[i][2]：第二次持有股票最多现金
dp[i][3]：第二次不持有股票所得最多现金

转移方程：
dp[i][0]=max(dp[i-1][0],-prices[i])：
第一次持有股票可能是今天之前就已经持有，也可能是今天买入（昨天是不持有股票状态）
注意和上一题的区别，因为本题最多完成两笔交易，意味着可以有0/1/2次交易。意味着第一次持有股票时，现金数必定是-prices[i]，而不能用所得利润取购入，因为第一次持有股票之前肯定没有交易完成。
dp[i][1]=max(dp[i-1][1],dp[i-1][0]+prices[i])：
第一次不持有股票可能是今天之前就已经卖出，也可能是今天卖出（昨天是持有股票状态）
dp[i][2]=max(dp[i-1][2],dp[i-1][1]-prices[i])：
第二次持有股票是建立在第一次股票已经卖出的情况下，所以使用第一次已经卖出股票的利润来买入第二次股票。也可能是今天之前已经第二次买入
dp[i][3]=max(dp[i-1][3],dp[i-1][2]+prices[i])：
第二次不持有股票建立在第二次股票已经买入的情况下。也可能今天之前就已经第二次卖出股票。取最大值保存最大利润

初始化dp：
dp[0][0]=-prices[0];//第0天第一次只能买入
dp[0][1]=0;//第0天第一次不持有股票的情况下只能不买入，所以是原始财产0
dp[0][2]=-prices[0];//第0天第一次买入后再同一天卖出，之后第二次买入，便是第二次持有股票了
dp[0][3]=0;//第0天第二次不持有股票，即第二次不买入

遍历顺序：先更新那些不需要作为其他dp值基础的值

滚动数组：
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        /*
        dp[i][0]：第一次持有股票最多现金
        dp[i][1]：第一次不持有股票所得最多现金
        dp[i][2]：第二次持有股票最多现金
        dp[i][3]：第二次不持有股票所得最多现金
        */
        vector<int>dp(4);

        //初始化dp
        dp[0]=-prices[0];
        dp[1]=0;
        dp[2]=-prices[0];
        dp[3]=0;

        //完善dp
        for(int i=1;i<prices.size();i++){         
            dp[3]=max(dp[3],dp[2]+prices[i]);
            dp[2]=max(dp[2],dp[1]-prices[i]);
            dp[1]=max(dp[1],dp[0]+prices[i]);
            dp[0]=max(dp[0],-prices[i]);//第一次持有股票之前没有交易完成，不能用利润去购买
        }

        return dp[3];
    }
};
```

### 27. LeetCode188. 买卖股票的最佳时机 IV

```cpp
思路：
1.dp[i][j]：第i天的状态为j时，所得最多现金
  j的状态：奇数为买，偶数为卖
    0.不操作，保证除了可以完成k次交易外，还可以完成0/1/2/3/.../k-1次交易，并不是固定k次交易，满足题意最多k次交易
    1.第一次买入
    2.第一次卖出
    3.第二次买入
    4.第二次卖出
    ……
  j的范围：2*k+1
2.dp[i][j+1]=max(dp[i-1][j+1],dp[i-1][j]-prices[i])：用上一次卖出股票的利润买入这次股票
  dp[i][j+2]=max(dp[i-1][j+2],dp[i-1][j+1]+prices[i])：用上一次买入股票后剩余现金加上当前股票值就是这次卖出所得利润
3.初始化dp
  dp[0][奇]=-prices[0];
  dp[0][偶]=0;

class Solution {
public:
    int maxProfit(int k, vector<int>& prices) {
        if(prices.size()==0||k==0){
            return 0;
        }
        //dp[i][j]：第i天的状态为j时，所得最多现金
        vector<vector<int>>dp(prices.size(),vector<int>(2*k+1));
        //初始化dp
        for(int j=1;j<2*k+1;j+=2){
            dp[0][j]=-prices[0];
        }
        //完善dp
        for(int i=1;i<prices.size();i++){
            for(int j=0;j<2*k-1;j+=2){
                dp[i][j+1]=max(dp[i-1][j+1],dp[i-1][j]-prices[i]);//可能今天之前就买入了
                dp[i][j+2]=max(dp[i-1][j+2],dp[i-1][j+1]+prices[i]);//可能今天直接就卖出了
                //j+2<2*k+1 => j<2*k-1
            }
        }
        //最终返回值是：最多完成k次交易后所得最多现金，可能只完成了k-j(j:0~k)次交易后，最大值延续到最后一天
        return dp[prices.size()-1][2*k];
    }
};
```

### 28. LeetCode309. 最佳买卖股票时机含冷冻期

```cpp
思路：
每天有3个状态：
0.持有股票
1.保持卖出股票状态（进入冷冻期前，一定卖出了股票，在冷冻期后，不一定就要买入股票（因为不确认是否一定有利润获取），所以保持卖出状态）
2.卖出股票状态（当天卖出了股票）
3.冷冻期
因为多了一个冷冻期，所以把不持有股票状态分开计算。1和2状态合起来就是不持有股票状态，如果统合起来为不持有股票状态，就分不清是今天之前的哪一天卖出了股票，所以也就无法确认今天能否买入股票。

1.dp[i][j]：第i天为j状态时的最多现金
2.初始化dp：
  dp[0][0]=-prices[0]
  dp[0][1]=0
  dp[0][2]=0
  dp[0][3]=0

class Solution {
public:
    int maxProfit(vector<int>& prices) {
        //dp[i][j]：第i天为j状态时的最多现金
        vector<vector<int>>dp(prices.size(),vector<int>(4));
        //初始化dp
        dp[0][0]=-prices[0];
        //完善dp
        for(int i=1;i<prices.size();i++){
            //dp[i][0]不能用dp[i-1][2]推得的原因：
            //dp[i-1][2]是第i-1天具体卖出的现金，卖出后的第一天是冷冻期，不能买入，只有过了一天冷冻期后才行
            dp[i][0]=max(dp[i-1][0],max(dp[i-1][1]-prices[i],dp[i-1][3]-prices[i]));
            dp[i][1]=max(dp[i-1][1],dp[i-1][3]);//保持卖出状态的上一个状态一定是冷冻期
            dp[i][2]=dp[i-1][0]+prices[i];//今天卖出
            dp[i][3]=dp[i-1][2];//具体卖出后的一天是冷冻期
        }
        return max(dp[prices.size()-1][1],max(dp[prices.size()-1][2],dp[prices.size()-1][3]));
    }
};
```

### 29. LeetCode714. 买卖股票的最佳时机含手续费

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/LeetCode-img/15.png)

```cpp
1.dp含义：
  dp[i][0]：第i天持有股票时的最多现金
  dp[i][1]：第i天不持有股票时的最多现金
2.转移方程：
  dp[i][0]=max(dp[i-1][0],dp[i-1][1]-prices[i])：
  (1)今天之前就已经持有股票
  (2)今天买入股票，因为不限制交易次数，所以为了记录总利润，用之前买卖股票所得利润买入股票
  当天价格越低，dp[i-1][1]-prices[i]就越大，买入后获取利润值更大
  
  dp[i][1]=max(dp[i-1][1],dp[i-1][0]+prices[i]-fee)：
  (1)今天之前就已经不持有股票
  (2)今天卖出股票，因为买入股票时已经在总利润里减去了买入价格，再加上卖出价格，差价就是利润了，注意还要减去手续费
  当天价格越高，dp[i-1][0]+prices[i]-fee就越大，利润也就更大
3.初始化dp：
  dp[0][0]=-prices[i]：第0天持有股票只能买入股票
  dp[0][1]=0：不持有股票只能不买，或者买入后卖出，由于交易有手续费，肯定是不买所得现金最多。

class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        //dp[i][0]：第i天持有股票时的最多现金
        //dp[i][1]：第i天不持有股票时的最多现金
        vector<vector<int>>dp(prices.size(),vector<int>(2));
        //初始化dp
        dp[0][0]=-prices[0];
        //完善dp
        for(int i=1;i<prices.size();i++){
            dp[i][0]=max(dp[i-1][0],dp[i-1][1]-prices[i]);
            dp[i][1]=max(dp[i-1][1],dp[i-1][0]+prices[i]-fee);
        }
        return dp[prices.size()-1][1];
    }
};

滚动数组：
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        vector<int>dp(2);
        dp[0]=-prices[0];
        for(int i=1;i<prices.size();i++){
            int pre=dp[0];
            dp[0]=max(dp[0],dp[1]-prices[i]);
            dp[1]=max(dp[1],pre+prices[i]-fee);
        }
        return dp[1];
    }
};
```

### 30. LeetCode300. 最长递增子序列

```cpp
思路：
1.dp含义：
dp[i]：集合nums从下标0到下标i并且以nums[i]结尾的最长递增子序列长度
(1)为什么能够以nums[i]结尾？因为每个递增子序列必定是以集合nums中的其中一个元素结尾。
(2)为什么一定要以nums[i]结尾？因为在递推时，我们需要与前一个递增子序列作比较，而只有知道尾元素大小才能有效地比较。
2.转移方程：
if(nums[j]<nums[i])dp[i]=max(dp[i],dp[j]+1);//j：0~(i-1)
3.初始化dp：
dp[0]=1;

方法一：动态规划
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        if(nums.size()<=1)return nums.size();
        //dp[i]：集合nums从下标0到下标i并且以nums[i]结尾的最长递增子序列长度
        vector<int>dp(nums.size(),1);//基础长度是1
        int res=0;
        //完善dp
        for(int i=1;i<nums.size();i++){
            for(int j=0;j<i;j++){
                if(nums[j]<nums[i])dp[i]=max(dp[i],dp[j]+1);//保留以nums[i]结尾最大长度
            }
            res=max(dp[i],res);//保留整体集合最长递增子序列长度
        }
        return res;
    }
};
时间复杂度：O(n^2)
空间复杂度：O(n)

方法二：贪心+动态规划
ends[i]：所有长度为i+1的递增子序列的尾元素最小值，且ends[i]必定小于ends[i+1]
为什么ends[i]<ends[i+1]？设以ends[i]、ends[i+1]结尾的递增子序列分别为subSequence1（长度为i+1），subSequence2（长度为i+2），
假设ends[i]>ends[i+1]，即subSequence1[i]>subSequence2[i+1]，由于递增，
所以subSequence1[i]>subSequence2[i+1]>subSequence2[i] => subSequence1[i]>subSequence2[i]
则 ends[i]=subSequence2[i]，有矛盾，所以假设不成立。

class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        if(nums.size()<=1)return nums.size();
        //ends[i]：所有长度为i+1的递增子序列的尾元素最小值，且ends[i]必定小于ends[i+1]
        vector<int>ends(nums.size());
        ends[0]=nums[0];
        int right=0;
        for(int i=1;i<nums.size();i++){
            //长度最长的递增子序列尾元素小于nums[i]，有效区域右移，并记录最小尾元素
            if(ends[right]<nums[i]){
                ends[++right]=nums[i];
            }
            else{
                int l=0;
                while(l<right&&ends[l]<nums[i]){//找到ends最左边比nums[i]大的尾元素
                    l++;
                }
                //退出循环有两种情况，只有第二种情况才能赋值
                if(ends[l]>nums[i])ends[l]=nums[i];
            }
        }
        //ends[right]是"长度为right+1"的递增子序列的最小尾元素
        return right+1;
    }
};
时间复杂度：O(n*logn)
空间复杂度：O(n)
```

### 31. LeetCode674. 最长连续递增序列

```cpp
1.dp[i]：以nums[i]结尾的连续递增序列长度
2.if(nums[i]>nums[i-1])dp[i]=dp[i-1]+1：因为是连续的，所以只需要考虑nums[i]是否比nums[i-1]大就行了（贪心）
3.初始化dp：全都初始化成1，因为至少包含nums[i]

动态规划：
class Solution {
public:
    int findLengthOfLCIS(vector<int>& nums) {
        if(nums.size()<=1)return nums.size();
        //dp[i]：以nums[i]结尾的连续递增序列长度
        vector<int>dp(nums.size(),1);
        int res=INT_MIN;
        //完善dp
        for(int i=1;i<nums.size();i++){
            if(nums[i]>nums[i-1])dp[i]=dp[i-1]+1;
            res=max(res,dp[i]);//记录最长连续递增序列长度
        }
        return res;
    }
};

贪心：
class Solution {
public:
    int findLengthOfLCIS(vector<int>& nums) {
        if(nums.size()<=1)return nums.size();
        int count=1;//记录过程中个递增序列长度
        int res=INT_MIN;//记录最长递增序列长度
        for(int i=1;i<nums.size();i++){
            if(nums[i]>nums[i-1])count++;
            else count=1;//重新取递增序列头元素
            res=max(res,count);
        }
        return res;
    }
};

与前一题的区别是：
本题dp[i]状态之和dp[i-1]有关，因为是连续的。
而前一题因为是不连续的，所以dp[i]状态和dp[0]/dp[1]/.../dp[i-1]都有关系
```

### 32. LeetCode18. 最长重复子数组

```cpp
1.dp[i][j]：以nums1[i]结尾的nums1和以nums2[j]结尾的nums2的重复子数组长度
2.if(nums1[i]==nums[j])dp[i][j]=dp[i-1][j-1]+1;
  else dp[i][j]=0;
3.初始化dp

普通动态规划二维表：
class Solution {
public:
    int findLength(vector<int>& nums1, vector<int>& nums2) {
        //长度是1，看唯一元素是否相同
        if(nums1.size()==1&&nums2.size()==1)return nums1[0]==nums2[0]?1:0;
        //dp[i][j]：以nums1[i]结尾的nums1和以nums2[j]结尾的nums2的重复子数组长度
        vector<vector<int>>dp(nums1.size(),vector<int>(nums2.size()));
        //记录最长重复子数组长度
        int res=0;
        //初始化dp
        for(int i=0;i<nums1.size();i++){
            if(nums1[i]==nums2[0])dp[i][0]=1;
            res=max(res,dp[i][0]);
        }
        for(int j=0;j<nums2.size();j++){
            if(nums2[j]==nums1[0])dp[0][j]=1;
            res=max(res,dp[0][j]);
        }
        //完善dp
        for(int i=1;i<nums1.size();i++){
            for(int j=1;j<nums2.size();j++){
                if(nums1[i]==nums2[j]){
                    dp[i][j]=dp[i-1][j-1]+1;
                }else{
                    //如果nums1[i]!=nums2[j]，那必不可能重复，因为子数组一定要包含nums1[i]和nums2[j]
                    dp[i][j]=0;
                }
                res=max(res,dp[i][j]);
            }
        }
        return res;
    }
};

观察上方代码，发现不仅要额外判断数组长度为1时的情况，还要在初始化dp时更新res，代码略显臃肿。
再看转移方程，我们可以多一行一列，基础值为0，但也因此有额外空间开销
1.dp[i][j]：以nums1[i-1]结尾的nums1和以nums2[j-1]结尾的nums2的重复子数组长度
2.if(nums1[i]==nums[j])dp[i][j]=dp[i-1][j-1]+1;
  else dp[i][j]=0;
3.初始化dp
改良动态规划二维表（代码行数更少）：
class Solution {
public:
    int findLength(vector<int>& nums1, vector<int>& nums2) {
        //dp[i][j]：以nums1[i-1]结尾的nums1和以nums2[j-1]结尾的nums2的重复子数组长度
        vector<vector<int>>dp(nums1.size()+1,vector<int>(nums2.size()+1));
        //记录最长重复子数组长度
        int res=0;
        //完善dp
        for(int i=1;i<nums1.size()+1;i++){
            for(int j=1;j<nums2.size()+1;j++){
                if(nums1[i-1]==nums2[j-1])dp[i][j]=dp[i-1][j-1]+1;
                else dp[i][j]=0;
                res=max(res,dp[i][j]);
            }
        }
        return res;
    }
};

滚动数组：
观察转移方程，dp是一行一行更新，并且是从左到右
class Solution {
public:
    int findLength(vector<int>& nums1, vector<int>& nums2) {
        vector<int>dp(nums2.size()+1);
        int res=0;
        for(int i=1;i<nums1.size()+1;i++){
            for(int j=nums2.size();j>=1;j--){
                //滚动数组应当优先更新那些不用来递推其他dp值的
                if(nums1[i-1]==nums2[j-1])dp[j]=dp[j-1]+1;
                else dp[j]=0;
                res=max(res,dp[j]);
            }
        }
        return res;
    }
};

总结：
实现出动态规划二维表后，观察转移方程，看是否能够利用滚动数组实现，一般来说一行一行遍历的都可以转成一维数组。
```

### 33. LeetCode1143. 最长公共子序列

```cpp
1.dp[i][j]：text1[0,i-1]和text2[0,j-1]的最长公共子序列长度
2.if(text[i]==text[j])dp[i][j]=dp[i-1][j-1]+1;
  else dp[i][j]=max(dp[i-1][j],dp[i][j-1]);

class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        //dp[i][j]：text1[0,i-1]和text2[0,j-1]的最长公共子序列长度
        vector<vector<int>>dp(text1.length()+1,vector<int>(text2.length()+1));
        //完善dp
        for(int i=1;i<text1.length()+1;i++){
            for(int j=1;j<text2.length()+1;j++){
                if(text1[i-1]==text2[j-1]){
                    dp[i][j]=dp[i-1][j-1]+1;
                }
                else{
                    //text1[i-1]!=text2[j-1]
                    //1.text1[i-2]和text[j-1]比较
                    //2.text1[i-1]和text[j-2]比较
                    //上述两种情况已经涵盖了所有可能性：text1[i-2]子序列涵盖了text1[i-3]所有子序列
                    //text1[i-1]和text2[j-1]是新出现的字符，所以必须带着
                    dp[i][j]=max(dp[i-1][j],dp[i][j-1]);//记忆化搜索
                }
            }
        }
        return dp[text1.length()][text2.length()];
    }
};
```

### 34. LeetCode1035. 不相交的线

```cpp
思路：
线只要在连接nums1和nums2的元素时按照相对同样的顺序就不会相交，所以本质上是求nums1和nums2最长公共子序列长度。

1.dp[i][j]：nums1[0,i-1]和nums2[0,j-1]的公共子序列长度
2.if(nums1[i-1]==nums2[j-1])dp[i][j]=dp[i-1][j-1]+1;
  else dp[i][j]=max(dp[i-1][j],dp[i][j-1]);

class Solution {
public:
    int maxUncrossedLines(vector<int>& nums1, vector<int>& nums2) {
        //dp[i][j]：nums1[0,i-1]和nums2[0,j-1]的公共子序列长度
        vector<vector<int>>dp(nums1.size()+1,vector<int>(nums2.size()+1));
        //完善dp
        for(int i=1;i<nums1.size()+1;i++){
            for(int j=1;j<nums2.size()+1;j++){
                if(nums1[i-1]==nums2[j-1])dp[i][j]=dp[i-1][j-1]+1;
                else dp[i][j]=max(dp[i][j-1],dp[i-1][j]);
            }
        }
        return dp[nums1.size()][nums2.size()];
    }
};
```

### 35. LeetCode53. 最大子数组和

```cpp
1.dp[i]：以nums[i-1]结尾的最大子数组和
2.if(dp[i-1]>0)dp[i]=dp[i-1]+nums[i];
  else dp[i]=nums[i-1];

class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        //dp[i]：以nums[i]结尾的最大子数组和
        vector<int>dp(nums.size()+1);
        int res=INT_MIN;
        //完善dp
        for(int i=1;i<nums.size()+1;i++){
            //只有前面子数组和为正，才对自己有帮助，才能够获取最大子数组和
            if(dp[i-1]>0)dp[i]=dp[i-1]+nums[i-1];
            else dp[i]=nums[i-1];
            res=max(res,dp[i]);
        }
        return res;
    }
};

dp[i]只与dp[i-1]有关，与dp[i-1]前的元素都无关，所以我们只需一个值即可
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int cur=0;
        int res=INT_MIN;
        for(int i=0;i<nums.size();i++){
            if(cur>=0){
                cur=cur+nums[i];
            }else{
                //cur<0，只会减少后续子数组和
                cur=nums[i];
            }
            res=max(res,cur);
        }
        return res;
    }
};
```



### 36. LeetCode392.判断子序列

```cpp
1.dp含义：
dp[i][j]：以s[i-1]结尾的字符串s，和以t[j-1]结尾的字符串t，相同子序列长度
注意：是判断s是否为t的子序列，所以t.length()>=s.length()；s一定要包含s[i-1]，t不一定要包含t[j-1]
    
2.转移方程：
只需要考虑删除字符串t的元素即可
if(s[i-1]==t[j-1])dp[i][j]=dp[i-1][j-1]+1;//t中新出现的字符能够和s[i-1]匹配
else dp[i][j]=dp[i][j-1];//无法匹配，t删除元素，继续匹配，然前一个字符去处理

3.初始化dp：
//初始化二维数组dp时就已经完成了
dp[0][j]=0;
dp[i][0]=0;

4.遍历顺序：
dp值从左上角递推而得
i：从上到下
j：从左至右

class Solution {
public:
    bool isSubsequence(string s, string t) {
        //dp[i][j]：以s[i-1]结尾的字符串s，和以t[j-1]结尾的字符串t，相同子序列长度
        vector<vector<int>>dp(s.length()+1,vector<int>(t.length()+1));
        //完善dp
        for(int i=1;i<s.length()+1;i++){
            for(int j=i;j<t.length()+1;j++){
                if(s[i-1]==t[j-1])dp[i][j]=dp[i-1][j-1]+1;
                else dp[i][j]=dp[i][j-1];
            }
        }
        return dp[s.length()][t.length()]==s.length();
    }
};

双指针法：
创建两个索引指针sIndex、tIndex分别指向s和t，如果t[tIndex]==s[sIndex]，那么两根指针同时后移；否则tIndex后移，直到sIndex扫完s
，恰好也符合子序列顺序。
class Solution {
public:
    bool isSubsequence(string s, string t) {
        int sIndex=0;//指向s字符的指针
        int tIndex=0;//指向t字符的指针
        while(sIndex<s.length()&&tIndex<t.length()){
            if(s[sIndex]==t[tIndex]){
                sIndex++;
                tIndex++;
            }else{
                tIndex++;
            }
        }
        //sIndex扫过的区域都是在t中以同样的顺序出现过的
        return sIndex==s.length();
    }
};
```

### 37. LeetCode115. 不同的子序列

```cpp
解读题意：从s中可以找出几个子序列恰好和t相同。s如何删除元素可以变成t
1.dp含义：
dp[i][j]：以s[i-1]结尾的字符串s中，以j[i-1]结尾的字符串t个数

2.转移方程：
只有出现新字符时（下一轮循环）才会有未考虑的情况，对于新出现字符只有两种考虑（使用或不使用）
if(s[i-1]==t[j-1])dp[i][j]=dp[i-1][j-1]+dp[i-1][j];//s[i-1]可使用也可不使用(模拟删除)，两种情况总和。t无法删除元素
else dp[i][j]=dp[i-1][j];//不匹配，不用考虑s[i-1]

3.初始化dp：
dp[0][j]=0;
dp[i][0]=1;
dp[0][0]=1;

4.遍历顺序：
i：从上到下
j：从左到右

class Solution {
public:
    int numDistinct(string s, string t) {
        //dp[i][j]：以s[i-1]结尾的字符串s中，以j[i-1]结尾的字符串t个数
        vector<vector<uint64_t>>dp(s.length()+1,vector<uint64_t>(t.length()+1));
        //初始化dp
        for(int i=0;i<s.length()+1;i++){
            dp[i][0]=1;
        }
        for(int j=1;j<t.length()+1;j++){
            dp[0][j]=0;
        }
        //完善dp
        for(int i=1;i<s.length()+1;i++){
            for(int j=1;j<t.length()+1;j++){
                if(s[i-1]==t[j-1])dp[i][j]=dp[i-1][j-1]+dp[i-1][j];
                else dp[i][j]=dp[i-1][j];
            }
        }
        return dp[s.length()][t.length()];
    }
};
```

### 38. LeetCode583. 两个字符串的删除操作

```cpp
思路：
二维数组，因为需要操作两个字符串

1.dp含义：
dp[i][j]：让以word1[i-1]结尾的字符串word1和以word2[j-1]结尾的字符串word2相同的最少操作数
2.转移方程：
对于新字符word1[i-1]和word2[j-1]只有相同和不相同两种情况
if(word1[i-1]==word[j-1])dp[i][j]=dp[i-1][j-1];//保留这两字符可以减少两步删除操作
else dp[i][j]=min(dp[i-1][j]+1,dp[i][j-1]+1,dp[i-1][j-1]+2);//二者选一个删除或都删除，取最小操作数
3.遍历顺序：
i：从上到下
j：从左到右
4.初始化dp：
dp[0][j]=j;//word1是空字符，word2必须删掉所有字符才能相同
dp[i][0]=i;

class Solution {
public:
    int minDistance(string word1, string word2) {
      //dp[i][j]：让以word1[i-1]结尾的字符串word1和以word2[j-1]结尾的字符串word2相同的最少操作数
      vector<vector<int>>dp(word1.length()+1,vector<int>(word2.length()+1));
      //初始化dp
      for(int i=0;i<=word1.length();i++){
          dp[i][0]=i;
      }
      for(int j=0;j<=word2.length();j++){
          dp[0][j]=j;
      }
      //完善dp
      for(int i=1;i<=word1.length();i++){
          for(int j=1;j<=word2.length();j++){
              if(word1[i-1]==word2[j-1])dp[i][j]=dp[i-1][j-1];
              else dp[i][j]=min(dp[i-1][j]+1,min(dp[i][j-1]+1,dp[i-1][j-1]+2));
          }
      }
      return dp[word1.length()][word2.length()];
    }
};

也可以求最长公共子序列长度，然后经过计算得出结果。
```

### 39. LeetCode72. 编辑距离

```cpp
思路：
虽然题目说让word1变成word2，但也可以让word2变成word1。因为删除word1的字符，等效于在word2中添加字符，所以删除操作包含了添加操作

1.dp含义：
dp[i][j]：让以word1[i-1]结尾的字符串word1和以word2[j-1]结尾的字符串word2相同的最少操作数
2.转移方程：
if(word1[i-1]==word2[j-1])dp[i][j]=dp[i-1][j-1];
else{
    dp[i][j]=min(dp[i-1][j]+1,dp[i][j-1]+1);//删除，这二者已经包含了dp[i-1][j-1]+2
    dp[i][j]=min(dp[i][j],dp[i-1][j-1]+1);//替换
}
3.遍历顺序：
i：从上到下
j：从左到右
4.初始化dp：
dp[0][j]=j;
dp[i][0]=i;

class Solution {
public:
    int minDistance(string word1, string word2) {
        //dp[i][j]：让以word1[i-1]结尾的字符串word1和以word2[j-1]结尾的字符串word2相同的最少操作数
        vector<vector<int>>dp(word1.length()+1,vector<int>(word2.length()+1));
        //初始化dp
        for(int i=0;i<=word1.length();i++){
            dp[i][0]=i;
        }
        for(int j=0;j<=word2.length();j++){
            dp[0][j]=j;
        }
        //完善dp
        for(int i=1;i<=word1.length();i++){
            for(int j=1;j<=word2.length();j++){
                if(word1[i-1]==word2[j-1])dp[i][j]=dp[i-1][j-1];
                else{
                    dp[i][j]=min(dp[i-1][j]+1,dp[i][j-1]+1);//删除，这二者已经包含了dp[i-1][j-1]+2
                    dp[i][j]=min(dp[i][j],dp[i-1][j-1]+1);//替换
                }
            }
        }
        return dp[word1.length()][word2.length()];
    }
};
```

### 40. LeetCode647. 回文子串

```cpp
1.dp含义：
dp[i][j]：s[i,j]是否为回文串
2.转移方程：
if(s[i]==s[j])dp[i][j]=dp[i+1][j-1];
3.遍历顺序：
i：从下到上
j：从左到右
4.初始化dp：
//因为新的字符总是从两边一起加入，所以字符串长度每次递增2
//初始化一个字符的情况：1/3/5/7
//初始化两个字符的情况：2/4/6/8
//这样就可以把所有情况都覆盖到了，所以要初始化一个字符和两个字符的情况，因为他们都是递推dp的基础值
dp[i][i]=true;
if(s[i]==s[i+1])dp[i][i+1]=true;

class Solution {
public:
    int countSubstrings(string s) {
        //dp[i][j]：s[i,j]是否为回文串
        vector<vector<int>>dp(s.length(),vector<int>(s.length()));
        int res=0;//有多少个true，就有多少个回文子串
        //初始化dp
        for(int i=0;i<s.length();i++){
            dp[i][i]=true;
            res++;
        }
        for(int i=0;i<s.length()-1;i++){
            if(s[i]==s[i+1]){
                dp[i][i+1]=true;
                res++;
            }
        }
        //完善dp
        for(int i=s.length()-3;i>=0;i--){
            for(int j=i+2;j<s.length();j++){
                if(s[i]==s[j])dp[i][j]=dp[i+1][j-1];
                if(dp[i][j])res++;
            }
        }
        return res;
    }
};

双指针法：节省空间
class Solution {
public:
    int extend(string&s,int i,int j){
        int res=0;
        while(i>=0&&j<s.length()&&s[i]==s[j]){
            res++;
            i--;
            j++;
        }
        return res;
    }

    int countSubstrings(string s) {
        int result=0;
        for(int i=0;i<s.length();i++){
            result+=extend(s,i,i);//以s[i]为中心点
            result+=extend(s,i,i+1);//以s[i,i+1]为中心点
        }
        return result;
    }
};
```

### 41. LeetCode516. 最长回文子序列

```cpp
1.dp含义：
dp[i][j]：s[i,j]最长回文子序列的长度
2.转移方程：
if(s[i]==s[j])dp[i][j]=dp[i+1][j-1]+2;
else dp[i][j]=max(dp[i+1][j],dp[i][j-1]);//因为是序列，所以可以删除
3.遍历顺序：
i：从下到上
j：从左到右
4.初始化dp
dp[i][i]=1;
if(s[i]==s[i+1])dp[i][i+1]=2;
else dp[i][i+1]=1;

class Solution {
public:
    int longestPalindromeSubseq(string s) {
        //dp[i][j]：s[i,j]最长回文子序列的长度
        vector<vector<int>>dp(s.length(),vector<int>(s.length()));
        //初始化dp
        for(int i=0;i<s.length();i++){
            dp[i][i]=1;
        }
        for(int i=0;i<s.length()-1;i++){
            if(s[i]==s[i+1])dp[i][i+1]=2;
            else dp[i][i+1]=1;
        }
        //完善dp
        for(int i=s.length()-3;i>=0;i--){
            for(int j=i+2;j<s.length();j++){
                if(s[i]==s[j])dp[i][j]=dp[i+1][j-1]+2;
                else dp[i][j]=max(dp[i+1][j],dp[i][j-1]);
            }
        }
        return dp[0][s.length()-1];
    }
};
```

## 十、单调栈

在一维数组中，要寻找任一元素的右边或者左边第一个自己大或小的元素的位置，就可以想到用单调栈了。——代码随想录

### 1. LeetCode739. 每日温度

```cpp
思路：
1.创建辅助栈stk和结果数组answer
2.遍历温度数组，把第i天的温度temperatures[i]入栈，栈顶到栈底单调递减
3.如果当前温度大于栈顶温度，则不断出栈，并把相隔天数(i-stk.top())写入到结果集中

class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        //结果集，因为对于后续没有温度更高的，结果是0，所以默认都为0
        vector<int>answer(temperatures.size());
        //辅助栈
        stack<int>stk;
        //遍历数组
        for(int i=0;i<temperatures.size();i++){
            while(!stk.empty()&&temperatures[i]>temperatures[stk.top()]){
                answer[stk.top()]=i-stk.top();
                stk.pop();
            }
            stk.push(i);//存入下标，既能知道天数，又可以知道温度
        }
        return answer;
    }
};

时间复杂度：O(n)，对于每个温度数组的元素下标，只有入栈和出栈各一次操作;
空间复杂度：O(n)，需要维护一个辅助栈
```

### 2. LeetCode496. 下一个更大元素 I

```cpp
思路：
1.创建辅助栈stk，结果集ans，映射表存储nums1中各元素和下标的对应
2.初始化结果集为-1
3.建立好nums1的映射关系
4.遍历nums2，找到nums1中的元素在nums2右边第一个比自己大的值，放入结果集中

卡点：因为nums1和nums2是元素方面的联系，所以可以通过哈希表来找到当前遍历元素在nums1中的对应下标

class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
        //结果集，默认-1
        vector<int>ans(nums1.size(),-1);
        //辅助栈
        stack<int>stk;//栈底到栈顶：单调递减
        //映射表
        unordered_map<int,int>recorded;
        //建立映射关系
        for(int i=0;i<nums1.size();i++){
            recorded[nums1[i]]=i;
        }
        //遍历nums2
        for(int i=0;i<nums2.size();i++){
            while(!stk.empty()&&nums2[i]>nums2[stk.top()]){
                if(recorded.count(nums2[stk.top()])>0){
                    ans[recorded[nums2[stk.top()]]]=nums2[i];
                }
                stk.pop();
            }
            stk.push(i);
        }
        return ans;
    }
};
```

### 3. LeetCode503. 下一个更大元素 II

```cpp
class Solution {
public:
    vector<int> nextGreaterElements(vector<int>& nums) {
        vector<int>ans(nums.size(),-1);//结果集，默认-1
        stack<int>stk;//辅助栈，栈底到栈顶：单调递减
        
        //当第二次遍历到最大值时，就可以结束循环了
        //第一次遍历到最大值：后面的元素还没遍历到，需要继续处理
        //第二次遍历到最大值时，最坏情况下，后面的所有元素右边比自己更大的数刚好就是最大值
        //由于所有位置的元素都有可能是最大值，所以把所有元素都遍历两次，nums的所有元素肯定都处理好了
        for(int i=0;i<nums.size()*2-1;i++){
            while(!stk.empty()&&nums[i%nums.size()]>nums[stk.top()]){
                ans[stk.top()]=nums[i%nums.size()];
                stk.pop();
            }
            stk.push(i%nums.size());
        }
        return ans;
    }
};
```

### 4. LeetCode42. 接雨水

```cpp
1.双指针：按列计算雨水面积
当前列的雨水面积=min(左边最大高度,右边最大高度)-当前列高度，因为宽度默认是1，取最小值是由于短板效应
因为我们只需要每一列柱子左边和右边的高度的最小值，所以可以用maxLeft和maxRight来记录，这样就避免了重复计算左右两边最大高度的操作
class Solution {
public:
    int trap(vector<int>& height) {
        int res=0;//结果
        vector<int>maxLeft(height.size());//左边最大高度
        vector<int>maxRight(height.size());//右边最大高度
        
        //动态规划
        maxLeft[0]=height[0];
        for(int i=1;i<height.size();i++){
            maxLeft[i]=max(maxLeft[i-1],height[i]);
        }
        maxRight[height.size()-1]=height[height.size()-1];
        for(int i=height.size()-2;i>=0;i--){
            maxRight[i]=max(maxRight[i+1],height[i]);
        }

        //计算雨水面积
        for(int i=1;i<height.size()-1;i++){
            res+=min(maxLeft[i],maxRight[i])-height[i];
        }
        return res;
    }
};

2.单调栈：按行计算雨水面积
辅助栈：栈底到栈顶（单调递减）
（1）当前高度大于栈顶对应高度：一旦出现更高的柱子，就出现凹槽了，需要计算雨水面积
（2）当前高度等于栈顶对应高度：栈顶元素弹出，当前元素入栈，因为我们需要相同高度的柱子的最右边那个来计算雨水面积
（3）当前高度小于栈顶对应高度：直接入栈

出现凹槽时，我们需要三个元素来计算面积：栈顶，栈顶的下一个元素，准备入栈的元素
栈顶：凹槽底下的柱子
栈顶的下一个元素：凹槽左边的柱子
准备入栈的元素：凹槽右边的柱子

class Solution {
public:
    int trap(vector<int>& height) {
        stack<int>stk;//辅助栈
        int res=0;//结果
        stk.push(0);//先让0入栈，因为已进入循环就要和stk.top()比较，所以先让0入栈
        for(int i=1;i<height.size();i++){
            if(height[i]<height[stk.top()])stk.push(i);
            else if(height[i]==height[stk.top()]){
                stk.pop();
                stk.push(i);
            }
            else{
                while(!stk.empty()&&height[i]>height[stk.top()]){
                    int mid=stk.top();
                    stk.pop();
                    if(!stk.empty()){
                        int h=min(height[stk.top()],height[i])-height[mid];
                        int w=i-stk.top()-1;
                        res+=h*w;
                    }
                }
                stk.push(i);
            }
        }
        return res;
    }
};

简化代码：
等于和小于的操作类似，而等于的更新栈顶元素不影响计算。如果栈顶和栈顶的下一个元素相同，则在大于的操作中h=0，不影响结果。等到栈顶下一个相等高度的柱子做凹槽底部时，h是正常的，w也不受影响，因为w只和stk.top()有关，和弹出的柱子无关
class Solution {
public:
    int trap(vector<int>& height) {
        stack<int>stk;//辅助栈
        int res=0;//结果
        stk.push(0);
        for(int i=1;i<height.size();i++){
            while(!stk.empty()&&height[i]>height[stk.top()]){
                int mid=stk.top();
                stk.pop();
                if(!stk.empty()){
                    int h=min(height[stk.top()],height[i])-height[mid];
                    int w=i-stk.top()-1;
                    res+=h*w;
                }
            }
            stk.push(i);
        }
        return res;
    }
};

卡点：没有考虑凹槽底部柱子高度，导致计算了很多多余的面积
```

### 5. LeetCode84. 柱状图中最大的矩形

```cpp
1.双指针：
leftFirst[i]：0~i-1第一个比柱子i低的下标
rightFirst[i]：i+1~height.size()-1第一个比i低的下标
把leftFirst[0]初始化为-1，把rightFirst初始化为height.size()，能够最低的柱子计算出正确的矩形

class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int res=0;
        vector<int>leftFirst(heights.size());
        vector<int>rightFirst(heights.size());
        leftFirst[0]=-1;
        for(int i=1;i<heights.size();i++){
            int t=i-1;
            while(t>=0&&heights[i]<=heights[t])t=leftFirst[t];
            leftFirst[i]=t;   
        }
        rightFirst[heights.size()-1]=heights.size();
        for(int i=heights.size()-2;i>=0;i--){
            int t=i+1;
            while(t<heights.size()&&heights[i]<heights[t])t=rightFirst[t];
            rightFirst[i]=t;
        }
        
        for(int i=0;i<heights.size();i++){
            int sum=heights[i]*(rightFirst[i]-leftFirst[i]-1);
            res=max(res,sum);
        }
        return res;
    }
};

本题和接雨水的唯一区别就是本题是找左右两边第一个小于的柱子
2.单调栈：
栈底到栈顶：单调递增，因为需要找的是左右两边第一个小于自己的柱子
height[i]>height[stk.top()]：入栈
height[i]==height[stk.top()]：更新栈顶
height[i]<height[stk.top()]：计算最大矩形

在数组前后都插入0，避免原始数组单调递增而导致没有计算矩形就结束循环了
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        stack<int>stk;
        int res=0;
        heights.insert(heights.begin(),0);
        heights.push_back(0);
        stk.push(0);

        for(int i=1;i<heights.size();i++){
            if(heights[i]>heights[stk.top()])stk.push(i);
            else if(heights[i]==heights[stk.top()])stk.top()=i;
            else{
                while(!stk.empty()&&heights[i]<heights[stk.top()]){
                    int h=heights[stk.top()];
                    stk.pop();
                    if(!stk.empty()){
                        int w=i-stk.top()-1;
                        res=max(res,h*w);
                    }
                }
                stk.push(i);
            }
        }
        return res;
    }
};

精简代码：
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        stack<int>stk;
        int res=0;
        heights.insert(heights.begin(),0);
        heights.push_back(0);
        stk.push(0);

        for(int i=1;i<heights.size();i++){
            while(!stk.empty()&&heights[i]<heights[stk.top()]){
                int h=heights[stk.top()];
                stk.pop();
                if(!stk.empty()){
                    int w=i-stk.top()-1;
                    res=max(res,h*w);
                }
            }
            stk.push(i);
        }
        return res;
    }
};
```

## 十一、数组额外题目

### 1.LeetCode1365. 有多少小于当前数字的数字

```cpp
思路：
1.使数组排序，从小到大
2.创建哈希表记录每个元素最左的下标

class Solution {
public:
    vector<int> smallerNumbersThanCurrent(vector<int>& nums) {
        //结果集合
        vector<int>vec=nums;
        //排序数组
        sort(vec.begin(),vec.end());
        //哈希表记录下标
        map<int,int>recorded;
        for(int i=nums.size()-1;i>=0;i--){//从右到左遍历数组，这样记录的就是最左边的下标
            recorded[vec[i]]=i;
        }
        //处理结果集
        for(int i=0;i<nums.size();i++){
            vec[i]=recorded[nums[i]];
        }
        return vec;
    }
};
```

### 2.LeetCode941. 有效的山脉数组

```cpp
思路：
1.利用bool变量up判断现在是否在上升期
2.当第一次下降时，把up置为false，且之后都必须为false。如果下降或平地之后还有上升过程，说明就是无效的山脉

方法一：状态法
class Solution {
public:
    bool validMountainArray(vector<int>& arr) {
        //山脉数组先递增，所以up先置为true
        bool up=true;
        int peak=0;
        //遍历山脉数组
        for(int i=1;i<arr.size();i++){
            if(arr[i-1]==arr[i]){//不是严格递增或递减，有平地，无效山脉
                return false;
            }
            else if(arr[i-1]<arr[i]){
                if(!up){//在非上升过程中遇到上升过程，无效山脉
                    return false;
                }
                peak=i;//记录顶点坐标
            }else{
                up=false;
            }
        }
        //能出循环，顶点坐标不是0且最终趋势是下降(即up==false)，就说明是有效的山脉数组
        return peak!=0&&!up;
    }
};

方法二：双指针法
class Solution {
public:
    bool validMountainArray(vector<int>& arr) {
        //左指针
        int left=0;
        //右指针
        int right=arr.size()-1;
        //注意不要越界
        while(left<arr.size()-1&&arr[left]<arr[left+1])left++;
        while(right>0&&arr[right]<arr[right-1])right--;

        //left和right必须都指向顶点，且山脉不是单纯上坡或下坡
        return left==right&&left!=0&&right!=arr.size()-1;
    }
};
```

### 3. LeetCode1207. 独一无二的出现次数

```cpp
class Solution {
public:
    bool uniqueOccurrences(vector<int>& arr) {
        unordered_map<int,int>recorded;//记录每个元素出现的次数
        set<int>count;//记录出现过的词频
        
        for(int i=0;i<arr.size();i++){
            recorded[arr[i]]++;
        }

        for(auto &p:recorded){
            if(count.find(p.second)==count.end()){//如果count不曾出现当前词频
                count.insert(p.second);
            }else{
                return false;
            }
        }
        
        return true;
    }
};
```

### 4. LeetCode283. 移动零

```cpp
双指针：
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        if(nums.size()==1)return;
        int left=0;//左边第一个为0的
        int right=0;//left右边第一个不为0的
        while(left<nums.size()&&right<nums.size()){
            while(left<nums.size()&&nums[left]!=0)left++;//最好把越界条件放在最前面
            right=left;
            while(right<nums.size()&&nums[right]==0)right++;//最好把越界条件放在最前面
            if(left<nums.size()&&right<nums.size())swap(nums[left],nums[right]);
        }
    }
};

class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int slowIndex=0;
        for(int fastIndex=0;fastIndex<nums.size();fastIndex++){
            if(nums[fastIndex]!=0){//先把不为0的值放在数组左边
                nums[slowIndex++]=nums[fastIndex];
            }
        }

        while(slowIndex<nums.size()){//后面赋0
            nums[slowIndex++]=0;
        }
    }
};
```

### 5. LeetCode189. 轮转数组

```cpp
思路：
假设以原数组的状态（即所有元素相对顺序一致）定义为有序，此种状态下，nums从0到nums.size()-1都是有序的
轮转后的数组：
1. 0~k-1有序
2. k~nums.size()-1有序
3. [0,k-1]和[k,nums.size()-1]各自有序，但二者在一起不有序

先将整体反转，这样就使得右边的能够到左边，左边的能够到右边，再以k为界限让两边局部反转，使得两边局部有序
注意：必须先让整体反转使各元素到了正确的区域，才能够局部反转使得局部有序

class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        k=k%nums.size();
        reverse(nums.begin(),nums.end());//反转整个数组
        reverse(nums.begin(),nums.begin()+k);//反转前k个元素[0,k-1]
        reverse(nums.begin()+k,nums.end());//反转后续所有元素[k,nums.size()-1]
    }
};
```

### 6. LeetCode724. 寻找数组的中心下标

```cpp
class Solution {
public:
    int pivotIndex(vector<int>& nums) {
        int sum=0;//数组总和
        for(auto&val:nums){
            sum+=val;
        }
        int leftSum=0;//中心下标左边元素和
        for(int i=0;i<nums.size();i++){
            //中心左边右边元素和rightSum=sum-leftSum-nums[i]，注意也要减去中心下标的值
            if(leftSum==sum-leftSum-nums[i])return i;
            leftSum+=nums[i];
        }
        return -1;
    }
};
```

### 7. LeetCode34. 在排序数组中查找元素的第一个和最后一个位置

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        if(nums.size()==1){
            vector<int>vec0(2,0);
            vector<int>vec1(2,-1);
            return (nums[0]==target)?vec0:vec1;
        }
        int targetIndex=-1;
        int left=0;
        int right=nums.size()-1;
        //二分查找
        while(left<=right){
            int mid=left+((right-left)>>1);
            if(nums[mid]==target){
                targetIndex=mid;
                break;
            }else if(nums[mid]>target){
                right=mid-1;
            }else{
                left=mid+1;
            }
        }
        if(targetIndex==-1){//说明未找到目标值
            return{-1,-1};
        }else{
            left=targetIndex;
            right=targetIndex;
while(left>=0&&nums[left]==target)left--;//找到左边边界
            while(right<=nums.size()-1&&nums[right]==target)right++;//找到右边边界
        }
        return {left+1,right-1};
    }
};
```

### 8. LeetCode922. 按奇偶排序数组 II

```cpp
class Solution {
public:
    vector<int> sortArrayByParityII(vector<int>& nums) {
        int evenIndex=0;//偶数下标
        int oddIndex=1;//奇数下标
        while(oddIndex<nums.size()&&evenIndex<nums.size()){
            while(oddIndex<nums.size()&&oddIndex%2==1&&nums[oddIndex]%2==1)oddIndex+=2;//元素合理就下一个奇数元素
            while(evenIndex<nums.size()&&evenIndex%2==0&&nums[evenIndex]%2==0)evenIndex+=2;//元素合理就下一个偶数元素
            if(oddIndex<nums.size()&&evenIndex<nums.size())swap(nums[oddIndex],nums[evenIndex]);//必须保证不越界
        }
        return nums;
    }
};
```

### 

```cpp
1.[left,right]，闭区间
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        //排序数组、O(logn) -> 二分查找
        int left=0;
        int right=nums.size()-1;
        while(left<=right){//因为target在[left,right]区间，所以left==right时，区间仍然有效
            int mid=left+((right-left)>>1);
            if(nums[mid]==target){
                return mid;
            }else if(nums[mid]<target){
                left=mid+1;
            }else{
                right=mid-1;
            }
        }
        // 分别处理如下四种情况
        //目标值等于数组中某一个元素  return middle;
        // 目标值在数组所有元素之前  [0, -1]，0 = -1 + 1 = right+1
        // 目标值插入数组中的位置 [left, right]，return  right + 1，因为mid是向下取整的
        // 目标值在数组所有元素之后的情况 [left, right]， 因为是右闭区间，所以 return right + 1
        return right+1;
    }
};

2.[left,right)，左闭右开区间
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int left=0;
        int right=nums.size();
        while(left<right){
            int mid=left+((right-left)>>1);
            if(nums[mid]==target){
                return mid;
            }else if(nums[mid]<target){
                left=mid+1;
            }else{
                right=mid;
            }
        }
        return right;
    }
};
```

## 十二、链表额外题目

### 1. LeetCode24. 两两交换链表中的节点

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        if(head==NULL||head->next==NULL){
            return head;
        }
        ListNode*node1=head;//交换节点1
        ListNode*node2=head->next;//交换节点2
        ListNode*pre=NULL;//前驱节点
        ListNode*next=node2->next;//后续节点，防止丢失链表
        while(node1&&node2){//node1和node2必须都不为空，才能交换，因为空指针无next
            node1->next=next;
            node2->next=node1;
            if(pre)pre->next=node2;//第一对节点无前驱节点
            else head=node2;//pre是空，说明在交换第一对节点，头指针更换为node2
            //更新节点信息
            pre=node1;
            node1=next;
            if(node1)node2=node1->next;
            if(node2)next=node2->next;
        }
        return head;
    }
};
```

**虽然有重复的题，但是每次做的感觉都不一样，都有不同的感悟**

### 2. LeetCode234. 回文链表

```cpp
方法一：
数组/字符串 模拟
提高效率：提前确定数组和字符串的长度，这样就不用resize

方法二：
反转后半部分字符串（快慢指针）
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        if(head==NULL||head->next==NULL){
            return true;
        }
        ListNode*slow=head;
        ListNode*fast=head;
        ListNode*pre=slow;//slow的前置节点
        //fast一定放在fast->next前面，当fast==NULL时，fast->next有异常，因为空指针没有next
        //并且因为是‘与’，所以判断fast为false后，就不会判断后面的了
        while(fast&&fast->next){
            pre=slow;
            slow=slow->next;
            fast=fast->next->next;
        }
        //如果链表长度是奇数，后半部分链表长度多1一个
        //长度奇数：fast指向最后一个节点
        //长度偶数：fast==NULL
        pre->next=NULL;//断开
        slow=reverseList(slow);//从slow开始的节点都是后半部分链表的

        fast=head;
        while(fast&&slow){
            if(fast->val!=slow->val)return false;
            fast=fast->next;
            slow=slow->next;
        }
        return true;
    }

        ListNode* reverseList(ListNode*head){//反转链表并返回新链表头
        ListNode*reverseHead=new ListNode(-1);
        reverseHead->next=NULL;
        ListNode*cur=head;
        ListNode*next=cur->next;
        while(cur){
            cur->next=reverseHead->next;
            reverseHead->next=cur;
            cur=next;
            //cur可能在等于next之后变为空，但不能在next为空时就停止循环，因为cur不为空，还有节点要反转
            if(cur)next=cur->next;
        }
        return reverseHead->next;
    }
};
```

### 3. LeetCode143. 重排链表

```cpp
方法一：数组模拟(双指针)
class Solution {
public:
    void reorderList(ListNode* head) {
        //特殊情况
        if(head->next==NULL||head->next->next==NULL)return;

        //计算链表长度
        int len=0;//链表长
        ListNode*cur=head;//临时链表头
        while(cur){
            len++;
            cur=cur->next;
        }
        vector<ListNode*>nodeVec(len,NULL);

        //将链表节点按序放入集合中
        cur=head;
        for(int i=0;i<len;i++){
            nodeVec[i]=cur;
            cur=cur->next;
        }

        //重排链表
        cur=head;
        //双指针
        int i=1;
        int j=len-1;
        int count=1;//计数
        while(i<=j){
            if((count&1)==1){
                cur->next=nodeVec[j--];
            }else{
                cur->next=nodeVec[i++];
            }
            cur=cur->next;
            count++;
        }
        //最后cur指向重排链表的最后一个节点
        cur->next=NULL;
    }
};

方法二：双向队列
通过双向队列一前一后弹出完成重排链表

方法三：切割并反转链表
class Solution {
public:
    void reorderList(ListNode* head) {
        if(head->next==nullptr||head->next->next==nullptr)return;
        
        //快慢指针
        ListNode*slow=head;
        ListNode*fast=head;
        ListNode*pre=nullptr;//slow的前驱节点
        while(fast&&fast->next){
            pre=slow;
            slow=slow->next;
            fast=fast->next->next;
        }
        //切割链表
        pre->next=nullptr;
        //出循环后，slow为后半段链表的链表头
        slow=reverseList(slow);
        fast=head->next;

        //重排链表
        ListNode*cur=head;
        int count=1;
        while(slow&&fast){
            if((count&1)==1){
                    cur->next=slow;
                    slow=slow->next;
            }else{
                    cur->next=fast;
                    fast=fast->next;
            }
            cur=cur->next;
            count++;
        }
        //slow链表的长度大于等于fast链表的长度
        cur->next=slow;
    }
    /*
    误区：
    while(slow||fast){
            if((count&1)==1){
                if(slow){
                    cur->next=slow;
                    slow=slow->next;
                }
            }else{
                if(fast){
                    cur->next=fast;
                    fast=fast->next;
                }
            }
            cur=cur->next;
            count++;
     }
     这样写会陷入死循环，因为即使fast为空时，按理说cur的next没有指向正确的方向，但是cur仍向前移动了，导致cur不断在slow链表里
     移动，cur最后不是在重排链表，而是在延长slow链表了，slow一直不为空，陷入死循环
    */

    ListNode*reverseList(ListNode*head){
        ListNode*reverseHead=new ListNode(-1);
        reverseHead->next=NULL;
        ListNode*cur=head;
        ListNode*next=cur->next;
        while(cur){
            cur->next=reverseHead->next;
            reverseHead->next=cur;
            cur=next;
            if(cur)next=cur->next;
        }
        return reverseHead->next;
    }
};
```

### 4. LeetCode141. 环形链表

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        if(head==nullptr||head->next==nullptr)return false;
        //快慢指针
        ListNode*fast=head->next->next;
        ListNode*slow=head;
        while(fast&&fast->next){
            if(fast==slow){//相遇，说明有环
                return true;
            }
            fast=fast->next->next;
            slow=slow->next;
        }
        //退出循环，说明链表无环
        return false;
    }
};
```

### 5. 面试题 02.07. 链表相交

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode*A=headA;
        ListNode*B=headB;
        while(A||B){
            if(A==B)return A;
            A=A?A->next:headB;
            B=B?B->next:headA;
            //这样先到尽头的指针可以先在长链表提前补上位移差
        }
        return nullptr;
    }
};
```

## 十三、哈希表额外题目

### 1. LeetCode205. 同构字符串

![](https://github.com/Jomocool/DataStructure_Algorithm/blob/main/LeetCode-img/16.png)

```cpp
思路：
1.创建映射表recorded，记录字符串s中各字符和字符串t的映射关系
2.遍历字符串s，如果已有映射关系(即已经加入映射表)，看是否和当前t字符串的字符映射；如果没有映射关系，判断当前t[i]是否已被映射，加入映射表，并确定好映射关系

class Solution {
public:
    bool isIsomorphic(string s, string t) {
        unordered_map<char,char>sTot;//映射表(x->t)
        unordered_map<char,char>tTos;//映射表(t->x)
        for(int i=0;i<s.length();i++){
            if(sTot.find(s[i])!=sTot.end()){//有映射关系
                if(sTot[s[i]]!=t[i])return false;
            }else{
                if(tTos.find(t[i])!=tTos.end()){//当前t[i]已被映射
                    if(tTos[t[i]]!=s[i])return false;//如果当前被映射t[i]的来源不是s[i]，说明被重复映射，无效
                }else{
                    //既无s[i]->t[i]，也无t[i]->s[i]
                    sTot[s[i]]=t[i];
                    tTos[t[i]]=s[i];
                }
            }
        }
        return true;
    }
};
本题需要注意的点是既要知道s->t，也要知道t->s，所以需要两个映射表
```

### 2. LeetCode1002. 查找共用字符

```cpp
思路：
1.创建数组模拟哈希表统计每个字符串词频，int recorded[26]={0};
2.创建数组hash记录整个字符串属猪的公共字符词频，取最小值

注意，想要整型数组 全部初始化为1的时候不能粗暴的设置为 
int a[5] = {1};    //[1, 0, 0, 0, 0]
因为 数组初始化列表中的元素个数小于指定的数组长度时， 不足的元素以默认值填补。

class Solution {
public:
    vector<string> commonChars(vector<string>& words) {
        vector<int>recorded(26,0);//初始化为0
        vector<int>hash(26,INT_MAX);//整体字符串数组公共字符词频，注意不能初始化为0，否则结果集永远都是空
        vector<string>res;//结果集
        for(int i=0;i<words.size();i++){
            for(int j=0;j<words[i].length();j++){
                recorded[words[i][j]-'a']++;
            }
            for(int k=0;k<26;k++){
                hash[k]=min(hash[k],recorded[k]);
                recorded[k]=0;//清零，不影响下一个字符串词频统计
            }
        }
        for(int i=0;i<26;i++){
            while(hash[i]!=0){
                string s(1,'a'+i);//转成字符串
                res.push_back(s);
                hash[i]--;
            }
        }
        return res;
    }
};

出现频率最小的字符才是决定性字符，要注意重复字符也要算入，所以记录词频的就是记录每个字符出现的次数，刚好把重复字符考虑到了
```

## 十四、字符串额外题目

### 1. LeetCode925. 长按键入

```cpp
思路：
对于typed[i]：
1.匹配：typed[i]==name[index]
2.长按字符：typed[i]!=name[index],but typed[i]==typed[i-1]
3.不匹配：typed[i]!=name[index] && typed[i]!=typed[i-1]
遍历typed，用index记录name当前匹配下标，如果typed[i]能够和name[index]匹配，同时后移；如果无法匹配且typed[i]==typed[i-1]，说明是长按字符，否则无效

class Solution {
public:
    bool isLongPressedName(string name, string typed) {
        int index=0;//记录name当前匹配下标
        for(int i=0;i<typed.length();i++){
            if(name[index]==typed[i]){
                index++;
                continue;
            }else{
                if(i>0&&typed[i]==typed[i-1]){//长按字符
                    continue;
                }else{
                    return false;
                }
            }
        }
        return index==name.length();//看name的全部字符是否都被匹配上了，因为只有被匹配了，index才会后移
    }
};
```

### 2. LeetCode844. 比较含退格的字符串

```cpp
思路：
创建新的字符串newS和newT，遇到#就pop_back（注意字符串不能为空，否则有异常），否则push_back
字符串模拟栈

class Solution {
public:
    string getString(string& s){
        string newStr="";
        for(int i=0;i<s.length();i++){
            if(s[i]!='#')newStr.push_back(s[i]);
            else if(!newStr.empty())newStr.pop_back();
        }
        return newStr;
    }

    bool backspaceCompare(string s, string t) {
        return getString(s)==getString(t);
    }
};

双指针法：从后向前遍历，遇到#就跳过
class Solution {
public:
    bool backspaceCompare(string s, string t) {
        int sSkipNum=0;//遍历s过程中，s遇到#的个数
        int tSkipNum=0;//遍历t过程中，t遇到#的个数
        int sIndex=s.length()-1;
        int tIndex=t.length()-1;
        while(1){
            //处理'#'
            while(sIndex>=0){
                if(s[sIndex]=='#')sSkipNum++;
                else{
                    //遇到不是#的字符，且sSkipNum>>0，说明在#的作用域中，是要删去的
                    if(sSkipNum>0)sSkipNum--;//说明当前还在#的作用域中，跳过
                    else break;//不能够再删除了，得退出循环去匹配了
                }
                sIndex--;
            }
            while(tIndex>=0){
                if(t[tIndex]=='#')tSkipNum++;
                else{
                    if(tSkipNum>0)tSkipNum--;
                    else break;
                }
                tIndex--;
            }

            //退出循环有两种可能性
            //1.sIndex/tIndex<0
            //2.#已经处理完了，break

            //为避免异常，先判断下标是否有效
            if(sIndex<0||tIndex<0)break;//s或t，遍历到头了

            //匹配字符
            if(s[sIndex]!=t[tIndex])return false;

            //还没退出循环，说明当前字符匹配成功，继续下一轮处理#
            sIndex--;
            tIndex--;
        }

        //退出整体循环后，如果sIndex和tIndex不是同时等于-1，说明其中有一个先遍历到头了，不匹配
        return sIndex==-1&&tIndex==-1;
    }
};
```

## 十五、二叉树额外题目

### 1. LeetCode129. 求根节点到叶节点数字之和

```cpp
回溯法：
注意本题不可以递归到空节点，因为空节点没有值，没有意义
class Solution {
public:
    int res=0;//定义全局变量记录最终结果
    int path=0;//记录根节点到每个叶子节点的值

    void backTracking(TreeNode*node){
        path=path*10+node->val;
        //当前节点是叶子节点
        if(node->left==nullptr&&node->right==nullptr){
            res+=path;//只有在叶子节点时才需要把path附加给res
        }
        //不能走到空指针
        if(node->left)backTracking(node->left);
        if(node->right)backTracking(node->right);
        path/=10;//回溯
    }

    int sumNumbers(TreeNode* root) {
        backTracking(root);
        return res;
    }
};
```

