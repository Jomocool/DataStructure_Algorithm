# 左神LeetCode

## 1.链表

```c++
哈希表：
unordered_set<key>
unordered_map<key,value>
以上二者在结构上没有太大区别。哈希表增删改查时间复杂度都是 O(1)
    
1.哈希表中的key是基础类型，在插入时按照值传递的方式;是自定义类型，就会按照地址传递方式(8bytes)
  eg. student Jomo = new student("Jomo",19),插入时,把Jomo看作指向一块区域的"指针"

有序表：
ordered_Set<key>
ordered_map<key,value>
与哈希表不同的是元素之间按key排序，且自定义类型需要重载比较器；增删改查时间复杂度 O(logn)
```

**面试时链表解题的方法论：**

1. 笔试：一切为了时间复杂度
2. 面试：考虑时间复杂度的同时，一定要找到空间最省方法

**重要技巧：**

1. 额外数据结构记录（哈希表等）
2. 快慢指针

### 1.1 234. 回文链表

```c++
方法一：栈
利用栈先进后出的特性，可以发现遍历链表的过程中把结点压栈，出栈时的顺序恰好就是链表的逆序了。
时间复杂度:O(n) 空间复杂度:O(n)

方法二：一半栈
与方法一大同小异，不过我们利用快慢指针将链表的后半段压入栈即可，节省了一半的空间。
时间复杂度:O(n) 空间复杂度:O(n)
快慢指针遍历链表coding:


方法三：反向链表
fast指针走完时，让后半部分结点的next指针指向每个结点的前一个结点，然后两边向中间遍历比较，最后恢复链表。
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        //如果为空链表或者结点个数=1，肯定是回文链表
        if(head==NULL||head->next==NULL)return true;
        //创建快慢指针
        ListNode*slow=head;
        ListNode*fast=head;
        while(fast->next!=NULL&&fast->next->next!=NULL){
            slow=slow->next;
            fast=fast->next->next;
        }
        //结束while循环后，slow指向中间结点，接下来就是改变后续链表方向
        ListNode*rightOne=slow->next;//记录链表方向开始变化的第一个结点
        slow->next=NULL;//mid->next=NULL
        ListNode*next=NULL;//前驱结点
        while(rightOne!=NULL){
            next=rightOne->next;//保存原链表中slow的下一个结点
            rightOne->next=slow;//让后面的结点指向前面
            slow=rightOne;//slow move
            rightOne=next;//rightOne move
        }
        //结束循环后，slow指向链表的最后一个结点
        ListNode*last=slow;//保存最后一个结点
        ListNode*temp=head;//创建头指针副本
        bool res=true;//记录返回值
        while(temp!=NULL&&slow!=NULL){
            if(temp->val!=slow->val){
                res=false;
                break;
            }
            temp=temp->next;
            slow=slow->next;
        }
        //恢复链表原貌
        ListNode*pre=last;
        next=last->next;
        last->next=NULL;
        last=next;
        while(last!=NULL){
            next=last->next;
            last->next=pre;
            pre=last;
            last=next;
        }
        return res;
    }
};
```

### 1.2 面试题 02.04. 分割链表

```c++
方法一：数组
方法二：小等大指针
1.创建6个结点指针：SH(Smaller Head),ST(Smaller Tail);EH,ET;GH,GT
2.遍历链表，将每个结点值与基准值比较后，放到对应指针区域
    eg.SH=NULL且ST=NULL，则SH=第一个符合要求的结点，ST=SH;SH!=NULL且ST!=NULL;ST->next=下一个符合
       的结点;ST=ST->next;ST->next=NULL;
3.注意考虑边界情况：没有小于等于或大于基准值的结点
3. ST->next=EH;
4. ET->next=GH;
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        ListNode*SH=NULL;//小头
        ListNode*ST=NULL;//小尾
        ListNode*EH=NULL;//等头
        ListNode*ET=NULL;//等尾
        ListNode*GH=NULL;//大头
        ListNode*GT=NULL;//大尾
        //遍历链表
        ListNode*temp=head;//创建头指针副本
        ListNode*next=NULL;
        while(temp!=NULL){
            next=temp->next;
            temp->next=NULL;
            if(temp->val<x){
                if(SH==NULL&&ST==NULL){
                    SH=temp;
                    ST=SH;
                }else{
                    ST->next=temp;
                    ST=ST->next;
                }
            }
            else if(temp->val==x){
                if(EH==NULL&&ET==NULL){
                    EH=temp;
                    ET=EH;
                }else{
                    ET->next=temp;
                    ET=ET->next;
                }
            }
            else if(temp->val>x){
                if(GH==NULL&&GT==NULL){
                    GH=temp;
                    GT=GH;
                }else{
                    GT->next=temp;
                    GT=GT->next;
                }
            }
            temp=next;
        }
        if(ST!=NULL){
            ST->next=EH;
            ET=ET==NULL?ST:ET;
        }
        if(ET!=NULL){
            ET->next=GH;
        }
        return SH==NULL?(EH==NULL?GH:EH):SH;
    }
};
```

### 1.3 138.复制带随机指针的链表

