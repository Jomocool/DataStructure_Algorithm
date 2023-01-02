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



