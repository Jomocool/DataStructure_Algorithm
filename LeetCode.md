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


## [4. 寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)

```cpp
思路：
将问题转化为寻找两个正序数组第k大的数

当nums1[i]>nums2[j]时，意味着nums1[i]也大于nums2[j]之前的所有元素，所以即便是nums2[j](当前最大的)也不可能是第k小的数，最多是第k-1小的数，因为nums[i]>nums[j]，无论如何还有一个比它更大的且总数为k。这里我有一个误区就是，既然nums1[i]在nums1前面有k/2个比它小的，且nums2[j]之前也有k/2比它小的，那它直接不就是第k小的吗？实际不是的，因为我们还没有和nums2[j]之后的元素作比较，该方法节省了比较的算力，不需要重复去比较。

class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int len1=nums1.size();
        int len2=nums2.size();
        int left=(len1+len2+1)/2;
        int right=(len1+len2+2)/2;
        //奇数偶数都考虑到了
        return (getKth(nums1,0,len1-1,nums2,0,len2-1,left)+getKth(nums1,0,len1-1,nums2,0,len2-1,right))*0.5;
    }

    int getKth(vector<int>&nums1,int start1,int end1,vector<int>&nums2,int start2,int end2,int k){
        int len1=end1-start1+1;
        int len2=end2-start2+1;
        //让len1的长度小于len2，能够保证有数组为空时，一定是nums1
        if(len1>len2)return getKth(nums2,start2,end2,nums1,start1,end1,k);
        if(len1==0)return nums2[start2+k-1];
        if(k==1)return min(nums1[start1],nums2[start2]);

        int i=start1+min(len1,k/2)-1;//防止越界
        int j=start2+min(len2,k/2)-1;
        if(nums1[i]>nums2[j]){
            return getKth(nums1,start1,end1,nums2,j+1,end2,k-(j-start2+1));//不除2，直接减去被删掉数的个数
        }
        else{
            return getKth(nums1,i+1,end1,nums2,start2,end2,k-(i-start1+1));
        }
    }
};

时空复杂度分析:
时间复杂度：O(log(m+n))  每次比较都删去了k/2个元素;
空间复杂度：O(1)  因为该递归是尾递归，编译器不会不停地堆栈;

尾递归：如果一个函数中所有递归形式的调用都出现在函数的末尾，我们称这个递归函数是尾递归的。当递归调用是整个函数体中最后执行的语句且它的返回值不属于表达式的一部分时，这个递归调用就是尾递归。尾递归函数的特点是在回归过程中不用做任何操作，这个特性很重要，因为大多数现代的编译器会利用这种特点自动生成优化的代码。
```

## [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

```cpp
思路：
回文子串其实也可以看作是对称的字符串，所以我们可以选择字符成为对称轴，然后从其开始不断向两边拓展
class Solution {
public:
    int start=0;//记录起始位置
    int maxLen=0;//记录最长回文子串长度

    void extend(string&s,int i,int j){
        while(i>=0&&j<=s.length()-1&&s[i]==s[j]){//只有两边字符都相同时才继续向两边扩展
            if(j-i+1>maxLen){//如果当前回文子串的长度更大
                start=i;
                maxLen=j-i+1;
            }
            i--;
            j++;
        }
    }

    string longestPalindrome(string s) {
        for(int i=0;i<s.length();i++){
            //由于一次增加两个字符，所以起始时要考虑奇数和偶数的字符
            extend(s,i,i);
            extend(s,i,i+1);
        }
        return s.substr(start,maxLen);
    }
};

时空复杂度分析:
时间复杂度：O(n^2);
空间复杂度：O(1);
```

## [6. N 字形变换](https://leetcode.cn/problems/zigzag-conversion/)

