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

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/1.png)

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

## 3.图

创建一套属于自己的模板，在自己的模板里把算法玩熟悉

**模板1：**

```c++
思路：
    1.存储顶点用vector<char>
    2.保存矩阵用vector<vector<int>>edges，表示边的关系
    
class Graph {
private:
	vector<char>vertex;//存储顶点集合
	vector<vector<int>>edges;//存储图对应的领接矩阵
	int numOfEdges;//边的数目
public:
	//构造器
	Graph(int n) {
		//初始化矩阵和vertexList
		for (int i = 0; i < n; i++) {
			edges.push_back(vector<int>(n));
		}
		numOfEdges = 0;
	}

	//图中常用的方法
	//返回节点个数
	int getNumOfVertex() {
		return vertex.size();
	}

	//得到边的数目
	int getNumOfEdges() {
		return numOfEdges;
	}

	//返回节点i（下标）对应的数据
	char getValueByIndex(int i) {
		return vertex[i];
	}

	//返回[x][y]的权值
	int getWeight(int x, int y) {
		return edges[x][y];
	}

	//显示图对应的矩阵
	void showGraph() {
		for (auto& link : edges) {
			cout << "[" << " ";
			for (auto& n : link) {
				cout << n << " ";
			}
			cout <<"]"<< endl;
		}
	}

	//插入节点
	void insertVertex(char vertex2) {
		vertex.push_back(vertex2);
	}

	//添加边
	void insertEdge(int x, int y, int weight) {
		edges[x][y] = weight;
		edges[x][y] = weight;
		numOfEdges++;
	}
};
```

### 3.1 图的广度优先遍历：队列

```c++
	//得到第一个邻接节点的下标w，如果存在就返回对应的下标，否则返回-1
	int getFirstNeighbor(int index) {
		for (int j = 0; j < vertex.size(); j++) {
			if (edges[index][j] > 0) {
				return j;
			}
		}
		return -1;
	}

	//根据前一个邻接节点的下标来获取下一个邻接节点
	int getNextNeighbor(int v1, int v2) {
		for (int j = v2 + 1; j < vertex.size(); j++) {
			if (edges[v1][j] > 0) {
				return j;
			}
		}
		return -1;
	}

//对一个节点进行广度优先遍历的方法
	void bfs(vector<bool>& isvisited, int i) {
		int u=0;//表示队列头的对应下标
		int w = 0;//邻接节点w
		//队列，记录节点访问的顺序
		queue<int>qu;
		//能调用该函数，说明该节点可访问，输出节点信息
		cout << getValueByIndex(i) << "=>";
		//标记为已访问
		isvisited[i] = true;
		//将节点加入队列
		qu.push(i);
		
		while (!qu.empty()) {
			//取出队列的头节点下标
			u = qu.front();
			qu.pop();
			//得到第一个邻接节点的下标
			w = getFirstNeighbor(u);
			while (w != -1) {//找到
				//判断w节点是否被访问过
				if (!isvisited[w]) {//未被访问过
					//输出节点信息
					cout << getValueByIndex(w) << "=>";
					//标记已被访问
					isvisited[w] = true;
					//入队
					qu.push(w);
				}
				//如果w已被访问过，那么就以u为前驱节点，找w后面的邻接节点
				w = getNextNeighbor(u, w);//体现出广度优先
			}
		}
	}

	//重载bfs，遍历所有节点都进行广度优先搜索
	void bfs() {
		for (int i = 0; i < vertex.size(); i++) {
			if (!isVisited[i]) {
				bfs(isVisited, i);
			}
		}
	}
```

### 3.2 图的深度优先遍历：递归

```c++
//得到第一个邻接节点的下标w，如果存在就返回对应的下标，否则返回-1
	int getFirstNeighbor(int index) {
		for (int j = 0; j < vertex.size(); j++) {
			if (edges[index][j] > 0) {
				return j;
			}
		}
		return -1;
	}

	//根据前一个邻接节点的下标来获取下一个邻接节点
	int getNextNeighbor(int v1, int v2) {
		for (int j = v2 + 1; j < vertex.size(); j++) {
			if (edges[v1][j] > 0) {
				return j;
			}
		}
		return -1;
	}

	//深度优先遍历算法
	//i第一次就是0
	void dfs(vector<bool>&isvisited,int i) {
		//首先访问该节点，输出
		cout << getValueByIndex(i) << "->";
		//将该节点设置成已访问
		isvisited[i] = true;
		//查找i节点的第一个邻接节点
		int w = getFirstNeighbor(i);
		while (w != -1) {
			if (!isvisited[w]) {
				dfs(isvisited, w);
			}
			w = getNextNeighbor(i, w);
		}
	}

	//重载dfs
	void dfs() {
		for (int i = 0; i < vertex.size(); i++) {
			if (!isVisited[i]) {
				dfs(isVisited, i);
			}
		}
	}
```

### 3.3 拓扑排序

确定做事的顺序：优先执行入度为0的点，然后擦拭掉该点的影响（即边）

**模板2：**

```c++
class Node;

class Edge{
public:
    int weight;//权重
    Node*from=NULL;//出发点
    Node*to;//终点
    
    Edge(int weight,Node*from,Node*to){
        this->weight=weight;
        this->from=from;
        this->to=to;
    }
};

class Node{
public:
    int value;//结点值
    int in;//入度
    int out;//出度
    vector<Node*>nexts;//相邻结点
    vector<Edge>edges;//相邻边
    
    Node(int value){
        this->value=value;
        in=0;
        out=0;
    }
};

class Graph{
public:
    map<int,Node*>nodes;
    set<Edge>edges;
};
```

**拓扑排序代码实现：**

```c++
list<Node*>sortedTopology(Graph&graph){
    map<Node*,int>inMap;
    //入度为0的点才可以进队列
    queue<Node*>zeroInQueue;
    for(auto it=graph.nodes){
        inMap.insert(it->second,it->first);
        if(it->second->in==0){
            zeroInQueue.push(it->second);
        }
    }
    //拓扑排序的结果，依次加入result
    List<Node*>result;
    while(!zeroInQueue.empty()){
        Node*cur=zeroInQueue.front();
        zeroInQueue.pop();
        result.push(cur);
        for(Node*next:cur.nexts){
            inMap[next]-=1;
            if(inMap[next]==0){
                zeroInQueue.push(next);
            }
        }
    }
    return result;
}
```

### 3.4 最小生成树

1.保证所有点连通，但不一定需要所有的边

2.既保证连通性，同时边的权值累加和最小

#### 3.4.1 Kruskal算法

**适用范围：**无向图

```c++
思路：
    1.一开始把所有点看作孤立点，即每个点自成一个集合

//使用INT_MAX表示两个顶点不连通
int INF = INT_MAX;

//创建一个类EData，它的对象实例就表示一条边
class EData {
public:
	char start;//边的起点
	char end;//边的终点
	int weight;//边的权值

	EData()
		:start(' '), end(' '), weight(0) {}

	EData(char start, char end, int weight) {
		this->start = start;
		this->end = end;
		this->weight = weight;
	}

	void toString() {
		cout << "start: " << this->start << " end: " << this->end << " weight: " << this->weight << endl;
	}
};

class KruskalCase {
private:
	int edgeNum = 0;//边的个数
	string vertexs;//顶点数组
	vector<vector<int>>matrix;//邻接矩阵
public:
	KruskalCase(string vertexs, vector<vector<int>>& matrix) {
		size_t vlen = vertexs.length();
		this->vertexs=vertexs;
        this->matrix=matrix;

		//统计边数
		for (size_t i = 0; i < vlen; i++) {
			for (size_t j = i + 1; j < vlen; j++) {
				if (this->matrix[i][j] != INF) {
					edgeNum++;
				}
			}
		}
	}

    //Kruskal算法
	vector<EData> kruskal() {
		int index = 0;//表示最后结果数组的索引
		vector<int>ends(this->edgeNum);//用于保存“已有最小生成树”中的每个顶点在最小生成树中的终点
		//创建结果数组，保存最后的最生成树
		vector<EData>rets;

		//获取图中所有边的集合，一共有12条边
		vector<EData>edges = getEdges();

		//按照边的权值大小进行排序（从小到大）
		sortEdges(edges);

		//遍历edges数组，将边添加到最小生成树中时，判断准备加入的边是否形成了回路，如果没有，就加入rets，否则不能加入
		for (int i = 0; i < edgeNum; i++) {
			//获取到第i条边的第一个顶点（起点）
			int p1 = getPosition(edges[i].start);
			//获取到第i条边的第二个顶点
			int p2 = getPosition(edges[i].end);

			//获取p1这个顶点在已有最小生成树中的终点
			int m = getEnd(ends, p1);
			//获取p2这个顶点在已有最小生成树中的终点
			int n = getEnd(ends, p2);
			if (m != n) {//不构成回路
				ends[m] = n;//设置m 在“已有最小生成树”中的终点
				rets.push_back(edges[i]);//有一条边加入到rets数组
			}
		}
		for (int i = 0; i < rets.size(); i++) {
			rets[i].toString();
		}
		return rets;
	}

	//打印邻接矩阵
	void print() {
		cout << "邻接矩阵为：" << endl;
		for (int i = 0; i < vertexs.length(); i++) {
			for (int j = 0; j < vertexs.length(); j++) {
				cout << matrix[i][j] << "\t";
			}
			cout << endl;
		}
	}

	//对边进行排序处理，冒泡排序
	void sortEdges(vector<EData>& edges) {
		for (int i = 0; i < edges.size(); i++) {
			for (int j = 0; j < edges.size() - i-1; j++) {
				if (edges[j].weight < edges[j + 1].weight) {
					EData temp = edges[j];
					edges[j] = edges[j + 1];
					edges[j + 1] = temp;
				}
			}
		}
	}

	int getPosition(char ch) {
		for (int i = 0; i < vertexs.length(); i++) {
			if (vertexs[i] == ch) {
				return i;
			}
		}
		//找不到，返回-1
		return -1;
	}

	//获取图中边，放到EData数组中，后面我们需要遍历该数组
	vector<EData> getEdges() {
		vector<EData>edges;
		for (int i = 0; i < vertexs.length(); i++) {
			for (int j = i + 1; j < vertexs.length(); j++) {
				if (matrix[i][j] != INF) {
					edges.push_back(EData(vertexs[i], vertexs[j], matrix[i][j]));
				}
			}
		}
		return edges;
	}

	//获取下标为i的顶点的终点,，用于判断两个顶点的终点是否相同
	//ends数组记录了各个顶点对应的终点，ends数组实在遍历过程中，逐步形成
	int getEnd(vector<int>& ends, int i) {
		while (ends[i] != 0) {
			i = ends[i];
		}
		return i;
	}
};

int main() {
	string vertexs = "ABCDEFG";
	vector<vector<int>>matrix = {
		{0,12,INF,INF,INF,16,14},
		{12,0,10,INF,INF,7,INF},
		{INF,10,0,3,5,6,INF},
		{INF,INF,3,0,4,INF,INF},
		{INF,INF,5,4,0,2,8},
		{16,7,6,INF,2,0,9},
		{14,INF,INF,INF,8,9,0}
	};

	//创建KruskalCase对象实例
	KruskalCase kruskalCase(vertexs, matrix);
	//输出构建的
	kruskalCase.print();
	kruskalCase.kruskal();
}
```

#### 3.4.2 Prim算法

```c++
思路：
    1.从已在连通集合中的点中，找到与之相关权值最小的边，依次连通下一个点。

class MGraph {
public:
	int verxs;//表示图的节点个数
	string data;//存放节点数据
	vector<vector<int>>weight;//存放边，邻接矩阵

	MGraph(int verxs) {//构造函数
		this->verxs = verxs;
		for (int i = 0; i < verxs; i++) {
			data.append(" ");
			weight.push_back(vector<int>(verxs));
		}
	}
};

//创建最小生成树->村庄的图
class MinTree {
	//创建图的邻接矩阵
public:
	void createGraph(MGraph& graph, int verxs, string data, vector<vector<int>>& weight) {
		for (int i = 0; i < verxs; i++) {//顶点
			graph.data[i] = data[i];
			for (int j = 0; j < verxs; j++) {
				graph.weight[i][j] = weight[i][j];
			}
		}
	}

	void showGraph(MGraph&graph) {
		for (auto& it0 : graph.weight) {
			for (auto& it1 : it0) {
				cout << it1 << " ";
			}
			cout << endl;
		}
	}

	//Prim算法，得到最小生成树
	void prim(MGraph&graph,int v) {//v表示从图的第几个顶点开始
		//visited标记顶点是否被访问过
		vector<int>visited(graph.verxs);

		//把当前顶点标记为已访问
		visited[v] = 1;

		//h1和h2记录两个顶点的下标
		int h1 = -1;
		int h2 = -1;
		int minWeight = 10000;//将minWeight初始成一个大数，在遍历过程中会被替换

		//因为不是单纯的一次性遍历图，而是需要回溯，每次都要重新从第一个已访问的顶点去分析与各个相邻点的权
        //重
		for (int k = 1; k < graph.verxs; k++) {//因为有graph.verxs个顶点，所以有graph.verx-1
            								   //条边
			//确定每一次生成的子图和那个顶点的距离最近
			for (int i = 0; i < graph.verxs; i++) {//i表示已访问过的顶点
				if (visited[i] == 1) {
					for (int j = 0; j < graph.verxs; j++) {//j表示还未访问过的顶点
						if (visited[j] == 0 && graph.weight[i][j] < minWeight) {
							minWeight = graph.weight[i][j];
							h1 = i;
							h2 = j;
						}
					}
				}
			}
			cout << "边<" << graph.data[h1] << "," << graph.data[h2] << ">" << "权值：" << minWeight << endl;
			//将h2设置为已访问
			visited[h2] = 1;
			//重新设置minWeight
			minWeight = 10000;
		}
	}
};

int main() {
	string data = "ABCDEFG";
	int verxs = data.length();

	//邻接矩阵，10000表示两个点不连通
	vector<vector<int>>weight = {
		{10000,5,7,10000,10000,10000,2},
		{5,1000,10000,9,10000,10000,3},
		{7,10000,10000,10000,8,10000,10000},
		{10000,9,10000,10000,10000,4,10000},
		{10000,10000,8,10000,10000,5,4},
		{10000,10000,10000,4,5,10000,6},
		{2,3,10000,10000,4,6,10000}
	};

	//创建MGraph对象
	MGraph graph(verxs);
	////创建一个MinTree对象
	MinTree minTree;
	minTree.createGraph(graph, verxs, data, weight);
	minTree.showGraph(graph);
	minTree.prim(graph, 1);
}
```

#### 3.4.3 Dijkstra算法

**Prim算法和Dijkstra算法十分相似**，惟一的区别是： Prim算法要寻找的是离已加入顶点距离最近的顶点； Dijkstra算法是寻找离固定顶点距离最近的顶点。

