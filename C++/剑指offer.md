# 剑指offer

## 数组

### 数组中重复的数字

- 题目描述：长度为n的数组在0～n-1范围内，有数字重复，找出任意一个

- 题解：遍历数组，如果nums[i] != i，则nums[i]放到nums[i]的位置上,即nums[i]与nums[nums[i]]交换位置

### 不修改数组找出重复的数字

- 题目描述：长度n+1的数组在1～n的范围内，找出一个重复数字
- 题解：二分的思路，如果左边多，end=middle；如果右边多，end=start；

### 二维数组中的查找

- 题目描述：行列有序，查找是否包含某个整数
- 题解：从右上角开始判断，如果右上角元素大，则j-1；如果右上角元素小，则i+1

## 字符串

### 替换空格

- 题目描述：将空格修改为“%23”
- 题解：计算空格，算出修改后字符串的大小，从后往前进行修改

## 链表

### 从尾到头打印指针

- 题目描述：从尾到头打印指针
- 题解：递归打印，先递归后打印 （或栈）

## 树

### 重建二叉树

- 题目描述：给定前序和中序，重建二叉树
- 题解：遍历前序，遍历节点为根节点，在中序中找到根节点，则根节点左边的节点均为该根节点的左子树，知道左子树的节点数量后，再到前序中找到左子树的范围，从而对左子树进行递归；右子树同理

### 二叉树的下一个节点

- 题目描述：树节点有左子节点和父节点，求中序的下一个节点
- 题解：如果当前节点有右子树，则寻找右子节点的左子树；如果当前节点没有右子树，则寻找父节点，如果父节点的左子树是当前节点，则返回父节点，否则寻找父节点的父节点

## 栈和队列

### 用两个栈实现队列

- 题目描述：两个栈实现队列
- 题解：一个栈添加元素，另一个栈删除元素，如果删除栈没元素，则将添加栈的元素push进去



## 递归和循环

### 斐波那契数列

- 题目描述：求斐波那契数列的第n项
- 题解：两个变量pre,cur，循环求解

### 跳台阶

- 题目描述：可以跳1个，也可以跳2个，上一个n阶楼梯，有多少种跳法
- 题解：动态规划，f(n) = f(n-1)+f(n-2)，即斐波那契数列

## 查找和排序

### 旋转数组的最小数字

- 题目描述：数组经过旋转，求最小数字
- 题解：二分判断，nums[mid] > nums[right] left=mid+1 特殊情况：left, mid,right节点相等时没法知道最小值在哪一半，需要遍历，另外，还有一种情况需要考虑：旋转后与旋转前一致

### Trick1 二分

​	正常二分找值：1)条件low<=high    2)low+((high-low)>>1).  3)low= mid +1.  high= mid-1

​	变形：

	- 当需要条件是low<high时,low的更新必须要mid+1,否则容易死循环；right可以=mid（mid的值不变化时=low，因此right=mid也会对right进行更新，但low如果=mid的话，可能会造成无法退出循环）
	- 如果有时候low 和 high不能 等于 mid + 1 或 mid - 1；即low和high只能等于mid时，可以在循环中刚开始判断一下如果low和high相差1，那么直接返回，以防出现死循环

## 回溯

### 矩阵中的路径

- 题目描述：字符矩阵和字符串，判断字符串是否在字符矩阵中
- 题解：新建递归函数，参数有visited,matrix,row,col,str,sPtr，返回上下左右四种情况的或

### Trick2 迷宫类回溯

递归函数的设计：参数必须有：迷宫矩阵，当前二维坐标x，y，visited矩阵，具体问题参数

大致思路是 

```C++
bool method(vector<vector<int>>& matrix,int x,int y,vector<vector<int>>& visited,...)
	退出条件判断
  flag = false;
	if（判断条件正确：包括当前坐标的限制，visited的限制，具体问题的限制三类） 
    1）设置visited值以及具体问题造成的其他值
  	2）flag = 上 || 下 || 左 || 右
  	3）if(!flag) 
  			进行回溯（还原visited值以及具体问题造成的其他值）
	return flag
```

递归回溯思路：

```c++
void recur(int level, int param){   //经验总结，参数经常有level，max_level，visited，res(判断推出时将结果加入res,不需要存储结果时可以不要该参数，直接退出即可)，cur(当前状态)
  if(level > MAX_LEVEL){   //退出条件
    return 
  }
  process_curent(level,param); //处理当前层
  
  recur(level+1,newParam); //递归下一层
  
  Backtrack(level,param); //恢复当前状态 
}
```

### 机器人的运动范围