```cpp
思路：
1.认识到总体一定是从左到右扫描字符串
2.模拟N字形时，其实就两种情况，一种是从上到下，另一种是从下到上
3.为防止重复，我的实现思路是从上到下时每一行都做处理，而从下到上时，只处理中间行，头行和尾行不做处理，因为这两行是两种情况的交界处
class Solution {
public:
    string convert(string s, int numRows) {
        vector<vector<char>>vec(numRows);//储存N字形
        string res;//结果字符串
        bool down=true;//是否从上到下
        for(int i=0;i<s.length();){
            if(down){
                int count=0;
                while(count<numRows){//注意对i的判断，防止溢出
                    if(i<s.length())vec[count].push_back(s[i++]);
                    count++;
                }
                down=false;//切换模式
            }
            else{
                int count=numRows-2;//不处理最后一行
                while(count>0){//不处理第一行
                    if(i<s.length())vec[count].push_back(s[i++]);
                    count--;
                }
                down=true;//切换模式
            }
        }
        for(int i=0;i<vec.size();i++){
            for(int j=0;j<vec[i].size();j++){
                res.push_back(vec[i][j]);//遍历N字形数组，并加入到结果字符串中
            }
        }
        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n)  i是不断后移的;
空间复杂度：O(n)  s有多少个字符就需要多少个位置;
```

## [7. 整数反转](https://leetcode.cn/problems/reverse-integer/)

```cpp
思路：
1.本题首先注意符号，其次就是反转后是否溢出
2.判断溢出需要提前预判

class Solution {
public:
    int reverse(int x) {
        int res=0;//结果
        while(x!=0){
            int offset=x%10;
            //关键点：判断溢出，需要提前判断
            if(abs(res)>2147483647/10||(abs(res)==2147483647/10&&(offset<-8||offset>7)))return 0;
            res=res*10+offset;
            x/=10;
        }
        return res;
    }
};

时空复杂度分析:
时间复杂度：O(logn)  x每次都除以10;
空间复杂度：O(1)  只用到res;
```

## [8. 字符串转换整数 (atoi)](https://leetcode.cn/problems/string-to-integer-atoi/)

```cpp
思路：
1.丢弃前导空格
2.检查正负号
3.读取数字字符直到非数字字符或s的结尾

class Solution {
public:
    int myAtoi(string s) {
        //1.丢弃前导空格
        int start=0;
        while(s[start]==' '){
            start++;
        }
        //2.检查正负号
        int flag=1;//默认1
        int count=0;//记录符号数，如果符号过多不符合要求，返回0
        if(s[start]=='+'){
            start++;
            count++;
        }
        if(s[start]=='-'){
            flag=-1;
            start++;
            count++;
        }
        if(count>1){//多余一个符号
            return 0;
        }
        //3.读取数字字符直到非数字字符或s的结尾
        int res=0;//结果
        while(start<s.length()&&s[start]<='9'&&s[start]>='0'){
            if(res==0&&s[start]=='0'){
                start++;
            }else{
                int offset=flag*(s[start]-'0');//因为下面不是判断绝对值了，所以直接乘符号
                //因为下面需要offset的真实大小，所以上面需要先乘符号，不能一直为正，否则过小时，无法正常输出，因为无法判断小于-8
                if(res>214748364||(res==214748364&&offset>7))return 2147483648-1;
                if(res<-214748364||(res==-214748364&&offset<-8))return -2147483648;
                //因为不是过大过小越界的返回值不同，所以不能一起用绝对值判断了
                res=res*10+offset;
                start++;
            }
        }
        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

## [9. 回文数](https://leetcode.cn/problems/palindrome-number/)

```cpp
思路：
1.判断正负，小于0不是回文数
2.注意存储x的值，因为后续还需要用x去作比较，所以不能修改x的值
3.但凡是反转值，就要注意越界问题
4.注意缩小tmp值

class Solution {
public:
    bool isPalindrome(int x) {
        int converse=0;//反转后的数
        int tmp=x;//因为后续还需要和x比较，所以不能修改x的值
        if(x<0)return false;//如果带负号，不是回文
        while(tmp>0){
            int offset=tmp%10;
            if(abs(converse)>214748364||(abs(converse)==214748364&&(offset<-8||offset>7))){
                return false;//越界了，肯定不等于x，因为x是int值
            }
            converse=converse*10+offset;
            tmp/=10;
        }
        return converse==x;
    }
};

时空复杂度分析:
时间复杂度：O(logn)  对于每次迭代，都将tmp除10;
空间复杂度：O(1)  只需要常数空间存放变量;