```c++
方法一：普通版
class VisitedVertex {
public:
	vector<int>already_arr;//记录各个顶点是否访问过 1-访问过，0-未访问过，会动态更新
	vector<int>pre_visited;//每个下标对应的值为前一个顶点的下标，会动态更新
	vector<int>dis;//记录出发顶点到其他所有顶点的距离，比如G为出发顶点，就会记录F到其他顶点的距离，会动态更新，求的最短距离就会存放到dis

	VisitedVertex(){}

	VisitedVertex(int length, int index) {//length：顶点个数  index:：起始顶点下标
		already_arr = vector<int>(length);
		pre_visited = vector<int>(length);
		dis = vector<int>(length);

		this->already_arr[index] = 1;//设置出发顶点被访问

		//初始化dis数组
		for (int i = 0; i < dis.size() - 1; i++) {
			this->dis[i] = 65535;
		}
	}

	//判断index顶点是否被访问过
	bool in(int index) {
		return already_arr[index] == 1;
	}

	//更新出发顶点到index顶点的距离
	void updateDis(int index, int len) {
		this->dis[index] = len;
	}

	//更新pre顶点的前驱顶点为index顶点
	void updatePre(int pre, int index) {
		this->pre_visited[pre] = index;
	}

	//返回出发顶点到index顶点的距离
	int getDis(int index) {
		return this->dis[index];
	}

	//继续访问未访问过的顶点
	int updateArr() {
		int min = 65535, index = 0;
		for (int i = 0; i < already_arr.size(); i++) {
			if (already_arr[i] == 0 && dis[i] < min) {
				min = dis[i];
				index = i;
			}
		}
		//更新index顶点被访问过
		already_arr[index] = 1;
		return index;
	}

	//显示最后的结果
	//即将三个数组的情况输出
	void show() {
		for (auto& it0 : already_arr) {
			cout << it0 << "\t";
		}
		cout << endl;
		for (auto& it1 : pre_visited) {
			cout << it1 << "\t";
		}
		cout << endl;
		for (auto& it2 : dis) {
			cout << it2 << "\t";
		}
		cout << endl;
	}
};

class Graph {
private:
	string vertrx;//顶点数组
	vector<vector<int>>matrix;//邻接矩阵
	VisitedVertex vv;//已经访问的顶点的集合
public:
	Graph(string vertrx, vector<vector<int>>& matrix) {
		this->vertrx = vertrx;
		this->matrix = matrix;
	}

	//显示图
	void showGraph() {
		for (auto& it0 : matrix) {
			for (auto& it1 : it0) {
				cout << it1 << "\t";
			}
			cout << endl;
		}
	}

	//Dijstra算法实现
	void dsj(int index) {//index：出发顶点
		vv=VisitedVertex(this->vertrx.length(), index);
		update(index);
		for (int j = 0; j < vertrx.length()-1; j++) {//因为出发顶点已经处理过了，所以处理的顶点数为总顶点数-1
			index = vv.updateArr();
			update(index);
		}
	}

	//更新index顶点到周围顶点的距离和周围顶点的前驱顶点
	void update(int index) {
		int len = 0;
		//根据遍历邻接矩阵的matrix[index]行
		for (int j = 0; j < matrix[index].size(); j++) {
			//len：出发顶点到index顶点的距离+从index顶点到j顶点的距离
			len = vv.getDis(index) + matrix[index][j];
			//如果j顶点没被访问过，并且len小于出发顶点到j顶点的距离，就需要更新
			if (!vv.in(j) && len < vv.getDis(j)) {
				vv.updatePre(j, index);//更新j顶点的前驱顶点为index顶点
				vv.updateDis(j, len);//更新出发顶点到j顶点的距离
			}
		}
	}

	void showDijstra() {
		vv.show();
	}
};

int main() {
	string vertrx = "ABCDEFG";

	int N = 65535;//表示不连通
	//邻接矩阵
	vector<vector<int>>matrix = {
		{N,5,7,N,N,N,2},
		{5,N,N,9,N,N,3},
		{7,N,N,N,8,N,N},
		{N,9,N,N,N,4,N},
		{N,N,8,N,N,5,4},
		{N,N,N,4,5,N,6},
		{2,3,N,N,4,6,N}
	};

	Graph graph(vertrx, matrix);

	graph.dsj(6);
	graph.showDijstra();
}

方法二：优化版（小根堆）
思路：
    1.设出发点到其他点的距离都为INT_MAX
    2.利用小根堆不断更新出发点到其他点的最小权重路径：不需要遍历了，直接访问堆顶即可
    3.系统的堆无法满足我们的要求：改变某一个值后重新排序变成小根堆
  
class Node;

class Edge {
public:
    int weight;//权重
    Node* from = NULL;//出发点
    Node* to;//终点

    Edge(int weight, Node* from, Node* to) {
        this->weight = weight;
        this->from = from;
        this->to = to;
    }
};

class Node {
public:
    int value;//结点值
    int in;//入度
    int out;//出度
    vector<Node*>nexts;//相邻结点
    vector<Edge>edges;//相邻边

    Node(int value) {
        this->value = value;
        in = 0;
        out = 0;
    }
};

class NodeRecord {
public:
    Node* node;
    int distance;

    NodeRecord(Node* node, int distance) {
        this->node = node;
        this->distance = distance;
    }
};

class NodeHeap {
private:
	vector<Node*>nodes;
    map<Node*, int>heapIndexMap;//存放结点对应索引
    map<Node*, int>distanceMap;//存放结点和起点的距离
    int size;

public:
    NodeHeap(int size) {
        nodes = vector<Node*>(size);
        this->size = 0;//设置为0，方便后续直接在nodes[size]插入新结点
    }

    bool isEmpty() {
        return size == 0;
    }

    void addOrUpdateOrIgnore(Node* node, int distance) {
        //update
        if (inHeap(node)) {
            distanceMap[node] = min(distanceMap[node], distance);
            insertHeapify(node,heapIndexMap[node]);
        }
        //add
        if (!isEntered(node)) {
            nodes[size] = node;
            heapIndexMap[node] = size;
            distance[node] = distance;
            insertHeapify(node,size++);
        }
        //ignore：什么也不做
    }

    NodeRecord pop() {
        NodeRecord nodeRecord(nodes[0], distanceMap[nodes[0]]);//记录堆顶信息
        swap(0, size - 1);
        heapIndexMap[nodes[size - 1]] = -1;
        distanceMap.erase(nodes[size - 1]);
        delete nodes[size - 1];
        //重新调整树结构
        heapify(0, --size);
        return nodeRecord;
    }

    void insertHeapify(Node*node,int index) {
        while (distanceMap[nodes[index]] < distanceMap[nodes[(index - 1) / 2]]) {
            swap(index, (index - 1) / 2);
            index = (index - 1) / 2;
        }
    }

    //从index开始调整
    void heapify(int index, int size) {
        int left = index * 2 + 1;
        int right = left + 1;
        //因为循环条件需要left，所以必须在循环之前就定义好left
        while (left < size) {
            //right要在冒号前，因为是&&，任意有一个不满足都会赋值后者，因此我们要保证“一致性”
            int smallest = right < size&& distanceMap[nodes[right]] < 
                distanceMap[nodes[left]] ? right : left;
            smallest = distanceMap[nodes[smallest]] < distanceMap[nodes[index]] ? smallest 
                : index;
            if (smallest == index)break;//如果父结点就是最小的，那么不用做任何处理
            swap(smallest, index);
            index = smallest;
            left = index * 2 + 1;
            right = left + 1;
        }
    }

    bool isEntered(Node* node) {
        return heapIndexMap.find(node) != heapIndexMap.end();
    }

    bool inHeap(Node* node) {
        return isEntered(node) && heapIndexMap[node] != -1;
    }

    void swap(int index1, int index2) {
        heapIndexMap[nodes[index1]] = index2;
        heapIndexMap[nodes[index2]] = index1;
        Node* tmp = nodes[index1];
        nodes[index1] = nodes[index2];
        nodes[index2] = tmp;
    }
};

map<Node*, int> djs2(Node* head, int size) {
    NodeHeap nodeHeap(size);
    nodeHeap.addOrUpdateOrIgnore(head, 0);
    map<Node*, int>res;
    while (!nodeHeap.isEmpty()) {
        NodeRecord record = nodeHeap.pop();
        Node* cur = record.node;
        int distance = record.distance;
        for (Edge& edge : cur->edges) {
            nodeHeap.addOrUpdateOrIgnore(edge.to, distance + edge.weight);
        }
        res[cur] = distance;//会不断更新成最小值，直到不能再更新时，nodeHeap会不断地pop而不会add了
    }
}
```
### 3.5 N皇后

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/9.png)

```c++
方法一：普通版
int num1(int n) {
	if (n < 1) {
		return 0;
	}
	vector<int>record(n);//表明在第i行的皇后在第几列
	return process1(0, record, n);
}

//判断i行j列的皇后是否有效
bool isValid(vector<int>& record, int i, int j) {//只用和i行之前的判断
	for (int k = 0; j < i; k++) {
		if (record[k] == j || abs(i - k) == abs(j - record[k]))return false;
	}
	return true;
}

//i表示目前来到了第i行
//record表示之前放过的皇后的位置
//n代表共有多少行
//返回值表示摆完所有的皇后，合理的摆法有多少种
int process1(int i, vector<int>& record, int n) {
	if (i == n)return 1;//终止行，i能到n，说明前面的都合法
	int res = 0;
	for (int j = 0; j < n; j++) {//测试第i行的所有列
		//找合理位置
		if (isValid(record, i, j)) {
			record[i] = j;
			res += process1(i + 1, record, n);
		}
	}
	return res;
}

方法二：常数优化版（利用位运算）
因为是位运算，所以尽量不要超过32位
int num2(int n) {
	if (n < 1||n>32) {
		return 0;
	}
	//生成一个二进制的数
	int limit = n == 32 ? -1 : (1 << n) - 1;
	//能在limit中，位上的值为1上尝试放皇后，且一开始没有限制，所以全为0
	return process2(limit, 0, 0, 0);
}

//colLim列的限制，1的位置不能放皇后，0的位置可以
//leftDiaLim左斜线的限制，1的位置不能放皇后，0的位置可以
//rightDiaLim右斜线的限制，1的位置不能放皇后，0的位置可以
int process2(int limit, int colLim, int leftDiaLim, int rightDiaLim) {
	if (colLim == limit) {//base case，colLim能递归到全为1，说明前面所有放置的皇后都合法
		return 1;
	}
	int mostRightOne = 0;
	// | 保留1，& 保留0，不要忘记最高位还有一个0，取反后变成1，需要让其和limit做&运算变回0
	int pos = limit & (~(colLim | leftDiaLim | rightDiaLim));//保留所有能够填皇后的列--1
	int res = 0;
	while (pos != 0) {
		//在最右侧放皇后
		mostRightOne = pos & (~pos + 1);
		pos = pos - mostRightOne;
		//同时还保留了之前行对下面行的限制，注：右移是逻辑右移，因为保证左边0不变
		res += process2(limit, colLim | mostRightOne, (leftDiaLim | mostRightOne) << 1, (rightDiaLim | mostRightOne) >> 1);
	}
	return res;
}
C++中，signed类型默认逻辑右移：即直接补0；unsigned类型默认算数右移：保留符号位0。重定义类型即可切换。
```


## 4.前缀树

何为前缀树？

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/2.png)

```c++
前缀树的功能：
    1.判断是否加入过某些字符串
    2.计算以某字符串为前缀的字符串有几个
    
//定义结点
class TrieNode {
public:
	//遍历所有字符串后
	int pass;//穿过该结点次数
	int end;//如果是终点，则end=1，从而可以用来判断是否加过某些字符串
	vector<TrieNode*>nexts;//存放该结点的后序结点，索引就是路
	//如果是多字符的情况，可以换成哈希表，key代表路，value代表路通往的结点

	TrieNode() {
		pass = 0;
		end = 0;
		//nexts[0]==NULL，没有走向a的路
		//nexts[0]!=NULL，有走向a的路
		nexts = vector<TrieNode*>(26);//a~z，路是提前建好的，通过指针是否为空来判断路的存在
	}
};

//定义前缀树
class Trie {
private:
	//根结点的pass值代表有多少个字符串以空串""作为前缀(所有字符串)，end代表有几个空串
	TrieNode* root;
	
public:
	Trie() {
		root = new TrieNode();
	}

	//插入字符串
	void insert(string word) {
		if (word.length() == NULL)return;
		TrieNode* node = root;
		node->pass++;
		int index = 0;
		for (int i = 0; i < word.length(); i++) {
			index = word[i] - 'a';//计算应走哪一条路
			if (node->nexts[index] == NULL)node->nexts[index] = new TrieNode();
			node = node->nexts[index];
			node->pass++;
		}
		//结束循环后，node指向word路径的终点
		node->end++;
	}

	//查询word这个单词加入过几次
	int search(string word) {
		if (word.length() == 0) {
			return root->end;
		}
		TrieNode* node = root;
		int index = 0;
		for (int i = 0; i < word.length(); i++) {
			index = word[i] - 'a';
			if (node->nexts[index] == NULL) {
				return 0;
			}
			node = node->nexts[index];
		}
		return node->end;
	}

	//所有加入的字符串中，有几个是以pre这个字符串作为前缀的
	int prefixNumber(string pre) {
		if (pre.length() == 0) {
			return root->end;
		}
		TrieNode* node = root;
		int index = 0;
		for (int i = 0; i < pre.length(); i++) {
			index = pre[i] - 'a';
			if (node->nexts[index] == NULL) {
				return 0;
			}
			node = node->nexts[index];
		}
		return node->pass;
	}

	void deleteWord(string word) {
		if (search(word) != 0) {//确认加入过word，才删除
			TrieNode* node = root;
			node->pass--;
			int index = 0;

			TrieNode* fatherNode = NULL;
			int deleteIndex = -1;
			set<TrieNode*>deleteSet;
			for (int i = 0; i < word.length(); i++) {
				index = word[i] - 'a';
				if (--node->nexts[index] == 0) {
					//记录第一个要删去结点的父节点
					fatherNode = fatherNode == NULL ? node : fatherNode;
					deleteIndex = deleteIndex == -1 ? index : deleteIndex;
					//记录所有要删去的结点
					deleteSet.insert(node->nexts[index]);
				}
				node = node->nexts[index];
			}
			node->end--;
			fatherNode->nexts[deleteIndex] = NULL;
			for (auto deleteNode : deleteSet) {
				deleteNode = NULL;
			}
		}
	}
};
删除结点时的误区：误以为只要将第一个pass=0的结点置空即可，实际上不对。因为后面还有结点pass=1，等待被删除，
    		   所以，应该把所有待删除结点放入到一个容器先，循环结束后再去遍历该容器一个一个置空结点。
```



## 5.贪心算法

### 5.1 定义

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/3.png)

### 5.2 1353. 最多可以参加的会议数目

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/4.png)

```c++
class Program {
public:
	int start;
	int end;

	Program(int start, int end) {
		this->start = start;
		this->end = end;
	}
};

bool programCompare(Program& p1, Program& p2) {
	return p1.end < p2.end;
}

int bestArrange(vector<Program>& programs, int timePoint) {
	sort(programs.begin(), programs.end(), programCompare);
	int res = 0;
	for (int i = 0; i < programs.size(); i++) {
		if (timePoint <= programs[i].start) {
			res++;
			timePoint = programs[i].end;
		}
	}
	return res;
}
```

### 5.3 解题套路

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/5.png)

**对数器：**对数器是通过用大量测试数据来验证算法是否正确的一种方式。

### 5.4 最小字典序

```c++
问题：将所有字符串拼接起来，如何保证拼接完之后的字符串字典序最低？
贪心策略：a.b<=b.a，a,b都为字符串
有效比较策略：原始数据相对位置无论怎么改变，排完序之后的数据位置是固定的
证明：
    To show:a.b<=b.a,b.c<=c.b => a.c<=c.a
    1. 把string看作k进制数，因为数才好比较=>a.b=a*k^b.length()+b
    2. k^b.length()=m(b)
    3. a.b<=b.a ：a*m(b)+b<=b*m(a)+a
    4. b.c<=c.b ：b*m(c)+c<=c*m(b)+b
    5. 由3、4可得：a*m(c)+c<=c*m(a)+a
    6. 以上只能证明策略可以排序出一个唯一序列
    7. 证明排好序后的序列任意两个字符串交换顺序，新的字符总串的字典序大于旧的
    8. 一路和相邻的字符串交换，利用数学归纳法
     
bool stringCompare(string a, string b) {
	return a + b <= b + a;
}

string lowestString(vector<string>& strs) {
	if (strs.size() == 0) {
		return "";
	}
	sort(strs.begin(), strs.end(), stringCompare);
	string res;
	for (int i = 0; i < strs.size(); i++) {
		res += strs[i];
	}
    return res;
}
```

### 5.5 分割金条

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/6.png)

```c++
赫夫曼编码
    
int lessMoney(vector<int>& vec) {
	priority_queue<int, vector<int>, greater<int>>pQ;
	for (int i = 0; i < vec.size(); i++) {
		pQ.push(vec[i]);
	}
	int sum = 0;
	int cur = 0;
	while (pQ.size() > 1) {
		cur = 0;
		cur += pQ.top();
		pQ.pop();
		cur += pQ.top();
		pQ.pop();
		sum += cur;
		pQ.push(cur);
	}
	return sum;
}
```

### 5.6 最大钱数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/7.png)

```c++
class Project {
public:
	int profit;
	int cost;

	Project(int profit, int cost) {
		this->profit = profit;
		this->cost = cost;
	}
};

class CostCompare {
public:
	bool compare(Project& p1, Project& p2) { return p1.cost < p2.cost; }
};

class ProfitCompare {
public:
	bool compare(Project& p1, Project& p2) { return p1.profit > p2.profit; }
};

int findMaximizedCapital(int k, int w, vector<int>& Profits, vector<int>& Capital) {
	priority_queue<Project, vector<Project>, CostCompare>minCostQ;//存放成本最小的
	priority_queue<Project, vector<Project>, ProfitCompare>maxProfitQ;//存放利润最大的
	for (int i = 0; i < Profits.size(); i++) {
		minCostQ.push(Project(Profits[i], Capital[i]));
	}
	for (int i = 0; i < k; i++) {
		while (!minCostQ.empty() && minCostQ.top().cost <= w) {
			maxProfitQ.push(minCostQ.top());
			minCostQ.pop();
		}
		if (maxProfitQ.empty())return w;//没有可以做的项目了
		w += maxProfitQ.top().profit;
		maxProfitQ.pop();
	}
	return w;
}
```

### 5.7 剑指 Offer 41. 数据流中的中位数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/8.png)

```c++
思路：
    1. 准备一个小根堆和大根堆
    2. 两个堆都为空时，默认放入大根堆
    3. 判断cur<=大根堆堆顶
       3.1 是：cur入大根堆
       3.2 否：cur入小根堆
    4. 判断两堆大小差距，超过2：较大的堆堆顶弹出放入另一个堆里
    5. 最终：较小的n/2个数在大根堆，较大的n/2个数在小根堆里

class MedianFinder{
public:
	priority_queue<int, vector<int>, less<int>>maxHeap;
	priority_queue<int, vector<int>, greater<int>>minHeap;

	void addNum(int num) {
		if (maxHeap.empty() || num <= maxHeap.top()) {
			maxHeap.push(num);
		}
		else {
			minHeap.push(num);
		}
		if (maxHeap.size() == minHeap.size() + 2) {
			minHeap.push(maxHeap.top());
			maxHeap.pop();
		}
		if (minHeap.size() == maxHeap.size() + 2) {
			maxHeap.push(minHeap.top());
			minHeap.pop();
		}
	}

	double findMedian() {
		int maxHeapSize=maxHeap.size();
        int minHeapSize=minHeap.size();
        if(((maxHeapSize+minHeapSize)&1)==0)return (maxHeap.top()+minHeap.top())*0.5;
        return maxHeapSize>minHeapSize?maxHeap.top():minHeap.top();
	}
};
```



