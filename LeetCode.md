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
双指针
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

## [17. 电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

```cpp
思路：递归
1.对于digits的一个数字就代表某一层递归，而每层递归需要遍历自己所代表的所有字符
2.每层递归处理path的方式是，在path加上自己当前所代表的字符后，交给下一层去处理
3.index下标表示当前所在层，刚好可以用来做base case的判断，如果经历过所有层递归后，就可以把path加入结果集了

class Solution {
public:
    vector<string>res;//结果集
    unordered_map<char,string>map{//映射表
        {'2',"abc"},{'3',"def"},{'4',"ghi"},{'5',"jkl"},
        {'6',"mno"},{'7',"pqrs"},{'8',"tuv"},{'9',"wxyz"}
    };
	
    //递归函数
    void traversal(string& digits,int index,string path){
        if(index==digits.length()){//遍历完所有数字后，path完型了，加入结果集
            res.push_back(path);
            return;
        }

        for(int j=0;j<map[digits[index]].length();j++){//遍历当前数字代表的所有字符
            traversal(digits,index+1,path+map[digits[index]][j]);//由于是交给下一层处理，所以idnex要加1
        }
    }

    vector<string> letterCombinations(string digits) {
        res.clear();
        if(digits.length()==0)return res;
        traversal(digits,0,"");
        return res;
    }
};

一开始有一个误区，就是在每一层递归中，外面多加了一个for循环，i:index->digits.length()，想着这样是交给后面处理，但实际上index+1时，下一层也会有index+1，这样就已经能够把digits遍历完了，不需要再额外处理了，不然会有很多不符合预期的额外结果。多加一层for循环可以找到以不同数字为起点的，以最后一个数字为终点的所有结果。

时空复杂度分析:
时间复杂度：O(n)  digits有多长，就有几层递归，每次递归的for循环只是常数个字符而已;
空间复杂度：O(n)  path的最终长度和digits一样;
```

## [18. 四数之和](https://leetcode.cn/problems/4sum/)

```cpp
思路和三数之和类似，唯一要注意的点是，四数之和可能会溢出，需要一个比int更大范围的数去保存

class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        sort(nums.begin(),nums.end());//先排序，集中等值元素
        vector<vector<int>>res;//结果集合
        if(nums.size()<4)return res;//如果nums元素个数小于3，nums.size()-3不符合预期
        for(unsigned int i=0;i<nums.size()-3;i++){
            if(i>0&&nums[i]==nums[i-1])continue;
            for(unsigned int j=i+1;j<nums.size()-2;j++){
                if(j>i+1&&nums[j]==nums[j-1])continue;
                int left=j+1;
                int right=nums.size()-1;
                while(left<right){
                    long sum=(long)nums[i]+nums[j]+nums[left]+nums[right];//将第一个数改为long类型后，后面和都是long类型了
                    if(sum==target){
                        res.push_back({nums[i],nums[j],nums[left],nums[right]});
                        while(left<right&&nums[left]==nums[left+1])left++;
                        while(left<right&&nums[right]==nums[right-1])right--;
                        left++;
                        right--;
                    }
                    else if(sum>target){
                        right--;
                    }
                    else{
                        left++;
                    }
                }
            }
        }

        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n^3);
空间复杂度：O(1);
```

## [19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

```cpp
思路：快慢指针
1.快指针先走n步
2.慢指针等快指针走完n步后开始移动
3.等快指针走到最后一个节点时，慢指针刚好就在倒数第n个结点的前一个节点了
4.由于要删掉倒数第n个节点，所以需要其前一个节点
注意：head没有前一个节点，所以需要额外判断

class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        if(!head->next){//如果只有一个节点，必定只能删除当前节点
            return nullptr;
        }
        ListNode* fast=head;
        while(n>0&&fast){
            fast=fast->next;
            n--;
        }

        if(!fast)return head->next;//如果fast是空，说明n==节点数，所以要删除的是头节点，直接从第二个节点开始返回即可

        ListNode*slow=head;
        while(fast->next){
            fast=fast->next;
            slow=slow->next;
        }
        
        slow->next=slow->next->next;

        return head;
    }
};

时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

## [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)

```cpp
class Solution {
public:
    bool isValid(string s) {
        stack<char>stk;//辅助栈
        unordered_map<char,char>map{
            {'(',')'},
            {'[',']'},
            {'{','}'}
        };
        for(int i=0;i<s.length();i++){
            if(map.find(s[i])!=map.end()){//左括号，入栈
                stk.push(s[i]);
            }
            else{//右括号
                if(stk.empty())return false;//有右括号，但是栈为空，说明没有对应左括号
                char c=stk.top();
                stk.pop();
                if(map[c]!=s[i])return false;
            }
        }
        return stk.empty();//栈里的左括号需要被全部匹配完
    }
};

时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n)  辅助栈;
```

## [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)

```cpp
思路：双指针
1.指针head1和head2分别指向链表1、2
2.谁小谁先移动，然后把较小的那个节点加入到新链表中
3.直到某一个指针为空时，把另一个还为空的链表插入到新链表尾部

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
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode*head1=list1;
        ListNode*head2=list2;
        ListNode*res=new ListNode(-1);//结果链表头的前驱节点
        ListNode*cur=res;//用于操作结果链表

        while(head1&&head2){//head1和head2都不为空
            if(head1->val<head2->val){
                cur->next=head1;
                head1=head1->next;
            }else{
                cur->next=head2;
                head2=head2->next;
            }
            cur=cur->next;
        }
        cur->next=head1?head1:head2;//谁不为空，就在尾部加入谁

        return res->next;//返回的是前驱节点的下一个，因为这才是我们添加的第一个节点
    }
};

时空复杂度分析:
时间复杂度：O(n+m)  list1和list2的长度和;
空间复杂度：O(1)  辅助链表属于返回结果，不算额外空间;
```

## [22. 括号生成](https://leetcode.cn/problems/generate-parentheses/)

```cpp
思路：递归
1.每一层让path添加左括号或右括号
2.注意括号要有效，所以当右括号数量和左括号持平时，一定只能加左括号了
3.避免添加的括号数过多导致递归停不下来，每次递归前加一个判断，避免括号过多

class Solution {
public:
    vector<string>res;

    void traversal(string path,int left, int right,int n){
        if(left==n&&right==n){
            res.push_back(path);
            return;
        }
        
        //括号数不能超过n，所以要在每次递归前判断当前括号数是否会溢出
        if(right==left){//左右括号数相同，只能加左括号
            if(left<=n-1)traversal(path+"(",left+1,right,n);
        }else{
            if(left<=n-1)traversal(path+"(",left+1,right,n);
            if(right<=n-1)traversal(path+")",left,right+1,n);
        }
    }

    vector<string> generateParenthesis(int n) {
        res.clear();
        traversal("",0,0,n);
        return res;
    }
};
	

时空复杂度分析:
时间复杂度：O((4^n)/(n^(1/2)));
空间复杂度：O(n)  额外空间大小取决于递归栈的深度，而深度等于n;
```

## [23. 合并 K 个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)

```cpp
思路：
1.和21题类似
2.建一个小顶堆存放当前需要比较的k个节点
3.//对于基础类型 默认是大顶堆
  priority_queue<int> a; 
  //等同于 priority_queue<int, vector<int>, less<int> > a;
  priority_queue<int, vector<int>, greater<int> > c;  //这样就是小顶堆

  std::vector<int> values{21, 22, 12, 3, 24, 54, 56};
  std::priority_queue<int> numbers {std::less<int>(),values};//56 54 24 22 21 12 3

  greater：越来越大，less：越来越小

4.我们需要小顶堆，所以应该是越来越大，这样最小的元素就在vector最前面，即栈顶

class Solution {
public:
    struct Cmp{
        bool operator()(ListNode* a,ListNode*b){
            return a->val>b->val;
        }
    };

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        priority_queue<ListNode*,vector<ListNode*>,Cmp>minHeap;
        ListNode*dummy=new ListNode(-1),*cur=dummy;
        for(int i=0;i<lists.size();i++){
            if(lists[i])minHeap.push(lists[i]);//装入各链表头
        }