- 题目描述：从(0,0)移动，可上下左右，但当前x,y坐标的位之和不能大于k，问可以到达多少格子
- 题解：新建递归函数，上下左右或

## 动态规划与贪婪算法

### 剪绳子

- 题目描述：长度为n的绳子剪成m段，问乘积最大是多少
- 题解：
  - 贪心：尽量剪成长度为3的，注意当剩下长度为4时，应剪成2 2
  - 动归：dp[i] = max{dp[x]*dp[i-x]}

## 位运算

### 二进制中1的个数

- 题目描述：一个数二进制位有多少个1
- 题解：
  - while(n)  左移1    32次   每次与n与
  - while (n).  n = n & ( n-1 )

## 代码完整性

### 数值的整数次方

- 题目描述：实现power(double base,int exponent)
- 题解: 考虑完整和高效：exponent的正负，幂的递归(要用到递归结果可以用变量将结果存储下来)  

### Trick3 递归

​	要重复用到递归结果的时候，可以将递归结果以返回值返回，或者使用参数(引用或指针)，然后重复利用，避免重复计算相同的递归式

```c++
// exponent需要是非负数
double PowerWithUnsignedExponent(double base,unsigned int exponent){
  if(exponent ==0) return 1;
  if(exponent ==1) return base;
  double result = PowerWithUnsignedExponent(base,exponent>>1);
  result *= result;  //重复使用
  if(exponent & 1){
    result*=base;
  }
  return result;
}
```

### 打印从1到最大的n位数

- 题目描述：打印从1到n位数的最大数
- 题解：n位字符数组+递归(排列)    打印时注意别打印开头的0

### 删除链表的节点

- 题目描述：O(1)时间删除节点
- 题解：将next节点复制到待删除节点，然后删除next节点即可(考虑尾节点)

### 删除链表中重复的节点

- 题目描述：排序链表，删除重复节点
- 题解：可以新建哑节点，使问题简单化

### 正则表达式匹配

- 题目描述：字符串s和模式p是否匹配
- 题解：
  - 递归：根据后一个字符是否是*来进行分支，如果是 *，判断当前字符是否与字符串当前字符匹配，或者当前字符是.，如果匹配成功，则可以不匹配字符串当前字符，也可以只匹配当前字符，或者匹配当前字符及后面字符三种情况，如果没匹配成功，则可以不匹配当前字符；如果不是 *，则判断当前字符和字符串当前字符即可
  - 动态规划：dp [ i ] [ j ]  表示字符串前i个字符和模式前j个字符是否匹配

### 表示数值的字符串

- 题目描述：判断字符串是否属于数值
- 题解：A[.[B]] [e|EC] 或者.B[e|EC].  分解函数=>检查是否是非负整数，检查是否是整数

### 调整数组顺序使奇数位于偶数前面

- 题目描述：让奇数位于数组前半部分
- 题解：使用快排的思路，双指针，O(n)

### Trick4 快排

```c++
void qSortCore(vector<int>& nums,int start,int end){
    if(start >= end) return;
    int pivot = nums[end];
    int left = start;
    int right = end-1;
    while(left <= right){
        while(left<=right && nums[left]<=pivot) left ++;
        while(left<=right && nums[right]>pivot) right--;
        if(left<right){
            int temp = nums[left];
            nums[left] = nums[right];
            nums[right] = temp;
        }
    }
    nums[end] = nums[left];
    nums[left] = pivot;
    qSortCore(nums,start,left-1);
    qSortCore(nums,left+1,end);
}
void qSort(vector<int>& nums){
    qSortCore(nums,0,nums.size()-1);
}
```





## 代码的鲁棒性

### 链表中倒数第k个节点

- 题目描述：倒数第k个节点
- 题解：双指针，前面的指针先走k步

### 链表中环的入口节点

- 题目描述：求链表中环的入口节点
- 题解：检测是否有环：前后指针，一个指针走一步，一个指针走两步，相遇则有环；相遇节点在环上，循环一圈就是环的节点个数n；两个指针，一个先走n步，另一个再走，相遇点就是环的入口节点

### 反转链表

- 题目描述：反转链表
- 题解：两个指针，pre和cur

### 合并两个排序的链表

- 题目描述：合并两个排序的链表
- 题解：while(list1 && list2).  while(list1).  while(list2)

### 树的子结构

- 题目描述：判断是否是树的子结构
- 题解：递归判断两个树是否相等

## 思路

### 二叉树的镜像

- 题目描述：求一棵树的镜像
- 题解：递归 => left = mirrorRecursively(right) right = mirrorRecursively(left)

### 对称二叉树

- 题目描述：判断两个树是否对称
- 题解：递归判断 left == right   right == left

### 顺时针打印矩阵