## 6.暴力递归

### 6.1 定义

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/10.png)

### 6.2 打印字符串子序列/子集（无序）

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/11.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/12.png)

```c++
方法一：普通版剑指 Offer 38. 字符串的排列
void process(string str, int i, string res) {
	if (i == str.length()) {//base case
		cout<<res<<endl;
		return;
	}
	string resKeep = res;
	resKeep += str[i];
	process(str, i + 1, resKeep);//选择当前字符
	string resNoInclude = res;
	process(str, i + 1, resNoInclude);//不选择当前字符
}

方法二：省空间版
void process(string str, int i) {
	if (i == str.length()) {
		cout << str << endl;
		return;
	}
	process(str, i + 1);//选择当前字符
	char tmp = str[i];
	str[i] = '\0';
	process(str, i + 1);//不选择当前字符
	str[i] = tmp;//恢复
}
```

### 6.3 打印字符串全排列

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/13.png)

```c++
//str[i...]范围上，所有的字符都可以在i位置上，后续都可以去尝试
//str[0...i-1]范围上，是之前做的选择
//请把所有的字符串形成的全排列，加入到res中
void process(string str, int i, vector<string>& res) {
	if (i == str.length()) {
		res.push_back(str);
	}
	for (int j = i; j < str.length(); j++) {
		swap(str[i], str[j]);
		process(str, i + 1, res);
		swap(str[i], str[j]);
	}
}
```

### 6.4 打印字符串全排列（不重复）

```c++
void process(string str, int i, vector<string>& res) {
	if (i == str.length()) {
		res.push_back(str);
	}
    vector<bool>visit(26);//判断26个字母是否被访问过
	for (int j = i; j < str.length(); j++) {
        if(!visit[str[j]-'a']){
            visit[str[j]-'a']=true;
            swap(str[i], str[j]);
			process(str, i + 1, res);
			swap(str[i], str[j]);
        }
	}
}
```

### 6.5 拿牌

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/14.png)

```c++
int win1(vector<int>& arr) {
	return max(f(arr, 0, arr.size() - 1), s(arr, 0, arr.size() - 1));
}

//先手
int f(vector<int>& arr, int L, int R) {
	if (L == R)return arr[L];
    //先手有主动权，必定让自己拿到最多的分数
	return max(arr[L] + s(arr, L + 1, R), arr[R] + s(arr, L, R - 1));
}

//后手
int s(vector<int>& arr, int L, int R) {
	if (L == R)return 0;
    //后手被动，对手肯定会让我们获取最低分数
	return min(f(arr, L + 1, R), f(arr, L, R - 1));
}
```

### 6.6 逆序栈

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/15.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/16.png)

```c++
//取出栈底
int f(stack<int>&stk) {
	int result = stk.top();
	stk.pop();
	if (stk.empty())return result;
	else {
		int last = f(stk);
		stk.push(result);
		return last;
	}
}

void reverse(stack<int>&stk){
    if(stk.empty())return;
    int i=f(stk);
    reverse(stk);
    stk.push(i);
}
```

### 6.7 数转字符串

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/17.png)

```c++
思路：
    s[i]之前有多少种结果是已经确定的
    1. s[i]=0，只能尝试与s[i-1]组合
    2. s[i]=1，一定可以和s[i+1]组合
    3. s[i]=2，判断s[i+1]是否大于6
    
//i之前的位置如果转化已经做过判断了且有效
int process(string str, int i) {
	//能到最后一个字符说明s[i]之前的字符转化都是有效的，因此已经固定有1种了
	if (i == str.length())return 1;
	if (str[i] == '0')return 0;
	if (str[i] == '1') {
		int res = process(str, i + 1);//i作为单独的部分
		if (i + 1 < str.length()) {
			res += process(str, i + 2);//(i,i+1)看作一个整体
		}
		return res;
	}
	if (str[i] == '2') {
		int res = process(str, i + 1);
		if (i + 1 < str.length() && str[i + 1] >= '0' && str[i + 1] <= '6') {
			res += process(str, i + 2);
		}
		return res;
	}
	//str[i]=='3'~'9'
	return process(str, i + 1);
}
```

### 6.8 载物

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/18.png)

```c++
思路：从左往右一个一个试，max(选择，不选择)

int process(vector<int>&weights, vector<int>&values, int i, int alreadyweight,int alreadyvalue, int bag) {
	if (alreadyweight > bag)return 0;//超重，该方案根本不应该存在
	if (i == weights.size())return alreadyvalue;//越界
	return max(process(weights, values, i + 1, alreadyweight,alreadyvalue, bag),
		process(weights, values, i + 1, alreadyweight + weights[i], alreadyvalue+ values[i],bag));
}
```



## 7.哈希函数与哈希表等

### 7.1 40亿个数出现次数最多的数

```c++
思路：
    只允许1GB内存
    1.遍历数组，利用哈希函数计算哈希值，再取模100
    2.先把取模为0的数放到0号文件的哈希表(记录次数)中
    3.循环遍历数组，依次处理1~99号文件
    4.平均每次只需要用到[(2^35)/100]GB
    5.使用哈希函数的目的：让值平均分布
```

### 7.2 随机返回池

```c++
template<typename K>
class RandomPool{
private:
	map<K, int>keyIndexMap;
	map<int, K>indexKeyMap;
	int size;
public:
	RandomPool() {
		size = 0;
	}

	void insert(K key) {
		if (keyIndexMap.find(key) == keyIndexMap.end()) {
			keyIndexMap[key] = size;
			indexKeyMap[size++] = key;
		}
	}

	void deleteKey(K key) {
		if (keyIndexMap.find(key) != keyIndexMap.end()) {
			int deleteIndex = keyIndexMap[key];
			int lastIndex = --size;
			K lastKey = indexKeyMap[lastIndex];
			keyIndexMap[lastKey] = deleteIndex;
			indexKeyMap[deleteIndex] = lastKey;
			keyIndexMap.erase(key);
			indexKeyMap.erase(lastIndex);
		}
	}

	//等概率随机返回
	K getRandom() {
		if (size == 0)_Throw_range_error("容器为空！");
		srand((unsigned)time(NULL));
		int randomIndex = rand() % size;
		return indexKeyMap[randomIndex];
	}
};
```

### 7.3 布隆过滤器

```c++
作用：通过查询黑名单，判断是否在黑名单里。还有爬虫驱虫
只需要实现加入和查询功能，空间占用少且允许一定失误率
思路：
    1.创建位图：bit map。即一个位图的格子占用一个bit位
    2.用基础类型实现：int[10]可储存320位信息
    
int arr[10];
int i=178;
//1.取第178位信息
int numIndex=178/32;//计算在哪个格子的数
int bitIndex=178%32;//计算在该数的第几位
int s=((arr[numIndex]>>(bitIndex))&1);//第178位的信息
//2.把第i位的状态改成1
arr[numIndex]=arr[numIndex]|(1<<(bitIndex));
//3.把第i位的状态改成0
arr[numIndex]=arr[numIndex]&(~(1<<bitIndex));

流程：每个URL通过k个哈希函数，对相应的位描黑，因此可能会把白判成黑，但黑一定是黑。
   
问题：什么时候可以用布隆过滤器
已有条件：n=样本量，p=失误率
1.看需不需要有删除行为， 2.看允不允许有失误率
样本大小无关紧要
m=-(n*lnp)/((ln2)^2)(bit)
k=ln2*(m/n)=0.7*(m/n)(个)，向上取整
p真=(1-e^(-(n*k真)/m真))^k真
```

### 7.4 一致性哈希原理

```c++
1.解决数据服务器怎么组织的问题
2.让高频中频低频都能作数据的划分，所以要选择合适的key使其能够达到均分
3.把存储的机器连接起来形成一个哈希环，经过哈希函数后，数据按顺时针的方式选择最近的机器
4.因为机器均分环，所以数据刚好均分
5.增加机器和迁出机器的迁移量低，因为只需向顺时针的第一个机器要数据即可
6.问题：
  6.1机器很少时，难把环均分
  6.2增加机器后，负载不均衡
7.用虚拟节点技术解决问题：
  3台机器去抢环上的点，即圈环上任一处，各机器的点各占 1/3；机器4加入时，去夺环上的点，这样夺的点中，另三台
   机器的点等分，就能做到环上的点被4台机器均分；机器4下机后，也是等比例的把点归还给其他3台数据。
8.管理负载：负载能力强的机器负责更多的点
```



## 8.有序表、并查表等

### 8.1 岛问题（并查集）

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/19.png)

```c++
方法一：普通版
思路：
    利用infect函数把一片1统一变成2，本质上是记录访问过的1
 
//"感染"函数
/*
* vec：必须传引用，因为修改的值需要保存，并且引用的是原容器的副本，所以不会修改原先内容
* i：行
* j：列
* m：行界限
* n：列界限
*/
void infect(vector<vector<int>>& vec, int i, int j, int m, int n) {
	if (i < 0 || i >= m || j < 0 || j >= m || vec[i][j] != 1)return;
	//vec[i][j]=1，修改成2
	vec[i][j] = 2;
    //由于for循环遍历的方向是从左到右，从上到下，因此每个1必定是从它的左边或上边抵达，因此只需向右和向下走
	infect(vec, i, j + 1, m, n);
	infect(vec, i + 1, j, m, n);
}
递归时间复杂度： O(N*M)

int countIslands(vector<vector<int>>vec) {//不传引用避免修改原容器里的内容
	int m = vec.size();//行
	int n = vec[0].size();//列
	int res = 0;//岛的数量
	for (int i = 0; i < m; i++) {
		for (int j = 0; j < n; j++) {
			if (vec[i][j] == 1) {
				//到达1处才开始感染，每调用一次感染岛数＋1
				res++;
				infect(vec, i, j, m, n);
			}
		}
	}
	return res;
}


并查集：向上指的结构
//样本进来包一层，叫做元素
template<typename V>
class Element{
public:
	V value;
	Element(V value) {
		this->value = value;
	}
};

//定义并查集结构
template<typename V>
class UnionFindSet {
public:
	//key元素对应value元素"圈"
	map<V, Element<V>>elementMap;
	//value元素是key元素的父
	map<Element<V>, Element<V>>fatherMap;
	//key元素是某个集合的代表元素，value是该集合的大小
	map<Element<V>, int>sizeMap;
	//并查集在构建时要求用户把样本都传过来
	UnionFindSet(list<V>ls) {
		for (V value : list) {
			Element<V>element = new Element<V>(value);
			elementMap[value] = element;
			fatherMap[element] = element;
			sizeMap[element] = 1;
		}
	}

	Element<V>findHead(Element<V>element) {
		stack<Element<V>>path;
		while (element != fatherMap[element]) {
			path.push(element);
			element = fatherMap[element];
		}
		//把结构变成扁平的，降低高度
		//不用考虑每个元素集合size的变化，因为改变的元素都不是代表元素
		while (!path.isEmpty()) {
			fatherMap[path.top] = element;
			path.pop();
		}
		return element;
	}

	bool isSameSet(V a, V b) {
		if (elementMap.find(a) != elementMap.end() && elementMap.find(b) != elementMap.end()) {
			return findHead(elementMap[a]) == findHead(elementMap[b]);
		}
		return false;
	}

	void union(V a, V b) {
		if (elementMap.find(a) != elementMap.end() && elementMap.find(b) != elementMap.end()) {
			Element<V>aF = findHead(elementMap[a]);
			Element<V>bF = findHead(elementMap[b]);
			if (aF != bF) {
				Element<V>big = sizeMap[aF] > sizeMap[bF] ? aF : bF;
				Element<V>small = big == aF ? bF : aF;
				fatherMap[small] = big;
				sizeMap[big] += sizeMap[small];
				sizeMap.erase(small);
			}
		}
	}
};

方法二：并行算法
问题1：切开图分配给不同的CPU之后，可能会把原本连通的切断。
解决1：记录切割边边界点的感染源。不同的感染源感染到的点分配在不同集合中，最后通过能否合并集合来算出实际岛数
```

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/20.png)

## 9.KMP、Manacher算法

### 9.1 KMP

```c++
解决str2是否为str1子串且如果是子串从str1哪个位置开始的问题
暴力方法时间复杂度： O(N*M)，N、M分别为str1和str2长度
KMP：
    思路：
    1.找str[i]字符前的字符串前缀后缀的最大匹配长度，且不能取整体
    2.需要str2每个字符的信息(该字符前的字符串的前缀后缀的最大匹配长度)放入next数组中
    3.人为规定next[0]=-1，next[1]=0
	4.实际代码过程中不是把str2[0]向后移，而是让str2中不同的字符索引Y=next[Y](跳到前缀的后一个字符)，但
      逻辑上相同
    
int kmp(string str1, string str2) {
	int i1 = 0;//str1中比对的位置
	int i2 = 0;//str2中比对的位置
	vector<int>next = getNext(str2);//O(M)
	//O(N)
	while (i1 < str1.length() && i2 < str2.length()) {
		if (str1[i1] == str2[i2]) {
			i1++;
			i2++;
		}else if(i2==0) {//next[i2]==-1
			//说明第一个字符就不等，str2中比对的位置已经无法往前跳了，让str1换一个开头去匹配
			i1++;
		}
		else {
			//str2比对的位置跳到前缀的后一个字符
			i2 = next[i2];
		}
	}
	//i1越界 或者 i2越界
	//如果匹配成功，i1到达str1中匹配的最后一个字符，i2也到达str2中最后一个字符，start=i1-i2.length()+1=i1-i2
	return i2 == str2.length() ? i1 - i2 : -1;
}

vector<int>getNext(string str) {
	if (str.length() == 1) {
		return { -1 };
	}
	vector<int>next(str.length());
	next[0] = -1;
	int i = 2;
	int cn = 0;//cn是用来和i-1字符比较的字符索引，且cn恰好等于next[i-1]
	while (i < next.size()) {
		if (str[i - 1] == str[cn]) {
			next[i++] = ++cn;
		}
		else if (cn > 0) {
			cn = next[cn];
		}
		else {
			next[i++] = 0;
		}
	}
	return next;
}
```

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/21.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/22.png)

### 9.2 Manacher算法

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/23.png)

```c++
1.经典方法：
依次以每个字符为中心，向两边扩，两边字符相同就继续向外扩，直至两边字符不同。
问题：当回文子串长度为偶数时，该方法失效。
解决：在字符串两边和每两个字符之间添加特殊符号'#'(并不一定要与原字符串的字符不同，因为都
     是"实"与"实"比，"虚"与"虚"比)，再依次遍历新字符去扩。
时间复杂度： O(n^2)

2.Manacher算法：
回文直径：从中心出发向两边扩，包含的子串长度。
回文半径：回文直径的一半。
(1)创建回文半径数组，每个字符都有回文半径。
(2)右边界(int)R记录每次扩到的最右边界，如果比上次扩得远则更新。
(3)中心点(int)C记录由哪个点扩到最右边界，和R同步更新。
(4)分类讨论：假设开始扩展的点为str[i]，以下我们简称为i
    4.1 i>=R：直接暴力扩(向两边扩)，无优化。
    4.2 i<R：
    	左边界L：R关于C的对称点
    	i关于C的对称点j
    	4.2.1 j的回文区域彻底属于(L,R)：i的回文区域长度与j相同
    	4.2.2 j的回文区域的左半区域部分在L左边：i的回文半径=R-i
    	4.2.3 j的回文区域左边界=L(压线)：至少有一个回文区域，但是不确定会不会更大。

string manacherString(string str) {
	string res;
	int index = 0;
	for (int i = 0; i < str.length() * 2 + 1; i++) {
		char c = (i & 1) == 0 ? '#' : str[index++];
		res.push_back(c);
	}
	return res;
}

int maxLcpsLength(string s) {
	if (s.length() == 0)return 0;
	string str = manacherString(s);
	vector<int>pArr(str.length());
	int C = -1;
	int R = -1;
	int maxLen = INT_MIN;
	for (int i = 0; i != str.length(); i++) {
		//先算出每种情况下不用至少不用验的区域，再去扩展
		pArr[i] = R > i ? min(pArr[2 * C - i], R - i) : 1;
		while (i + pArr[i]<str.length() && i - pArr[i]>-1) {
			if (str[i + pArr[i]] == str[i - pArr[i]]) pArr[i]++;
			else break;
		}
		if (i + pArr[i] > R) {
			R = i + pArr[i];
			C = i;
		}
		maxLen = max(maxLen, pArr[i]);
	}
	return maxLen - 1;
}
时间复杂度分析：
由于四个分支中，i都是增加，R要么增加要么不变，总增长幅度是2n，所以时间复杂度是 O(n)
Manacher算法保证了i和R都是增长状态，所以可以把时间复杂度从O(n^2)优化到O(n)
```

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/24.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/25.png)

### 9.3 窗口更新最大最小值

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/26.png)