        while(!minHeap.empty()){
            ListNode*top=minHeap.top();minHeap.pop();
            cur->next=top;
            cur=cur->next;
            top=top->next;
            if(top)minHeap.push(top);//如果top的下一个节点不为空，就加入小顶堆
        }

        return dummy->next;
    }
};

时空复杂度分析:
时间复杂度：O(nlogk)  n是最长链表的长度，k是链表数，有几条链表就需要比较几次，小顶堆深度为logk;
空间复杂度：O(k)  每次都是进出各一个;

这题主要是找各链表当前指向的节点值的最小值，利用小顶堆的高效查找效率很方便的就找到最小值了，查找时间是O(1)，因为就是minHeap.top()，但是push到小顶堆的时间是取决于深度。
```

## [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)

```cpp
思路：
1.增加虚拟头节点，防止丢失整条链表
2.需要有三个指针，第一个pre指向当前待交换节点对的前驱节点，第二个first指向当前待交换节点对的第一个节点，第三个last指向当前待交换节点对第二个节点
3.开始交换
  3.1 first->next=last->next;
  3.2 last->next=first;
  3.3 pre->next=last;
注意：1和2的相对顺序一定不能反，否则会丢失last->next
4.移动指针到下一个要处理的地方
  4.1 pre=first;
  4.2 first=pre->next;
  4.3 last=first->next;
5.注意奇数个节点和偶数个节点的最后情况

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
    ListNode* swapPairs(ListNode* head) {
        if(head==nullptr||head->next==nullptr)return head;//如果节点数小于2，就没有交换的必要了

        ListNode*dummy=new ListNode(-1),*pre=dummy;
        dummy->next=head;
        ListNode*first=head;
        ListNode*last=head->next;

        //奇数个节点时，最后一对节点的last为空，不需要交换了
        //偶数个节点时，交换完最后一对节点后，first和last都为空
        while(first&&last){
            first->next=last->next;
            last->next=first;
            pre->next=last;

            pre=first;
            first=pre->next;//这里不用加空指针判断，因为pre不会为空，因为没有节点的前驱节点是nullptr
            if(first)last=first->next;//多加一个判断条件，防止空指针异常
        }

        return dummy->next;
    }
};

时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

## [25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)

```cpp
思路：
1.其实就是多次反转长度为k的链表
2.虚拟头节点指向整条链表，防止丢失
3.每次将k个节点的链表反转前，先让最后一个节点的next指向空，否则会出现next指向不明的情况

class Solution {
public:
    //翻转链表
    ListNode*reverse(ListNode*head){
        ListNode*dummy=new ListNode(-1);
        ListNode*cur=head;
        ListNode*next=cur->next;
        while(cur){
            cur->next=dummy->next;
            dummy->next=cur;
            cur=next;
            if(cur)next=cur->next;
        }
        return dummy->next;
    }

    ListNode* reverseKGroup(ListNode* head, int k) {
        ListNode*dummy=new ListNode(-1),*cur=dummy;
        ListNode*beginNode=head;//当前起始节点
        ListNode*endNode=head;//当前截止节点
        ListNode*nextBeginNode=beginNode;//下一次翻转起始节点
        
        while(true){
            for(int i=0;i<k-1;i++){//移动k-1次就已经是第k个节点了
                if(endNode==nullptr)break;
                endNode=endNode->next;
            }
            //如果endNode是空，说明最后不够k个了，不用翻转了
            if(endNode==nullptr)break;

            nextBeginNode=endNode->next;//记录下一次开始翻转的节点
            endNode->next=nullptr;//截止节点的next指向空

            cur->next=reverse(beginNode);//开始反转，并让cur指向翻转后的头节点

            cur=beginNode;//此时，beginNode是翻转后的链表的最后一个节点，刚好是我们所需的前驱节点
            beginNode=nextBeginNode;//跳到下一次开始的地方
            endNode=beginNode;//重置endNode
        }

        cur->next=nextBeginNode;//最后没被翻转的链表需要补充到尾部
        return dummy->next;
    }
};

时空复杂度分析:
时间复杂度：O(n)  (n/k)*k;
空间复杂度：O(1);
```

## [26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)

```cpp
思路：快慢指针
1.确定数组中不同的元素个数，假设为n
2.slow的大小可以理解成当前已经填入到nums前面的不重复的元素个数，而对应的索引自然就是下一个不重复的元素需要填入的下标
3.fast可以理解成去寻找不重复元素的索引

class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int n=1;//第一个元素必然不会重复，所以n初始化成1，即至少有一个不重复的值

        //由于是有序数组，所以相同元素集中在一块
        for(int i=1;i<nums.size();i++){
            if(nums[i-1]!=nums[i])n++;
        }
        
        //第一个元素必然不会重复，所以不处理
        int slow=1;
        int fast=1;
        while(slow<n){
            while(nums[fast]==nums[fast-1])fast++;
            nums[slow++]=nums[fast++];//重写覆盖掉前面重复的值，不仅要移动slow，也要移动fast，不然fast会卡住
        }

        nums.resize(n);
        return nums.size();
    }
};

时空复杂度分析:
时间复杂度：O(n)  fast和slow一直在向后移，而不会重复计算;
空间复杂度：O(1);
```

## [27. 移除元素](https://leetcode.cn/problems/remove-element/)

```cpp
思路：
和26题类似

class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int count=0;//和val等值的元素的个数
        for(int i=0;i<nums.size();i++){
            if(nums[i]==val)count++;
        }

        int n=nums.size()-count;//新数组长度
        int slow=0;
        int fast=0;
        while(slow<n){
            while(nums[fast]==val)fast++;//跳过等于val的元素
            nums[slow++]=nums[fast++];
        }

        nums.resize(n);
        return n;
    }
};

时空复杂度分析:
时间复杂度：O(n)  fast和slow一直在向后移，而不会重复计算;
空间复杂度：O(1);
```

## [28. 找出字符串中第一个匹配项的下标](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/)

```cpp
思路：kmp算法

class Solution {
public:
    vector<int>next;//next[i]，字符串从0~i-1的最长公共前后缀长度

    void setNext(string& needle){
        if(needle.length()==1){
            next={-1};
            return;
        }
        next.resize(needle.length());
        next[0]=-1;
        next[1]=0;
        int i=2;
        int cn=0;
        while(i<next.size()){
            if(needle[i-1]==needle[cn]){
                next[i++]=++cn;
            }
            //如果字符不匹配，但当前公共前后缀大于0的话就让当前字符needle[i]和更前面的最长公共前后缀作比较
            else if(cn>0){//{[()()][()()]}，即让最左边的小括号子串的下一个字符和needle[i]比较
                cn=next[cn];
            }
            else{
                next[i++]=0;
            }
        }
    }

    int strStr(string haystack, string needle) {
        next.clear();
        setNext(needle);

        int index=0;
        int j=0;
        while(index<haystack.length()&&j<needle.length()){
            if(haystack[index]==needle[j]){
                index++;
                j++;
            }else if(j==0){//说明第一个字符就匹配不成功
                index++;
            }else{
                j=next[j];//j之前都是匹配成功的，下一次直接从略过匹配成功的子串，跳到匹配不成功的位置
            }
        }

        return j==needle.length()?index-j:-1;
    }
};

时空复杂度分析:
时间复杂度：O(n+m)  n是haystack的长度，m是needle的长度，至多需要遍历两字符串各一次;
空间复杂度：O(m)  needle字符串的前缀函数;
```

## [29. 两数相除](https://leetcode.cn/problems/divide-two-integers/)

