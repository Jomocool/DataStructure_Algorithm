# 剑指offer

## 第一天

### 剑指 Offer 09. 用两个栈实现队列

```c++
class CQueue {
private:
    stack<int>stk_in,stk_out;//stk_in:存数据;stk_out:取数据
    void inToOut(){
        while(!stk_in.empty()){//将stk_in中存入的数据放入stk_out
            stk_out.push(stk_in.top());
            stk_in.pop();
        }
    }
public:
    CQueue(){}
    
    void appendTail(int value) {
        stk_in.push(value);
    }
    
    int deleteHead() {
        if(stk_out.empty()){
            if(stk_in.empty()){
                return -1;//如果stk_out和stk_in都为空，说明数据已全被取出，此时才可以返回-1
            }
            inToOut();//stk_in中还有数据，将其中的数据拷入stk_out
        }
        int val=stk_out.top();
        stk_out.pop();
        return val;
    }
};
```

### 剑指 Offer 30. 包含min函数的栈

```c++
class MinStack {
private:
    stack<int>stkA,stkB;//stkA用来储存所有元素；sktB用来储存stkA中的部分元素，这些元素保持非严格降序
public:
    /** initialize your data structure here. */
    MinStack() {}
    
    void push(int x) {
        stkA.push(x);//只要有数据就插入stkA
        if(stkB.empty()){
            stkB.push(x);//如果stkB为空，就插入
        }else if(!stkB.empty()&&x<=stkB.top()){
            stkB.push(x);//如果x小于stkB的栈顶元素，就插入
        }
        //这里可能很多人会写成
        /*if(x<=stkB.top()||stkB.empty()){
            stkB.push(x);
        }*/
        //但提交代码的时候会发现这样写会报错，地址冲突了。
        //原因是，当stkB为空时，我们去判断条件时，还会访问到stkB的栈顶，此时属于越界访问了，因此起冲突
        //如果换成上面的代码，在else if语句判断时，但判断出stkB为空时，因为是‘与’，就直接是false了，就		   不会再访问到stkB的栈顶然后去和x比较。
    }
    
    void pop() {
        if(stkA.top()==stkB.top()){//如果stkA和stkB栈顶元素相同，则同时删去，保持元素一致性
            stkA.pop();
            stkB.pop();
        }else{
            stkA.pop();
        }
        //插入stkB中的元素在stkB中的顺序必定是和在stkA中的顺序一致
    }
    
    int top() {
        return stkA.top();
    }
    
    int min() {
        return stkB.top();
    }
};
```



## 第二天

### 剑指 Offer 06. 从尾到头打印链表

```c++
class Solution {
public:
    vector<int> reversePrint(ListNode* head) {
        vector<int>vec1;//从头到尾接收链表
        vector<int>vec2;//存储逆序链表的值
        while(head){
            vec1.push_back(head->val);
            head=head->next;
        }
        for(int i=vec1.size()-1;i>=0;i--){
            vec2.push_back(vec1[i]);
        }
        return vec2;
    }
};
```

### 剑指 Offer 24. 反转链表

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        //处理边界情况
        if(!head||head->next==NULL){//如果无节点或只有一个节点，直接返回head即可
            return head;
        }
        ListNode*reverseNode=new ListNode(-1);//前驱节点，防止头指针丢失
        reverseNode->next=NULL;
        ListNode*cur=head;//遍历链表
        while(cur){
            ListNode*next=cur->next;//提前储存好cur在链表中的下一个节点，不然会丢失
            cur->next=reverseNode->next;
            reverseNode->next=cur;
            cur=next;//后移一个节点
        }
        return reverseNode->next;//注意是返回反转链表头节点的下一个节点
    }
};
```

### 剑指 Offer 35. 复杂链表的复制

![image text](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/Jz_Offer-img/1.png)

```c++
方法一：哈希表
class Solution {
public:
    Node* copyRandomList(Node* head) {
        if(!head)return head;//如果是空指针，直接返回
        unordered_map<Node*,Node*>dic;//创建哈希表
        Node*cur=head;//cur用于遍历链表
        while(cur){
            dic[cur]=new Node(cur->val);
            cur=cur->next;
        }
        cur=head;//回溯
        while(cur){
            dic[cur]->next=dic[cur->next];
            dic[cur]->random=dic[cur->random];
            cur=cur->next;
        }
        return dic[head];
    }
};

