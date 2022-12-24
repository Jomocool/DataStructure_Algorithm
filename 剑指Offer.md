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

## 第八天

### 剑指 Offer 10- I. 斐波那契数列

```c++
动态规划法：
    由于整个过程中只需要3个数:f(n)=f(n-1)+f(n-2)，所以我们只需要不断的更新这三个数即可
class Solution {
public:
    int fib(int n) {
        if(n<2)return n;
        int a=0;
        int b=1;
        int sum=0;
        for(int i=0;i<n;i++){
            sum=(a+b)%1000000007;
            a=b;
            b=sum;
        }
        //注意这里是return a，而不是return sum
        return a;
    }
};
```

![image text](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/Jz_Offer-img/2.png)

### 剑指 Offer 10- II. 青蛙跳台阶问题

```c++
动态规划：
    由于青蛙上台阶的方法有两种：1级或2级，所以我们以第1级台阶和第2级台阶的方法数为基础，向上迭加，所以i从1开始，因为开始的台阶级数为1。
class Solution {
public:
    int numWays(int n) {
        int a=1;
        int b=2;
        int sum=0;
        for(int i=1;i<n;i++){
            sum=(a+b)%1000000007;
            a=b;
            b=sum;
        }
        return a;
    }
};
```

![image text](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/Jz_Offer-img/3.png)

### 剑指 Offer 63. 股票的最大利润

```c++
由于股票只有在购入之后才能卖出，因此我们可以用变量cost来记录当前最低购入值，用变量profit来记录在最低购入点之前的最大利润，从而可以和在最低购入点购入之后再卖出的最大利润值作比较，不断地更新迭代。
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int cost=INT_MAX,profit=0;
        //遍历每天的股票价格
        for(auto&price:prices){
            //找最低购入点
            cost=min(cost,price);
            //计算在最低购入点买入后的最大利润
            profit=max(profit,price-cost);
        }
        return profit;
    }
};
```

**第八天总结：**动态规划并不一定要用二维数组，仔细观察真正用到的变量是哪几个，比如这三题用到的变量几乎都是dp[i],dp[i-1],dp[i-2]这三个变量，因此为了省略空间开销，我们可以用三个变量通过循环来实现动态规划的效果。



## 第九天

### 剑指 Offer 42. 连续子数组的最大和

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int maxSum=nums[0];
        int pre=0;
        for(int sum:nums){
            sum+=max(pre,0);
            pre=sum;
            maxSum=max(maxSum,sum);
        }
        return maxSum;
    }
};
```

![image text](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/Jz_Offer-img/4.png)

### 剑指 Offer 47. 礼物的最大价值

```c++
思路：
    由于只能向下或者向右走，因此除了第一行和第一列之外，其他格子都可以从它左边或者上边的格子抵达。
    dp[i][j]=:
			  1.dp[i][j]:i=0,j=0
              2.dp[i-1][j]:i!=0,j=0
              3.dp[i][j-1]:i=0,j!=0
              4.max(dp[i-1][j],dp[i][j-1])+grid[i][j]
class Solution {
public:
    int maxValue(vector<vector<int>>& grid) {
        int row=grid.size();//棋盘行数
        int column=grid[0].size();//棋盘列数
        //dp[i][j]=从grid[0][0]到grid[i-1][j-1]的最大礼物价值
        vector<vector<int>>dp(row+1,vector<int>(column+1));
        //多一行一列可以让代码更简洁，且这一行一列的值都是0，所以不影响边界情况
        for(int i=1;i<row+1;i++){
            for(int j=1;j<column+1;j++){
                dp[i][j]=max(dp[i-1][j],dp[i][j-1])+grid[i-1][j-1];
            }
        }
        return dp[row][column];
    }
};
```

**第九天总结：**弄明白每个状态是由哪个状态推出来的，并且考虑特殊情况。



## 第十天：动态规划（中等）

### 剑指 Offer 46. 把数字翻译成字符串

```c++
动态规划：
方法一：字符串遍历
    1.先将num转成字符串，方便后续处理，因为我们需要精确到每一位数字
    2.状态定义：dp[i]:以s[i-1]为结尾的数字（即从头开始总共i位数）的翻译方案数量,
    3.转移方程：s[i-1]s[i]组成的两位数字可以被翻译，则dp[i]=dp[i-1]+dp[i-2];否则，dp[i]=dp[i-1]
    4.注意：如果s[i-1]=0，组成的两位数无法被翻译；因此翻译范围：10~25(包括边界)
    5.定义初始状态：dp[0]=dp[1]=1;dp[0]表示无数字的时候。
    6.那么dp[0]=1从何而来呢？
      6.1 s[0]和s[1]组成两位数属于10~25时，dp[2]=2，而dp[2]=dp[1]+dp[0] => dp[0]=1;
	  6.2 s[0]和s[1]组成两位数不属于10~25时，dp[2]=dp[1]=1，不关dp[0]的事
    7.由于从始至终我们只需要3个变量：dp[i],dp[i-1],dp[i-2]，因此我们用变量c,b,a分别去对应这三个状态，
      从而减少空间的开销