```cpp
思路：位运算
1.处理边界条件
2.准备candidates数组，提前记录好divisor左移的结果，空间换时间
3.遍历candidates数组，如果当前dividend小于candidates[i]，说明有(1<<i)个divisor。然后要注意因为我们一开始就取反了，所以越小反而绝对值越大。

class Solution {
public:
    int divide(int dividend, int divisor) {
        //被除数为最小值
        if(dividend==INT_MIN){
            if(divisor==1)return INT_MIN;
            if(divisor==-1)return INT_MAX;
        }
        //除数为最小值
        if(divisor==INT_MIN){
            return dividend==INT_MIN?1:0;
        }

        if(dividend==0){
            return 0;
        }

        //将所有正数取反，因为负数的范围更大
        //异号为负
        bool flag=(dividend>0&&divisor<0)||(dividend<0&&divisor>0);
        dividend=dividend>0?-dividend:dividend;
        divisor=divisor>0?-divisor:divisor;
		
        /*
        关键点：
        eg.39/3=13=(1101)，所以39=3*2^3+3*2^2+3*2^0用数组存放3,3+3,(3+3)+(3+3)……,candidates[i]=divisor*2^i，如果dividend小于candidates[i]，说明有(1<<i)个divisor。注意因为是负数，所以越小绝对值反而越大
        */
        vector<int>candidates={divisor};
        while(candidates.back()>=dividend-candidates.back()){//防止溢出
            candidates.push_back(candidates.back()+candidates.back());
        }
        
        int res=0;//商
        for(int i=candidates.size()-1;i>=0;i--){
            if(candidates[i]>=dividend){
                res|=(1<<i);
                dividend-=candidates[i];
            }
        }

        return flag?-res:res;
    }
};

时空复杂度分析:
时间复杂度：O(logn)  遍历candidates;
空间复杂度：O(logn)  candidates的大小;
```

## [30. 串联所有单词的子串](https://leetcode.cn/problems/substring-with-concatenation-of-all-words/)

```cpp
思路：滑动窗口+哈希表
class Solution {
public:
    vector<int> findSubstring(string s, vector<string>& words) {
        vector<int>res;//结果集
        if(s.length()==0||words.size()==0)return res;
        
        //key:words[i],value:words[i]的词频
        unordered_map<string,int>words_map;
        for(int i=0;i<words.size();i++){
            words_map[words[i]]++;
        }

        int w=words[0].length();//单个单词的大小
        int len=w*words.size();//窗口大小

        //由于对顺序没有要求，那只需要保证子串中每个单词的词频和要求的一致即可
        unordered_map<string,int>count_map;
        for(int start=0;start+len<=s.length();start++){
            count_map.clear();//注意每个新的窗口都要先清空原先的记录
            int j=0;
            for(;j<len;j+=w){
                string substr=s.substr(start+j,w);
                if(words_map[substr]){
                    count_map[substr]++;
                    //词频不匹配
                    if(count_map[substr]>words_map[substr])break;
                }else{//没有当前单词，也不匹配
                    break;
                }
            }
            if(j==len){//匹配完全了
                res.push_back(start);
            }
        }

        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n*m)  n是s长度，m是窗口大小;
空间复杂度：O(n)  n是words大小，哈希表存放词频;
```

## [31. 下一个排列](https://leetcode.cn/problems/next-permutation/)

```cpp
思路：
1.从右到左找到第一个元素的下标j，特点是这个元素大于其右边的元素
解释：因为当一组元素是从左到右递减排列时，已经就是最大排列了，因为最大的数放在了最高位，必然是最大的
2.从j的右边找到大于nums[j]并且最接近nums[j]的元素下标，然后和nums[j]交换
解释：为什么不从左边找呢？因为左边的权重更大，动左边的必然会引起巨大的变化，就不是题目要求的下一个排列了。
3.可能会出现一种情况，即整个数组是递减的，说明是最大排列，返回最小排列即可，用反转比排序快
4.交换完后，需要把j之后的元素都反转成从左到右的递增序列，这样才能确保是最小的，也就最接近上一个排列了
解释：eg.最接近12378654的下一个排列是12384567

class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int j=nums.size()-2;
        while(j>=0&&nums[j]>=nums[j+1]){//等于也需要左移，我们需要找到严格小于其右边元素的元素下标
            j--;
        }

        //结束循环后，j右边的元素是递减的
        if(j==-1){
            //说明整个nums数组是递减的，也就是最大排列了，没有更大的，反转后变最小，返回
            reverse(nums.begin(),nums.end());
            return;
        }
        
        int index=0;
        int minDiff=INT_MAX;
        int cur=nums.size()-1;
        while(cur>j){
            int diff=nums[cur]-nums[j];
            if(nums[cur]>nums[j]&&diff<minDiff){//一定要保证nums[j]交换后变大了，因为后面还有反转
                index=cur;
                minDiff=diff;
            }
            cur--;
        }

        swap(nums[j],nums[index]);
        reverse(nums.begin()+j+1,nums.end());
    }
};

时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(1);
```

## [32. 最长有效括号](https://leetcode.cn/problems/longest-valid-parentheses/)

```cpp
思路：
1.有效括号串的条件：
  1.1任意[前缀]中'('的数量>=')'的数量
  1.2'('的数量==')'的数量
2.找到合法的有效括号子串，就需要找到其左右边界
  2.1右边界可通过性质1(1.1)决定，找到右边界后需要重置左右括号数量，然后继续找下一段的右边界
  2.2可将字符串分成n段，在段内可通过栈进行左右括号匹配，根据栈内剩余左括号数量可以确定右边界
3.右括号匹配前判断一次栈是否空，匹配后判读一次栈是否空
  3.1匹配前判断栈是为了防止无左括号可以匹配，导致弹栈异常，同时可以知道右括号多于左括号了，找到右边界
  3.2匹配后判断栈是为了计算结果
4.当某段前缀的右括号数量更多时，其必定不能再和后面的括号串组成连续有效括号串了，所以我们以此为依据分段，互不干扰

class Solution {
public:
    int longestValidParentheses(string s) {
        stack<int>stk;//存放左括号的下标
        int res=0;//结果

        //start是右边界
        for(int i=0,start=-1;i<s.length();i++){
            if(s[i]=='(')stk.push(i);//碰到左括号直接入栈
            else{//碰到右括号
                if(!stk.empty()){//如果栈不为空，和左括号匹配
                    stk.pop();
                    if(stk.empty()){
                        //说明整个栈的左括号都匹配完了，上一次空栈是在前一段的右边界
                        res=max(res,i-start);
                    }else{
                        //stk.top()就是左边界
                        res=max(res,i-stk.top());
                    }
                }else{
                    //说明右括号大于左括号了，找到当前段的右边界了
                    start=i;
                }
            }
        }

        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n);
空间复杂度：O(n);
```

## [33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/)

```cpp
思路：
1.旋转后的数组可看作两个升序数组，且前段升序数组的任何一个值都大于后段升序数组的所有值
2.特殊值是左边升序数组的最小值，即nums[0]，依此和nums[mid]比较来判断当前是在前段有序数组还是后段有序数组
3.将数组一分为二，其中一定有一个是有序的，另一个可能是有序，也能是部分有序。此时有序部分用二分法查找。部分有序部分再一分为二，其中一个一定有序，另一个可能有序，可能无序。循环下去。———— LeetCode用户@HaominYuan

class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left=0;
        int right=nums.size()-1;
        while(left<=right){
            int mid=left+((right-left)>>1);//外括号很重要，保证先右移再加
            if(nums[mid]==target)return mid;
            //和nums[0]或nums[nums.size()-1]比较时才加等号，因为target已经和nums[mid]比较过了，不可能相等才进入下面分支，其次是边界值可能就是target，需要向其靠近
            else if(nums[0]<=nums[mid]){//在前段有序数组，注意这里的等号，因为nums[0]也属于前半段
                if(nums[0]<=target&&target<nums[mid]){//[0,mid]是有序的，target属于这部分
                    right=mid-1;
                }else{//target不属于[0,mid]，去[mid+1,right]二分查找，这部分可能有序也可能无序
            //target<nums[0]||target>nums[mid]，而mid右边既有小于nums[0]的部分，也有大于nums[mid]的部分
                    left=mid+1;
                }
            }else{//在后段有序数组
                if(nums[mid]<target&&target<=nums[nums.size()-1]){//[mid,nums.size()-1]是有序的，target属于这部分
                    left=mid+1;
                }else{//target不属于[mid,nums.size()-1]，可能属于[left,mid-1]
           //target<nums[mid]||target>nums[nums.size()-1]，mid左边既有小于nums[mid]的部分，也有大于nums[nums.size()-1]的部分
                    right=mid-1;
                }
            }
        }
        return -1;
    }
};

时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```