```c++
窗口：
    1. 左边界L、右边界R只能向右移动
    2. L<R
    3. 可以选择L或R二者之一往右动
    
思路：利用双端队列实现：存索引（即能知道值，还能知道索引）
    1. R往右走时，从尾部添加新元素，并保证从头到尾严格单调递增，不满足就删掉队尾元素
    2. L往右走时，如果队头元素与过期元素相同则删去，否则不管
    3. 下标大且对应的值也大，那么就可以把前面比它小的元素删去了，因为前面小的元素再也没可能成为最大值了
       原因：因为索引大且值大的元素比前面那些元素晚过期
    
双端队列维持的信息：如果目前窗口不再添加新元素，而L依次往右动，谁会依次成为最大值，最大值优先级信息。

最大值：
vector<int>getMaxWindow(vector<int>vec, int w) {
	if (vec.size() == 0 || w < 1 || vec.size() < w)return {};
	deque<int>qmax;
	vector<int>res(vec.size() - w + 1);
	int index = 0;
	for (int i = 0; i < vec.size(); i++) {
		while (!qmax.empty() && vec[qmax.back()] <= vec[i]) {//不满足严格递减的删去
			qmax.pop_back();
		}
		//删掉一定数量的元素后加入新元素
		qmax.push_back(vec[i]);
		if (qmax.front() == i - w) {//因窗口右移过期
			qmax.pop_front();
		}
		//i至少遍历到索引为2的元素窗口才处理过3个元素
		if (i >= w - 1) {
			res[index++] = vec[qmax.front()];
		}
	} 
	return res;
}
最小值：双端队列从头到尾严格递增，逻辑与求最大值类似。

时间复杂度分析：
每个元素进出队列最多各一次，所以是 O(n)
```

### 9.4 单调栈

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/27.png)

```c++
左右两边离得最近的比当前值大的元素的索引：
思路：
    1.创建一个栈，从底到顶按从小到大排序，添加的元素对应的索引。
    2.如果准备添加的元素不符合顺序，那么就删栈顶，同时我们知道栈里所有数右边离得最近且比这个数小的元素就是
      准备新加的元素，左边离得最近且比这个数小的元素就是该元素在栈中底下的那一个元素。
    3.遍历结束后，若栈中还有数，就进入清算阶段，弹出所有元素并处理。右边都没有比该元素小的元素。
    
//有重复元素的，把重复元素的索引按顺序放入vector中
vector<vector<int>>getNearLess(vector<int>vec) {
	//创建二维数组，第一列存放左边最近小数索引，第二列存放右边最近小数索引
	vector<vector<int>>res(vec.size(), vector<int>(2));
	//单调栈
	stack<vector<int>>stk;
	//遍历数组
	for (int i = 0; i < vec.size(); i++) {
		//如果栈不为空，且待添加数不符合单调性，那就要删去栈顶，同时更新删去元素的信息
		while (!stk.empty() && vec[stk.top()[0]] > vec[i]) {
			vector<int>popIs = stk.top();
			stk.pop();
			//stk.top()[stk.top().size() - 1]，相同元素的索引中最后一个加进来的。
			int leftLessIndex = stk.empty() ? -1 : stk.top()[stk.top().size() - 1];
			//可能有多个索引，所以要遍历完
			for (int popi : popIs) {
				res[popi][0] = leftLessIndex;
				res[popi][1] = i;
			}
		}
		if (!stk.empty() && vec[stk.top()[0]] == vec[i]) {
			stk.top().push_back(i);
		}
		else {
			vector<int>indexs;
			indexs.push_back(i);
			stk.push(indexs);
		}
	}
	//清算
	while (!stk.empty()) {
		vector<int>popIs = stk.top();
		stk.pop();
		int leftLessIndex = stk.empty() ? -1 : stk.top() [stk.top().size() - 1];
		for (int popi : popIs) {
			res[popi][0] = leftLessIndex;
			res[popi][1] = -1;
		}
	}
	return res;
}
时间复杂度分析：
每个元素进栈出栈的次数都各位1次，所有时间复杂度是 O(n)
```

### 9.5 最大指标

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/28.png)

```c++
思路：单调栈
    1.遍历数组，利用单调栈的信息，求每次以vec[i]作为最小值的子数组（范围是vec[i]左边最近小数到右边最近小
      数）的指标，并记录遍历过程中的最大指标。

int max2(vector<int>vec) {
	int size = vec.size();
	vector<int>sums(size);
	sums[0] = vec[0];
	for (int i = 1; i < size; i++) {
		sums[i] = sums[i - 1] + vec[i];
	}
	int maxVal = INT_MIN;
	stack<int>stk;
	for (int i = 0; i < size; i++) {
		while (!stk.empty() && vec[stk.top()] >= vec[i]) {
			int j = stk.top();
			stk.pop();
			//栈为空，说明弹出的元素左边没有比它更小的了
			maxVal = max(maxVal, (stk.empty() ? sums[i - 1] : (sums[i - 1] - sums[stk.top()])) * vec[j]);
		}
		stk.push(i);
	}
	while (!stk.empty()) {
		int j = stk.top();
		stk.pop();
		//栈为空，说明弹出的元素左边没有比它更小的了
		maxVal = max(maxVal, (stk.empty() ? sums[size - 1]: (sums[size - 1] - sums[stk.top()])) * vec[j]);
	}
	return maxVal;
}
```



## 10.二叉树的Morris遍历等

### 10.1 二叉树结点间的最大距离

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/29.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/31.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/30.png)

```c++
最大距离分析：
    1.根结点不参与
      1.1最大距离在根结点左树
      1.2最大距离在根结点右树
    2.根结点参与
      从根结点左树最远的结点到右树最远的结点：由高度决定
    3.最大距离=上述3种情况的最大值

class Node {
public:
	int value;
	Node* left;
	Node* right;
	Node(int data) {
		this->value = data;
	}
};

class Info {
public:
	int maxDistance;
	int height;
	Info(int dis, int h) {
		maxDistance = dis;
		height = h;
	}
};

//返回以x为头的整棵树，两个信息
Info process(Node* x) {
	if (x == NULL)return Info(0, 0);
	Info leftInfo = process(x->left);
	Info rightInfo = process(x->right);
	//info
	int p1 = leftInfo.maxDistance;
	int p2 = rightInfo.maxDistance;
	int p3 = leftInfo.height + 1 + rightInfo.height;
	int maxDistance = max(p3, max(p1, p2));
	int height = max(leftInfo.height, rightInfo.height) + 1;
	return Info(maxDistance, height);
}

int maxDistance(Node* head) {
	return process(head).maxDistance;
}
```

### 10.2 派对的最大快乐值

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/32.png)

```c++
可能性分类：每个员工在上级的影响下决定来与不来，所以分成两种情况即可。

class Info {
public:
	int laiMaxHappy;
	int buMaxHappy;
	Info(int lai, int bu) {
		laiMaxHappy = lai;
		buMaxHappy = bu;
	}
};

class Employee {
public:
	int happy;
	vector<Employee>nexts;//直接下属
};

Info process(Employee x) {
	if (x.nexts.empty())return Info(x.happy, 0);//x是基层员工
	int lai = x.happy;//x来的情况下，整棵树的最大收益的初始值
	int bu = 0;//x不来的情况下，整棵树最大收益的初始值
	//遍历直接下属，提取信息
	for (Employee next : x.nexts) {
		//取一名下属员工
		Info info = process(next);
		//如果x来，那么x的直系下属就不来
		lai += info.buMaxHappy;
		//如果x不来，x的直系下属可来也可不来，取最大值
		bu += max(info.laiMaxHappy, info.buMaxHappy);
	}
	return Info(lai, bu);
}

int maxHappy(Employee boss) {
	Info headInfo = process(boss);
	return max(headInfo.laiMaxHappy, headInfo.buMaxHappy);
}
递归实际上就是暴力地罗列出所有可能性(可能性是我们提前想出来的)，然后在所有情况中选择自己想要的，由局部组合成
整体。
```

### 10.3 Morris遍历

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/33.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/34.png)

```c++
class Node {
public:
	int value;
	Node* left;
	Node* right;
	Node(int val){
		this->value = value;
		this->left = NULL;
		this->right = NULL;
	}
};

void morris(Node* head) {
	if (head == NULL)return;
	Node* cur = head;
	Node* mostRight = NULL;
	while (cur != NULL) {//流程
		mostRight = cur->left;//mostRight暂时作cur的左孩子
		if (mostRight != NULL) {//有左子树
			//由于会手动更改右指针，所以多加一个判断条件
			while (mostRight->right != NULL && mostRight != cur) {
				mostRight = mostRight->right;
			}
			//mostRight变成了cur左子树最右的节点
			if (mostRight->right == NULL) {
				//第一次来到cur节点
				mostRight->right = cur;
				cur = cur->left;
				continue;
			}
			else {
				//第二次来到cur节点，恢复指针
				mostRight->right = NULL;
			}
		}
		cur = cur->right;
	}
}

先序：
    1.只经过一次的节点，直接打印
    2.经过两次的节点，只有第一次才打印
void morrisPre(Node* head) {
	if (head == NULL)return;
	Node* cur = head;
	Node* mostRight = NULL;
	while (cur != NULL) {//流程
		mostRight = cur->left;//mostRight暂时作cur的左孩子
		if (mostRight != NULL) {//有左子树，说明会遍历两次该节点
			//由于会手动更改右指针，所以多加一个判断条件
			while (mostRight->right != NULL && mostRight != cur) {
				mostRight = mostRight->right;
			}
			//mostRight变成了cur左子树最右的节点，第一次遍历时才打印
			if (mostRight->right == NULL) {
				//第一次来到cur节点
				cout << cur->value << " ";
				mostRight->right = cur;
				cur = cur->left;
				continue;
			}
			else {
				//第二次来到cur节点，恢复指针
				mostRight->right = NULL;
			}
		}
		else {//没有左子树的情况，说明只会遍历一次该节点，直接打印
			cout << cur->value << " ";
		}
		cur = cur->right;
	}
}

中序：
    1.只经过一次的节点，直接打印
    2.经过两次的节点，只有第二次才打印
void morrisIn(Node* head) {
	if (head == NULL)return;
	Node* cur = head;
	Node* mostRight = NULL;
	while (cur != NULL) {//流程
		mostRight = cur->left;//mostRight暂时作cur的左孩子
		if (mostRight != NULL) {//有左子树，说明会遍历两次该节点
			//由于会手动更改右指针，所以多加一个判断条件
			while (mostRight->right != NULL && mostRight != cur) {
				mostRight = mostRight->right;
			}
			//mostRight变成了cur左子树最右的节点
			if (mostRight->right == NULL) {
				//第一次来到cur节点
				mostRight->right = cur;
				cur = cur->left;
				continue;
			}
			else {
				//第二次来到cur节点，恢复指针
				mostRight->right = NULL;
			}
		}
		else {//没有左子树的情况，说明只会遍历一次该节点，直接打印
			cout << cur->value << " ";
		}
		//第二次遍历完之后跳出循环直接打印该节点
		cout << cur->value << " ";
		cur = cur->right;
	}
}

后序：
    1.只有在第二次遍历到能被二次遍历的节点时才做事：逆序打印其左树的右边界
    2.最后单独逆序打印整棵树的右边界
Node* reverseEdge(Node* from) {
	Node* pre = NULL;
	Node* next = NULL;
	while (from != NULL) {
		next = from->right;
		from->right = pre;
		pre = from;
		from = next;
	}
	return pre;
}

//以x为头的树，逆序打印这棵树的右边界
void printEdge(Node* x) {
	Node* tail = reverseEdge(x);
	Node* cur = tail;
	while (cur != NULL) {
		cout << cur->value << " ";
		cur = cur->right;
	}
	reverseEdge(tail);
}

void morriPos(Node* head) {
	if (head == NULL)return;
	Node* cur = head;
	Node* mostRight = NULL;
	while (cur != NULL) {//流程
		mostRight = cur->left;//mostRight暂时作cur的左孩子
		if (mostRight != NULL) {//有左子树，说明会遍历两次该节点
			//由于会手动更改右指针，所以多加一个判断条件
			while (mostRight->right != NULL && mostRight != cur) {
				mostRight = mostRight->right;
			}
			//mostRight变成了cur左子树最右的节点
			if (mostRight->right == NULL) {
				//第一次来到cur节点
				mostRight->right = cur;
				cur = cur->left;
				continue;
			}
			else {
				//第二次来到cur节点，恢复指针
				mostRight->right = NULL;
				//逆序打印左树右边界
				printEdge(cur->left);
			}
		}
		cur = cur->right;
	}
	//单独逆序打印整棵树右边界
	printEdge(head);
}

判断搜索二叉树：
bool isBST(Node* head) {
	if (head == NULL)return true;
	Node* cur = head;
	Node* mostRight = NULL;
	int preValue = INT_MIN;
	while (cur != NULL) {//流程
		mostRight = cur->left;//mostRight暂时作cur的左孩子
		if (mostRight != NULL) {//有左子树，说明会遍历两次该节点
			//由于会手动更改右指针，所以多加一个判断条件
			while (mostRight->right != NULL && mostRight != cur) {
				mostRight = mostRight->right;
			}
			//mostRight变成了cur左子树最右的节点
			if (mostRight->right == NULL) {
				//第一次来到cur节点
				mostRight->right = cur;
				cur = cur->left;
				continue;
			}
			else {
				//第二次来到cur节点，恢复指针
				mostRight->right = NULL;
			}
		}
		if (cur->value <= preValue)return false;
		preValue = cur->value;
		cur = cur->right;
	}
}
```

**最优解：**

如果要做第三次信息的强整合，用递归；否则，用Morris



## 11.大数据题目等

**大数据题目的解题技巧：**

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/36.png)

### 11.1 未出现过的数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/35.png)

```c++
1.需要2^32个bit去记录每个数存在与否，如果存在在就让相应bit位的值改为1
2.进阶：3KB
3KB/4=750B，向下取整就是512=2^9，申请数组int arr[512]，记录词频
将2^32个数分成512组，int len=2^32/2^9，遍历文件数组，arr[nums[i]/len]++，由于只有40亿个数，所以必定
有的小于len，那么就在该小于len的范围上继续分成512份，直到找到一个数为止。
```

### 11.2 所有重复的URL

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/37.png)

```c++
补充题：
    1.创建若干个大根堆(按照词频排序)
    2.取所有大根堆的堆顶放入总堆(大根堆)
    3.取总堆堆顶(剩余词汇中词频最大)放入返回容器中
    4.找到刚刚被取出总堆的所在原堆，并在其中删去它，然后取该堆的新堆顶放入总堆中，周而复始，直到返回容器中
      有100个词汇
```

### 11.3 找数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/38.png)

```c++
思路：
    1.用两个bit记录x出现得次数：00(0次),01(1次),10(2次),11(>2次)
    2.大小：((2^32)*2)/8=2^30(Byte)=1GB
    
补充题：
    1.10KB/4B=2500B，向下取整就是2048=2^11，申请数组int arr[2048]，记录每个范围出现次数
    2.将2^32个数分成2048组，int len=2^32/2^11，遍历文件数组，arr[nums[i]/len]++
    3.去找第二十亿个数所在范围即可，然后再分组，直到找到第二十亿个数
```

### 11.4 无序变有序

```c++
将10GB无序数通过5GB内存转变成有序数：5GB/16=5*2^26=>2^28
创建大根堆，遍历数组，每次都找最小的2^28个数，记录数和对应的词频，然后按照词频放入文件中，最多循环2^6次
```

### 11.5 位运算

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/39.png)

```c++
//1->0
//0->1
int flip(int n) {
	return n ^ 1;
}

//n是非负数，返回1
//n是负数，返回0
int sign(int n) {
	return flip((n >> 31) & 1);
}

//考虑了c溢出
int getMax(int a, int b) {
	int c = a - b;
	int sa = sign(a);
	int sb = sign(b);
	int sc = sign(c);
	int difSab = sa ^ sb;//a和b符号不一样，为1；一样，为0
	int sameSab = flip(difSab);//a和b符号一样，为1；不一样，为0
	int returnA = difSab * sa + sameSab * sc;//返回a：a、b同号&&a-b>=0 || a，b不同号&&a>0
	int returnB = flip(returnA);//返回b：不返回a就一定返回b
	return a * returnA + b * returnB;
}
```

### 11.6 判断一个32位正数是不是2的幂、4的幂

```c++
2的幂：x中只能右一个1
方法一：找最右1与原数对比
x最右1：x&(~x+1)
bool is2Power(int x){
    return x==(x&(~x+1));
}

方法二：x-1
bool is2Power(int x){
    return (x&(x-1))==0;
}

4的幂：x是4的幂必然也是2的幂
1.判断x是否只有一个1
2.1只能在 0/2/4/8/.../2n位上
3.x&0101010101010101...01：即让x和在偶数位上全是1的数作与运算，如果不等于0，那么就是4的幂；否则不是。
bool 4Power(int x){
    return (x&(x-1)==0)&&(x&0x55555555)!=0;
}
```

### 11.7 a和b的加减乘除

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/40.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/41.png)