```c++
方法一：哈希表
方法二：插入克隆结点
因为位置的相对关系已经确定，所以我们可以精确的找到克隆结点和旧结点的位置，又因为旧结点和克隆结点是成对存在的，因此有点类似于哈希表。
class Solution {
public:
    Node* copyRandomList(Node* head) {
        //创建哈希表记录对应的克隆结点
        unordered_map<Node*,Node*>st;
        //创建头指针副本
        Node*temp=head;
        //插入哈希表
        while(temp!=NULL){
            st.insert(make_pair(temp,new Node(temp->val)));
            temp=temp->next;
        }
        //同步克隆结点和原结点的信息
        temp=head;
        while(temp!=NULL){
            //注意克隆结点的指针指向的必须是克隆结点，不能是原结点
            st[temp]->next=st[temp->next];
            st[temp]->random=st[temp->random];
            temp=temp->next;
        }
        return st[head];
    }
};
```

### 1.4 剑指 Offer II 022. 链表中环的入口节点

![image-20221112154049200](C:\Users\86135\OneDrive\文档\数据结构与算法\zsLeetCode-img\1.png)

```c++
方法一：哈希表
利用哈希表快速查元素的特性。
方法二：快慢指针
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        //如果链表为空或者只有1个结点，肯定无法构成环
        if(head==NULL||head->next==NULL||head->next->next==NULL)return NULL;
        //创建快慢指针
        ListNode*fast=head->next->next;
        ListNode*slow=head->next;
        //第一次相遇
        while(fast!=slow){//注意如果把fast!=slow放在循环条件，那就不能一开始就让fast和slow都指向head
            if(fast->next==NULL||fast->next->next==NULL){
                return NULL;
            }
            slow=slow->next;
            fast=fast->next->next;
        }

        fast=head;

        while(slow!=fast){
            slow=slow->next;
            fast=fast->next;
        }
        return slow;
    }
};
```



## 2.二叉树

### 2.1 98. 验证二叉搜索树

```c++
方法一：中序遍历
看是否升序。不需要数组，因为只需要当前值和前一个值作比较
class Solution {
    long preVal=LONG_MIN;
public:
    bool isValidBST(TreeNode* root) {
        //空树
        if(root==NULL)return true;
        //判断左子树
        bool isLeftBST=isValidBST(root->left);
        //左子树不是BST，那整棵树都不是
        if(!isLeftBST)return false;
        //当前值小于前值，说明不是升序，不是BST
        if(root->val<=preVal){
            return false;
        }else{
            preVal=root->val;//更新preVal
        }
        //判断右子树
        return isValidBST(root->right);
    }
};
```

### 2.2 958. 二叉树的完全性检验

```c++
方法一：宽度遍历
1.碰到任一结点有右子结点而没有左子结点。返回false
2.在1不违规的情况下，遇到了第一个左右子结点不全(缺一即可)，那么后续所以结点都必须是叶子结点
class Solution {
public:
    bool isCompleteTree(TreeNode* root) {
        //空树
        if(!root)return true;
        //创建队列
        queue<TreeNode*>qu;
        //加入根结点
        qu.push(root);
        //判断已经遇到左右子结点不全的情况
        bool leaf=false;
        TreeNode*l=NULL;
        TreeNode*r=NULL;
        while(!qu.empty()){
            TreeNode*head=qu.front();
            qu.pop();
            l=head->left;
            r=head->right;
            if((l==NULL&&r!=NULL)||leaf&&!(l==NULL&&r==NULL))return false;
            if(l)qu.push(l);
            if(r)qu.push(r);
            if(l==NULL||r==NULL)leaf=true;
        }
        return true;
    }
};
```

**二叉树的套路：**

利用左右子树的信息，罗列各种可能性，树型DP

### 2.3 判断满二叉树

符合树型DP的才可以用以下套路

```c++
思路：nodes=2^h-1
//创建一个类，保存结点信息
class Info{
public:
    int height;
    int nodes;
    
    Info(int h,int n):
    height(h),nodes(n){}
};

//递归分析函数
Info f(Node*x){
    if(x==NULL)return Info(0,0);
    Info leftData=f(x->left);
    Info rightData=f(x->right);
    int height=max(leftData.height,rightData.height)+1;
    int nodes=leftData.nodes+rightData.nodes+1;
    return new Info(height,nodes);
}

bool isF(Node*root){
    if(root==NULL)return true;
    Info data=f(head);
    return data.nodes==((1<<data.height)-1);//左移几次就是乘以2的几次方
}
```

### 2.4 剑指 Offer 55 - II. 平衡二叉树

```c++
思路：判断平衡二叉树需要左子树和右子树的高度和平衡状况，符合树型DP
class ReturnType{
public:
    bool isBalance;
    int height;
    
    ReturnType(bool isB,int h):
    isBalance(isB),height(h){}
};

ReturnType process(Node*x){
    if(x==NULL)return ReturnType(true,0);
    //处理左子树
    ReturnType leftData=process(x->left);
    //处理右子树
    ReturnType rightData=process(x->right);
    //处理本结点
    bool isBalance=leftData.isBalance&&rightData.isBalance&&abs(leftData.height-rightData.height)<2;
    int height=max(leftData.height,rightData.height)+1;
    //返回本层结点的信息给上一层
    return ReturnType(isBalance,height);
}

bool isB(Node*root){
    return process(root).isBalance;
}
```