方法二：拼接+拆分
class Solution {
public:
    Node* copyRandomList(Node* head) {
        if(!head)return head;
        Node*cur=head;
        while(cur){
            Node*next=cur->next;
            cur->next=new Node(cur->val);
            cur->next->next=next;
            cur=next;
        }
        cur=head;
        while(cur){
            if(cur->random){//注意random指针可能指向NULL
               cur->next->random=cur->random->next;//复制节点的任何指针都要指向复制节点，而不能指向原节点
            }
            cur=cur->next->next;
        }
        cur=head->next;
        Node*pre=head;
        Node*res=head->next;//记录合成链表中的head->next，因为分开后的链表中head指针指的是原链表了
        while(cur->next){
            pre->next=pre->next->next;
            cur->next=cur->next->next;
            pre=pre->next;
            cur=cur->next;
        }
        pre->next=NULL;//单独处理原链表尾部，因为此时还未指向NULL，因为不能修改原链表，所以置空
        return res;
    }
};
```



## 第三天

### 剑指 Offer 05. 替换空格

```c++
class Solution {
public:
    string replaceSpace(string s) {
        int count=0;//记录空格数
        for(auto&c:s){
            if(c==' ')count++;
        }
        int len=s.size();//记录s的长度
        s.resize(len+2*count);//在原来字符串的基础上进行扩充
        for(int i=len-1,j=s.length()-1;i!=j;i--,j--){
            if(s[i]!=' '){//如果不是空格的话就直接copy
                s[j]=s[i];
            }else{//如果是空格的话用"%20"覆盖，所以j也要向前走两格
                s[j-2]='%';
                s[j-1]='2';
                s[j]='0';
                j-=2;
            }
        }
        return s;
    }
};
```

### 剑指 Offer 58 - II. 左旋转字符串

```c++
方法一：字符串切片
class Solution {
public:
    string reverseLeftWords(string s, int n) {
        s.append(s.substr(0,n));
        return s.substr(n);
    }
};

方法二：遍历字符串
class Solution {
public:
    string reverseLeftWords(string s, int n) {
        string res="";
        int len=s.length();
        for(int i=n;i<s.length()+n;i++){
            res+=s[i%len];//利用取模可以取到s的前n个字符
        }
        return res;
    }
};
```



## 第四天

### 剑指 Offer 03. 数组中重复的数字

```c++
方法一：哈希表
class Solution {
public:
    int findRepeatNumber(vector<int>& nums) {
        unordered_map<int,int>hs;//记录nums中各数出现的次数
        for(int i=0;i<nums.size();i++){//遍历nums
            if(hs.find(nums[i])==hs.end()){
                hs[nums[i]]=1;//如果在哈希表中没出现过，那就在哈希表中新建
            }
            else{
               return nums[i];//说明该数出现过一次了，重复了，直接返回即可
            }
        }
        return -1;
    }
};

class Solution {
public:
    int findRepeatNumber(vector<int>& nums) {
        unordered_map<int,bool>map;//创建哈希表
        for(auto&num:nums){
            if(map[num])return num;//如果出现过，返回该数
            map[num]=true;//没出现过，就置为true
        }
        return -1;
    }
};

方法二：交换
思路：
    注意条件（在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内），说明索引和值的对应关系是一对多的。所以我们遍历nums，将前面的索引与值通过swap(nums[i],nums[nums[i]])一一交换之后，如果后面再出现重复的数字，那么就会满足条件(nums[i]==nums[nums[i]])，这时就出现第一个重复的数字了。
class Solution {
public:
    int findRepeatNumber(vector<int>& nums) {
        int i=0;//索引，用于遍历nums
        while(i<nums.size()){
            if(nums[i]==i){
                i+=1;//如果值和索引相同，就让i后移一位
                continue;
            }
            //如果值和索引不同，并且此时的值和以该值为索引所对应的值相等的话，说明该值重复，应当返回
            if(nums[i]==nums[nums[i]])return nums[i];
            swap(nums[i],nums[nums[i]]);//上述条件都不满足的话，就将该值交换到以该值为索引的位置上
        }
        return -1;
    }
};