- 题目描述：顺时针打印矩阵
- 题解：限制并更新上下左右

### 包含min函数的栈

- 题目描述：O(1)时间计算栈的min
- 题解：两个栈，一个栈存正常值，另一个栈存当前最小值，每次添加和移除元素时，两个栈同时更新

### 栈的压入，弹出序列

- 题目描述：栈中数字均不相等，判断弹出序列和压入序列是否匹配
- 题解：新建一个栈，模拟入栈，设置弹出栈序列的指针，每次判断弹出栈指针相应元素与栈顶是否相等。如果元素入栈结束，弹出栈指针还未指向结尾，则返回false；否则返回true

### 面试题32 从上到下打印二叉树

- 分层打印：层序遍历
- 之字形打印：采用双栈层次遍历，也可以采用deque

### 面试题33 二叉搜索树的后序遍历序列

- 题目描述：判断某序列是否是二叉树的后序遍历序列
- 题解：递归判断是否序列分为两截，前部分都小于尾节点，后部分都大于尾节点

### 面试题34 二叉树和为某一值的路径

- 题目描述：打印二叉树和为某一值的所有路径
- 题解：递归，记得要回溯

## 分解复杂问题

### 复杂链表的复制

- 题目描述：链表多了一个指针randomPtr，对链表进行复制

- 题解：

  1）在原先链表每个节点后面复制一个节点；

  2）根据原链表的randomPtr，给新复制的节点设置randomPtr；

  3）拆解原链表和复制链表

### 二叉搜索树与双向链表

- 题目描述：将二叉搜索树转变为双向有序链表
- 题解：中序遍历，两个指针，一个指针指向头节点，一个指针指向前一个节点，最后返回头节点即可

### 序列化二叉树

- 题目描述：前序序列化二叉树，并反序列化回到原树
- 题解：

### 字符串的排列

- 题目描述：字符串所有排列
- 题解：固定一个，递归后面一串，记得回溯

### Trick 全排列

```c++
void permuteCore( vector<vector<int>>& res,vector<int>& nums, int d){
        if(nums.size()-1 == d){
            res.push_back(vector<int>(nums));
            return;
        }
        for(int i = d ;i< nums.size() ;i++){ //当前位置的元素轮换放置
            int temp = nums[d];
            nums[d] = nums[i];
            nums[i] = temp;
            permuteCore(res,nums,d+1);
            temp = nums[d];
            nums[d] = nums[i];
            nums[i] = temp;

        }
    }

    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> res;
        if(nums.size()==0) return res;
        permuteCore(res,nums,0);
        return res;
    }
```



## 优化时间和空间

### 面试题39 数组中出现次数超过一半的数字

- 题目描述：找出这个数字
- 题解：
  - 中位数思路，第k小
  - 遍历一遍，两个变量，一个记录次数，一个记录当前字符，小兵对打

### 面试题40 最小的k个数

- 题目描述：找出最小的k个数
- 题解：
  - 快速选择 第k小
  - 使用堆排，默认priority_queue为大堆，即priority_queue<int,vector<int>,less<int>>

### 面试题41 数据流中的中位数

- 题目描述：O(1)得到中位数
- 题解：两个堆，一个最大堆，一个最小堆

### 连续子数组的最大和

- 题目描述：一个数组，求连续子数组的最大和
- 题解：求出尾元素是当前元素的最大和，遍历求最大

### 1～n整数中1出现的次数

- 题目描述：求出十进制数中1出现的次数
- 题解：每n位数里包含的1次数是固定的，且有递推公式f(n) = 10*f(n-1)+pow(10,n-1)，然后根据最高位进行分支：大于1时，等于1时

### 把数组排成最小的数

- 题目描述：将一个数组中的数连接起来组成最小的数
- 题解：将数组中的元素进行排序，默认排序会出现为题，即30 > 3,但30应该排在前面，因此重新定义比较函数，两个数分别前后进行比较

### 把数字翻译成字符串

- 题目描述：1对应a，2对应b...计算一个数字对应多少种翻译方法
- 题解：动态规划

### 礼物的最大价值

- 题目描述：从矩阵左上角走到右下角，计算加起来的最大值
- 题解：动态规划 dp[ i ] [ j ] = matrix[ i ] [ j ] + max{dp[ i ] [ j-1 ],dp[ i-1 ] [ j ]}

### 最长不含重复字符的子字符串

- 题目描述：最长不含重复字符的子字符串
- 题解：两个指针，要注意的是，当出现重复字符的时候，前面那个指针要比较原重复字符的位置和当前指针的位置，选择较大的，否则会出错

### 丑数