## [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

```cpp
思路：
和二分查找一样，只不过在找到target值时，向左右扩展找到其第一个和最后一个位置

class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        int len=(int)nums.size();//长度，转为int型
        int left=0;//左边界
        int right=len-1;//右边界

        while(left<=right){
            int mid=left+((right-left)>>1);
            if(nums[mid]==target){
                int i=mid;
                int j=mid;
                while(i>=0&&nums[i]==target)i--;//向左扩展
                while(j<len&&nums[j]==target)j++;//向右扩展
                return {i+1,j-1};//注意边界，当前i,j所在位置的元素都不等于target，或者越界了，向内缩进一个
            }else if(nums[mid]<target){
                left=mid+1;
            }else{
                right=mid-1;
            }
        }

        return {-1,-1};
    }
};

时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```

## [35. 搜索插入位置](https://leetcode.cn/problems/search-insert-position/)

```cpp
思路：二分查找
nums为[无重复元素]的[升序]数组

class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        //处理边界
        if(target<=nums[0])return 0;
        if(target>nums[nums.size()-1])return nums.size();
        if(target==nums[nums.size()-1])return nums.size()-1;
        
		//上述边界都不符合要求，说明要插入的索引在[1,nums.size()-2]，在其中二分查找
        int left=1;
        int right=nums.size()-2;
        int mid=0;
        while(left<=right){
            mid=left+((right-left)>>1);
            if(nums[mid]==target){//如果存在target，直接返回其下标
                return mid;
            }else if(nums[mid]<target){
                if(nums[mid+1]>=target){//nums[mid]<target<=nums[mid+1]
                    return mid+1;
                }else{//target>nums[mid+1]>nums[mid]，所以要插入的索引在[mid+1,right]
                    left=mid+1;
                }
            }else{
                //这里nums[mid-1]不能等于target，否则返回值就不匹配了
                if(nums[mid-1]<target){//nums[mid-1]<target<nums[mid]
                    return mid;
                }else{//target<nums[mid-1]<nums[mid]，所以插入的索引在[left,mid-1]
                    right=mid-1;
                }
            }
        }
		
        //进入循环就必然有返回值，能到这一步说明没有进入循环，只有一种可能
		//即nums.size()==2，且target不处于边界，所以nums[0]<target<nums[1]，因此返回1
        return 1;
    }
};

优化代码：
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int l=0,r=nums.size()-1;
        while(l<=r){
            int mid=l+((r-l)>>1);
            if(nums[mid]<target)l=mid+1;//这里只判断小于很关键
            else r=mid-1;
        }
        return l;
    }
};
解释：
1.循环过程中：根据if判断条件，l左边都是小于target值的元素，r右边都是大于等于target值的元素
2.循环结束后：l=r+1，nums[0~r]<target<=nums[l~nums.size()-1]，所以返回l即可，考虑边界条件：
  2.1 target<nums[0]，r=-1，l=0，返回l符合要求
  2.2 target>nums[nums.size()-1]，r=nums.size()-1，l=nums.size()，返回l符合要求

时空复杂度分析:
时间复杂度：O(logn);
空间复杂度：O(1);
```

## [36. 有效的数独](https://leetcode.cn/problems/valid-sudoku/)

```cpp
思路：蛮力法
按数独规则去判断即可

class Solution {
public:
    bool isValidSudoku(vector<vector<char>>& board) {
        int row=board.size();
        int col=board[0].size();

        vector<bool>isExist(row,false);//判断数字是否已经出现
        //判断行是否合格
        for(int i=0;i<row;i++){
            isExist.assign(isExist.size(),false);
            for(int j=0;j<col;j++){
                if(board[i][j]=='.')continue;
                int t=board[i][j]-'1';//数字从1开始，而数组从0开始，所以减1对应下标
                if(isExist[t])return false;
                isExist[t]=true;
            }
        }

        //判断列是否合格
        for(int j=0;j<col;j++){
            isExist.assign(isExist.size(),false);
            for(int i=0;i<row;i++){
                if(board[i][j]=='.')continue;
                int t=board[i][j]-'1';//数字从1开始，而数组从0开始，所以减1对应下标
                if(isExist[t])return false;
                isExist[t]=true;
            }
        }

        //判断九宫格
        for(int i=0;i<row;i+=3){
            for(int j=0;j<col;j+=3){
                isExist.assign(isExist.size(),false);
                for(int x=i,countx=0;countx<3;x++,countx++){
                    for(int y=j,county=0;county<3;y++,county++){
                        if(board[x][y]=='.')continue;
                        int t=board[x][y]-'1';
                        if(isExist[t])return false;
                        isExist[t]=true;
                    }
                }
            }
        }

        //行，列，九宫格都有效
        return true;
    }
};

时空复杂度分析:
时间复杂度：O(1)  遍历81个格子;
空间复杂度：O(1)  isExist大小是9，常数空间;
```

## [37. 解数独](https://leetcode.cn/problems/sudoku-solver/)

```cpp
思路：DFS
1.定义3个数组rows、cols和cells分别记录各行、各列和各九宫格的数字出现情况
2.递归回溯

class Solution {
public:
    bool rows[9][9];//9行9个数
    bool cols[9][9];//9列9个数
    bool cells[3][3][9];//行列各分成3个九宫格，共有9个九宫格，9个数

    bool dfs(vector<vector<char>>&board,int row,int col){
        //超过第9列，跳到下一行并重置列
        if(col==9)row++,col=0;
        
        //超过第9行，说明数独已经填好了
        if(row==9)return true;

        //当前格已经填好数字了，不能修改
        if(board[row][col]!='.')return dfs(board,row,col+1);

        //枚举1~9，填入空格中
        for(int digit=0;digit<9;digit++){
            //当前行、列或九宫格出现过digit，说明不可以填，直接看下一个数
            if(rows[row][digit]||cols[col][digit]||cells[row/3][col/3][digit])continue;

            //可以填
            rows[row][digit]=cols[col][digit]=cells[row/3][col/3][digit]=true;
            board[row][col]=digit+'1';
            if(dfs(board,row,col+1))return true;

            //填了之后但是后面无法解出数独，回溯，填下一个数
            rows[row][digit]=cols[col][digit]=cells[row/3][col/3][digit]=false;
            board[row][col]='.';
        }

        //所有数都无法解出的话，说明当前路径走不通
        return false;
    }

    void solveSudoku(vector<vector<char>>& board) {
        //完善辅助数组
        memset(rows,0,sizeof(rows));
        memset(cols,0,sizeof(cols));
        memset(cells,0,sizeof(cells));
        for(int i=0;i<9;i++){
            for(int j=0;j<9;j++){
                if(board[i][j]=='.')continue;
                int t=board[i][j]-'1';
                rows[i][t]=cols[j][t]=cells[i/3][j/3][t]=true;
            }
        }

        dfs(board,0,0);
    }
};

分析：
每一格就代表着一层递归，而每一层的处理逻辑是枚举1~9，填入当前格子后，然后交给下一个格子去处理，而下一个格子只需要让列加1，因为列加到9之后通过前面的if判断会自动跳到下一行的第一列。其次就是边界条件的判断了，如果已经超过棋盘的行数了，说明已经填完所有格子了，已经找到解了，返回true
```

## [38. 外观数列](https://leetcode.cn/problems/count-and-say/)

```cpp
思路：
1.base case：n==1
2.每一层先获取前一层的字符串，再去分析

class Solution {
public:
    string traversal(int n){
        if(n==1)return "1";

        string preItem=traversal(n-1);
        
        string res;//结果字符串
        int index=0;
        while(index<preItem.length()){
            int count=1;//记录连续数字的长度，即有几个数
            while(index<preItem.length()-1&&preItem[index]==preItem[index+1]){
                index++;
                count++;
            }
            res.push_back(count+'0');//先添加词频
            res.push_back(preItem[index]);//再添加数字
            index++;//后移index，因为当前指向连续数字的最后一个
        }

        return res;
    }

    string countAndSay(int n) {
        return traversal(n);
    }
};

时空复杂度分析:
时间复杂度：O(nm)  n是给定的正整数，m为生成字符串的最大长度;
空间复杂度：O(m)  生成字符串的最大长度;
```