class Solution {
public:
    int translateNum(int num) {
        //先将num转字符串，方便后续处理
        string numStr=to_string(num);
        //定义dp[0]
        int a=1;
        //定义dp[1]
        int b=1;
        //从i=2开始，即开始算dp[2]，即以s[2-1]结尾的数
        for(int i=2;i<=numStr.length();i++){
            string tmpStr=numStr.substr(i-2,2);
            int tmpVal=atoi(tmpStr.c_str());
            int c=tmpVal>=10&&tmpVal<=25?a+b:b;
            a=b;
            b=c;
        }
        return b;
    }
};

方法二：数字求余
相比于方法一，方法二少了额外的整数转字符串的空间开销，只用了常数个变量就完成了
class Solution {
public:
    int translateNum(int num) {
        int a=1,b=1;
        //从右向左计算翻译数量
        while(num>9){
            int c=num%100>=10&&num%100<=25?a+b:a;
            b=a;
            a=c;
            num/=10;
        }
        return a;
    }
};
```

### 剑指 Offer 48. 最长不含重复字符的子字符串

```c++
方法一：滑动窗口
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        //用于判断字符c是否已经存在于哈希表中了
        unordered_set<char>st;
        //s的长度
        int length=s.length();
        //记录最长无重复字符子串的长度
        int maxLen=0;
        //用于滑动
        int right=-1;
        //遍历字符串
        for(int i=0;i<length&&right+1<length;i++){
            if(i!=0){
                st.erase(s[i-1]);
            }
            while(right+1<length&&st.count(s[right+1])==0){
                st.insert(s[right+1]);
                right++;
            }
            maxLen=max(maxLen,right-i+1);
        }
        return maxLen;
    }
};
```

## 第十一天：双指针（简单）

### 剑指 Offer 18. 删除链表的节点

```c++
class Solution {
public:
    ListNode* deleteNode(ListNode* head, int val) {
        //边界情况
        if(head->val==val)return head->next;
        ListNode*cur=head;
        //pre是cur的前驱结点
        ListNode*pre=cur;
        while(cur->val!=val){
            pre=cur;
            cur=cur->next;
        }
        pre->next=pre->next->next;
        return head;
    }
};
```

### 剑指 Offer 22. 链表中倒数第k个节点

```c++
class Solution {
public:
    ListNode* getKthFromEnd(ListNode* head, int k) {
        ListNode*slow=head,*fast=head;
        int t=0;
        //让fast先走k步，然后当fast走完整个链表时，slow刚好走到倒数第k个结点
        while(fast){
            if(t>=k)slow=slow->next;
            fast=fast->next;
            t++;
        }
        return slow;
    }
};
```

**第十一天总结：**利用双指针可以补上链表长度差。



## 第十二天：双指针（简单）

### 剑指 Offer 25. 合并两个排序的链表

```c++
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        //创建新链表，用于指向合并的链表
        ListNode*res=new ListNode(-1);
        ListNode*temp=res;
        while(l1&&l2){
            //记录两指针遍历到的结点值
            int x=l1->val;
            int y=l2->val;
            if(x<=y){//如果l1的值更小或等于l2的值，那么temp的next指针就指向l1
                temp->next=l1;
                l1=l1->next;
            }else{//如果l2的值更小，那么temp的next指针就指向l2
                temp->next=l2;
                l2=l2->next;
            }
            temp=temp->next;
        }
        //处理还未遍历完的链表
        temp->next=l1?l1:l2;
        return res->next;
    }
};
```

### 剑指 Offer 52. 两个链表的第一个公共节点

```c++
方法一：计算长度法
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        //记录链表A的长度
        int lenA=0;
        //记录链表B的长度
        int lenB=0;
        //创建A，B链表头指针副本
        ListNode*tempA=headA;
        ListNode*tempB=headB;

        //计算链表A，B的长度
        while(tempA){
            lenA++;
            tempA=tempA->next;
        }
        while(tempB){
            lenB++;
            tempB=tempB->next;
        }

        //找长链表的头指针
        ListNode*headLonger=lenA>lenB?headA:headB;
        ListNode*headShorter=headLonger==headA?headB:headA;
        /*犯错点：一开始我写成了：
        ListNode*headLonger=lenA>lenB?headA:headB;
		ListNode*headShorter=lenA<lenB?headA:headB;
		这里会导致：当lenA==lenB时，headLonger=headShorter
		导致后面while循环的判断中出问题，因为指针相等，所以不进入循环。
		两种出错的情况：
					1.A、B相交时：返回的不是相交结点
					2.A、B不相交且长度相等时，直接返回headB，因为headLonger=headShorter=headB
        */

        //找长度差
        int dif=lenA>lenB?(lenA-lenB):(lenB-lenA);

        //让长链表的头指针先走，补上位移差
        for(int i=0;i<dif;i++){
            headLonger=headLonger->next;
        }

        //在距离相交点同样位移差的结点出发
        while(headLonger!=headShorter&&headLonger&&headShorter){
            headLonger=headLonger->next;
            headShorter=headShorter->next;
        }

        return headShorter;
    }
};