优化：反转一半数字
注意：
1.负号
2.如果x的最后一位是0，那么第一位也要是0，这样x只能等于0，注意x是合法的int值
3.判断已经反转了一半的前提是前半部分值小于等于后半部分反转值

class Solution {
public:
    bool isPalindrome(int x) {
        if(x<0||(x%10==0&&x!=0))return false;
        int converse=0;
        while(converse<x){
            converse=converse*10+x%10;
            x/=10;
        }
        //如果x的长度是偶数，那么二者长度一样
        //如果x的长度是奇数，converse长度多1，可以舍弃最后一个
        return converse==x||converse/10==x;
    }
};

时空复杂度分析:
时间复杂度：O(logn)  对于每次迭代，都将tmp除10;
空间复杂度：O(1)  只需要常数空间存放变量;
```

## [10. 正则表达式匹配](https://leetcode.cn/problems/regular-expression-matching/)

**参考视频：**[LeetCode 每日一题10. 正则表达式匹配 | 手写图解版思路 + 代码讲解](https://www.bilibili.com/video/BV1Br4y1v7SA?vd_source=cc4b252c81c96c974da321a0c636cc6e)

![](https://github.com/Jomocool/DataStructure_Algorithm/blob/main/LeetCode%E9%A1%BA%E5%BA%8F%E5%88%B7-img/1.png)

```cpp
思路：
1.字符串s只包含a-z的小写字母
2.模式串p只包含a-z的小写字母和*、.
3.动态规划表dp，dp[i][j]：p[0,j]是否涵盖s[0,i]
4.分类讨论：
  4.1 p[j]!='*'：dp[i][j]=dp[i-1][j-1]&&s[i]==s[j]
  4.2 p[j]=='*'：dp[i][j]=dp[i][j-2]||(dp[i-1][j-2]&&s[i]==p[j-1])||(dp[i-2][j-2]&&s[i-1]==p[j-1]&&s[i]==p[j-1])||......
      可由上图化简为 dp[i][j]=dp[i][j-2]||(dp[i-1][j]&&s[i]==p[j-1])
  注：因为*匹配多个p[j-1]，所以后面都是和p[j-1]比较。a*可以转为{ε,a,aa,aaa,......}
5.注意边界，如果越过边界了肯定无法匹配了

class Solution {
public:
    bool isMatch(string s, string p) {
        int n=s.length();
        int m=p.length();
        //前面加一个空格是为了避免繁琐的初始化
        s=' '+s;
        p=' '+p;
        vector<vector<bool>>f(n+1,vector<bool>(m+1));
        f[0][0]=true;

        //s="",p=".*"
        for(int i=0;i<=n;i++){
            for(int j=1;j<=m;j++){//无需处理第0列大于第0行的部分，因为空正则表达式无法匹配非空字符串
                //如果p的下一个字符是*，那就让*处理，因为*包括了当前字符单独匹配的情况
                if(j+1<=m&&p[j+1]=='*')continue;
                if(p[j]!='*'){
                    f[i][j]=i&&j&&f[i-1][j-1]&&(s[i]==p[j]||p[j]=='.');//i,j>0
                }else{
                    f[i][j]=(j>=2&&f[i][j-2])||(i&&j&&f[i-1][j]&&(s[i]==p[j-1]||p[j-1]=='.'));
                }
            }
        }

        return f[n][m];
    }
};

在代码中，在字符串s和p前添加空格是为了避免在处理边界情况时需要进行繁琐的初始化。具体来说，我们需要初始化f数组的第0行和第0列，分别表示s为空字符串和p为空正则表达式的情况。在处理第0列时，如果不在p的前面添加空格，那么我们需要单独处理p的第一个字符，即p[0]是否为"*"。在处理第0行时，如果不在s的前面添加空格，那么我们需要单独处理s为空字符串的情况。为了避免这些特殊情况的处理，我们可以在s和p的前面添加空格，这样f[0][0]就表示空字符串和空正则表达式的匹配。此外，由于空格不在s和p中出现，所以在匹配过程中不会对结果产生影响。