方法三：集中法
class Solution {
public:
    int findRepeatNumber(vector<int>& nums) {
        sort(nums.begin(),nums.end());//先让nums排好序，这样相同的数会集中在一起
        for(int i=0;i<nums.size()-1;i++){
            if(nums[i]==nums[i+1])return nums[i];//直接遍历判断相邻的数是否重复即可
        }
        return -1;
    }
};
```

### 剑指 Offer 53 - I. 在排序数组中查找数字 I

```c++
方法一：二分查找
class Solution {
public:
    int search(vector<int>& nums, int target) {
        return findIndex(nums,target)-findIndex(nums,target-1);
    }

    //该函数本质上是找target在有序数组nums中合适的插入点或者是在相同数的右边界
    int findIndex(vector<int>&nums,int target){
        int i=0,j=nums.size()-1;
        while(i<=j){
            //找中间数，右移比除要快，而且(j-i)可以防止溢出，比如j+i可能大于2^32-1
            int m=i+((j-i)>>1);
            //target比中间值大，说明右边界在m的右边
            if(nums[m]<=target)i=m+1;
            //不然就在m的左边
            else j=m-1;
        }
        return i;
    }
};
```

### 剑指 Offer 53 - II. 0～n-1中缺失的数字

```c++
方法一：暴力枚举法
class Solution {
public:
    int missingNumber(vector<int>& nums) {
        //如果最后一个值与下标匹配得上，就说明前面没有缺，因此是数组后面缺一个n-1(即数组长度)
        if(nums[nums.size()-1]==nums.size()-1)return nums.size();
        //遍历数组找到第一个值与下标不匹配的值，然后返回下标(因为下标与本应在这的值相等)
        for(int i=0;i<nums.size();i++){
            if(i!=nums[i])return i;
        }
        return -1;
    }
};

方法二：二分查找
Key point:边界的值与下标不匹配，就意味着该边界左边或右边的元素都不匹配
class Solution {
public:
    int missingNumber(vector<int>& nums) {
        //i为左边界，j为右边界
        int i=0,j=nums.size()-1;
        while(i<=j){
            //找中间数
            int m=i+((j-i)>>1);
            //左边界左边的值与下标一一匹配，因此左边界右移
            if(nums[m]==m)i=m+1;
            //不匹配，右边界左移
            else j=m-1;
        }
        return i;
    }
};
```

**第四天总结：**当碰到排序数组查找的题目，可以优先考虑二分查找，因为二分算法就是专门针对排好序的结构的。



## 第五天

### 剑指 Offer 04. 二维数组中的查找

```c++
方法一：遍历二分
class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>>& matrix, int target) {
        //遍历二维数组
        for(auto&nums:matrix){
            //定义左右双指针
            int left=0,right=nums.size()-1;
            while(left<=right){
                //定义中间数索引
                int mid=left+((right-left)>>1);
                //如果有相等的，直接返回true
                if(nums[mid]==target)return true;
                //如果中间数小于target，说明target在右半区，left右移
                else if(nums[mid]<target)left=mid+1;
                //如果中间数大于target，说明target在左半区，right左移
                else right=mid-1;
            }
        }
        //遍历完整个二维数组后都没有符合要求的数，就返回false
        return false;
    }
};

方法二：类二叉树
    思路：我们现在将目光聚集于矩阵中最后一行的第一个元素，往上走值变小，往右走值变大。
         假设i为行数，j为列数，最初我们将标志数定为左下角的数，让target不断地于标志数比较。
    	 如果target更大，标志数就要往更大的方向走（即右边，j++）。
    	 如果target更小，标志数就要往更小的方向走（即上边，i--）。
    	 注意：虽然往左边走值也变小，但是由于前面的比较我们已经摒弃了前一列，因此不能向左回去比较。