- 题目描述：只包含因子2，3，5的数
- 题解：从1开始，不断乘2，3，5，使用set去重，使用堆进行丑数有序输出

### 第一个只出现一次的字符

- 题目描述：第一个只出现一次的字符
- 题解：使用map存储，只出现一次的存坐标，出现多次的存-1

### 数组中的逆序对

- 题目描述：输出数组中的逆序对
- 题解：使用归并排序，边排序边求对数

### 两个链表的第一个公共节点

- 题目描述：两个链表的第一个公共节点
- 题解：
  - 各遍历一遍，记录长度之差，长链表先走
  - 使用栈

### 在排序数组中查找数字

- 题目描述：输出某个数字在排序数组中出现的次数
- 题解：二分变形，找到最左和最右

### Trick5 二分变形

```c++
//求第一个等于target的数
int searchCore(vector<int>& nums, int target){
        int l = 0;
        int r = nums.size()-1;
        int index = -1;
        while(l<=r){ //注意<=
            int mid = l+((r-l)>>1) ; //注意超出范围
            if(nums[mid] == target) {
                if(mid>0 && nums[mid-1]==target){
                    r = mid -1; //注意-1
                }else {
                    index = mid;
                    break;
                }
            }
            else if(nums[mid] > target) r = mid - 1; //注意-1
            else l = mid + 1; //注意+1
        }
        return index;
    }
```

### 0～n-1缺失的数

- 题目描述：有个数缺了，找出
- 题解：二分

### 数组中数值和下标相等的元素

- 题目描述：找出下标与数相等的数
- 题解：二分

### 二叉搜索树中第k大节点

- 题目描述：第k大节点
- 题解：中序（右 root 左）

### 二叉树的深度

- 题目描述：深度
- 题解：递归max（left，right）+1

### 判断平衡二叉树

- 题目描述：判断是否是平衡二叉树
- 题解：后序，以防递归重复

### 数组中数字出现的次数

- 题目描述：数组中只出现一次的两个数字
- 题解：所有数异或之后，即得到两个数字的异或值，通过结果中位为1的地方，将数组分为两组，两组分别异或，得到最终结果

### 数组中唯一只出现一次的数字

- 题目描述：除了一个数字外，每个数字都出现三次，找出这个数字
- 题解：位数相加，除以3即可得到该数

### 和为s的数

- 题目描述：输入一个递增排序数组和一个数字s，输出一对
- 题解：双指针

### 和为s的连续正数序列

- 题目描述：打印出所有和为s的连续正数序列
- 题解：从1开始遍历，当sum超过s时，start++；小于s时，end++

### 队列的最大值

- 题目描述：一个数组和滑动窗口的大小，输出所有滑动窗口的最大值
- 题解：使用deque，每次首元素都是滑动窗口目前最大的，deque存储坐标，当滑动窗口移出该最大元素，即移除即可

### 队列的最大值

- 题目描述：定义一个队列的最大值函数
- 题解：采用deque，每次添加元素时，如果当前元素大于deque的首元素，则将首元素换为当前元素；每次删除元素时，如果当前元素等于deque的首元素，则移除掉该首元素

### 面试题60 n个骰子的点数

- 题目描述：点数出现的概率
- 题解：动态规划 等于前六个元素的值

### 扑克牌中的顺子

- 题目描述：0满天飞，判断是否是顺子
- 题解：排序，扫描，判断有几个0，并判断有几个间隔，如果0>=间隔，则是顺子

### 圆圈中最后剩下的数字

- 题目描述：每次移除第m个元素，最后剩下谁

- 题解：

  - 使用循环链表
  - 将原先元素的第m个元素推出后，将原先的元素映射到0～n-1，通过映射后的值推出映射前的值，由于知道了最后移除的元素映射后必定是0，因此可以推出最一开始元素的下标，从而输出。递推公式为：f(n,m) = (f(n-1,m)+m)%n

  ![image-20200303190126621](%E5%89%91%E6%8C%87offer.assets/image-20200303190126621.png)

### 股票的最大利润

- 题目描述：买卖一次，输出最大利润
- 题解：遍历时，一直维护当天之前的最小值，以及当天卖出的最大值

### stoi

- 将字符串转变为数

### 二叉树公共节点

- 题目描述：二叉树的公共节点
- 题解：后序遍历。如果节点在左右子树，或在根与左/右子树，返回根

```c++
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root == nullptr || p == root || q == root) return root;
        TreeNode* left = lowestCommonAncestor(root->left,p,q);
        TreeNode* right = lowestCommonAncestor(root->right,p,q);
        if(left == nullptr) return right;
        if(right == nullptr) return left;
        return root;
    }
```