时空复杂度分析:
时间复杂度：O(nm)  动态规划表f;
空间复杂度：O(nm)  完善f;
```

## [11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

```cpp
思路：双指针
1.短板效应：盛水容器的宽由短板决定。
2.假设双指针分别指向两边的某一个块板，计算完容积后，移动指针，有两种移法
  2.1移动短板：长度一定减少，但是宽度可能增加，有几率盛更多的水
  2.2移动长板：长度一定减少，宽度要么不变，要么减少，容积一定减小
  2.3综上，我们只能移动短板
3.记录最大容积

class Solution {
public:
    int maxArea(vector<int>& height) {
        int left=0;
        int right=height.size()-1;
        int maxCover=0;
        while(left<right){
            maxCover=max(maxCover,(right-left)*(min(height[left],height[right])));//记录最大容积
            height[left]<height[right]?left++:right--;//移动短板
        }
        return maxCover;
    }
};

时空复杂度分析:
时间复杂度：O(n)  遍历height数组;
空间复杂度：O(1)  常数个变量记录需要的值;
```

## [12. 整数转罗马数字](https://leetcode.cn/problems/integer-to-roman/)

```cpp
思路：
1.记录每位的罗马数字格式，然后拼接而成
2.1<=num<=3999
3.用字符串数组记录各数位上对应数字的罗马数字格式

class Solution {
public:
    string intToRoman(int num) {
        string thounds[]={"","M","MM","MMM"};
        string hundreds[]={"","C","CC","CCC","CD","D","DC","DCC","DCCC","CM"};
        string tens[]={"","X","XX","XXX","XL","L","LX","LXX","LXXX","XC"};
        string ones[]={"","I","II","III","IV","V","VI","VII","VIII","IX"};

        return thounds[num/1000]+hundreds[num%1000/100]+tens[num%100/10]+ones[num%10/1];
    }
};

时空复杂度分析:
时间复杂度：O(1)  计算量与数字大小无关;
空间复杂度：O(1)  只需要用到常数个字符串数组;
```

## [13. 罗马数字转整数](https://leetcode.cn/problems/roman-to-integer/)

```cpp
思路：
1.如果当前罗马字符小于右边相邻的罗马字符，则减，否则加

class Solution {
public:
    int romanToInt(string s) {
        unordered_map<char,int>map{
            {'I',1},
            {'V',5},
            {'X',10},
            {'L',50},
            {'C',100},
            {'D',500},
            {'M',1000}
        };
        int res=0;
        for(int i=0;i<s.length();i++){
            if(i==s.length()-1){
                res+=map[s[i]];//如果i是最后一位了，肯定是加
                break;//结束循环，不然会向下继续运算
            }
            if(map[s[i]]<map[s[i+1]]){
                res-=map[s[i]];
            }else{
                res+=map[s[i]];
            }
        }
        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n)  需要遍历字符串s;
空间复杂度：O(1)  只需要用到常数个变量;
```

## [14. 最长公共前缀](https://leetcode.cn/problems/longest-common-prefix/)

```cpp
思路：纵向比较法
1.以strs[0]为基准字符串

class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        for(int i=0;i<strs[0].length();i++){//以strs[0]为基准
            char c=strs[0][i];
            for(int j=1;j<strs.size();j++){//strs[0][i]是不等的字符，所以不拷贝
                if(i>strs[j].length()-1||strs[j][i]!=c)return strs[0].substr(0,i); 
            }
        }
        //如果能够出循环，说明遍历完了strs[0]，即strs[0]是最短的字符串，直接返回它即可
        return strs[0];
    }
};

时空复杂度分析:
时间复杂度：O(mn)  其中m是字符串数组中的字符串的平均长度，n是字符串的数量。最坏情况下，字符串数组中的每个字符串的每个字符都会被比较一次;
空间复杂度：O(1);
```

## [15. 三数之和](https://leetcode.cn/problems/3sum/)

```cpp
1.蛮力法
思路：
3层for循环依次找到i、j、k，有两点需要注意，因为不能有重复的集合，为了方便跳过重复的元素，我们提前排好序，让等值的集中在一起，但不能简单的判断当前值等于前一个值就跳过，因为所属范围不同，也就不等价。比如[0,0,0]，如果单纯的判断为当前值等于前一个值就跳过的话那么结果集就是空的了，实际上不为空，所以结果错误了，因为在第一层for循环的时候，就一直跳直接结束了，不会进入第二层for循环。我们可以把等值的元素看作一整块，对于每一块我们只需要其第一个元素即可，剩余的直接跳过，所以在每层for循环多加一个判断，如果不是每一块的第一个元素，并且该元素属于当前块，则直接跳过，这样就可以保证没有重复的结果了。