class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>>& matrix, int target) {
        //定义标志数坐标
        int i=matrix.size()-1;
        int j=0;
        while(i>=0&&j<matrix[0].size()){
            if(matrix[i][j]==target)return true;
            //向下一列走
            else if(matrix[i][j]<target)j++;
            //向上一行走
            else i--;
        }
        return false;
    }
};
```

### 剑指 Offer 11. 旋转数组的最小数字

```c++
方法一：二分查找旋转点
class Solution {
public:
    int minArray(vector<int>& numbers) {
        //定义左右边界
        int i=0,j=numbers.size()-1;
        while(i<=j){
            //定义中间数
            int m=i+((j-i)>>1);
            //如果nums[m]>nums[j]，m肯定在左排序，让左边界右移
            if(numbers[m]>numbers[j])i=m+1;
            //如果nums[m]<nums[j]，m肯定在右排序，让右边界左移
            else if(numbers[m]<numbers[j])j=m;//j=m-1的话可能会错过旋转点
            
       //nums[m]==nums[j]，[i,m]或[m,j]之间全是相同元素，二者也可同时成立，此时遍历[i,j]找最小值即可
            else{
                int x=i;
                for(int k=i+1;k<j;k++){
                    x=numbers[k]<numbers[x]?k:x;
                }
                return numbers[x];
            }
        }
        return numbers[j];
    }
};
```

### 剑指 Offer 50. 第一个只出现一次的字符

```c++
方法一：有序哈希表
class Solution {
public:
    char firstUniqChar(string s) {
        vector<char>keys;//存储哈希表关键字
        unordered_map<char,bool>dic;//如果字符char出现次数大于1，second值为false
        //遍历字符串并将各个字符装入哈希表
        for(char&c:s){
            if(dic.find(c)==dic.end()){//没找到c，说明是第一次，true
                dic[c]=true;
                keys.push_back(c);//插入出现过的字符，且不重复，从而形成有序态
            }
            else dic[c]=false;
        }
        for(char&c:keys){
            if(dic[c])return c;
        }
        return ' ';
    }
};
```



## 第六天

### 剑指 Offer 32 - I. 从上到下打印二叉树

```c++
class Solution {
public:
    vector<int> levelOrder(TreeNode* root) {
        //将遍历到的结点值填入容器vec中
        vector<int>vec;
        //如果是空树，直接返回空容器即可
        if(!root)return vec;
        //将树结点按从上到下，从左到右的顺序储存到队列qu中
        queue<TreeNode*>qu;
        qu.push(root);
        while(!qu.empty()){//当qu不为空时才能进入循环，因为qu还有结点未处理
            //取队列的第一个节点
            TreeNode*node=qu.front();
            //删去队列头
            qu.pop();
            //将队列头的结点值插入vec中
            vec.push_back(node->val);
            //先左（左子结点）后右（右子结点）地插入队列
            if(node->left)qu.push(node->left);//左子结点不为空才能插入
            if(node->right)qu.push(node->right);//右子结点不为空才能插入
        }
        return vec;
    }
};
```

### 剑指 Offer 32 - II. 从上到下打印二叉树 II

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
         //用来存入各行结点
        vector<vector<int>>res;
        //如果是空树，直接返回空容器即可
        if(!root)return res;
        //将树结点按从上到下，从左到右的顺序储存到队列qu中
        queue<TreeNode*>qu;
        qu.push(root);
        while(!qu.empty()){//当qu不为空时才能进入循环，因为qu还有结点未处理
            //创建容器vec来存储该行的结点，每循环一次就是新的一行，所以要重新定义vec
            vector<int>vec;
            //遍历qu（储存的是当前行的结点），并将结点值插入vec中
            for(int i=qu.size();i>0;i--){//用for循环可以避免取到下一行的结点
/*如果是：for(int i=0;i<qu.size();i++)，因为循环过程中，qu有插入有删除，所以qu的大小在变化，因此循环判断是动态的，并非我们初始预期的；如果将i先初始化成qu的长度，那么循环次数是不变的，避免将下一行的结点卷进来*/
                //取队列头
                TreeNode*node=qu.front();
                //取完之后删去
                qu.pop();
                //将结点值插入vec
                vec.push_back(node->val);
                //将子结点加入qu
                if(node->left)qu.push(node->left);
                if(node->right)qu.push(node->right);
            }
            //遍历完当前行后，vec中存储的当前行所有结点值，将其插入res中
            //加入判断条件是防止加入空集合
            if(vec.size()>0)res.push_back(vec);
        }
        return res;
    }
};
```

### 剑指 Offer 32 - III. 从上到下打印二叉树 III