方法二：自动补距离法
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode*A=headA,*B=headB;
        /*当A或B切换到另一条链表时，此时在长链表还剩下没走的结点数就是两条链表的结点差，
        又因为在长链表的指针还有剩余结点没走完，就给了从短链表到长链表的那根指针补上位移差的机会，
        等到原本就在长链表的指针转到短链表时，二者距离相交结点或者空指针的距离是一样的*/
        while(A!=B){
            A=A?A->next:headB;
            B=B?B->next:headA;
        }
        return A;
    }
};
```



## 第十三天：双指针（简单）

### 剑指 Offer 21. 调整数组顺序使奇数位于偶数前面

```c++
方法一：左右指针互换法
class Solution {
public:
    vector<int> exchange(vector<int>& nums) {
        int i=0,j=nums.size()-1;
        while(i<j){
            //奇数二进制表达必定有一个2^0，即第一位肯定是1，和000...01作与运算，结果=1
            while(i<j&&(nums[i]&1)==1)i++;
            //偶数二进制表达必定没有2^0，即第一位肯定是0，和000...01作与运算，结果=0
            while(i<j&&(nums[j]&1)==0)j--;
            swap(nums[i],nums[j]);
        }
        return nums;
    }
};

方法二：博主自成版
class Solution {
public:
    vector<int> exchange(vector<int>& nums) {
        int left=0;
        int right=nums.size()-1;
        while(left<right){
            if(nums[left]%2==1&&nums[right]%2==1)left++;
            else if(nums[left]%2==0&&nums[right]%2==0)right--;
            else if(nums[left]%2==1&&nums[right]%2==0){
                left++;
                right--;
            }
            else if(nums[left]%2==0&&nums[right]%2==1){
                swap(nums[left],nums[right]);
                left++;
                right--;
            }
        }
        return nums;
    }

    void swap(int& a,int& b){
        int temp=a;
        a=b;
        b=temp;
    }
};
```

### 剑指 Offer 57. 和为s的两个数字

```c++
方法一：左右指针法
    思路：因为要想让和nums[i]+nums[j]增大或减小都各只有一种方法，那就是i++或j--，因此我们每次循环都是在
    	 不断缩小范围去逼近target
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        int i=0,j=nums.size()-1;
        while(i<j){
            if(nums[i]+nums[j]==target)return {nums[i],nums[j]};
            //这里用减法是防止溢出
            else if(nums[i]<target-nums[j])i++;
            else if(nums[i]>target-nums[j])j--;
        }
        return {nums[i],nums[j]};
    }
};
```

### 剑指 Offer 58 - I. 翻转单词顺序

```c++
class Solution {
public:
    string reverseWords(string s) {
        string res;
        for(int j=s.length()-1;j>=0;j--){
            //找单词的最后一个字符
            if(s[j]==' ')continue;
            int i=j;
            //找单词的第一个字符的前一个空格
            while(i>=0&&s[i]!=' ')i--;
            res.append(s.substr(i+1,j-i));
            res.append(" ");
            j=i;
        }
        //因为加入最后一个单词后，也会多加一个空格，所以我们需要删掉最后一个字符（即空格）
        if(!res.empty())res.pop_back();
        return res;
    }
};
```

**第十三天总结：**将双指针设置成相应的左右边界，从而限定范围。

## 第十四天：搜索与回溯算法（中等）

### 剑指 Offer 12. 矩阵中的路径

```c++
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        rows=board.size();
        clos=board[0].size();
        for(int i=0;i<rows;i++){
            for(int j=0;j<clos;j++){
                //从i,j点开始
                if(dfs(board,word,i,j,0))return true;
            }
        }
        return false;
    }

private:
    int rows,clos;
    bool dfs(vector<vector<char>>&board,string word,int i,int j,int k){
        if(i>=rows||i<0||j>=clos||j<0||board[i][j]!=word[k])return false;
        if(k>=word.length()-1)return true;
        board[i][j]='\0';//标记已被访问
        //有一条路径能通就行
        bool res=dfs(board,word,i-1,j,k+1)||dfs(board,word,i+1,j,k+1)||dfs(board,word,i,j-1,k+1)||dfs(board,word,i,j+1,k+1);
        //写回去
        board[i][j]=word[k];
        return res;
    }
};
```

### 面试题13. 机器人的运动范围

```c++
思路：
    模式识别：对于搜索问题（周游矩阵），递归实现的深度优先（DFS）和利用“队列”实现的广度优先（BFS）优先被考			   虑。
    状态：当前坐标（i,j）看作状态，考虑机器人从（i,j）出发能到达哪些格子。
    子问题：递归搜索（i-1,j）,（i+1,j）,（i,j-1）,（i,j+1）
    回溯：需要记录格子是否已经被访问，如果被访问过，说明该格子被包含在了子问题中，回溯到上一个格子时，就不
    	 能重复记录了。
    总结：到达每个格子时，要做的就是三件事：
    	 1.满足要求，格子数++
    	 2.更改状态为已访问
    	 3.解决子问题
    
