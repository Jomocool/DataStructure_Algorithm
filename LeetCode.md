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