## [39. 组合总和](https://leetcode.cn/problems/combination-sum/)

```cpp
思路：回溯递归
1.每一层递归的处理逻辑是，从当前元素开始累加到target
2.每个元素可以取0到无数次，截止循环的条件是当前值取了太多次以至于超过target了，这时就该结束了
3.base case是target已经等于0的话就加入path到结果集中
4.这里我们是让target减去已经加入路径集的元素

class Solution {
public:
    vector<vector<int>>res;//结果集
    vector<int>path;//路径集

    //回溯递归，index代表当前要选取哪个索引对应的元素了
    void backTracking(vector<int>&candidates,int index,int target){
        if(target<0)return;//target小于0，代表当前累加和过大了，后面都不需要了，直接剪枝

        if(target==0){//target等于0，代表当前累加和满足要求，加入结果集中，也不需要累加后面的值了，因为都是正数
            res.push_back(path);
            return;

        }
        if(index>candidates.size()-1)return;//越界且累加和还不够，但也无法继续累加了

        //开始累加
        for(int i=0;i*candidates[index]<=target;i++){
            for(int count=0;count<i;count++){
                path.push_back(candidates[index]);
            }
            backTracking(candidates,index+1,target-candidates[index]*i);
            //回溯
            for(int count=0;count<i;count++){
                path.pop_back();
            }
        }
    }

    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        res.clear();
        path.clear();
        backTracking(candidates,0,target);
        return res;
    }
};

时空复杂度分析:
时间复杂度：O(nm)  n是candidates大小，m是target/candidates[index]的最大值;
空间复杂度：O(target)  除答案数组外，空间复杂度取决于递归的栈深度，最坏情况下需要递归target层;
```

## [40. 组合总和 II](https://leetcode.cn/problems/combination-sum-ii/)

```cpp
思路：
本题和上一题的区别在于candidates数组中有重复的元素且每个数字只能使用一次（即不能出现同一下标元素重复添加的情况），解题思路其实和三数之和类似，先排序数组，然后递归

卡点：
	不知道如何避免重复组合。
解决方法：
	换个角度理解，元素的重复次数是该元素的可取次数，相当于在前一题的基础上，每个元素可取无数次变成了有限次，这样就不会出现重复组合了，因为虽然元素值相同，但是意义不同，第一个元素相当于当前元素的代表值，其后面连续相等的元素只是可选的备用值，用于凑数，而不会继续选择相同值元素当作代表值再次递归。举个例子，比如[1,1,2,2,3]，组合成5，每组重复元素的首元素是代表值，绝对不会出现[1(1),2(2),2(3)](小括号是下标)的情况，因为1(1)不可能成为代表值，即意味着不可能出现有1(1)而没有1(0)的情况，所以就避免了[1(0),2(2),2(3)]和[1(1),2(2),2(3)]的重复情况；同理，[2(2),3(4)]和[2(3),3(4)]也不会重复出现

class Solution {
public:
    vector<vector<int>>res;//结果集
    vector<int>path;//路径集合

    void backTracking(vector<int>&candidates,int index,int target){
        if(target==0){
            res.push_back(path);
            return;
        }

        //由于candidates的元素都是正整数，并且升序，所以如果出现以下情况，再怎么处理后面的元素也无法组成target，提前结束（剪枝）
        if(index==candidates.size()||target<0||candidates[index]>target){//一定要优先判断数组越界情况
            return;
        }

        //记录当前元素的备用个数
        int cnt=1;
        while(index<candidates.size()-1&&candidates[index]==candidates[index+1]){
            cnt++;
            index++;
        }
        
        for(int i=0;i<=cnt&&candidates[index]*i<=target;i++){
            backTracking(candidates,index+1,target-candidates[index]*i);
            path.push_back(candidates[index]);//当前的push是用于下一循环的，因为取0次时，不能添加当前元素
        }

        for(int i=0;i<=cnt&&candidates[index]*i<=target;i++){//回溯，添加了多少次，就删掉多少次
            path.pop_back();
        }
    }

    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        res.clear();
        path.clear();
        sort(candidates.begin(),candidates.end());//先排序数组
        backTracking(candidates,0,target);
        return res;
    }
};

时空复杂度分析:
时间复杂度：O(nm)  n是candidates大小，m是cnt的最大值;
空间复杂度：O(target)  除答案数组外，空间复杂度取决于递归的栈深度，最坏情况下需要递归target层;
```

## [41. 缺失的第一个正数](https://leetcode.cn/problems/first-missing-positive/)

![image-20230531131547845](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230531131547845.png)

```cpp
思路：原地交换数组元素
让数组呈现如下规律，nums[i]=i+1，处理完之后，遍历数组，第一个不符合要求的元素，其对应位置应该出现的元素就是所需要的答案
1.交换时，当前处理的元素应该∈[1,n]，否则会越界
2.注意当前元素不能和相同大小的元素交换，否则会陷入死循环
3.由于我们只需要找到正整数，而下标恰好从0开始，所以刚好像一张映射表，因为如果使用哈希表的话，空间复杂度就是O(n)不符合要求
4.缺失的第一个正整数必定属于[1,n]区间或者就是n+1，所以只需要处理处于[1,n]范围的元素，过小或者过大的数不影响

关键：
1.交换停止时的规则
2.把数组当作一个哈希表

class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        int n=nums.size();
        for(int i=0;i<n;i++){
            while(nums[i]>=1&&nums[i]<=n&&nums[i]!=nums[nums[i]-1]){
                swap(nums[i],nums[nums[i]-1]);
            }
        }

        for(int i=0;i<n;i++){
            if(nums[i]!=i+1)return i+1;
        }

        return n+1;
    }
};

时空复杂度分析：
时间复杂度：O(n)；
空间复杂度：O(1);
```

## [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/)

```cpp
思路1：单调栈
单调栈用于处理“离a最近又比a高（或低）这类问题

1.创建一个单调递减栈，即栈底到栈顶降序，这里往栈里存放元素下标，这样既可以获取高度也可以求出宽度
2.若当前柱子高度大于栈顶柱子高度，因为是递减栈，所以栈顶柱子处会有一个凹槽，其上放可以形成积水，计算面积
3.若当前柱子高度小于栈顶柱子高度，不确定后面是否有比当前柱子更高的柱子能够形成凹槽，直接入栈
这里利用单调栈的原因是可以快速判断是否可以形成凹槽以及凹槽的左右边界

class Solution {
public:
    int trap(vector<int>& height) {
        stack<int>stk;//单调栈
        int area=0;//雨水面积

        stk.push(0);//先让第一根柱子入栈

        for(int i=1;i<height.size();i++){
            while(!stk.empty()&&height[i]>height[stk.top()]){
                int mid=stk.top();stk.pop();//取出中间柱子
                //再次弹栈，所以要判断栈是否为空，如果是空的，意味着左边没有更高的柱子能够让中间柱子上方形成凹槽
                if(!stk.empty()){
                    //计算高度差
                    int h=min(height[i],height[stk.top()])-height[mid];
                    //计算宽度
                    int w=i-stk.top()-1;
                    //计算面积
                    area+=h*w;
                }
            }
            stk.push(i);//入栈
        }

        return area;
    }
};

时空复杂度分析：
时间复杂度：O(n)  每个元素只会出栈入栈各一次；
空间复杂度：O(n)  单调栈的大小不超过数组长度;


思路2：动态规划

class Solution {
public:
    int trap(vector<int>& height) {
        vector<int>left_Max(height.size());//左边柱子最大高度
        vector<int>right_Max(height.size());//右边柱子最大高度

        for(int i=1;i<height.size();i++){
            left_Max[i]=max(left_Max[i-1],height[i-1]);
        }
        for(int j=height.size()-2;j>=0;j--){
            right_Max[j]=max(right_Max[j+1],height[j+1]);
        }

        int area=0;
        for(int i=1;i<height.size()-1;i++){
            int h=min(left_Max[i],right_Max[i])-height[i];
            if(h>0)area+=h;
        }

        return area;
    }
};

时空复杂度分析：
时间复杂度：O(n)；
空间复杂度：O(n);


思路3：双指针
优化动态规划的空间复杂度，由于left_Max和right_Max数组每个元素只用到了一次，只需要用两个双指针和两个变量即可。
维护两个指针left和right，两个变量left_Max和right_Max，left只能向右移，right只能向左移，二者在移动过程中顺便更新left_Max和right_Max

隐形大小关系：
lRight_Max>=rRight_Max
lLeft_Max<=rLeft_Max

class Solution {
public:
    int trap(vector<int>& height) {
        int left=0;
        int right=height.size()-1;
        int lLeft_Max=0;
        int rRight_Max=0;
        int area=0;

        while(left<right){
            lLeft_Max=max(lLeft_Max,height[left]);
            rRight_Max=max(rRight_Max,height[right]);
            if(lLeft_Max<rRight_Max){
                area+=lLeft_Max-height[left++];
            }else{
                area+=rRight_Max-height[right--];
            }
        }

        return area;
    }
};
```