方法一：深度优先探索
    1.定义辅助函数：计算数位之和
    2.定义递归函数
      2.1递归出口：（越界||数位和大于k||已被访问）return 0;
	3.更新当前状态，标记(i,j)已被访问
    4.解决子问题：向格子四周探索
        
class Solution {
public:
    //定义辅助函数计算数位之和
    int bitSum(int n){
        int sum=0;
        while(n!=0){
            sum+=n%10;
            n/=10;
        }
        return sum;
    }

    //定义深度优先探索函数
    int dfsDiscover(vector<vector<bool>>&state,int i,int j,int k){
        //退出递归条件
        if(i<0||j<0||i>=state.size()||j>=state[0].size()||bitSum(i)+bitSum(j)>k||state[i][j])return 0;
        //标记(i,j)已被访问
        state[i][j]=true;
        //解决子问题，向四周探索，因为能到这一步说明(i,j)符合要求，要加上这个格子，因此在后面+1
        return dfsDiscover(state,i-1,j,k)+dfsDiscover(state,i+1,j,k)+dfsDiscover(state,i,j-1,k)+dfsDiscover(state,i,j+1,k)+1;
    }

    int movingCount(int m, int n, int k) {
        //定义状态容器，记录每个格子的访问情况
        vector<vector<bool>>state(m,vector<bool>(n));
        //从原点出发
        return dfsDiscover(state,0,0,k);
    }
};

方法二：广度优先探索
    思路：
    	1.将数位之和大于k的格子看作是障碍物，就可以把这题当作一道传统的搜索题目了。
    	2.由于从原点出发，因此所有连通的格子必然都是从左边或上边的格子连通而得，因此，我们将搜索方向缩减为
    	  向右或向下即可。
    	3.广度优先探索是利用队列实现的，与队列相匹配的就是while循环了。
    
class Solution {
public:
    //辅助函数，计算数位之和
    int bitSum(int n){
        int sum=0;
        while(n!=0){
            sum+=n%10;
            n/=10;
        }
        return sum;
    }

    int movingCount(int m, int n, int k) {
        //k=0的话，只有原点满足要求，返回1个格子数即可
        if(!k)return 1;
        //创建队列，放入(i,j)格子
        queue<pair<int,int>>qu;
        //创建向下和向右的方向数组
        int di[2]={1,0};
        int dj[2]={0,1};
        //创建状态二维数组
        vector<vector<int>>state(m,vector<int>(n));
        //将原点放入qu
        qu.push(make_pair(0,0));
        //记录符合要求的格子数，已经有一个原点了，因此初始值为1
        int ans=1;
        //标记原点已被访问
        state[0][0]=1;
        //广度优先探索
        while(!qu.empty()){
            //取qu队列头，因为是符合要求的格子，从该格子出发向下向右探索
            auto[i,j]=qu.front();   
            //取完之后删去
            qu.pop();
            //向下向右探索，两次循环
            for(int c=0;c<2;c++){
                int ti=i+di[c];
                int tj=j+dj[c];
                //不满足要求直接继续循环，不用做下面的操作
                if(ti>=m||tj>=n||state[ti][tj]||bitSum(ti)+bitSum(tj)>k)continue;
                //如果满足要求，加入队列、标记已访问、ans++
                qu.push(make_pair(ti,tj));
                state[ti][tj]=1;
                /*
                误区：一开始的时候是在取完队列头之后才标记已访问，这样做会导致某些格子重复计算在ans内
                	 因为队列后面的格子在加入时就已经计算了ans++，如果此时不标记已访问，那么队列前面
                	 那些格子在走到队列后面的那些格子时又会再加一遍。所以ans++和标记已访问应该同步。
                */
                ans++;
            }
        }
        return ans;
    }
};

方法三：动态规划
    思路：考虑到除了边界的格子之外，每个格子都可以从它左边或上边的格子抵达，这与动态规划不谋而合
    转移方程：
    		1.dp[i][j]=1;i=0,j=0
			2.dp[i][j]=dp[i][j-1];i=0,j!=0
    		3.dp[i][j]=dp[i-1][j];i!=0,j=0
    		4.dp[i][j]=dp[i-1][j]|dp[i][j-1];i!=0,j!=0

class Solution {
public:
    //辅助函数，计算数位之和
    int bitSum(int n){
        int sum=0;
        while(n!=0){
            sum+=n%10;
            n/=10;
        }
        return sum;
    }