### 2.5 剑指 Offer 68 - II. 二叉树的最近公共祖先

```c++
方法一：哈希表
	思路：找到每个结点的父结点
void process(TreeNode*root,unordered_map<TreeNode*,TreeNode*>&fatherMap){
    if(root==NULL)return;
    fatherMap[root->left]=root;
    fatherMap[root->right]=root;
    process(root->left,fatherMap);
    process(root->right,fatherMap);
}

//返回o1,o2的最近公共祖先
TreeNode*lca(TreeNode*root,TreeNode*o1,TreeNode*o2){
    unordered_map<TreeNode*,TreeNode*>fatherMap;
    fatherMap[root]=root;
    process(root,fatherMap);
    unordered_set<TreeNode*>set1;
    TreeNode*cur=o1;
    //让cur向上走，把所有祖先放入set1
    while(cur!=fatherMap[cur]){
        set1.insert(cur);
        cur=fatherMap[cur];
    }
    //把root放入set1
    set1.insert(root);
    //让o2向上走，如果走到的结点存在于set1中，那就是最近公共祖先
    TreeNode*res=o2;
    while(set1.find(res)==set1.end()){
        res=fatherMap[res];
    }
    return res;
}

方法二：
1. o1是o2的lca，或o2是o1的lca
2. o1和o2不互为公共祖先，往上汇聚才能找到
    
函数实际上只返回NULL或o1或o2或o1、o2的最近共同祖先。
结点左右都为空的话该结点必定也返回空，叶子结点的left和right必定为空，会导致返回空层层向上传递直至o1、o2
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root==NULL||root==p||root==q)return root;
        TreeNode*left=lowestCommonAncestor(root->left,p,q);
        TreeNode*right=lowestCommonAncestor(root->right,p,q);
        if(left!=NULL&&right!=NULL)return root;//当left和right分别等于o1、o2时
        return left!=NULL?left:right;
    }
};
```

### 2.6 面试题 04.06. 后继者

```c++
1. x有右树：x的后继结点是其右树的最左结点
2. x无右树：一路往父结点找，当发现某个结点是其父结点的左子结点时，该父结点就是x的后继结点
   2.1注意考虑x是不是整棵树的最右结点
3.不考虑左树的原因：后续结点不会出现在左树中
class Solution {
public:
    TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) {
        unordered_map<TreeNode*,TreeNode*>fatherMap;
        fatherMap[root]=NULL;
        process(root,fatherMap);
        TreeNode*parent=fatherMap[p];
        if(p->right!=NULL)return getLeftMost(p->right);
        else{//无右树
            while(parent!=NULL&&parent->left!=p){
                p=parent;
                parent=fatherMap[p];
            }
        }
        return parent;
    }

    void process(TreeNode*root,unordered_map<TreeNode*,TreeNode*>&fatherMap){
        if(root==NULL)return;
        fatherMap[root->left]=root;
        fatherMap[root->right]=root;
        process(root->left,fatherMap);
        process(root->right,fatherMap);
    }

    TreeNode*getLeftMost(TreeNode*node){
        if(node==NULL)return node;
        while(node->left!=NULL){
            node=node->left;
        }
        return node;
    }
};
```

### 2.7 剑指 Offer II 048. 序列化与反序列化二叉树

内存里的一棵树如何变成字符串形式，又如何从字符串形式变成内存里的树

```c++
class Codec {
public:
    void rserialize(TreeNode* root, string& str) {
        if (root == nullptr) {
            str += "null,";
        } else {
            str += to_string(root->val) + ",";
            rserialize(root->left, str);
            rserialize(root->right, str);
        }
    }

    string serialize(TreeNode* root) {
        string ret;
        rserialize(root, ret);
        return ret;
    }

    TreeNode* rdeserialize(list<string>& dataArray) {
        if (dataArray.front() == "null") {
            dataArray.erase(dataArray.begin());
            return nullptr;
        }

        TreeNode* root = new TreeNode(stoi(dataArray.front()));
        dataArray.erase(dataArray.begin());
        root->left = rdeserialize(dataArray);
        root->right = rdeserialize(dataArray);
        return root;
    }

    TreeNode* deserialize(string data) {
        list<string> dataArray;
        string str;
        for (auto& ch : data) {
            if (ch == ',') {
                dataArray.push_back(str);
                str.clear();
            } else {
                str.push_back(ch);
            }
        }
        if (!str.empty()) {
            dataArray.push_back(str);
            str.clear();
        }
        return rdeserialize(dataArray);
    }
};
```

### 2.8 微软面试：折纸问题

```c++
思路：
除了最上和最下两条折痕，每条折痕的上下两条折痕分别为凹凸
void printProcess(int i,int N,bool down){
    if(i>N)return;
    //左子结点
    printProcess(i+1,N,true);
    down?cout<<"凹":cout<<"凸";
    //右子结点
    printProcess(i+1,N,false);
}
```