```c++
1.加法：
a^b：a和b无进位相加
(a&b)<<1：a和b产生的进位
(a^b)^(a&b),(a^b)&((a&b)<<1)
不断重复上述操作直到产生的进位等于0，结果就出来了
int add(int a, int b) {
	int sum = a;
	while (b != 0) {
		sum = a ^ b;//无进位相加的结果
		b = (a & b) << 1;//进位信息
		a = sum;
	}
	return sum;
}

2.减法：减数取反加1，放入加法中实现
int negNum(int n) {
	return add(~n, 1);
}

int minus(int a, int b) {
	return add(a, negNum(b));
}

3.乘法：
int multi(int a, int b) {
	int res = 0;
	while (b != 0) {
		if ((b & 1) != 0) {
			res = add(res, a);
		}
		a <<= 1;//a左移，b的1一直处于最低位，实际上a应该是在和b的高位1相乘，所以a要左移扩大
		b >>= 1;//b右移
	}
	return res;
}

4.除法：a/b
让b左移但不超过a：0111100/0000101，让b左移3位，00001也相应左移3位：01000，意味着0111100可以有01000个
000101。但实际代码是让a右移，因为右移更安全，左移可能会溢出。
x=a-b：让b左移但不超过x，直到减完b之后等于0。
bool isNeg(int n) {
	return n < 0;
}

int divide(int a, int b) {
	int x = isNeg(a) ? negNum(a) : a;
	int y = isNeg(b) ? negNum(b) : b;
	int res = 0;
	for (int i = 31; i > -1; i = minus(i, 1)) {
		if ((x >> i) >= y) {//左移y可能会溢出，x右移更安全
			res |= (1 << i);
			x = minus(x, y << i);
		}
	}
	return isNeg(a) ^ isNeg(b) ? negNum(res) : res;
}
```

## 12.暴力递归

### 12.1 机器人移动

**问题：**

int N：N个格子(1~N)

int s：开始位置

int E：结束位置

int k：机器人必须走k步

机器人移动方式：向左或向右一格，但不能越界

求：总共有几种方式，用k步从s到e？

```c++
方法一：普通版
/*
N：一共是N个位置
E：终点
rest：还剩rest步要走，rest初始值其实就是k
cur：当前位置
反回方法数
*/
int f1(int N, int E, int rest, int cur) {
	if (rest == 0) {
		return cur == E ? 1 : 0;
	}
	//防止越界，强行干预机器人移动
	if (cur == 1) return f1(N, E, rest - 1, 2);
	if (cur == N)return f1(N, E, rest - 1, N - 1);
	//中间位置，机器人即可向左走也可向右走
	return f1(N, E, rest - 1, cur - 1) + f1(N, E, rest - 1, cur + 1);
}

int walkWays1(int N, int E, int S, int K) {
	return f1(N, E,K,S);
}
方法一的问题：由于f1函数只和rest、cur这两个变量有关，而每个格子（除边界格子）又可能从左边或右边来，因此子
    		过程可能会有重复的f1函数。
时间复杂度分析：
机器人既可往左也可往右，类似二叉树，总共有k步，所以是一棵高度为k的二叉树，那么时间复杂度就是 O(2^k)

方法二：优化版(利用可变参数做二维表)
int f2(int N, int E, int rest, int cur,vector<vector<int>>&dp) {
	if (dp[rest][cur] != -1) {//如果算过，直接返回算过的返回值
		return dp[rest][cur];
	}
	//缓存未命中
	if (rest == 0) {
		dp[rest][cur] = cur == E ? 1 : 0;
		return dp[rest][cur];
	}
	//防止越界，强行干预机器人移动
	if (cur == 1) {
		dp[rest][cur] = f2(N, E, rest - 1, 2, dp);
	}
	else if (cur == N) {
		dp[rest][cur] = f2(N, E, rest - 1, N-1, dp);
	}
	else {//中间位置，机器人即可向左走也可向右走
		dp[rest][cur]= f2(N, E, rest - 1, cur - 1, dp) + f2(N, E, rest - 1, cur + 1, dp);
	}
	return dp[rest][cur];
}

int walkWays2(int N, int E, int S, int K) {
	vector < vector<int>>dp(K + 1, vector<int>(N + 1,-1));
	return f2(N, E,K,S,dp);
}
时间复杂度分析：
dp的大小是k*N，而求每个dp的时间复杂度是 O(1)，因此，总时间复杂度就是 O(k*N)
```

### 12.2 最少硬币数

给定不同面额的硬币 coins (硬币数各只有一个)和一个总金额 amount。编写一个函数来计算可以凑成

总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。

```c++
方法一：普通版
//返回组成rest的最少硬币数
int process1(vector<int>& coins, int index, int rest) {
	if (rest < 0) {//该枚硬币不可取，删去
		return -1;
	}
	if (rest == 0) {//该枚硬币取完之后刚好组成aim，但因为已经算过该枚硬币了，所以这里返回0
		return 0;
	}
	if (index == coins.size()) {//越界，因此多算了一枚，要删去
		return -1;
	}
	int p1 = process1(coins, index + 1, rest);//不取
	int p2Next = process1(coins, index + 1, rest - coins[index]);//取
	if (p1 == -1 && p2Next == -1) {
		return -1;//无效解
	}
	else {
		if (p1 == -1)return 1 + p2Next;
		if (p2Next == -1)return p1;
	}
	return min(p1, 1 + p2Next);
}

int minCoins1(vector<int>& coins, int aim) {
	return process1(coins, 0, aim);
}

方法二：优化版
int process2(vector<int>& coins, int index, int rest,vector<vector<int>>&dp) {
	if (rest < 0) {//该枚硬币不可取，删去
		return -1;
	}
	if (dp[index][rest] != -2) {
		return dp[index][rest];
	}
	if (rest == 0) {//该枚硬币取完之后刚好组成aim，但因为已经算过该枚硬币了，所以这里返回0
		dp[index][rest] = 0;
	}
	if (index == coins.size()) {//越界，因此多算了一枚，要删去
		dp[index][rest] = -1;
	}
	else {
		int p1 = process2(coins, index + 1, rest, dp);//不取
		int p2Next = process2(coins, index + 1, rest - coins[index], dp);//取
		if (p1 == -1 && p2Next == -1) {
			dp[index][rest] = -1;//无效解
		}
		else {
			if (p1 == -1) {
				dp[index][rest] = 1 + p2Next;
			}
			else if (p2Next == -1) {
				dp[index][rest] = p1;
			}
			else {
				dp[index][rest] = min(p1, 1 + p2Next);
			}
		}
	}
	return dp[index][rest];
}

int minCoins2(vector<int>& coins, int aim) {
	vector<vector<int>>dp(coins.size() + 1, vector<int>(aim + 1, -2));
	return process2(coins, 0, aim, dp);
}

方法三：动态规划
int minCoins3(vector<int>coins, int aim) {
	int N = coins.size();
	vector<vector<int>>dp(N + 1, vector<int>(aim + 1));
	for (int col = 1; col <= aim; col++) {
		dp[N][col] = -1;
	}
	for (int index = N - 1; index >= 0; index--) {
		for (int rest = 1; rest <= aim; rest++) {
			int p1 = dp[index + 1][rest];
			int p2Next = -1;
			if (rest - coins[index] >= 0) {//防止越界
				p2Next = dp[index + 1][rest - coins[index]];
			}
			if (p1 == -1 && p2Next == -1) {
				dp[index][rest] = -1;
			}
			else {
				if (p1 == -1) {
					dp[index][rest] = p2Next + 1;
				}
				else if (p2Next == -1) {
					dp[index][rest] = p1;
				}
				else {
					dp[index][rest] = min(p1, p2Next + 1);
				}
			}
		}
	}
	return dp[0][aim];
}
```

### 12.3 马到终点的方法数

**问题：**

给定一个中国象棋棋盘，马走日，从(0,0)到(a,b)必须走k步的方法数有多少？

```c++
思路：
    可以反着来，从（a，b）到（0，0），step递减，最终看step=0时，是否到达（0，0）

//防止越界访问数组
int getValue(vector<vector<vector<int>>>& dp, int step, int row, int col) {
	if (row < 0 || row>9 || col < 0 || col>8)return 0;
	return dp[step][row][col];
}

int dpWays(int i, int j, int step) {
	if (j < 0 || j>8 || i < 0 || i>9 || step < 0) {
		return 0;
	}
	vector<vector<vector<int>>>dp(step + 1, vector<vector<int>>(10, vector<int>(9)));
	dp[0][0][0] = 1;
	for (int h = 1; h <= step; h++) {//层
		for (int r = 0; r < 10; r++) {
			for (int c = 0; c < 9; c++) {
				dp[h][r][c] = getValue(dp, h - 1, r - 1, c + 2);
				dp[h][r][c] = getValue(dp, h - 1, r - 1, c - 2);
				dp[h][r][c] = getValue(dp, h - 1, r + 1, c + 2);
				dp[h][r][c] = getValue(dp, h - 1, r + 1, c - 2);
				dp[h][r][c] = getValue(dp, h - 1, r - 2, c + 1);
				dp[h][r][c] = getValue(dp, h - 1, r - 2, c - 1);
				dp[h][r][c] = getValue(dp, h - 1, r + 2, c + 1);
				dp[h][r][c] = getValue(dp, h - 1, r + 2, c - 1);
			}
		}
	}
	return dp[step][i][j];
}
```

### 12.4 最少方法数

给定不同面额的硬币 coins (硬币数各有任意个)和一个总金额 amount。编写一个函数来计算可以凑成

总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。

```c++
方法一：递归版
int process(vector<int>& coins, int index, int rest) {
	if (index == coins.size()) {
		return rest == 0 ? 1 : 0;
	}
	int ways = 0;
	//张数过大就没意义了，因为已经超出rest了
	for (int zhang = 0; coins[index] * zhang < rest; zhang++) {
		ways += process(coins, index + 1, rest - coins[index] * zhang);
	}
	return ways;
}

方法二：动态规划
int way2(vector<int>coins, int aim) {
	if (coins.size() == 0)return 0;
	int N = coins.size();
	vector<vector<int>>dp(N + 1, vector<int>(aim + 1));
	dp[N][0] = 1;
	for (int index = N - 1; index >= 0; index--) {
		for (int rest = 0; rest <= aim; rest++) {
			int ways = 0;
			for (int zhang = 0; coins[index] * zhang <= rest; zhang++) {
				ways += dp[index + 1][rest - coins[index] * zhang];
			}
			dp[index][rest] = ways;
		}
	}
	return dp[0][aim];
}
时间复杂度： O(N*aim^2)

优化版：用临近值代替枚举
int way3(vector<int>coins, int aim) {
	if (coins.size() == 0)return 0;
	int N = coins.size();
	vector<vector<int>>dp(N + 1, vector<int>(aim + 1));
	dp[N][0] = 1;
	for (int index = N - 1; index >= 0; index--) {
		for (int rest = 0; rest <= aim; rest++) {
			dp[index][rest] = dp[index+1][rest];
            if(rest-coins[index]){//防止越界
                dp[index][rest]+=dp[index][rest-coins[index]]
            }
		}
	}
	return dp[0][aim];
}
时间复杂度： O(N*aim)
```

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/42.png)

**总结：**

递归（定）=> 记忆搜索 => 严格位置依赖

1. 可变参数范围
2. 目标值
3. base case
4. 中间范围的依赖关系
5. 求值顺序



## 13.有序表

1. **BST:**

   - 红黑树

   - AVL(平衡二叉搜索树)

   - SBT(Size Balanced Tree)

2. **list:**

   - 跳表(skip list)

时间复杂度：O(logN)

### 13.1 AVL

```c++
每添加一个节点都要判断一次左右子树的高度差然后相应地进行左右旋转

当右子树高度大于左子树高度时，且高度差大于1：
左旋转：(本质：让一个更适中的数作根节点使得两边的节点数差不多，以此达到平衡)
    1.创建一个新的节点，值等于当前根节点的值：Node*newNode=new Node(root->value);
    2.把新节点的左子树设置为当前节点的左子树：newNode->left=left;
    3.把新节点的右子树设置为当前节点的右子树的左子树：newNode->right=right->left;
    4.把当前节点的值换为右子节点的值：value=right->value;
    5.把当前节点的右子树设置成右子树的右子树：right=right->right;
    6.把当前节点的左子树设置为新节点：left=newNode;

当左子树高度大于左子树高度时，且高度差大于1：
右旋转：
    1.创建一个新的节点，值等于当前根节点的值：Node*newNode=new Node(root->value);
    2.把新节点的右子树设置为当前节点的右子树：newNode->rihgt=right;
    3.把新节点的左子树设置为当前节点的左子树的右子树：newNode->left=left->right;
    4.把当前节点的值换为左子节点的值：value=left->value;
    5.把当前节点的左子树设置成左子树的左子树：left=left->left;
    6.把当前节点的左子树设置为新节点 right=newNode;


问题分析：
    1.符合右旋转条件
    2.如果当前节点的左子树的右子树高度大于它的左子树的左子树的高度
    3.先对当前节点的左节点进行左旋转
    4.再对当前节点进行右旋转
    
//创建节点
class Node {
public:
	int value;
	Node* left = NULL;
	Node* right = NULL;
	
	Node(int value) {
		this->value = value;
	}

	//返回左子树高度
	int leftheight() {
		if (left == NULL) {
			return 0;
		}
		return left->height();
	}
	//返回右子树高度
	int rightheight() {
		if (right == NULL) {
			return 0;
		}
		return right->height();
	}

	//返回以该节点为根节点的树的高度
	int height() {
		return max(left ? left->height() : 0, right ? right->height() : 0)+1 ;
	}

	//左旋转方法
	void leftRotate() {
		//创建新的节点，以当前根节点的值
		Node* newNode = new Node(value);
		//把新节点的左子树设置为当前节点的左子树
		newNode->left = left;
		//把新的节点的右子树设置为当前节点的右子树的左子树
		newNode->right = right->left;
		//把当前节点的值替换成右子节点的值
		value = right->value;
		//把当前节点的右子树设置成右子树的右子树
		right = right->right;
		//把当前节点的左子树设置成新的节点
		left = newNode;
	}

	void rightRotate() {
		Node* newNode = new Node(value);
		newNode->right = right;
		newNode->left = left->right;
		value = left->value;
		left = left->left;
		right = newNode;
	}

	//查找要删除的节点
	Node* research(int value) {
		if (value == this->value) {//就是该节点
			return this;
		}
		else if (value < this->value) {//如果查找的值小于当前节点，向左子树递归
			if (this->left == NULL)return NULL;
			return this->left->research(value);
		}
		else {
			if (this->right == NULL)return NULL;
			return this->right->research(value);
		}
	}

	//查找要删除节点的父节点
	Node* searchParent(int value) {
		if ((this->left != NULL && this->left->value == value) ||
			(this->right != NULL && this->right->value == value)) {
			return this;
		}
		else {
			//如果查找的值小于当前节点的值，并且当前节点的左子节点不为空
			if (value < this->value && this->left != NULL) {
				return this->left->searchParent(value);//向左子树递归查找
			}
			else if (value >= this->value && this->right != NULL) {
				return this->right->searchParent(value);
			}
			else {
				return NULL;//没有父节点
			}
		}
	}

	//添加节点
	//递归的形式添加节点，注意需要满足树的要求
	void add(Node* node) {
		if (!node) return;

		//判断传入的节点的值和当前节点值得关系
		if (node->value < this->value) {
			//如果当前节点的左子节点为空
			if (this->left == NULL) {
				this->left = node;
			}
			else {
				//递归向左子树添加
				this->left->add(node);
			}
		}
		else {//添加的节点的值大于当前节点的值
			if (this->right == NULL) {
				this->right = node;
			}
			else {
				//递归向右子树添加
				this->right->add(node);
			}
		}

		//当添加完一个节点后，如果：（右子树的高度-左子树高度）>1，左旋转
		if (rightheight() - leftheight() > 1) {
			//如果它的右子树的左子树高度大于它的右子树的右子树的高度
			if (right != NULL && right->leftheight() > right->rightheight()) {
				right->rightRotate();
			}

			leftRotate();

			return;//直接return，不然还要接下去做判断，以免引起不必要的bug
		}

		//当添加完一个节点，如果（左子树的高度-右子树的高度）>1，右旋转
		if (leftheight() - rightheight() > 1) {
			//如果它的左子树的右子树高度大于它的左子树的左子树的高度
			if (left != NULL && left->rightheight() > left->leftheight()) {
				//先对当前这个节点的左节点进行左旋转
				left->leftRotate();
			}
			rightRotate();

			return;
		}
	}
    
    //1.返回以node为根节点的树的最小节点的值
	//2.删除以node为根节点的树的最小节点
	int delRightTreeMin(Node* node) {//node当作一棵树的根节点，返回以node为根节点的树的最小
        							 //节点的值
		Node* target = node;
		//循环查找左节点，就会找到最小值
		while (target->left != NULL) {
			target = target->left;
		}
		//这时target就指向了最小节点
		//删除最小节点，最小节点必然无子树，所以不会调用本函数
		delNode(target->value);
		return target->value;
	}
    
    //删除节点
	void delNode(int value) {
		if (root == NULL)return;

		//1.需要先去找到要删除的节点targetNode
		Node* targetNode = search(value);
		//2.如果没有找到要删除的节点
		if (targetNode == NULL) {
			return;
		}
		//如果该树只有一个节点，且存在值为value的节点，那必然是root
		if (root->left == NULL && root->right == NULL) {
			root = NULL;
			return;
		}

		//寻找targetNode的父节点
		Node* parent = searchParent(value);

		//删除叶子节点
		if (targetNode->left == NULL && targetNode->right == NULL) {
			//判断targetNode是父节点的左子节点。还是右子节点
			if (parent->left != NULL && parent->left->value == value) {
				parent->left = NULL;
			}
			else if (parent->right != NULL && parent->right->value == value) {
				parent->value = NULL;
			}
		}
		else if (targetNode->left!=NULL&&targetNode->right!=NULL) {//删除有两棵子树的节点
			int minVal = delRightTreeMin(targetNode->right);
			targetNode->value = minVal;
		}
		else {//删除只有一棵子树的节点，关键：当树只剩两个节点时，要注意父节点可能为空
			//如果要删除的节点有左子节点
			if (targetNode->left != NULL) {
				if (parent != NULL) {
					//如果targetNode是parent的左子节点
					if (parent->left->value == value) {
						parent->left = targetNode->left;
					}
					else {//targetNode是parent的右子节点
						parent->right = targetNode->left;
					}
				}
				else {
					root = targetNode->left;
				}
			}
			else {
				if (parent != NULL) {
					//如果targetNode是parent的左子节点
					if (parent->left->value == value) {
						parent->left = targetNode->right;
					}
					else {//targetNode是parent的右子节点
						parent->right = targetNode->right;
					}
				}
				else {
					root = targetNode->right;
				}
			}
		}
	}
};
```