![image-20230601162240316](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230601162240316.png)

[407. 接雨水 II](https://leetcode.cn/problems/trapping-rain-water-ii/)

```cpp
思路：从外向内，类似BFS
最外层影响次外层，次外层影响次次外层，影响的方式是木桶效应

1.将最外层的柱子放入优先队列，根节点是外层中最矮的柱子，所以优先队列中的元素要包含高度这个属性
2.不断的取出最矮的柱子并看其上下左右是否可以灌水，然后标记其上下左右已经灌过水并加入优先队列
3.如果当前柱子的周围有一个比自己矮的且未被处理过的柱子，在这个柱子上就会有一个凹槽可以灌水，且由于木桶效应，当前柱子就是最矮的那根，所以灌水量也确定了

关键点：为什么遇到比自己矮的柱子就可以灌水，这个较矮的柱子周围难道没有更矮的吗？
解答：分两种情况
（1）较矮的柱子在外层，由于我们是按优先队列顺序处理的，所以这个柱子肯定比当前柱子更先处理过，所以不会有水
（2）较矮的柱子在内层，由于当前柱子已经是外层柱子中最矮的一个了（优先队列堆顶），如果较矮的柱子未被处理过，意味着其周围的柱子都比当前柱子高，因为如果比当前柱子矮的话，由于优先队列的顺序，这个较矮的柱子必然已经被处理过了。

注意，注水之后的柱子高度变了

class Solution {
public:
    typedef pair<int,int> point;

    int trapRainWater(vector<vector<int>>& heightMap) {
        //处理特殊情况
        if(heightMap.size()<=2||heightMap[0].size()<=2)return 0;

        int m=heightMap.size();
        int n=heightMap[0].size();
        priority_queue<point,vector<point>,greater<point>>pq;//优先队列
        vector<vector<bool>>visit(m,vector<bool>(n,false));//访问记录表
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(i==0||i==m-1||j==0||j==n-1){//最外层
                    pq.push({heightMap[i][j],i*n+j});
                    visit[i][j]=true;
                }
            }
        }

        int res=0;
        int dirs[]={-1,0,1,0,-1};//用于找上下左右
        while(!pq.empty()){
            point cur=pq.top();pq.pop();//取柱子
            for(int k=0;k<4;k++){
                int x=cur.second/n+dirs[k];
                int y=cur.second%n+dirs[k+1];
                if(x>=0&&x<m&&y>=0&&y<n&&!visit[x][y]){//坐标合法且未被访问过
                    if(heightMap[x][y]<cur.first){
                        res+=cur.first-heightMap[x][y];
                    }
                    visit[x][y]=true;
                    //如果cur.first更大，说明[x][y]柱子被注水了，需要增加高度
                    pq.push({max(cur.first,heightMap[x][y]),x*n+y});
                }
            }
        }

        return res;
    }
};
```

## [43. 字符串相乘](https://leetcode.cn/problems/multiply-strings/)

![image-20230603222706206](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230603222706206.png)

```cpp
思路：手算乘法
tip：int通过+'0'转成char(0的ASCII码后移int大小恰好是int的ASCII码，int得是0~10)，char通过-'0'转成int，因为字符做加减运算时是其ASCII码大小

1.创建数组vec存储乘数，长度最长为num1.length()+num2.length()，预留vec[0]给最高位的进位；eg.99*99=9801
2.遍历num2，假设遍历到num2[i]，然后依次从num1低位乘，遍历num1，假设遍历到num1[j]，乘数放入vec[i+j]
3.最后处理进位将vec转成结果字符串res

class Solution {
public:
    string multiply(string num1, string num2) {
        //处理边界条件
        if(num1[0]=='0'||num2[0]=='0')return "0";

        int m=num1.length();
        int n=num2.length();
        vector<int>vec(m+n);
        
        //处理乘数
        for(int i=n-1;i>=0;i--){
            int a=num2[i]-'0';
            for(int j=m-1;j>=0;j--){
                int b=num1[j]-'0';
                vec[i+j+1]+=a*b;//最高一个vec[0]留给最高的进位
            }
        }

        //处理进位，vec[0~vec.size()-1]是高位到低位
        for(int i=vec.size()-1,carry=0;i>=0;i--){
            vec[i]+=carry;
            carry=vec[i]/10;
            vec[i]%=10;
        }

        //将数组转成结果字符串
        string res(vec.size(),'0');
        for(int i=0;i<vec.size();i++){
            res[i]=vec[i]+'0';
        }

        return res[0]=='0'?res.substr(1,res.length()-1):res;//如果最高的进位是0，只需返回后面的内容即可
    }
};

时空复杂度分析:
时间复杂度：O(nm)  m、n分别是num1和num2的长度;
空间复杂度：O(n+m)  vec的大小

本题的流程主要是从两个字符串的地位相乘，将得到的乘数添加到vec对应位置上，用数组是因为数组可以存放多位数，然后开始处理进位，无非就是把低位多于10的部分传给高一位，然后自身取个位数，关键点在于vec[0]预留最高位的进位，最后转成字符串，但0不能在最高位。
```

## [44. 通配符匹配](https://leetcode.cn/problems/wildcard-matching/)

```cpp
思路：动态规划
1.s仅由小写字母组成
2.p仅由小写字母、'?'、'*'组成
  2.1?：匹配任意单个字符
  2.2*：匹配任意字符串
3.动态规划表dp：dp[i][j] (s[0~i]能否和p[0~j]匹配)
  3.1 p[j]!='*'：dp[i][j]=dp[i-1][j-1]&&(s[i]==p[j]||p[j]=='?');
  3.2 p[j]=='*'：dp[i][j]=dp[i][j-1]||dp[i-1][j-1]||dp[i-2][j-1]……||dp[0][j-1]，即*匹配0、1、2……i个字符
				 dp[i-1][j]=dp[i-1][j-1]||dp[i-2][j-1]||……||dp[0][j-1]，和dp[i][j]后面部分相同，简化dp[i][j]
      			 dp[i][j]=dp[i][j-1]||dp[i-1][j];

一开始的弄错的地方是以为碰到了'*'需要往后去匹配，因为我还是没明白dp[i][j]的含义，实际上dp[i][j]的i和j索引，就已经限定了结束索引，即s必须以s[i]结束，p必须以p[j]结束，所以不可能再向后扩展了。而且我需要利用前面得到的结果，必然也是往前找，不可能往后找，因为后面是待处理的

class Solution {
public:
    bool isMatch(string s, string p) {
        int m=s.length();
        int n=p.length();
        //两个字符串前都加入一个相同的字符，不影响结果的同时还避免了繁琐的初始化
        s=' '+s;
        p=' '+p;

        vector<vector<bool>>dp(m+1,vector<bool>(n+1,false));
        dp[0][0]=true;

        for(int i=0;i<=m;i++){
            //dp[i][0](i>=1的情况下，都是false，因为一个空格无法匹配两个字符)
            for(int j=1;j<=n;j++){
                if(p[j]!='*'){//额外判断边界，因为递推公式只在边界内成立
                    dp[i][j]=i>=1&&j>=1&&dp[i-1][j-1]&&(s[i]==p[j]||p[j]=='?');
                }else{//额外判断边界，因为递推公式只在边界内成立
                    dp[i][j]=(j>=1&&dp[i][j-1])||(i>=1&&dp[i-1][j]);//二级结论
                }
            }
        }

        return dp[m][n];
    }
};

时空复杂度分析:
时间复杂度：O(nm);
空间复杂度：O(nm);
```

## [45. 跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/)

```cpp
思路：贪心算法
1.定义两个变量start和end，标记着可活动范围
2.在这可活动范围中，找到下一跳的最大可达位置
3.如果活动范围已经覆盖nums.size()-1的话，说明以当前步数就可以到达最后一格了，又因为每次我们都是位移最大范围，所以必然是最小步数

class Solution {
public:
    int jump(vector<int>& nums) {
        //起始位置在0，还没跳出去，所以还不知道下一跳的最大可达位置在哪里
        int minStep=0;
        //当前活动范围是上次活动范围跳出来得到的
        int start=0;//当前活动范围的起始位置
        int end=0;//当前活动范围的结束位置

        //如果当前活动范围已经超过最后一格的话，说明从上一个活动范围中的下一跳就可以抵达终点了
        while(end<nums.size()-1){
            int maxPos=0;//下一跳最大可达位置，即下一个活动范围的end
            //从当前活动范围找maxPos
            for(int i=start;i<=end;i++){//贪心
                maxPos=max(maxPos,i+nums[i]);
            }
            //更新活动范围
            start=end+1;
            end=maxPos;
            minStep++;
        }

        return minStep;
    }
};

时空复杂度分析:
时间复杂度：O(n)  start和end都是一直向后走的，没有重复过;
空间复杂度：O(1)  只需要常数个变量记录特殊数据;
```

## [46. 全排列](https://leetcode.cn/problems/permutations/)

![image-20230606220707452](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230606220707452.png)

```cpp
思路：DFS
1.枚举每个位置每个元素都出现一次的所有情况
2.记录元素使用情况，即不能在同一个组合的不同索引中出现相同的两个元素

class Solution {
public:
    vector<vector<int>>res;
    vector<bool>isUsed;
    vector<int>path;

    void dfs(vector<int>&nums,int index){
        if(index>=nums.size()){
            res.push_back(path);
            return;
        }

        for(int i=0;i<nums.size();i++){//枚举所有元素在当前位置出现的情况
            if(!isUsed[i]){//如果未被用过
                path.push_back(nums[i]);
                isUsed[i]=true;
                dfs(nums,index+1);
                path.pop_back();//回溯
                isUsed[i]=false;
            }
        }
    }

    vector<vector<int>> permute(vector<int>& nums) {
        res.clear();
        isUsed.clear();
        path.clear();

        isUsed=vector<bool>(nums.size(),false);

        dfs(nums,0);

        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n*n!)  叶子结点个数为n!（第一个位置有n种选择，第二个位置有n-1种选择……），我们需要将O(n)的时间将答案复制到结果数组中;
空间复杂度：O(n)  递归深度为n;
```

## [47. 全排列 II](https://leetcode.cn/problems/permutations-ii/)

![image-20230606224947613](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230606224947613.png)

```cpp
思路：
与上题基本一致，但是本题给的nums序列中允许重复数字的出现。
上图中，就是因为1的相对顺序搞反了，导致重复的排列出现。因为会在前面排列不变的基础上，在当前相同位置上重复选择了值相同但下标不同的元素，结果排列只看值不看下标，所以虽然排列并不能说是完全一样，但由于值相同，所以结果排列相同，因此重复了
1.只要保证所有相同元素的相对顺序不变，就不会出现重复序列了

class Solution {
public:
    vector<vector<int>>res;
    vector<int>path;
    vector<bool>isUsed;

    //index当前选择数字的位置
    void dfs(vector<int>&nums,int index){
        if(index==nums.size()){
            res.push_back(path);
            return;
        }

        //枚举所有数字
        for(int i=0;i<nums.size();i++){
            if(!isUsed[i]){//如果未出现
                //以相同元素的相对顺序查重，即在前面排列不变的情况下，假设当前位置已经选择过i号位的1，那么就不能再继续选择i+1号位的1了
                if(i!=0&&nums[i]==nums[i-1]&&!isUsed[i-1])continue;
                path.push_back(nums[i]);
                isUsed[i]=true;
                dfs(nums,index+1);
                path.pop_back();
                isUsed[i]=false;
            }
        }
    }

    vector<vector<int>> permuteUnique(vector<int>& nums) {
        res.clear();
        path.clear();
        isUsed.clear();
        
        sort(nums.begin(),nums.end());//排序数组，让相同元素扎堆
        isUsed=vector<bool>(nums.size(),false);

        dfs(nums,0);

        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n*n!)  叶子结点个数为n!（第一个位置有n种选择，第二个位置有n-1种选择……），我们需要将O(n)的时间将答案复制到结果数组中;
空间复杂度：O(n)  递归深度为n;
```

## [48. 旋转图像](https://leetcode.cn/problems/rotate-image/)

![image-20230608113701435](https://md-jomo.oss-cn-guangzhou.aliyuncs.com/IMG/image-20230608113701435.png)

```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n=matrix.size();
        //左上角
        pair<int,int>leftUp{0,0};
        //右下角
        pair<int,int>rightDown{n-1,n-1};
        
        //由于是正方形，所以左上角和右下角在处理完时只有两种情况，一是在同一个位置，二是在相邻对角，且左上角在下方
        while(leftUp.first<rightDown.first&&leftUp.second<rightDown.second){
            int left=leftUp.first;
            int up=leftUp.second;
            int right=rightDown.first;
            int down=rightDown.second;
			
            //处理时，把上下左右各当作一个独立块，所以会发现循环中矩阵变化的值中总有一个轴是不变的
            for(int i=0;i<right-left;i++){
                int tmp=matrix[up][left+i];
                matrix[up][left+i]=matrix[down-i][left];//up轴不变，对应上
                matrix[down-i][left]=matrix[down][right-i];//left轴不变，对应左
                matrix[down][right-i]=matrix[up+i][right];//down轴不变，对应下
                matrix[up+i][right]=tmp;//right轴不变，对应右
            }
			
            //内缩
            leftUp.first+=1;
            leftUp.second+=1;
            rightDown.first-=1;
            rightDown.second-=1;
        }
    }
};

时空复杂度分析:
时间复杂度：O(n^2) n是矩阵的边长，因为每个格子都遍历了一遍;
空间复杂度：O(1) 只用了常数个变量存储必要的信息，对角;
```

## [49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)

```cpp
思路：
本题的要求是将字母异位词分到同一个组中，（字母异位词 是由重新排列源单词的所有字母得到的一个新单词。），通俗一点讲就是只要两个不相同的单词的字母出现频率相同，那它们就是字母异位词

关键点：字母异位词排序的结果都是一样的，比如[abc,bac,cba]各字符串排完序的结果都是abc，所以可以创建一张哈希表，key是排完序对应的字符串，value是排完序的字符串等于key的字符串

class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string,vector<string>>record;
        
        for(int i=0;i<strs.size();i++){
            string key=strs[i];
            sort(key.begin(),key.end());//排序字符串
            record[key].push_back(strs[i]);//如果哈希表中不存在key，则会先创建对应关系后，再添加字符串
        }

        vector<vector<string>>res;
        for(auto it:record){//遍历哈希表，获取到每一项的字符串集合
            vector<string>vec;
            for(string& str:it.second){//遍历当前字符串集合，它们属于同一组
                vec.push_back(str);
            }
            res.push_back(vec);
        }

        return res;
    }
};

时空复杂度分析:
时间复杂度：O(nklogk)  n是strs的大小，k是最长字符串的长度，klogk是排序时间;
空间复杂度：O(nk)  哈希表大小;
```

## [50. Pow(x, n)](https://leetcode.cn/problems/powx-n/)

```cpp
思路：快速幂
把n看作2进制数，假设计算2^5，5的2进制是101，2^5=2^(101)，2^(2^2+2^0)=2(2^2)*2(2^0)=16*2=32

class Solution {
public:
    double myPow(double x, int n) {
        long N=n;//如果n小于0，需要先转成正数，但可能会越界，所以用long接收
        if(N<0)N=-N;

        double res=1.0;
        while(N>0){
            if(N&1==1)res*=x;
            x*=x;//x^(2^0) -> x^(2^1) -> x^(2^2) -> x^(2^3)，恰好对应着每一位的2进制数大小
            N>>=1;
        }

        return n<0?1.0/res:res;//如果n小于0，需要返回倒数
    }
};
```

## [51. N 皇后](https://leetcode.cn/problems/n-queens/)

```cpp
思路：回溯法
class Solution {
public:
    vector<string>chessBoard;//棋盘
    vector<vector<string>>res;//有效棋盘集合

    bool isValid(int row,int col){
        //第0行无论放哪一列都是合法的，因为后面行都还没放皇后，必然不会冲突
        if(row==0)return true;

        //只需要判断是否和上方几行冲突，因为下面几行都还没有放置皇后

        //同一列不能有皇后
        for(int i=row-1;i>=0;i--){
            if(chessBoard[i][col]=='Q')return false;
        }

        //左上斜线
        int k=1;
        while(row-k>=0&&col-k>=0){
            if(chessBoard[row-k][col-k]=='Q')return false;
            k++;
        }
        k=1;
        //右上斜线
        while(row-k>=0&&col+k<chessBoard.size()){
            if(chessBoard[row-k][col+k]=='Q')return false;
            k++;
        }

        return true;
    }

    void backTracking(int row){
        //如果能超过棋盘最后一行，说明最后一行的皇后也放置了且合法，就代表当前棋盘摆放皇后有效，加入结果集
        if(row==chessBoard.size()){
            res.push_back(chessBoard);
            return;
        }

        //遍历当前行所有列，找到一个适合放置皇后的位置
        for(int j=0;j<chessBoard.size();j++){
            if(isValid(row,j)){
                chessBoard[row][j]='Q';//放置皇后
                backTracking(row+1);
                chessBoard[row][j]='.';//回溯
            }
        }
    }

    vector<vector<string>> solveNQueens(int n) {
        res.clear();
        chessBoard.clear();
        chessBoard=vector<string>(n,string(n,'.'));

        backTracking(0);

        return res;
    }
};

时空复杂度分析:
时间复杂度：O(n!)  n是皇后的数量，第一行有n个格子放置皇后，第二行有n-1个格子放置皇后……，总情况数量是n*(n-1)*(n-2)……=n!
空间复杂度：O(n^2)   递归深度不超过n，chessBoard的大小为n^2
```

## [52. N 皇后 II](https://leetcode.cn/problems/n-queens-ii/)

```cpp
思路：回溯法
本题的做法和上题完全相同，只不过是加入有效棋盘变成了记录有效棋盘数

class Solution {
public:
    vector<string>chessBoard;//棋盘
    int count;

    bool isValid(int row,int col){
        //第0行无论放哪一列都是合法的，因为后面行都还没放皇后，必然不会冲突
        if(row==0)return true;

        //只需要判断是否和上方几行冲突，因为下面几行都还没有放置皇后

        //同一列不能有皇后
        for(int i=row-1;i>=0;i--){
            if(chessBoard[i][col]=='Q')return false;
        }

        //左上斜线
        int k=1;
        while(row-k>=0&&col-k>=0){
            if(chessBoard[row-k][col-k]=='Q')return false;
            k++;
        }
        k=1;
        //右上斜线
        while(row-k>=0&&col+k<chessBoard.size()){
            if(chessBoard[row-k][col+k]=='Q')return false;
            k++;
        }

        return true;
    }

    void backTracking(int row){
        //如果能超过棋盘最后一行，说明最后一行的皇后也放置了且合法，就代表当前棋盘摆放皇后有效，加入结果集
        if(row==chessBoard.size()){
            count++;
            return;
        }

        //遍历当前行所有列，找到一个适合放置皇后的位置
        for(int j=0;j<chessBoard.size();j++){
            if(isValid(row,j)){
                chessBoard[row][j]='Q';//放置皇后
                backTracking(row+1);
                chessBoard[row][j]='.';//回溯
            }
        }
    }

    int totalNQueens(int n) {
        chessBoard.clear();
        chessBoard=vector<string>(n,string(n,'.'));
        count=0;
        backTracking(0);

        return count;
    }
};

时空复杂度分析:
时间复杂度：O(n!)  n是皇后的数量，第一行有n个格子放置皇后，第二行有n-1个格子放置皇后……，总情况数量是n*(n-1)*(n-2)……=n!
空间复杂度：O(n^2)   递归深度不超过n，chessBoard的大小为n^2
```

## [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

```cpp
思路：动态规划
dp[i]：以nums[i]结尾的子数组的最大和，如果dp[i-1]即以nums[i-1]结尾的最大子数组和大于0，则可以连接dp[i-1]涉及的子数组，加上正数必然比自身更大，如果其小于0，那么就不能连接了，因为和会更小

class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        if(nums.size()==1)return nums[0];//边界情况

        vector<int>dp(nums.size(),0);
        dp[0]=nums[0];
        int maxSum=dp[0];
        for(int i=1;i<nums.size();i++){
            //如果前面子数组最大和大于0，则连接，否则自成一个子数组
            dp[i]=dp[i-1]>0?dp[i-1]+nums[i]:nums[i];
            maxSum=max(maxSum,dp[i]);
        }

        return maxSum;
    }
};

时空复杂度分析:
时间复杂度：O(n)
空间复杂度：O(n)

优化空间复杂度至O(1)：
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int maxSum=INT_MIN;
        int cur=0;

        for(int i=0;i<nums.size();i++){
            cur>0?cur+=nums[i]:cur=nums[i];
            maxSum=max(cur,maxSum);
        }

        return maxSum;
    }
};
```

## [54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        int m=matrix.size();
        int n=matrix[0].size();

        pair<int,int>leftUp{0,0};//左上角
        pair<int,int>rightDown{m-1,n-1};//右下角
        
        vector<int>res(m*n,0);
        int cur=0;
        while(leftUp.first<=rightDown.first&&leftUp.second<=rightDown.second){
            int up=leftUp.first;//上边界
            int left=leftUp.second;//左边界
            int down=rightDown.first;//下边界
            int right=rightDown.second;//右边界

            //避免只剩最后一个中心格导致重复打印的情况，用else if，避免重复处理，且其他情况下同一行或同一列不可能同时发生
            if(up==down){//只剩一行
                for(int i=left;i<=right;i++){
                    res[cur++]=matrix[up][i];
                }
                break;//及时退出循环，避免下面又重复打印
            }
            else if(left==right){//只剩一列
                for(int i=up;i<=down;i++){
                    res[cur++]=matrix[i][left];
                }
                break;//及时退出循环，避免下面又重复打印
            }

            //注意不要涉及其他边界的打印，例如上边界不涉及右边界，右边界不涉及下边界，下边界不涉及左边界，左边界不涉及上边界，不然可能会重复打印
            //上边界从左到右打印
            for(int i=left;i<right;i++){
                res[cur++]=matrix[up][i];
            }
            //右边界从上到下打印
            for(int i=up;i<down;i++){
                res[cur++]=matrix[i][right];
            }
            //下边界从右到左打印
            for(int i=right;i>left;i--){
                res[cur++]=matrix[down][i];
            }
            //左边界从下到上打印
            for(int i=down;i>up;i--){
                res[cur++]=matrix[i][left];
            }

            leftUp.first++;
            leftUp.second++;
            rightDown.first--;
            rightDown.second--;
        }

        return res;
    }
};

时空复杂度分析:
时间复杂度：O(m*n)  矩阵的每个元素都被扫描了一遍;
空间复杂度：O(1)  只用了常数个变量记录两个角的一些信息;
```