    int movingCount(int m, int n, int k) {
        if(!k)return 1;
        //记录满足要求的格子，如果满足条件，标记为1
        vector<vector<int>>vec(m,vector<int>(n));
        int ans=1;
        vec[0][0]=1;
        //利用两层for循环可以保证不会重复访问格子
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(i==0&&j==0||bitSum(i)+bitSum(j)>k)continue;
                //用|=原因：只要上边或左边有一个格子是1，那么(i,j)就是可抵达的
                //从上边格子抵达
                if(i-1>=0)vec[i][j]|=vec[i-1][j];
                //从左边格子抵达
                if(j-1>=0)vec[i][j]|=vec[i][j-1];
                ans+=vec[i][j];
            }
        }
        return ans;
    }
};
```

**第十四天总结：**搜索与回溯算法关键点在于每个格子必须由它周围的格子抵达，不能有独立的格子。



## 第十五天：搜索与回溯算法（中等）

### 剑指 Offer 34. 二叉树中和为某一值的路径

```c++
class Solution {
public:
    //创建全局变量，避免递归传参，提高效率
    vector<vector<int>>ret;//放入路径
    vector<int>path;//放入该路径的结点

    //深度递归探索，先序
    void dfs(TreeNode*node,int tar){
        //退出递归条件
        if(!node)return;
        path.emplace_back(node->val);
        tar-=node->val;
        if(!node->left&&!node->right&&tar==0){
            ret.emplace_back(path);
            //emplace_back()优于push_back()，因为不需要拷贝之后再放入
        }
        dfs(node->left,tar);
        dfs(node->right,tar);
        //关键！递归回溯过程中，为了避免路径包含了不属于自身的结点，要在最后删掉加入的结点值
        path.pop_back();
    }
    vector<vector<int>> pathSum(TreeNode* root, int target) {
        dfs(root,target);
        return ret;
    }
};
```

### 剑指 Offer 36. 二叉搜索树与双向链表

```c++
思路：
    1.由于是从小到大排列好，又因为在二叉排序树中，所以必然要用中序遍历。
    2.因为是双向链表且有链表头，所以需要一个前驱结点指针。
    3.边缘情况，递归完后，头尾结点实际上还没有连接起来，所以我们需要额外处理。
    
class Solution {
public:
    Node* treeToDoublyList(Node* root) {
        //边界情况
        if(!root)return root;
        dfs(root);
        //递归完之后，head的前驱结点为空，pre指向tail结点
        head->left=pre;
        pre->right=head;
        return head;
    }

    //创建pre、head指针
    Node*pre=NULL;
    Node*head=NULL;
    //中序递归
    void dfs(Node*cur){
        //退出递归的条件：当cur越过叶子结点时
        if(!cur)return;
        dfs(cur->left);
        if(pre!=NULL)pre->right=cur;
        else head=cur;//如果前驱结点为空，那cur必然是头结点
        //cur的前驱结点是pre
        cur->left=pre;
        //都处理完之后，让pre后移
        pre=cur;
        dfs(cur->right);
    }
};
```

### 剑指 Offer 54. 二叉搜索树的第k大节点

```c++
class Solution {
public:
    int k,ans;
    int kthLargest(TreeNode* root, int k) {
        this->k=k;
        dfs(root);
        return ans;
    }

    //深度递归，因为是从大到小，因此顺序是右根左
    void dfs(TreeNode*node){
        if(!node)return;
        //先遍历大的结点
        dfs(node->right);
        if(k==0)return;
        //第k大的数就是当k=1时的结点值
        if(--k==0)ans=node->val;
        dfs(node->left);
    }
};
```

**第十五天总结：**递归传参可以考虑用全局变量，提高效率。


## 第十六天：排序（简单）

### 剑指 Offer 45. 把数组排成最小的数

```c++
class Solution {
public:
    string minNumber(vector<int>& nums) {
        vector<string>strs;
        for(int i=0;i<nums.size();i++){
            strs.push_back(to_string(nums[i]));
        }
        quickSort(strs,0,strs.size()-1);
        string res;
        for(auto&str:strs){
            res.append(str);
        }
        return res;
    }