### 13.2 SB树

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/43.png)

### 13.3 红黑树

```c++
红黑树：
    1.每个结点不是红就是黑
    2.头结点，叶节点(最下面的空白区域)必须为黑
    3.红结点之间不相邻
    4.从头部cur出发，每条到结束的路黑结点一样多。因为是红黑交替，所以可以保证长度大概一致
```

### 13.4 跳表

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/44.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/45.png)



## 14.中级提升班1

### 14.1 窗口不回退

#### 14.1.1 最多覆盖点

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/46.png)

```c++
方法一：贪心
思路：
    1.把绳子的右端依次放在每个点上，记录所覆盖过的最多点数
    2.确定绳子右端后，找大于等于绳子左端的最左位置

//返回大于等于value的最左的值对应的索引
int nearestIndex(vector<int>& vec, int R, int value) {
	int left = 0;
	int index = R;
	while (left < R) {
		int mid = left + ((R - left) >> 1);
		if (vec[mid] >= value) {
			index = mid;
			R = mid - 1;
		}
		else {
			left = mid + 1;
		}
	}
	return index;
}

int maxPoint1(vector<int>& vec, int L) {
	int res = 1;
	for (int i = 0; i < vec.size(); i++) {
		int nearest = nearestIndex(vec, vec[i], vec[i] - L);
		res = max(res, i - nearest + 1);//索引差+1即为覆盖点数
	}
	return res;
}
时间复杂度： O(N*logN)
    
方法二：滑动窗口
思路：把绳子的左端依次放在每个点上，维持一个窗口，记录所覆盖过的最多点数
    
int maxPoint2(vector<int>& vec, int L) {
	int res = 1;
	int R = 0;
	//R没必要回退
	for (int i = 0; i < vec.size(); i++) {
		while (vec[R] - vec[i] <= L)R++;
		res = max(res, R - i + 1);
	}
	return res;
}
时间复杂度： O(N)，因为左边界右边界都不需要回退
```

### 14.2 打表法

####  14.2.1 买苹果

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/47.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/48.png)

```c++
思路：
需要n个苹果，尽量都买8个每袋的可以保证袋数最小，然后余下的苹果用6个每袋的解决。但是，当余下的苹果大于24时，
就不需要再试了。假设余下27个，27=24+3；再递减，余下35个，35=24+11；余下43个，
43=24+19。都是前面的基础类型出现过的，因为24是6和8的最小公倍数。24的那部分可以用8搞定，用6搞定袋数更多，不符合题意要求。
    
int minBags1(int apple) {
	if (apple < 0)return -1;
	int bag6 = -1;
	int bag8 = apple / 8;
	int rest = apple - 8 * bag8;
	while (bag8 >= 0 && rest < 24) {
		int bagUse6 = rest % 6 == 0 ? (rest / 6) : -1;
		if (bagUse6 != -1) {
			bag6 = bagUse6;
			break;
		}
		rest = apple - 8 * (--bag8);
	}
	return bag6 == -1 ? -1 : bag6 + bag8;
}

总结规律后：打表法
int minBagAwesome(int apple){
    if((apple&1)!=0)return -1;
    if(apple<18){
        return apple==0?0:(apple==6||apple==8)?1:(apple==12||apple==14||appple==16)?2:-1;
    }
    return (apple-18)/8+3;
}
```

#### 14.2.2 先后手吃草

**问题：**

两只羊每次只能选择吃4的幂次方草，给定n草，在先后手顺序吃草的情况下，谁刚好把草吃光就赢。

```c++
string winner1(int n) {
	//0    1    2    3   4
	//后  先   后  先  先
	if (n < 5) {
		return (n == 0 || n == 2) ? "后手" : "先手";
	}
	//n>=5时
	int base = 1;//先手决定吃的草
	while (base <= n) {//试吃1/4/16……份草，看各个结果，有一个能赢就行。
		//当前先手吃掉base份草，留下n-base份草给后手
		//母过程的“先手”是子过程的“后手”
		if (winner1(n - base) == "后手")return "先手";
		if (base>n/4)break;//防止base*4后溢出，即防止base大于INT_MAX
		base *= 4;
	}
	return "后手";
}

总结规律后：打表法
string winner2(int n){
    if(n%5==0||n%5==2)return "先手";
    return "后手";
}
```

### 14.3 预处理

#### 14.3.1 染色

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/49.png)

```c++
int minPaint(string s) {
	if (s.length() == 0 || s.length() == 1)return 0;
	int len = s.length();
	vector<int>leftG(len);//记录左半部分有多少G
	vector<int>rightR(len);//记录右半部分有多少R
	leftG[0] = s[0] == 'G' ? 1 : 0;
	rightR[0] = s[len - 1] == 'R' ? 1 : 0;
	for (int i = 1; i < len - 1; i++) {
		leftG[i] = leftG[i - 1] + (s[i] == 'G' ? 1 : 0);
		rightR[len - 1 - i] = rightR[len - i] + (s[len - 1 - i] == 'R' ? 1 : 0);
	}
    //注意边界，因为循环时i不达到边界
	leftG[len - 1] = leftG[len - 2] + (s[len - 1] == 'G' ? 1 : 0);
	rightR[0] = rightR[1] + (s[0] == 'R' ? 1 : 0);
	int res = rightR[0];
	for (int j = 0; j < len; j++) {
		res = min(res, leftG[j] + rightR[j]);//不会重复包含，因为s[j]要么是R要么是G
	}
	return res;
}
```

#### 14.3.2 最大正方形边长

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/50.png)

```c++
长方形子矩阵数:(N^4)/2，在总矩阵找两个点，可能性各为n^2，两点可确定一个矩阵（对角线），最多重复一次。
正方形子矩阵数:(N^3)/2，n^2*n=n^3，因为第二个点有特殊要求

创建辅助结构：
    1.right：记录二维数组每行中每个数右边（包含自己）有多少个连续的1，如果自身为0，那么直接就是0个
    2.down：每个数下方有多少个连续的1，逻辑和right类似
    
void setBorderMap(vector<vector<int>>& m, vector<vector<int>>& right, vector<vector<int>>& down) {
	int r = m.size();
	int c = m[0].size();
	if (m[r - 1][c - 1] == 1) {
		right[r - 1][c - 1] = 1;
		down[r - 1][c - 1] = 1;
	}
	//处理右边
	for (int i = r - 2; i != -1; i--) {
		if (m[i][c - 1] == 1) {
			right[i][c - 1] = 1;
			down[i][c - 1] = down[i + 1][c - 1] + 1;
		}
	}
	//处理下边
	for (int i = c - 2; i != -1; i--) {
		if (m[r - 1][i] == 1) {
			right[r - 1][i] = right[r - 1][i + 1] + 1;
			down[r - 1][i] = 1;
		}
	}
	//处理中间
	for (int i = r - 2; i != -1; i--) {
		for (int j = c - 2; j != -1; j--) {
			if (m[i][j] == 1) {
				right[i][j] = right[i][j + 1] + 1;
				down[i][j] = down[i + 1][j] + 1;
			}
		}
	}
}

bool hasSizeOfBorder(int size, vector<vector<int>>& right, vector<vector<int>>& down) {
	for (int i = 0; i != right.size() - size + 1; i++) {
		for (int j = 0; j != right[0].size() - size + 1; j++) {
			if (right[i][j] >= size && down[i][j] >= size && right[i + size - 1][j] >= size && down[i][j + size - 1] >= size) {
				return true;
			}
		}
	}
	return false;
}

int getMaxSize(vector<vector<int>>& m) {
	vector<vector<int>> right(m.size(),vector<int>(m[0].size()));
	vector<vector<int>> down(m.size(), vector<int>(m[0].size()));
	setBorderMap(m, right, down);
	for (int size = min(m.size(), m[0].size()); size != 0; size--) {
		if (hasSizeOfBorder(size, right, down)) {
			return size;
		}
	}
	return 0;
}
```

#### 14.3.3  加工函数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/51.png)

```c++
//等概率返回0和1的函数
int r01(){
    int res=0;
    do{
        res=f();
    }while(res==3);
 return res<3?0:1; //等分数字，小于的范围返回0,；大于的范围返回1；多余的那一个数重做。  
}

//1~7
int g(){
    int res=0;
    do{
        res=(r01()<<2)+(r01()<<1)+r01();
    }while(res==7);//大于所需范围的重做
    return res+1;//因为res是从0开始的，所以加上c之后就属于需要等概率返回的范围了
}

//调用两次f，如果是01返回0，如果是10就返回1，其他重做。因为这两种情况等概率，都是p(1-p)
int g(){
    int res=0;
    do{
        res=(f()<<1)+f();
    }while(res==0||res==3);
    return res-1;
}
```

## 15.中级提升班2

### 15.1 动态规划

#### 15.1.1 二叉树结构数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/52.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/53.png)

```c++
方法一：递归
int process(int n) {
	if (n < 0)return 0;
	if (n == 0)return 1;
	if (n == 1)return 1;
	if (n == 2)return 2;
	int res = 0;
	for (int leftNum = 0; leftNum <= n - 1; leftNum++) {
		int leftWays = process(leftNum);
		int rightWays = process(n - 1 - leftNum);
		res += leftWays * rightWays;
	}
	return res;
}

方法二：动态规划
int numTrees(int n) {
	if (n < 2)return 1;
	vector<int>dp(n+1);
	dp[0] = 1;
	for (int i = 1; i <= n; i++) {//结点个数为i时
		for (int j = 0; j <= i-1; j++) {//左侧结点个数为j-1，右侧结点个数为i-j
			dp[i] += dp[j] * dp[i - j - 1];
		}
	}
	return dp[n];
}
```

#### 15.1.2 完整括号字符串

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/54.png)

```c++
完整括号字符串定义：每个左括号都有右括号与之配对，且每个右括号都有左括号与之配对

思路：
    1.定义变量count，遍历字符串，碰到'('，count++，碰到')'，count--
    2.遍历过程中，如果count<0，说明')'过多
    3.遍历结束后，count必须等于0，否则可以判断'('过多

int needParenttheses(string str) {
	int count = 0;
	int ans = 0;
	for (int i = 0; i < str.length(); i++) {
		if (str[i] == '(')count++;
		else {
			if (count == 0)ans++;
			else count--;
		}
	}
	return count + ans;
}
```

### 15.2 差值为k的数字对

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/55.png)

```c++
vector<pair<int, int>>numPair(vector<int>& vec) {
	map<int, int>m;
	vector<pair< int, int>>res;
	for (int i = 0; i < vec.size(); i++) {
			m[vec[i]] = vec[i] + 2;//map的性质决定了不会有重复元素
	}
	for (auto& p : m) {//map是一个大的容器，里面装着多个pair
		if (m.count(p.second) != 0) {
			res.push_back(p);
		}
	}
	return res;
}
```

### 15.3 平均值变大

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/56.png)

```c++
int maxOps(vector<int>vec1, vector<int>vec2) {
	double sum1 = 0;
	for (int i = 0; i < vec1.size(); i++) {
		sum1 += (double)vec1[i];
	}
	double sum2 = 0;
	for (int i = 0; i < vec2.size(); i++) {
		sum2 += (double)vec2[i];
	}
	if (avg(sum1, vec1.size()) == avg(sum2, vec2.size()))return 0;
	vector<int>vecMore;
	vector<int>vecLess;
	double sumMore = 0;
	double sumLess = 0;
	if (avg(sum1, vec1.size()) > avg(sum2, vec2.size())) {
		vecMore = vec1;
		sumMore = sum1;
		vecLess = vec2;
		sumLess = sum2;
	}
	else {
		vecMore = vec2;
		sumMore = sum2;
		vecLess = vec1;
		sumLess = sum1;
	}
	sort(vecMore.begin(), vecMore.end());
	set<int>setLess;
	for (int num : vecLess) {
		setLess.insert(num);
	}
	int moreSize = vecMore.size();
	int lessSize = vecLess.size();
	int ops = 0;
	for (int i = 0; i < vecMore.size(); i++) {
		double cur = (double)vecMore[i];
		if (cur<avg(sumMore, moreSize) && cur>avg(sumLess, lessSize) && setLess.count(vecMore[i]) == 0) {
			sumMore -= cur;
			sumLess += cur;
			moreSize--;
			lessSize++;
			setLess.insert(vecMore[i]);
			ops++;
		}
	}
	return ops;
}

double avg(double sum, int size) {
	return sum / (double)size;
}
```

### 15.4 最长有效括号子串

```c++
dp[i]:子串以i位置字符为结尾时，最长的有效长度。如果s[i]='('，dp[i]=0

int maxLength(string s){
    if(s.length()==0)return 0;
    int res=0;
    int pre=0;
    vector<int>dp(s.length());
    for(int i=1;i<s.length();i++){
        if(s[i]==')'){//s[i]是右括号时才有得配对
            pre=i-dp[i-1]-1;//与s[i]配对的左括号索引pre
        	if(pre>=0&&s[pre]=='('){
            	dp[i]=dp[i-1]+2+(pre>0?dp[pre-1]:0);//还要加上能被左括号接上的一段
        	}
            res=max(res,dp[i]);
        }
    }
    return res;
}
```

### 15.5 无序栈变有序(栈顶为最大元素)

```c++
void sortStack(stack<int>&stk){
    if(stk.size()==0)return;
    stack<int>temp;
    while(!stk.empty()){
        int val=stk.top();
        stk.pop();
        while(!temp.empty()&&temp.top()<val){
            stk.push(temp.top());
            temp.pop();
        }
        temp.push(val);
    }
    while(!temp.empty()){
        stk.push(temp.top());
        temp.pop();
    }
}
```

### 15.6 有序二维数组找数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/57.png)

```c++
思路：
    1. 从右上角开始找
    2. matrix[i][j]<aim，i++
    3. matrix[i][j]>aim，j--
```

### 15.7 最多1的行

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/58.png)

## 16.中级提升班3

### 16.1 平分

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/59.png)

```c++
思路：
    1.求出总物品数sum，判断sum%n==0?
    2.分析单个机器，每个机器的左右两边可能缺物品也可能多物品
    3.所需物品数等于avg*m
    4.多：往外扔；少：往外拿
      4.1左右皆负(l,r)：至少|l|+|r|次，因为机器一次只能扔一个物品
      4.2左右皆正：至少max(l,r)次，因为可以一起接收物品
      4.3左负右正、左正右负：至少max(|l|,|r|)次
    5.遍历求每个位置的机器调用次数，所需次数最大的机器目标达成时，其他机器也一定完成了。
    6.因此总次数=所有机器中的最大次数
    
int minOps(vector<int>&arr){
    if(arr.size()==0){
        return 0;
    }
    int size=arr.size();
    int sum=0;
    for(int i=0;i<size;i++){
        sum+=arr[i];
    }
    if(sum%size!=0){
        return -1;
    }
    int avg=sum/size;
    int leftSum=0;
    int ans=0;
    for(int i=0;i<size;i++){
        //负需要输入，正需要输出
        int leftRest=leftSum-i*avg;
        int rightRest=(sum-leftSum-arr[i])-(size-i-1)*avg;
        if(leftRest<0&&rightRest<0){
            ans=max(ans,abs(leftRest)+abs(rightRest));
        }else{
            ans=max(ans,max(abs(leftRest,rightRest)));
        }
        leftSum=arr[i];
    }
    return ans;
}
```

### 16.2 螺旋打印矩阵

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/60.png)