class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        sort(nums.begin(),nums.end());//排序数组，让相等的值集中在一起
        vector<vector<int>>res;//结果集
        
        for(int i=0;i<nums.size()-2;i++){
            if(nums[i]>0)break;//nums[i]是三者中最小的，如果其大于0，后面的必然都大于0
            if(i!=0&&nums[i]==nums[i-1])continue;
            for(int j=i+1;j<nums.size()-1;j++){
                if(nums[i]+nums[j]>0)break;//二者和大于0，说明二者最大值必然大于0，而nums[k]是三者最大的，必然也大于0，三者和肯定也大于0
                if(j-1>i&&nums[j]==nums[j-1])continue;
                for(int k=j+1;k<nums.size();k++){
                    if(nums[i]+nums[j]+nums[k]>0)break;
                    if(k-1>j&&nums[k]==nums[k-1])continue;
                    if(nums[i]+nums[j]+nums[k]==0){
                        vector<int>vec(3);
                        vec[0]=nums[i];
                        vec[1]=nums[j];
                        vec[2]=nums[k];
                        res.push_back(vec);
                    }
                }
            }
        }

        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n^3)  3层for循环，超出时间限制;
空间复杂度：O(1)  返回结果不算额外空间;

2.双指针
引入：
对于一个排好序的数组，可以利用双指针法，找到和为0的二元组。
(1)如果和为0，两边都需要跳过重复的元素
(2)如果和小于0，左边跳过重复的元素
(3)如果和大于0，右边跳过重复的元素

思路：
通过引入，我们可以把-nums[i]当作我们的目标值，从而找到三元组

class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        sort(nums.begin(),nums.end());//排序是关键
        vector<vector<int>>res;
        for(int i=0;i<nums.size()-2;i++){
            if(i>0&&nums[i]==nums[i-1])continue;//去重nums[i]
            int left=i+1;
            int right=nums.size()-1;
            while(left<right){
                if(nums[i]+nums[left]+nums[right]==0){
                    res.push_back({nums[i],nums[left],nums[right]});
                    while(left<right&&nums[left]==nums[left+1])left++;//去重nums[left]
                    while(left<right&&nums[right]==nums[right-1])right--;//去重nums[right]
                    //这样不用额外考虑right是最后一位时的判断情况
                    //此时left在当前块的最后一位
                    //此时right在当前块的第一位
                    //各自再移动一位就到下一块了
                    left++;
                    right--;
                }
                else if(nums[i]+nums[left]+nums[right]<0){//nums[left]偏小
                    left++;
                }
                else{//nums[right]偏大
                    right--;
                }
            }
        }
        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n^2)  双指针提高了时间效率;
空间复杂度：O(1)  返回结果不算额外空间;
```

## [16. 最接近的三数之和](https://leetcode.cn/problems/3sum-closest/)

```cpp
思路：
1.最接近：保留差的绝对值
2.排序
3.偏小右移左指针，偏大左移右指针

class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {
        int res=nums[0]+nums[1]+nums[2];//用数组的元素初始化res，因为本身也是数组的三数之和去和其他三数之和比较
        sort(nums.begin(),nums.end());
        for(int i=0;i<nums.size()-2;i++){
            int left=i+1;
            int right=nums.size()-1;
            while(left<right){
                int sum=nums[i]+nums[left]+nums[right];//三数之和
                if(sum==target)return sum;
                if(abs(sum-target)<abs(res-target))res=sum;
                if(sum<target)left++;//偏小
                if(sum>target)right--;//偏大
            }
        }
        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n^2)  双指针提高了时间效率;
空间复杂度：O(1)  返回结果不算额外空间;
```