    //快排
    void quickSort(vector<string>&strs,int l,int r){
        if(l>=r)return;
        int i=l,j=r;
        while(i<j){
            /*
            以strs[l]为基准元素，从后向前找到一个小的（应放前面），
            从前向后找到一个大的（应放后面），二者交换就到了各自应该在的位置
            */
            while(strs[j]+strs[l]>=strs[l]+strs[j]&&i<j)j--;
            while(strs[i]+strs[l]<=strs[l]+strs[i]&&i<j)i++;
            swap(strs[i],strs[j]);
        }
        //出循环后，i=j。
        swap(strs[i],strs[l]);//更改基准值，不然str[l]一直都是同一个值
        quickSort(strs,l,i-1);
        quickSort(strs,i+1,r);
    }
};
```

### 剑指 Offer 61. 扑克牌中的顺子

```c++
class Solution {
public:
    bool isStraight(vector<int>& nums) {
        //记录大小王个数
        int joker=0;
        //排序
        sort(nums.begin(),nums.end());
        for(int i=0;i<4;i++){
            if(nums[i]==0)joker++;
            else if(nums[i]==nums[i+1])return false;
        }
        //nums[4]是MAX，nums[joker]是第一张不是大小王的牌，即MIN
        return nums[4]-nums[joker]<5;
    }
};
```

**第十六天总结：**快速排序并不一定是固定的模板，可以灵活改变一些条件，灵活运用。

​							顺子题：一开始我的想法是判断总差值是否为4，但实际上可以通过大小王来弥补，不

​											一定要用差值法，也可以用填补法，只要有足够的大小王就可以弥补。



## 第十七天：排序（中等）

### 剑指 Offer 40. 最小的k个数

```c++
方法一：快速排序
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) {
        quickSort(arr,0,arr.size()-1);
        vector<int>res;
        for(int i=0;i<k;i++){
            res.push_back(arr[i]);
        }
        return res;
    }

    //快排
    void quickSort(vector<int>&arr,int left,int right){
        if(left>=right)return;
        int i=left,j=right;
        while(i<j){
            while(arr[j]>=arr[left]&&i<j)j--;
            while(arr[i]<=arr[left]&&i<j)i++;
            swap(arr[i],arr[j]);
        }
        //更改哨兵值，要让数组左边是小于等于原哨兵值arr[left]的元素，数组右边是大于等于原哨兵值得元素
        swap(arr[i],arr[left]);
        quickSort(arr,left,i-1);
        quickSort(arr,i+1,right);
    }
};

方法二：基于快速排序的数组划分
    思路：因为题目只要求返回最小的k个数，且对这k个数的顺序不做要求，因此只需划分为最小的k个数和其他数字两部
    	 分。所以，在每次哨兵划分完之后，判段基准值arr[i]在数组中的索引是否等于k。
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) {
        if(k>=arr.size())return arr;
        return quickSort(arr,0,arr.size()-1,k);
    }

    //快排
    vector<int> quickSort(vector<int>&arr,int left,int right,int k){
        int i=left,j=right;
        while(i<j){
            while(arr[j]>=arr[left]&&i<j)j--;
            while(arr[i]<=arr[left]&&i<j)i++;
            swap(arr[i],arr[j]);
        }
        //更改哨兵值
        swap(arr[i],arr[left]);
        if(i>k)quickSort(arr,left,i-1,k);
        if(i<k)quickSort(arr,i+1,right,k);
        vector<int>res;
        res.assign(arr.begin(),arr.begin()+k);
        return res;
    }
};
```

### 剑指 Offer 41. 数据流中的中位数

```c++
class MedianFinder {
    priority_queue<int,vector<int>,less<int>>queMin;
    priority_queue<int,vector<int>,greater<int>>queMax;
public:
    /** initialize your data structure here. */
    MedianFinder() {} 
    
    void addNum(int num) {
        if(queMin.size()==0||num<=queMin.top()){
            queMin.push(num);
            if(queMax.size()+1<queMin.size()){
                queMax.push(queMin.top());
                queMin.pop();
            }
        }else{
            queMax.push(num);
            if(queMax.size()>queMin.size()){
                queMin.push(queMax.top());
                queMax.pop();
            }
        }
    }
    
    double findMedian() {
        return queMin.size()>queMax.size()?queMin.top():(queMax.top()+queMin.top())/2.0;
    }
};
```



## 第十八天：搜索与回溯算法（中等）

### 剑指 Offer 55 - I. 二叉树的深度

```c++
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(root==NULL)return 0;
        //能到当前步骤，说明该层是确实存在的，因此+1
        return max(maxDepth(root->left),maxDepth(root->right))+1;
    }
};
```

### 剑指 Offer 55 - II. 平衡二叉树

```c++
 //向左右子节点要信息
class Info{
public:
    int height;
    bool isB;

    Info(int height,bool isB){
        this->height=height;
        this->isB=isB;
    }
};

class Solution {
public:
    bool isBalanced(TreeNode* root) {
        Info res=process(root);
        return res.isB;
    }

    Info process(TreeNode*x){
        if(x==NULL){//base case
            return Info(0,true);
        }
        Info leftInfo=process(x->left);
        Info rightInfo=process(x->right);
        int height=max(leftInfo.height,rightInfo.height)+1;
        //左右子树都平衡且左右子树高度差不超过1的情况下，以该节点为根的树才平衡
        bool isB=leftInfo.isB&&rightInfo.isB&&abs(leftInfo.height-rightInfo.height)<=1;
        return Info(height,isB);
    }
};
```



## 第十九天：搜索与回溯算法（中等）

### 剑指 Offer 64. 求1+2+…+n

```c++
方法一：短路法
class Solution {
public:
    int sumNums(int n) {
        int sum=n;
        sum&&(sum+=sumNums(n-1));
        return sum;
    }
};