```c++
思路：
    1.找对角两点，按顺时针方向打印
    2.对角两点沿对角线方向向内收缩
    3.两点行号相同时：向右打印
    4.两点列号相同时：向下打印
    5.两点错过时停止

//辅助打印函数
void printEdge(vector<vector<int>>&m,int ia,int ja,int ib,int jb){
    if(ia==ib){
        for(int i=ja;i<=jb;i++){
            cout<<m[ia][i]<<" ";
        }
    }else if(ja==jb){
        for(int i=ia;i<=ib;i++){
            cout<<m[i][ja]<<" ";
        }
    }else{
        int curR=ia;
        int curC=ja;
        while(curC!=jb){
            cout<<m[ia][curC++]<<" ";
        }
        while(curR!=ib){
            cout<<m[curR++][jb]<<" ";
        }
        while(curC!=ja){
            cout<<m[ib][curC--]<<" ";
        }
        while(curR!=ia){
            cout<<m[curR--][ja]<<" ";
        }
    }
}

void spiralOrderPrint(vector<vector<int>>&m){
    int tR=0;
    int tC=0;
    int dR=m.size()-1;
    int dC=m[0].size()-1;
    while(tR<=dR&&tC<=dC){
        printEdge(m,tR++,tC++,dR--,dC--);
    }
}
```

### 16.3 转动正方形矩阵

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/61.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/62.png)

```c++
void rotateEdge(vector<vector<int>>&m,int tR,int tC,int dR,int dC){
    int times=dC-tC;
    int tmp=0;
    for(int i=0;i<times;i++){
        tmp=m[tR][tC+i];
        m[tR][tC+i]=m[dR-i][tC];
        m[dR-i][tC]=m[dR][dC-i];
        m[dR][dC-i]=m[tR+i][dC];
        d[tR+i][dC]=tmp;
    }
}

void rotate(vector<vector<int>>&m){
    int tR=0;
    int tC=0;
    int dR=m.size()-1;
    int dC=m[0].size()-1;
    while(tR<dR){
        rotate(m,tR++,tC++,dR--,dC--);
    }
}
```

### 16.4 zigzag打印矩阵

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/63.png)

```c++
思路：
    1. 两点A，B从左上角出发
    2. A往下走，不能再往下就往右走；B往右走，走到不能再往右就往下走。
    3. 两点每次同时动一步
    4. 如果两点之间有斜线，就不断地从左下到右上然后从右上到左下交替打印
    
void printLevel(vector<vector<int>>&m,int tR,int tC,int dR,int dC,bool f){
    if(f){
        while(tR!=dR+1){
            cout<<m[tR++][tC--]<<" ";
        }
    }else{
        while(dR!=tR-1){
            cout<<m[dR--][dC++]<<" ";
        }
    }
}

void printMatrixZigZag(vector<vector<int>>&m){
    int ar=0;
    int ac=0;
    int br=0;
    int bc=0;
    int endR=m.size()-1;
    int endC=m[0].size()-1;
    bool fromUp=false;
    while(ar!=endR+1){
        printLevel(m,ar,ac,br,bc,fromUp);
        ar=ac==endC?ar+1:ar;
        ac=ac==endC?ac:ac+1;
        br=br==endR?br:br+1;
        bc=br==endR?bc+1:bc;
        fromUp=!fromUp;
    }
}
```

### 16.5 最少操作步骤数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/64.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/65.png)

```c++
思路：
    1. n是质数，只调用操作2。因为操作1无法拼成质数长度的a
    2. n不是质数，n=a*b*c*d，（a，b，c，d）是质数，且已经是最优顺序
    3. a-2次操作2,1次操作1，b-2次操作2,1次操作1，c-2次操作2,1次操作1，d-2次操作2,1次操作1。
    4.总次数等于a+b+c+d-质数因子个数

bool isPrim(int n){
    if(n<2){
        return false;
    }
    int max=(int)sqrt((double)n);
    for(int i=2;i<=max;i++){
        if(n%i==0){
            return false;
        }
    }
    return true;
}
    
vector<int>divsSumAndCount(int n){
    int sum=0;
    int count=0;
    for(int i=2;i<=n;i++){
        while(n%i==0){
            sum+=i;
            count++;
            n/=i;
        }
    }
    return {sum,count};
}

int minOps(int n){
    if(n<2)return 0;
    if(isPrim(n)){
        return n-1;
    }
    vector<int>divSumAndCount=divsSumAndCount(n);
    return divSumAndCount[0]-divSumAndCount[1];
}
```

### 16.6 出现次数最多的前k个数

```c++
思路：
    1.准备一个小根堆
    2.当堆里的元素个数为k个时，堆顶元素就是门槛
    3.哈希表中只有大于堆顶元素的才能入堆
    
提升：实时显示
class Node{
public:
    string str;
    int times;
    Node(string s,int t){
        this->str=s;
        this->times=t;
    }
}

class TopKRecord{
private:
    map<string,Node>strNodeMap;
    vector<Node>heap;
    int index;
    map<Node,int>nodeIndexMap;
public:
    TopKRecord(int k){
        heap=vector<int>(k);
        heapSize=0;
    }
    
    void add(string str){
        Node curNode;
        int preIndex=-1;
        //当前str对应的节点对象
        if(strNodeMap.count(str)==0){//str第一次出现
            curNode=Node(str,1);
            strNodeMap[str]=curNode;
            nodeIndexMap[curNode]=-1;
        }else{//str并非第一次出现
            curNode=strNodeMap[str];
            curNode.times++;
            preIndex=nodeIndexMap[curNode];
        }
        //当前str对应的节点对象是否在堆上
        if(preIndex==-1){
            if(heapSize==heap.size()){//堆满，和门槛判断
                if(heap[0].times<curNode.times){
                    nodeIndexMap[heap[0]]=-1;
                    nodeIndexMap[curNode]=0;
                    heap[0]=curNode;
                    heapify(0,heapSize);//调整堆
                }
            }else{//堆没满，直接进，再用heapInsert
                nodeIndexMap[curNode]=heapSize;
                heap[heapSize]=curNode;
                heapInsert(heapSize++);
            }
        }else{//已经在堆上，str对应节点的times增加之后调整堆
            heapify(preIndex,heapSize);
        }
    }
    
    void heapInsert(int index){
        while(index!=0){
            int parent=(index-1)/2;
            if(heap[index].times<heap[parent].times){
                swap(parent,index);
                index=parent;
            }else{
                break;
            }
        }
    }
    
    void heapify(int index,int heapSize){
        int l=index*2+1;
        int r=l+1;
        int smallest=index;
        while(l<heapSize){
            if(heap[l].times<heap[index].times){
                smallest=l;
            }
            if(r<heapSize&&heap[r].times<heap[smallest].times){
                smallest=r;
            }
            if(smallest!=index){
                swap(smallest,index);
            }else{
                break;
            }
            index=smallest;
            l=index*2+1;
            r=l+1；
        }
    }
    
    void swap(int index1,int index2){
        nodeIndexMap[heap[index1]]=index2;
        nodeIndexMap[heap[index2]]=index1;
        Node temp=heap[index1];
        heap[index1]=heap[index2];
        heap[index2]=temp;
    }
}
```

## 17.中级提升班4

### 17.1 自定义栈

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/66.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/67.png)

```c++
思路：
    1. 定义两个栈Data和Min
    2. Data正常放入数据
    3. Min在有新数据放入Data时，也会被重复放入当前Data中的最小值，所以Data中的最小值保存在Min栈顶
    4. Data做一次pop操作时，Min也做一次pop操作
```

### 17.2 队列成栈，栈成队列

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/68.png)

```c++
队列成栈：
    class TwoQueueStack {
private:
	queue<int>qu;
	queue<int>help;
public:
	void push(int pushInt) {
		qu.push(pushInt);
	}

	//交换的原因：push操作是把元素插入到qu队列中，让qu一直是保留多个元素的队列，才能保证元素的相对顺序。
	int top() {
		if (qu.empty()) {
			throw runtime_error("Stack is empty");
		}
		while (qu.size() != 1) {
			help.push(qu.front());
			qu.pop();
		}
		int res = qu.front();
		qu.pop();
		swap();
		return res;
	}

	void pop() {
		if (qu.empty()) {
			throw runtime_error("Stack is empty");
		}
		while (qu.size() >1) {
			help.push(qu.front());
			qu.pop();
		}
		qu.pop();
		swap();
	}

	void swap() {
		queue<int>temp = qu;
		qu = help;
		help = temp;
	}
};

栈成队列：
    1. 定义两个栈push和pop
    2. 用户push新数据时就放入push栈
    3. 用户pop时先判断pop栈是否为空，然后再把push栈的全部数据弹出到pop栈中
    
class TwoStackQueue {
private:
	stack<int>stackPush;
	stack<int>stackPop;
	
public:
	void push(int pushInt) {
		stackPush.push(pushInt);
		dao();
	}

	void pop() {
		if (stackPop.empty() && stackPush.empty()) {
			throw runtime_error("Queue is empty!");
		}
		dao();
		stackPop.pop();
	}

	int front() {
		if (stackPop.empty() && stackPush.empty()) {
			throw runtime_error("Queue is empty!");
		}
		dao();
		return stackPop.top();
	}

	void dao() {
		if (stackPop.empty()) {//pop栈为空push栈才能倒数据
			while (!stackPush.empty()) {//一次性倒空
				stackPop.push(stackPush.top());
				stackPush.pop();
			}
		}
	}
};

因为队列和栈可以互通，所以图的深度或广度优先遍历既可以用栈实现，也可以用队列实现。
```

### 17.3 动态规划的空间压缩技巧

#### 17.3.1 最小路径和

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/69.png)

```c++
思路：
根据转移方程决定需要几行数组
    
int minPathSum(vector<vector<int>>& m) {
	if (m.size() == 0)return 0;
	int more = max(m.size(), m[0].size());
	int less = min(m.size(), m[0].size());
	bool rowmore = more == m.size();
	vector<int>arr(less);
	arr[0] = m[0][0];
	for (int i = 1; i < less; i++) {
		arr[i] = arr[i - 1] + (rowmore ? m[0][i] : m[i][0]);
	}
	for (int i = 1; i < more; i++) {
		arr[0] = arr[0] + (rowmore ? m[0][i] : m[i][0]);
		for (int j = 1; j < less; j++) {
			arr[j] = min(arr[j - 1], arr[j]) + (rowmore ? m[i][j]:m[j][i]);
		}
	}
	return arr[less - 1];
}
```

#### 17.3.2 装水

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/70.png)

```c++
思路：
只关心每个位置上方能够留下多少水
V[i]=max(min(左max,右max)-arr[i],0)
    1. 利用双指针L=1、R=arr.size()-2指向数组左右两侧
    2. 定义两变量maxLeft、maxRight，分别记录L、R扫过区域的最大值
    3. 计算水量all：瓶颈由小的确定，哪边高度低就移动哪边的指针
      3.1 maxLeft<maxRight：all+=maxLeft-arr[L]>0?maxLeft-arr[L]:0;右移L，更新maxLeft
      3.2 maxRight<maxLeft：all+=maxRight-arr[R]>0?maxRight-arr[R]:0;左移R，更新maxRight
      3.3 maxRight==maxLeft：all+=maxLeft-arr[L]>0?maxLeft-arr[L]:0;右移L，更新maxLeft；左移
    	  R，更新maxRight
    4. maxLeft是L位置左边真实的最大值，maxRight是R位置右边真实的最大值，所以当哪边真实的最大值更小时，
       对应那边指针的水量才可以计算，因为真实的瓶颈我们已经知道了。
    
int getWater(vector<int>& arr) {
	if (arr.size() < 3) return 0;
	int L = 1;
	int R = arr.size() - 2;
	int maxLeft = arr[0];
	int maxRight = arr[arr.size() - 1];
	int all = 0;
	while (L <= R) {
		if (maxLeft <= maxRight) {
			all += max(maxLeft - arr[L], 0);
			maxLeft = max(maxLeft, arr[L++]);
		}
		else {
			all += max(maxRight - arr[R], 0);
			maxRight = max(maxRight, arr[R--]);
		}
	}
	return all;
}
```

### 17.4 最大绝对差值

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/71.png)

```c++
思路：
    1.遍历数组找到最大值maxVal
    2.由于maxVal是全局最大值，所以maxVal必定是左部分最大值或者右部分最大值
    3.又因为无论怎么划分，arr[0]和arr[arr.size()-1]必定分别在左部分和右部分
    4.所以左部分的最大值要么是arr[0]，要么就是比arr[0]更大的数，右部分同理
    5.最终答案就是：max(maxVal-arr[0],maxVal-arr[arr.size()-1])
```

### 17.5 互为旋转词

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/72.png)

```c++
思路：
    1. 判断a、b长度是否一样
    2. a+=a;
    3. 利用kmp算法判断b是否为a的子串
```

### 17.6 咖啡机问题

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/73.png)

```c++
(1)vector<int>arr：数组大小代表咖啡机数量，数据代表冲一杯咖啡所需时间
(2)int n：n个人要喝咖啡，每人一杯
(3)int a：洗杯器（洗一个杯子所需时间a），只能串行，一次只能洗一个杯子
(4)int b：咖啡杯自然挥发变干净的时间
问题：从第一个人开始泡咖啡开始，到所有杯子都变干净至少需要多少时间？

思路：
    1.创建一个小根堆，如上图所示，第一个数代表开始时刻，第二个数代表所需时间，按结束时间早晚来排序。
    2.每个人拿完咖啡(取堆顶)后，更新小根堆里的数据，例如第一个人拿完后，(0,2)更新成(2.2)
    3.将每个人拿到咖啡的时间放入drinks数组中，该数据也是开始洗杯子的时间
    
//洗杯子
int process(vector<int>&drinks,int a,int b,int index,int washLine){
    if(index==drinks.size()-1){
        return min(max(washLine,drinks[index])+a,drinks[index]+b);
    }
    //用洗杯器
    int wash=max(washLine,drinks[index])+a;//洗完该杯子的结束时间，喝完才能洗
    int next1=process(drinks,a,b,index+1,wash);//洗完剩下杯子所需时间
    int p1=max(wash,next1);//全部事情做完所需时间
    
    //自然风干
    int dry=drink[index]+b;
    int next2=process(drinks,a,b,index+1,washLine);
    int p2=max(dry,next2);
    
    return min(p1,p2);//p1和p2是当前杯子选择不同干净的方式所需的总时间，返回最小的即可
}

由于上述递归只有两个可变参数，因此可以利用一张二维表动态规划完成
index:0~n-1
washLine:0~drinks[n-1]+n*a（准备足够用的表即可）
```

### 17.7 任意相邻两数之积是4的倍数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/74.png)

```c++
思路：
    1.统计奇数个数(a)、偶数个数（2的数量(b)、包含4因子的数(c)）
    2.讨论b：
      2.1 b==0：奇4奇4奇4奇4奇4奇4奇4奇4……；a==1(c>=1),a>1(c>=a-1)
      2.2 b==1：a==0(c>=1),a==1(c>=1),a>1(c>=a)
      2.3 b>1：22222224奇4奇4奇4奇4……；a==0(c>=0),a==1(c>=1),a>1(c>=a)=>c>=a
    3.if(b==0)return a==1?c>=1:c>=a-1;
	  else if(b==1)return a==0?c>=1:(a==1?c>=1:c>=a)
	  else(b>1)return c>=a;
```

## 18.中级提升班5

### 18.1 斐波那契套路

#### 18.1.1 F(N)=F(N-1)+F(N-2)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/75.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/83.png)

```c++
线性代数：
1. 利用初始项可以吧a,b,c,d算出来
2. a=b=c=1,d=0
3. 假设由abcd组成的矩阵为D
4. |F(N),F(N-1)|=|1,1|*D^(n-2)
5. 问题转成如何计算一个矩阵的幂次方
  5.1 先理解O(logn)计算一个数的幂次方
//时间复杂度为O(logn)计算num的n次方
int power(int num, int n) {
	int res = 1;
	int t = num;
	while (n > 0) {//把n看作2进制数，哪一位是1，就让res乘等于对应的10的幂次方
		if ((n & 1) == 1) {//当前位(最低位)是1
			res *= t;
		}
		n >>= 1;
		t *= num;
	}
	return res;
}
  5.2 计算矩阵的幂次方：如上图所示
6.所以完成了O(logn)计算斐波那契数
//返回矩阵m1、m2相乘的结果
vector<vector<int>>muliMatrix(vector<vector<int>>& m1, vector<vector<int>>& m2) {
	vector<vector<int>>res(m1.size(), vector<int>(m2[0].size()));
	for (int i = 0; i < m1.size(); i++) {
		for (int j = 0; j < m2[0].size(); j++) {
			for (int k = 0; k < m2.size(); k++) {
				res[i][j] += m1[i][k] + m2[k][j];
			}
		}
	}
	return res;
}

vector<vector<int>>matrixPower(vector<vector<int>>& m, int p) {
	vector<vector<int>>res(m.size(), vector<int>(m[0].size()));
	for (int i = 0; i < res.size(); i++) {//令矩阵对角线全为1，其他都为0
		res[i][i] = 1;
	}
	vector<vector<int>>t = m;
	for (; p != 0; p >>= 1) {
		if ((p & 1) == 1) {
			res = muliMatrix(res, t);
		}
		t = muliMatrix(t, t);
	}
	return res;
}

//O(logn)计算斐波那切数
int fi(int n) {
	if (n < 1)return 0;
	if (n == 1 || n == 2) return 1;
	vector<vector<int>>base = { {1,1},
													{1,0} };
	vector<vector<int>>res = matrixPower(base, n - 2);
	/*
	| F(N), F(N - 1) |= | 1, 1 | *D ^ (n - 2)
	假设D^(n-2)={{X,Y},{P,Q}}
	X+P=F(N)
	Y+Q=F(N-1)
	*/ 
	return res[0][0] + res[1][0];//X=res[0][0],P=res[1][0]
}
```