```c++
方法一：变向插入法
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        //创建res储存各行结点值
        vector<vector<int>>res;
        if(!root)return res;
        //创建qu储存结点
        queue<TreeNode*>qu;
        qu.push(root);
        //定义变量level记录当前层数
        int level=1;
        while(!qu.empty()){
            //创建vec记录当前行结点值
            vector<int>vec;
            for(int i=qu.size();i>0;i--){
                //取队列头
                TreeNode*node=qu.front();
                //删去队列头
                qu.pop();
                if(level%2==1){
                    //奇数层从前往后插数
                    vec.push_back(node->val);
                }else{
                    //偶数层从后向前插数
                    vec.insert(vec.begin(),node->val);
                }
                if(node->left)qu.push(node->left);
                if(node->right)qu.push(node->right);
            }
            //加入判断条件是防止加入空集合
            if(vec.size()>0)res.push_back(vec);
            //层数增加
            level++;
        }
        return res;
    }
};

方法二：双栈
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
         //创建res储存各行结点值
        vector<vector<int>>res;
        if(!root)return res;
        //创建双栈
        stack<TreeNode*>stk1;//处理奇数层
        stack<TreeNode*>stk2;//处理偶数层
        
        stk1.push(root);
        //只有当stk1和stk2都为空时才能结束循环
        while(!stk1.empty()||!stk2.empty()){
            vector<int>vec1;
            while(!stk1.empty()){
                //取栈顶元素
                TreeNode*node1=stk1.top();
                //删去栈顶元素
                stk1.pop();
                vec1.push_back(node1->val);
                if(node1->left)stk2.push(node1->left);
                if(node1->right)stk2.push(node1->right);
            }
            //加入判断条件是防止加入空集合
            if(vec1.size()>0)res.push_back(vec1);

            vector<int>vec2;
            while(!stk2.empty()){
                //取栈顶元素
                TreeNode*node2=stk2.top();
                //删去栈顶元素
                stk2.pop();
                vec2.push_back(node2->val);
                if(node2->right)stk1.push(node2->right);
                if(node2->left)stk1.push(node2->left);
            }
            //加入判断条件是防止加入空集合
            if(vec2.size()>0)res.push_back(vec2);
        }
        return res;
    }
};
```

## 第七天

### 剑指 Offer 26. 树的子结构

```c++
class Solution {
public:
    bool isSubStructure(TreeNode* A, TreeNode* B) {
        /*先序递归树A，因为要从根结点开始判断
          A&&B：AB都不为空
          recur(A,B)：判断以A结点值是否等于B结点值
          isSubStructure(A->left,B)：判断以A的左子结点为根结点的树是否包含树B
          isSubStructure(A->right,B)：判断以A的右子结点为根结点的树是否包含树B
        */
        return (A&&B)&&(recur(A,B)||isSubStructure(A->left,B)||isSubStructure(A->right,B));
    }

    bool recur(TreeNode*A,TreeNode*B){
        //如果递归到B为空，且前面还没返回，说明A已经包含了B
        if(!B)return true;
        //如果A超过叶子结点且B不为空或者A的值不等于B的值，说明A不包含B
        if(!A||A->val!=B->val)return false;
        //前面条件都不满足时，往下层递归
        return recur(A->left,B->left)&&recur(A->right,B->right);
    }
};
```

### 剑指 Offer 27. 二叉树的镜像

```c++
方法一：递归
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root) {
        if(!root)return root;
        TreeNode*temp=root->left;
        root->left=mirrorTree(root->right);
        root->right=mirrorTree(temp);
        return root;
    }
};

方法二：栈
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root) {
        if(!root)return root;
        stack<TreeNode*>stk;
        //将根结点压栈
        stk.push(root);
        while(!stk.empty()){
            //取栈顶元素
            TreeNode*node=stk.top();
            //消去
            stk.pop();
            if(node->left)stk.push(node->left);
            if(node->right)stk.push(node->right);
            TreeNode*temp=node->left;
            node->left=node->right;
            node->right=temp;
        }
        return root;
    }
};
```

### 剑指 Offer 28. 对称的二叉树

```c++
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        return root==NULL?true:recur(root->left,root->right);
    }

    bool recur(TreeNode*L,TreeNode*R){//L和R为对称的两结点
        if(!L&&!R)return true;
        if(!L||!R||L->val!=R->val)return false;
        return recur(L->left,R->right)&&recur(L->right,R->left);
    }
};
```

**第七天总结：**判断二叉树几何特性一般都用递归