方法二：
class Solution {
public:
    int sumNums(int n) {
        char arr[n][n+1];
        return sizeof(arr)>>1;
    }
};
```

### 剑指 Offer 68 - I. 二叉搜索树的最近公共祖先

```c++
class Solution {
public:
    void process(TreeNode*root,map<TreeNode*,TreeNode*>&fatherMap){
        if(root==NULL)return;
        fatherMap[root->left]=root;
        fatherMap[root->right]=root;
        process(root->left,fatherMap);
        process(root->right,fatherMap);
    }

    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(p==NULL||q==NULL)return NULL;

        map<TreeNode*,TreeNode*>fatherMap;
        fatherMap[root]=NULL;
        process(root,fatherMap);

        set<TreeNode*>set1;
        TreeNode*cur=p;
        while(cur!=NULL){
            set1.insert(cur);
            cur=fatherMap[cur];
        }

        TreeNode*res=q;
        while(set1.find(res)==set1.end()){
            res=fatherMap[res];
        }
        return res;
    }
};
```

### 剑指 Offer 68 - II. 二叉搜索树的最近公共祖先

```c++
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root==NULL||root==p||root==q)return root;
        TreeNode*left=lowestCommonAncestor(root->left,p,q);
        TreeNode*right=lowestCommonAncestor(root->right,p,q);
        //当left和right第一次分别等于o1、o2时，此时的root就是二者最近的公共祖先
        if(left!=NULL&&right!=NULL)return root;
        return left!=NULL?left:right;//如果二者有一个不为空，那么就返回那个不为空的
    }
};
```



## 第二十天 分治算法（中等）

### 剑指 Offer 07. 重建二叉树

![image text](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/Jz_Offer-img/5.png)

```c++
class Solution {
public:
    map<int,int>dic;//用于映射中序遍历元素对应的索引
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        if(preorder.size()==0||inorder.size()==0)return NULL;
        for(int i=0;i<inorder.size();i++){
            dic[inorder[i]]=i;
        }
        return f(preorder,0,preorder.size()-1,inorder,0,inorder.size()-1);
    }

    /*
    l1：前序数组子数组的左边界，而前序遍历的第一个元素即为根节点
    r1：前序数组子数组的右边界
    l2：中序数组子数组的左边界
    r2：中序数组子数组的右边界
    */
    TreeNode*f(vector<int>&preorder,int l1,int r1,vector<int>&inorder,int l2,int r2){
        if(l1>r1&&l2>r2)return NULL;

        //创建根节点
        TreeNode*root=new TreeNode(preorder[l1]);
        //处理其左右子树
        int i=dic[preorder[l1]];//找到根节点在中序遍历数组中的位置，以区分左右子树
        root->left=f(preorder,l1+1,l1+(i-l2),inorder,l2,i-1);
        root->right=f(preorder,l1+(i-l2)+1,r1,inorder,i+1,r2);
        return root;
    }
};
```

### 剑指 Offer 16. 数值的整数次方

```c++
利用进制的进制快速幂计算
class Solution {
public:
    double myPow(double x, int n) {
        double res=1.0;
        long y=n;//2^32无法用int类型保存
        if(n<0){
            y=-y;
            x=1.0/x;
        }
        while(y>0){
            if(y%2==1)res*=x;
            //假设n的二进制位全都是1的话
            //x^n=(x^1)*(x^2)*(x^4)*(x^8)*...*(x^(2^m))
            //所以x的进制基础从1开始，而不是0
            x*=x;
            y>>=1;
        }
        return res;
    }
};
```

### 剑指 Offer 33. 二叉搜索树的后序遍历序列

```c++
方法一：
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) {
        if(postorder.size()==0)return true;
        return f(postorder,0,postorder.size()-1);
    }

    bool f(vector<int>&postorder,int i,int j){
        if(i>=j)return true;
        int root=postorder[j];
        int p=i;
        //获取第一个比大于或等于root元素的位置，也是左右子树的分割位置
        while(postorder[p]<root)p++;
        //看p~j-1位置中是否有小于root的元素，有的话就不符合
        for(int k=p;k<j;k++){
            if(postorder[k]<root)return false;
        }
        return f(postorder,i,p-1)&&f(postorder,p,j-1);
    }
};