#### 18.1.2 F(N)=3F(N-1)-4F(N-3)+6F(N-5)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/76.png)

```c++
|5*5|：5*5的矩阵
```

#### 18.1.3 生牛问题

```c++
一头牛从出生到成熟可生牛的时间为3年，假设第1年只有1头牛，那么第n年总共有几头牛？
PS：牛不会死，一头牛每年只生一只牛
F(N)=F(N-1)+F(N-3)
F(N-1)：前一年留下的牛
F(N-3)：三年前的牛全都成熟后生下的新牛数
所以，利用3阶矩阵解决问题
|F(N),F(N-1),F(N-2)|=|F(3),F(2),F(1)|*D^(n-2)
```

#### 18.1.4 达标串数量

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/77.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/78.png)

```c++
思路：
    1. 求长度为i的达标串数量
    2. 第0位上的肯定是1，所以看后面i-1位
      2.1 s[1]=1：达标串数量为 F(i-1)
      2.2 s[1]=0，那么s[2]必定是1：达标串数量为 F(i-2)
    3. F(i)=F(i-1)+F(i-2)
```

#### 18.1.5 取最少的木棍

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/79.png)

```c++
思路：
保留斐波那切数，其他全删去
```

### 18.2 背包问题

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/80.png)

```c++
思路：
动态规划二维表：行（各零食体积），列（1~w）
dp[i][j]=dp[i-1][j]+dp[i][j-arr[i]];//不选择i零食的情况+选择i零食的情况
```

### 18.3 找工作

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/81.png)

```c++
思路：
    1.创建有序表，难度不同的按难度从小到大，难度相同的为一组，按照报酬从大到小排
    2.同组的只留组长（报酬高的）
    3.把难度提升但是报酬不提高的工作删去
    4.最终只留下难度和报酬单调递增的工作

class Job {
public:
	int money;
	int hard;
	
	Job(int money, int hard) {
		this->money = money;
		this->hard = hard;
	}
};

class JobComparator {
public:
	bool compare(Job o1, Job o2) {
		return o1.hard != o2.hard ? o2.hard - o1.hard : o1.money - o2.money;
	}
};

vector<int>getMoneys(vector<Job>job,vector<int>ability){
	sort(job.begin(), job.end(), JobComparator());
	//难度为key的工作，最优工资是value
	map<int, int>map;
	map[job[0].hard] = job[0].money;
	Job pre = job[0];
	for (int i = 0; i < job.size(); i++) {
		if (job[i].hard != pre.hard && job[i].money > pre.money) {//前者只保留组长，后者删去不保证单调性的工作
			pre = job[i];
			map[job[i].hard] = job[i].money;
		}
	}
	vector<int>ans(ability.size());
	for (int i = 0; i < ability.size(); i++) {
		auto it = map.upper_bound(ability[i]);//找到第一个大于该能力值的工作
		if (it != map.begin()) {
			it--;
			ans[i] = it->second;
		}
		else {//找不到匹配的工作
			ans[i] = 0;
		}
	}
	return ans;
}
```

### 18.4 判断是否符合人类正常书写

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/82.png)

```c++
思路：
    1.除了数字之外，只允许出现负号
    2.如果有负号，只会出现在开头，且只出现一次
    3.负号必须跟着数字，且不可以是0
    4.如果开头字符是0，后面不能有其他数字
    5.判断越界：将字符串转成数字，且都转为负数，因为|INT_MIN|-|INT_MAX|=1
    6.定义两个变量：int minq=INT_MIN/10;int minr=INT_MIN%10;用于判断是否越界
    7.if(res<minq||res==minq&&cur<minr)，判断溢出条件
```

## 19.中级提升班6

### 19.1 目录路径

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/84.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/85.png)

```c++
思路：
    1.创建前缀树，将相应文件名转成结点放入前缀树中
    2.深度优先遍历前缀树打印文件名
    
class Node {
public:
	string name;
	map<string, Node*>nextMap;//map默认有序，nextMap储存着文件路径

	Node(string name) {
		this->name = name;
	}
};

vector<string> stringSplit(string str, const char split)
{
	istringstream iss(str);	// 输入流
	string token;			// 接收缓冲区
	vector<string>res;
	while (getline(iss, token, split))	// 以split为分隔符
	{
		res.push_back(token); // 放入res中
	}
	return res;
}

Node* generateFolderTree(vector<string>& folderPaths) {
	Node* head = new Node("");
	for (string foldPath : folderPaths) {
		vector<string>paths = stringSplit(foldPath, '\\');
		Node* cur = head;
		for (int i = 0; i < paths.size(); i++) {
			if (cur->nextMap.count(paths[i]) == 0) {
				cur->nextMap[paths[i]] = new Node(paths[i]);
			}
			cur = cur->nextMap[paths[i]];
		}
	}
	return head;
}

string get2nSpace(int n) {
	string res = "";
	for (int i = 1; i < n; i++) {
		res += "  ";
	}
	return res;
}

void printProcess(Node* head, int level) {
	if (level != 0) {
		cout << get2nSpace(level) << head->name << endl;
	}
	for (auto next : head->nextMap) {
		printProcess(next.second, level + 1);
	}
}

void print(vector<string>& folderPaths) {
	if (folderPaths.size() == 0)return;
	Node* head = generateFolderTree(folderPaths);
	printProcess(head, 0);
}
```

### 19.2 二叉树经典套路

#### 19.2.1 搜索二叉树转双向链表

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/86.png)

```c++
思路：
    1.返回以每个结点为根结点时，双向链表的头和尾
    2.左子结点的尾的right指针指向其父结点(父结点的left指针反指)，父结点的right指针指向右子结点的头(右子       节点的头的left指针反指)，然后返回左子结点的头和右子结点的尾，作为自己的头和尾
    
class Node {
public:
	Node* left;
	Node* right;
	int value;

	Node(int value) {
		this->value = value;
		left = NULL;
		right = NULL;
	}
};

//搜索二叉树转化成双向链表后，返回头和尾的信息
class Info {
public:
	Node* start;
	Node* end;

	Info(Node* start, Node* end) {
		this->start = start;
		this->end = end;
	}
};

Info process(Node* x) {
	if (x == NULL)return Info(NULL, NULL);
	Info leftHeadEnd = process(x->left);
	Info rightHeadEnd = process(x->right);
	if (leftHeadEnd.end != NULL) {//左子结点的尾的next指针指向其父结点
		leftHeadEnd.end->right = x;
	}
	//父结点的left指针反指
	x->left = leftHeadEnd.end;
	//父结点的right指针指向右子结点的头
	x->right = rightHeadEnd.start;
	if (rightHeadEnd.start != NULL) {//右子节点的头的left指针反指
		rightHeadEnd.start->left = x;
	}
	return Info(leftHeadEnd.start ? leftHeadEnd.start : x, rightHeadEnd.end ? rightHeadEnd.end : x);
}

Node* convert(Node* head)
{
	return process(head).start;
}
```

#### 19.2.2 最大搜索子树节点个数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/87.png)

```c++
提升版：不返回节点个数，返回头节点
class Node {
public:
	Node* left;
	Node* right;
	int value;

	Node(int value) {
		this->value = value;
		left = NULL;
		right = NULL;
	}
};

class Info {
public:
	Node* maxBSTHead;
	bool isBST;
	int minVal;
	int maxVal;
	int maxBSTSize;

	Info(Node* head, bool is, int min, int max, int size) {
		this->maxBSTHead = head;
		this->isBST = is;
		this->minVal = min;
		this->maxVal = max;
		this->maxBSTSize = size;
	}
};

Info process(Node* x) {
	if (x == NULL) {//不能影响后续判断，左树最大值小于x的值，所以max设置为INT_MIN；min同理
		return Info(NULL, true, INT_MAX, INT_MIN, 0);
	}
	Info leftInfo = process(x->left);
	Info rightInfo = process(x->right);
	//用完黑盒后，自身也要完善黑盒
	int minVal = x->value;
	int maxVal = x->value;
	if (x->left) {
		minVal = min(minVal, leftInfo.minVal);
		maxVal = max(maxVal, leftInfo.maxVal);
	}
	if (x->right) {
		minVal = min(minVal, rightInfo.minVal);
		maxVal = max(maxVal, rightInfo.maxVal);
	}

	int maxBSTSize = 0;
	Node* maxBSTHead = NULL;
	if (x->left) {
		maxBSTSize = leftInfo.maxBSTSize;
		maxBSTHead = leftInfo.maxBSTHead;
	}
	if (x->right&&rightInfo.maxBSTSize>maxBSTSize) {
		maxBSTSize = rightInfo.maxBSTSize;
		maxBSTHead = rightInfo.maxBSTHead;
	}

	bool isBST = false;
	if ((x->left == NULL || leftInfo.isBST) &&(x->right == NULL || rightInfo.isBST)) {
		if ((x->left == NULL || leftInfo.maxVal < x->value)&&(x->right == NULL || rightInfo.minVal > x->value)) {
			isBST = true;
			maxBSTHead = x;
			int leftSize = x->left ? leftInfo.maxBSTSize : 0;
			int rightSize = x->right ? rightInfo.maxBSTSize : 0;
			maxBSTSize = leftSize + 1 + rightSize;
		}
	}
	return Info(maxBSTHead, isBST, minVal, maxVal, maxBSTSize);
}

Node*getMaxBSTHead(Node*head){
    if(head==NULL)return NULL;
    return process(head).maxBSTHead;
}
```

### 19.3 最大累加和问题

#### 19.3.1 历史最高分

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/88.png)

```c++
思路：
    1. 定义两个变量cur，maxVal
    2. cur+=arr[i]，maxVal跟踪cur不断更新
    3. 如果cur<0，令cur=0；否则继续循环
    
int maxSum(vector<int>& arr) {
	if (arr.size() == 0)return 0;
	int maxVal = INT_MIN;
	int cur = 0;
	for (int i = 0; i < arr.size(); i++) {
		cur += arr[i];
		maxVal = max(maxVal, cur);
		//cur<0，说明i之前的任何位置到当前位置的累加和都小于0，已经没有保留的必要了
		cur = cur < 0 ? 0 : cur;
	}
	return maxVal;
}
```

#### 19.3.2 子矩阵最大累加和

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/89.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/90.png)

```c++
思路：
    1. n行矩阵
    2. 只包含0~0；0~1；……；0~n-1；1~1；……；1~n-1；……；n-1~n-1行中，累加和最大在哪一项里
    3. 由于子矩阵是矩形的特性，可将每行的数据合并后，找子数组最大累加和，降低维度
    
int maxSum(vector<vector<int>>&m) {
	if (m.size() == 0 || m[0].size() == 0)return 0;
	int maxVal = INT_MIN;
	int cur = 0;
	vector<int>s;
	for (int i = 0; i < m.size(); i++) {
		s = vector<int>(m[i].size());
		for (int j = i; j < m.size(); j++) {
			cur = 0;
			for (int k = 0; k < s.size(); k++) {
				s[k] += m[j][k];
				cur += s[k];
				maxVal = max(maxVal, cur);
				cur = cur < 0 ? 0 : cur;
			}
		}
	}
	return maxVal;
}
```

## 20.中级提升班7

### 20.1 最少路灯数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/91.png)

```c++
方法一：贪心
int minLight(string str) {
	if (str.length() == 0)return 0;
	int light = 0;
	int index = 0;
	while (index < str.length()) {
		if (str[index] == 'X')index++;
		else {
			if (index+1==str.length() || str[index + 1] == 'X') {
				light++;
				index++;
			}
			else {
				light++;
				index += 3;//保证每个遍历到的'.'的前一个位置都没有灯，不会影响后续的判断
			}
		}
	}
	return light;
}
```

### 20.2 给定先序中序，找到后序

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/92.png)

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/93.png)

```c++
思路：
    1. 找到pre[i](当前根节点，用于区分左右树)在in数组中对应的位置find
    2. in数组中find左边对应左树，右边对应右树
    3. 对于二叉树，要有把整数拆分成各棵小树的想象力
    
//利用pre[prei...prej]，结合in[ini...inj]
//填写好pos[posi...posj]
void setPos(vector<int>& pre, vector<int>& in, vector<int>&pos, 
	int prei, int prej, 
	int ini, int inj, 
	int posi, int posj
    map<int,int>m) {
	if (prei > prej)return;
	if (prei == prej) {//只剩最后一个数，直接填
		pos[posi] = pre[prei];
	}
	pos[posj] = pre[prei];//先序遍历的第一个是后序遍历的最后一个
	int find = ini;
	for (; find <= inj; find++) {//找到当前根节点在in数组中的位置
		if (in[find] == pre[prei])break;
	}
	//左树
	setPos(pre, in, pos, prei + 1, prei + find - ini, ini, find - 1, posi, posi + find - ini-1);
	//右树，每次递归只搞定pos[j]，所以posj每次减1即可
	setPos(pre, in, pos, prei + find - ini + 1, prej, find + 1, inj, posi + find - ini, posj-1);
}

优化：优化找find的步骤，可用哈希表
void setPos(vector<int>& pre, vector<int>& in, vector<int>&pos, 
	int prei, int prej, 
	int ini, int inj, 
	int posi, int posj
    map<int,int>&m) {
	if (prei > prej)return;
	if (prei == prej) {//只剩最后一个数，直接填
		pos[posi] = pre[prei];
	}
	pos[posj] = pre[prei];//先序遍历的第一个是后序遍历的最后一个
	int find = m[pre[i]];//找到当前根节点在in数组中的位置
	//左树
	setPos(pre, in, pos, prei + 1, prei + find - ini, ini, find - 1, posi, posi + find - ini-1,m);
	//右树，每次递归只搞定pos[j]，所以posj每次减1即可
	setPos(pre, in, pos, prei + find - ini + 1, prej, find + 1, inj, posi + find - ini, posj-1,m);
}

vector<int>getPos(vector<int>&pre,vector<int>&in){
    int len=pre.size();
    vector<int>Pos(len);
    map<int,int>m;
    for(int i=0;i<len;i++){
        map[in[i]]=i;
    }
    setPos(pre,in,pos,0,len-1,0,len-1,0,len-1,0,len-1,m);
    return pos;
}
```

### 20.3 完全二叉树的节点个数

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/94.png)

```c++
方法一：遍历树(O(N))

方法二：
思路：
    1.一直往左子树移动，找到层数h
    2.判断右树的最左节点高度
      2.1等于h：左树肯定是满二叉树(高度为h-1)，n=(2^(h-1)-1)+1+右树节点数(继续递归)
      2.2小于h：右树肯定是满二叉树(高度为h-2)，n=(2^(h-2)-1)+1+左树节点数(继续递归)

class Node {
public:
	Node* left;
	Node* right;
	int value;

	Node(int value) {
		this->value = value;
		left = NULL;
		right = NULL;
	}
};

int mostLeftLevel(Node* node, int level) {
	while (node->left != NULL) {
		node = node->left;
		level++;
	}
	return level;
}

//node在level层，h是总深度，全局变量不变
int bs(Node* node, int level, int h) {
	if (level == h) return 1;
	if (mostLeftLevel(node->right, level + 1) == h) {//2.1
		return (1 << (h - level)) + bs(node->right, level + 1, h);
	}
	else {//2.2
		return (1 << (h - level - 1)) + bs(node->left, level + 1, h);//高度差不超过1，所以右树的高度肯定只比左树高度小1
	}
}
时间复杂度分析：
每次递归只选一侧移动，每次递归遍历（h-n）个节点，所以时间复杂度是O(h^2)，h=logN，O(h^2)=O((logN)^2)
```

### 20.4 最长递增子序列

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/95.png)

```c++
子序列：从左到右可以不连在一起
eg.{3,1,2,6,3,4}=>{1,2,3,4}

思路：动态规划
ends[i]：所有长度为i+1的递增子序列中最小结尾值，如果有更小的值的dp[i]满足长度要求，之前的位置需要被更新
ends数组的有效区域是单调递增的，因为长度为i的子序列最小结尾必定小于长度为i+1的子序列最小结尾，如果不满足，
就意味着长度为i+1的子序列中有更小的长度为i的子序列最小结尾(因为长度为i+1的子序列是包括长度为i的子序列的)
    
int maxSubsequence(vector<int> arr) {
	if (arr.size() <= 1)return arr.size();
	vector<int>ends(arr.size());
	ends[0] = arr[0];
	int right = 0;
	for (int i = 1; i < arr.size(); i++) {
		if (ends[right] < arr[i]) {//ends没有比arr[i]更大的
			right++;
			ends[right] = arr[i];
		}
		else {
			int l = 0;
			while (l <= right && ends[l] <= arr[i])l++;//找到ends最左边比arr[i]大的
			ends[l] = arr[i];
		}
	}
	//由于ends的特性，所以ends有多少有效区域就意味着最长子序列有多长
	return right + 1;
}
```

### 20.5 被3整除

![](https://github.com/Jomocool/Data_Structure_Algorithm/blob/main/zsLeetCode-img/96.png)

```c++
10%3可以等效看成(1+0)%3，所以也可以反向操作回去，直接算等差数列求和取模3即可
```