方法二：单调栈
正着看：左右根
倒着看：根右左
1.挨着的两个数如果arr[i]<arr[i+1]，那么arr[i+1]一定是arr[i]的右子节点
2.如果arr[i]>arr[i+1]，那么arr[i+1]一定是arr[0]……arr[i]中某个节点的左子节点，并且这个值是大于arr[i+1]中最小的。因为在右子树中，根节点和右子节点总是在左子节点前面，因此arr[i]<arr[i+1]，当第一次出现arr[i]>arr[i+1]时，必定是从右子树跳到左子树的交界处。且arr[i+1]的前面所有值只包含了其根节点和该根节点的右子树所有值，显然根节点是这些值中最小的。
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) {
        if(postorder.size()==0)return true;
        stack<int>stk;
        int parent=INT_MAX;
        for(int i=postorder.size()-1;i>=0;i--){
            int cur=postorder[i];
            //当出现倒序时，说明从右子树跳到左子树了，找到当前值得父节点
            while(!stk.empty()&&stk.top()>cur){
                parent=stk.top();
                stk.pop();
                //弹出栈的节点不用管了，因为每一个只有栈底的节点才是当前值父节点，其他值只是父的右子树
                //而后续遍历道的节点的父节点不可能是父节点的右子树上的节点
            }
            //如果是升序直接入栈，不进while循环
            if(cur>=parent)return false;
            stk.push(cur);
        }
        return true;
    }
};
```

## 第二十一天：位运算（简单）

### 剑指 Offer 15. 二进制中1的个数

```c++
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int count=0;
        while(n>0){
            if(n%2==1)count++;
            n>>=1;
        }
        return count;
    }
};
```

### 剑指 Offer 65. 不用加减乘除做加法

```c++
class Solution {
public:
    int add(int a, int b) {
        int sum=0;
        while(b!=0){
            sum=a^b;//a和b的无进位相加
            b=(unsigned)(a&b)<<1;//LeetCode负数不支持左移
            a=sum;
        }
        return sum;
    }
};
```



## 第二十二天：位运算（中等）

### 剑指 Offer 56 - I. 数组中数字出现的次数

```c++
class Solution {
public:
    vector<int> singleNumbers(vector<int>& nums) {
        //假设单独出现的两个数分别为x和y
        int eor=0;//整个数组的异或和
        for(int i=0;i<nums.size();i++){
            eor^=nums[i];
        }

        //此时eor=x^y，找到x和y第一个不相同的位
        int m=1;
        while((m&eor)==0){
            m<<=1;
        }

        //遍历数组，将数组分成两部分
        //nums1[i]&m==0
        //nums2[i]&m==1
        //x和y必然分开出现在以上两数组中，所以再找数组中一个单独出现的数即可
        int x=0;
        int y=0;
        for(int i=0;i<nums.size();i++){
            if((nums[i]&m)==0){
                x^=nums[i];
            }else{
                y^=nums[i];
            }
        }
        return {x,y};
    }
};
```

### 剑指 Offer 56 - II. 数组中数字出现的次数

```c++
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        //记录数组中所有数各数位1的个数
        int arr[32]={};//由于int类型只有32位，所以数组大小为32即可
        for(int i=0;i<nums.size();i++){
            for(int move=0;move<32;move++){
                if(((nums[i]>>move)&1)==1){
                    arr[move]++;
                }
            }
        }

        int single=0;
        for(int i=0;i<32;i++){
            //由于除了要找的数外其他数都有3个，那每个位上1的个数必然是3n或3n+1
            single+=(arr[i]%3)<<i;
        }
        return single;
    }
};
```



## 第二十三天：数学（简单）

### 剑指 Offer 39. 数组中出现次数超过一半的数字

```c++
摩尔投票法：将nums[i]看作支持的号码牌，sum则是票数
1.当sum=0时，选举下一个数当众数，遍历数组
2.如果当前数和众数相等，则众数票数增加，sum++
3.如果不等，众数票数就减少，sum--
4.当sum减少到0时，说明支持者不够多，那意味着它不是众数，重新选举下一个数为众数

class Solution {
public:
    int majorityElement(vector<int>& nums) {
        if(nums.size()<=2){
            return nums[0];
        }

        //先假定众数是nums[0]，众数票数sum=1
        int x=nums[0];
        int sum=1;
        for(int i=1;i<nums.size();i++){
            if(sum==0){//说明要重新选举众数了
                x=nums[i];
                sum=1;
            }else{
                if(nums[i]==x){//和众数一样，向众数投票
                    sum++;
                }else{//不一样则抵消
                    sum--;
                }
            }
        }
        return x;
    }
};
```

### 剑指 Offer 66. 构建乘积数组

```c++
正反遍历数组：
class Solution {
public:
    vector<int> constructArr(vector<int>& a) {
        //由于b[i]抛弃的是a[i]，因此我们可以通过正反遍历a数组达到只缺a[i]的效果
        vector<int>b(a.size());
        //遍历前初始化成1
        int mul=1;
        for(int i=0;i<a.size();i++){
            b[i]=mul;
            mul*=a[i];
        }

        //遍历前重新初始化成1
        mul=1;
        for(int i=a.size()-1;i>=0;i--){
            b[i]*=mul;
            mul*=a[i];
        }

        return b;
    }
};
```

