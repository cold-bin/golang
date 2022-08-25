# `Leecode`刷题笔记

## day01

### [移除元素](https://leetcode.cn/problems/remove-element/)

- 题目

  给你一个数组 `nums` 和一个值`val`，你需要 原地 移除所有数值等于`val`的元素，并返回移除后数组的新长度。

  不要使用额外的数组空间，你必须仅使用 `O(1) `额外空间并 原地 修改输入数组。

  元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。


  - 暴力题解

    这个题目暴力的解法就是两层for循环，一个for循环遍历数组元素 ，第二个for循环更新数组。

    ```java
    class Solution {
        public int removeElement(int[] nums, int val) {
            int j=0;
            int i=0;
            int len=nums.length;
            for(i=0;i<len;i++){
                if(val==nums[i]){
                    //将之后的元素全部往前挪移覆盖一位
                    for(j=i+1;j<len;j++){
                        nums[j-1]=nums[j];
                    }
                    //将数组索引也往前挪
                    i--;
                    len--;
                }
            }
           return len;
        }
    }
    ```

  - 双指针法求解

    由于题目要求删除数组中等于`val`的元素，因此输出数组的长度一定小于等于输入数组的长度，我们可以把输出的数组直接写在输入数组上。可以使用双指针：右指针`right` 指向当前将要处理的元素，左指针 `left` 指向下一个将要赋值的位置。

    如果右指针指向的元素不等于 `val`，它一定是输出数组的一个元素，我们就将右指针指向的元素复制到左指针位置，然后将左右指针同时右移；

    如果右指针指向的元素等于 `val`，它不能在输出数组里，此时左指针不动，右指针右移一位。

    整个过程保持不变的性质是：区间 `[0,left)`中的元素都不等于 `val`。当左右指针遍历完输入数组以后，`left `的值就是输出数组的长度。

    ```java
    class Solution {
        public int removeElement(int[] nums, int val) {
            int n = nums.length;
            int left = 0;//指输出元素索引
            for (int right = 0; right < n; right++){
                if (nums[right] != val) {
                    //不等于删除值，则将这个元素添加至输出元素
                    nums[left] = nums[right];
                    left++;
                }
            }
            return left;
        }
    }
    ```


## day02

### [最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

- 题目

  给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。**子数组** 是数组中的一个连续部分。

- 暴力破解

- 找规律

  对于含有正数的序列而言,最大子序列肯定是正数,**所以头尾肯定都是正数**.我们可以从第一个正数开始算起,每往后加一个数便更新一次和的最大值;当当前和成为负数时,则表明此前序列无法为后面提供最大子序列和,因此必须重新确定序列首项

  ```java
  class Solution {
      public int maxSubArray(int[] nums) {
          int maxSum = Integer.MIN_VALUE;//最大和
          int thisSum = 0;//当前和
          int len = nums.length;
          for(int i = 0; i < len; i++) {
              thisSum += nums[i];
              if(maxSum < thisSum) {
                  maxSum = thisSum;
              }
              //如果当前和小于0则归零，因为对于后面的元素来说这些是减小的。于是归零，意即从此处算开始最大和
              if(thisSum < 0) {
                  thisSum = 0;
              }
          }
          return maxSum;
      }
  }
  ```

### [加一](https://leetcode.cn/problems/plus-one/)

- 题目

  给定一个由 整数 组成的 非空 数组所表示的非负整数，在该数的基础上加一。最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。你可以假设除了整数 0 之外，这个整数不会以零开头。

- 找规律

  将这个看作是两个数组相加，辅助数组始终要比给定数组多一位，用来承载可能两数之和比给定数组多一位。两数相加，先从末尾相加，依次往前，满十进一

  ```java
  class Solution {
      public int[] plusOne(int[] digits) {
          int len1=digits.length;
          int len2=len1+1;
          int i,j;
          int[] arr=new int[len2];
          arr[len2-1]=1;
          for(i=len1-1,j=len2-1;;i--,j--){
              int sum=digits[i]+arr[j];
              if(sum>9){
                  //进1
                  arr[j-1]++;
                  //取余
                  arr[j]=(sum)%10;
              }else{
                  arr[j]=sum;
              }
              //跳出条件放到此处，防止少算一次
              if (i==0){
                  break;
              }
          }
          //根据首位判断是否输出首位
          if (arr[0]==0){
              for (i=1,j=0;i<len2;){
                  digits[j++]=arr[i++];
              }
              return digits;
          }
          return arr;
      }
  }
  ```


## day03

#### [合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/)

- 题目

  给你两个按 非递减顺序 排列的整数数组 nums1 和 nums2，另有两个整数 m 和 n ，分别表示 nums1 和 nums2 中的元素数目。请你 合并 nums2 到 nums1 中，使合并后的数组同样按 非递减顺序 排列。

  注意：最终，合并后数组不应由函数返回，而是存储在数组 nums1 中。为了应对这种情况，nums1 的初始长度为 m + n，其中前 m 个元素表示应合并的元素，后 n 个元素为 0 ，应忽略。nums2 的长度为 n 。

- 方法一：先合并再排序（**缺点，没有利用两数组已经排序的条件**）

  ```java
  class Solution {
          public void merge(int[] nums1, int m, int[] nums2, int n) {
            //先合并末尾数据
          for (int i = m, j = 0; i < nums1.length && j < n; i++, j++) {
              nums1[i] = nums2[j];
          }
          //按递增排序(希尔排序)
          for (int gap = nums1.length / 2; gap > 0; gap /= 2) {//步长
              //对每组进行直接插入排序
              for (int i = gap; i < nums1.length; i++) {
                  int index = i;
                  int value = nums1[i];
                  while (index - gap >= 0 && nums1[index - gap] > value) {
                      nums1[index] = nums1[index - gap];
                      index -= gap;
                  }
                  nums1[index] = value;
              }
          }
      }
  }
  ```

- 方法二：双指针

  方法一没有利用数组nums1与 nums2已经被排序的性质。为了利用这一性质，我们可以使用双指针方法。这一方法将两个数组看作队列，每次从两个数组头部取出比较小的数字放到结果中。如下面的动画所示：

  ![gif1](https://raw.githubusercontent.com/liuhaibin123456789/img-for-cold-bin-blog/master/img/202205300009037.gif)

  我们为两个数组分别设置一个指针p1与p2来作为队列的头部指针。代码实现如下：

  ```java
  class Solution {
      public void merge(int[] nums1, int m, int[] nums2, int n) {
          int p1 = 0, p2 = 0;
          int[] sorted = new int[m + n];
          int cur;
          while (p1 < m || p2 < n) {
              if (p1 == m) {
                  cur = nums2[p2++];
              } else if (p2 == n) {
                  cur = nums1[p1++];
              } else if (nums1[p1] < nums2[p2]) {
                  cur = nums1[p1++];
              } else {
                  cur = nums2[p2++];
              }
              sorted[p1 + p2 - 1] = cur;
          }
          for (int i = 0; i != m + n; ++i) {
              nums1[i] = sorted[i];
          }
      }
  }
  ```


## day04

#### [59. 螺旋矩阵 II](https://leetcode.cn/problems/spiral-matrix-ii/)

- 题目

  给你一个正整数 `n` ，生成一个包含 `1` 到 `n2` 所有元素，且元素按顺时针顺序螺旋排列的 `n x n` 正方形矩阵 `matrix` 。

  **示例 1：**

  ![img](https://raw.githubusercontent.com/liuhaibin123456789/img-for-cold-bin-blog/master/img/59.%20%E8%9E%BA%E6%97%8B%E7%9F%A9%E9%98%B5%20II--leecode.jpeg)

  ```
  输入：n = 3
  输出：[[1,2,3],[8,9,4],[7,6,5]]
  ```

  **示例 2：**

  ```
  输入：n = 1
  输出：[[1]]
  ```

  **提示：**

  - `1 <= n <= 20`

- 方法一：暴力破解，环形遍历-->也即按层模拟

  ```java
  package other;
  
  import java.util.Arrays;
  
  class Solution5 {
      public static void main(String[] args) {
          System.out.println(Arrays.deepToString(generateMatrix(4)));
      }
  
      public static int[][] generateMatrix(int n) {
          int[][] arr = new int[n][n];
          int val = 1;//表示填入值
          int i = 0, j = 0;//arr数组索引，i为行标，j列标
          int tempN1 = n;//表示每列或每行在某环的行走最大值
          int tempN2 = 0;//表示每列或每行在某环的行走最小值
  
          for (int l = 0; l < n - n / 2; l++) {//环数
              while (j < tempN1) {
                  arr[i][j] = val++;
                  j++;
              }
              j--;//还原j值
              i++;//下一个数
              while (i < tempN1) {
                  arr[i][j] = val++;
                  i++;
              }
              i--;//还原i值
              j--;//下一个数
              while (j >= tempN2) {
                  arr[i][j] = val++;
                  j--;
              }
              j++;//还原j值
              i--;//下一个数
              while (i > tempN2) {
                  arr[i][j] = val++;
                  i--;
              }
              i++;//还原i数
              j++;//指向下一个数字
              tempN1--;
              tempN2++;
          }
          return arr;
      }
  }
  ```


## day05

#### [209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)

- 题目

  给定一个含有 n 个正整数的数组和一个正整数 target 。

  找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。

- 方法一：暴力破解（超时）

  题目要求求出和大于等于`target`的长度最小的连续子数组，于是乎，就使用一个“窗口”从长度最小的1开始到整个数组的长度遍历，每次遍历都要“框出”子数组的和并比较，满足要求就返回。

  ```java
      public int minSubArrayLen(int target, int[] nums) {
          int left=0;//每个窗口的左索引
          int sum=0;
          for(int cap=1;cap<=nums.length;cap++){//滑动窗口大小
              for(left=0;left<nums.length-cap+1;left++){//左索引位置
                  sum=0;
                  //计算窗口内的和
                  for(int i=left;i<left+cap;i++){
                      sum+=nums[i];
                  }
                  if (sum>=target) return cap;
              }
          }
          return 0;
  }
  ```

- 方法二：滑动窗口

  使用左右指针 `left` 和 `right`，`left` 和 `right` 之间的长度即为滑动窗口的大小（即连续数组的大小）。如果滑动窗口内的值 `sum >= target`，维护连续数组最短长度，`left` 向右移动，缩小滑动窗口。如果滑动窗口内的值 `sum < target`，则 `right` 向有移动，扩大滑动窗口。

  ```java
      public int minSubArrayLen(int target, int[] nums) {
          int left=0;
          int right=0;
          int sum=0;
          int len=Integer.MAX_VALUE;
          while(right<nums.length){
              sum+=nums[right];
              while(sum>=target){
                  if(len>right-left+1) len=right-left+1;//与上次的进行比较，保留最短结果
                  sum-=nums[left++];
              }
              right++;
          }
          if (len==Integer.MAX_VALUE) return 0;
          return len;
  }
  ```


## day06

#### [977. 有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/)

- 题目

  给你一个按 非递减顺序 排序的整数数组 nums，返回 每个数字的平方 组成的新数组，要求也按 非递减顺序 排序。

  示例 1：

  ```txt
  输入：nums = [-4,-1,0,3,10]
  输出：[0,1,9,16,100]
  解释：平方后，数组变为 [16,1,0,9,100]
  排序后，数组变为 [0,1,9,16,100]
  ```

  示例2：

  ```txt
  输入：nums = [-7,-3,2,3,11]
  输出：[4,9,9,49,121]
  ```

- 方法一：直接排序

  最简单的方法就是将数组`nums`中的数平方后直接排序。

  ```java
  class Solution {
      public int[] sortedSquares(int[] nums) {
          int[] ans = new int[nums.length];
          for (int i = 0; i < nums.length; ++i) {
              ans[i] = nums[i] * nums[i];
          }
          Arrays.sort(ans);
          return ans;
      }
  }
  ```

- 方法二：双指针

  同样地，我们可以使用两个指针分别指向位置 `0` 和 `n-1`，每次比较两个指针对应的数，选择较大的那个逆序放入答案并移动指针。这种方法无需处理某一指针移动至边界的情况，读者可以仔细思考其精髓所在。

  ```java
  class Solution {
      public int[] sortedSquares(int[] nums) {
          int n = nums.length;
          int[] ans = new int[n];
          for (int i = 0, j = n - 1, pos = n - 1; i <= j;) {
              if (nums[i] * nums[i] > nums[j] * nums[j]) {
                  ans[pos] = nums[i] * nums[i];
                  ++i;
              } else {
                  ans[pos] = nums[j] * nums[j];
                  --j;
              }
              --pos;
          }
          return ans;
      }
  }
  ```


## day07

#### [19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

- 题目

  给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

  **示例 1：**

  ![img](https://raw.githubusercontent.com/liuhaibin123456789/img-for-cold-bin-blog/master/img/202206262154020.jpeg)

  ```
  输入：head = [1,2,3,4,5], n = 2
  输出：[1,2,3,5]
  ```

  **示例 2：**

  ```
  输入：head = [1], n = 1
  输出：[]
  ```

  **示例 3：**

  ```
  输入：head = [1,2], n = 1
  输出：[1]
  ```

   

  **提示：**

  - 链表中结点的数目为 `sz`
  - `1 <= sz <= 30`
  - `0 <= Node.val <= 100`
  - `1 <= n <= sz`

   

  **进阶：**你能尝试使用一趟扫描实现吗？

- 解决思路

  #### 前言

  在对链表进行操作时，一种常用的技巧是添加一个哑节点（dummy node），它的 next 指针指向链表的头节点。这样一来，我们就不需要对头节点进行特殊的判断了。

  例如，在本题中，如果我们要删除节点 y，我们需要知道节点 y 的前驱节点 x，并将 x 的指针指向 y 的后继节点。但由于头节点不存在前驱节点，因此我们需要在删除头节点时进行特殊判断。但如果我们添加了哑节点，那么头节点的前驱节点就是哑节点本身，此时我们就只需要考虑通用的情况即可。（**注意：此处头节点存放了有意义的数据**）

  特别地，在某些语言中，由于需要自行对内存进行管理。因此在实际的面试中，对于「是否需要释放被删除节点对应的空间」这一问题，我们需要和面试官进行积极的沟通以达成一致。下面的代码中默认不释放空间。

  #### 方法一：计算链表长度

  **思路与算法**

  一种容易想到的方法是，我们首先从头节点开始对链表进行一次遍历，得到链表的长度 L*L*。随后我们再从头节点开始对链表进行一次遍历，当遍历到第 L-n+1个节点时，它就是我们需要删除的节点。

  > 为了与题目中的 n 保持一致，节点的编号从 1 开始，头节点为编号 1 的节点。

  为了方便删除操作，我们可以从哑节点开始遍历 L-n+1个节点。当遍历到第 L-n+1 个节点时，**它的下一个节点**就是我们需要删除的节点，这样我们只需要修改一次指针，就能完成删除操作。

  ![p1](https://assets.leetcode-cn.com/solution-static/19/p1.png)

  **代码**

  ```java
  class Solution {
      public ListNode removeNthFromEnd(ListNode head, int n) {
          ListNode dummy = new ListNode(0, head);
          int length = getLength(head);
          ListNode cur = dummy;
          for (int i = 1; i < length - n + 1; ++i) {
              cur = cur.next;
          }
          cur.next = cur.next.next;
          ListNode ans = dummy.next;
          return ans;
      }
  
      public int getLength(ListNode head) {
          int length = 0;
          while (head != null) {
              ++length;
              head = head.next;
          }
          return length;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(L)，其中 L是链表的长度。
  - 空间复杂度：O(1)。

  #### 方法二：栈

  **思路与算法**

  我们也可以在遍历链表的同时将所有节点依次入栈。根据栈「先进后出」的原则，我们弹出栈的第 n 个节点就是需要删除的节点，并且目前栈顶的节点就是待删除节点的前驱节点。这样一来，删除操作就变得十分方便了。

  ![img](https://assets.leetcode-cn.com/solution-static/19/1.png)

  

  **代码**

  ```java
  class Solution {
      public ListNode removeNthFromEnd(ListNode head, int n) {
          ListNode dummy = new ListNode(0, head);
          Deque<ListNode> stack = new LinkedList<ListNode>();
          ListNode cur = dummy;
          while (cur != null) {
              stack.push(cur);
              cur = cur.next;
          }
          for (int i = 0; i < n; ++i) {
              stack.pop();
          }
          ListNode prev = stack.peek();
          prev.next = prev.next.next;
          ListNode ans = dummy.next;
          return ans;
      }
  }
  ```

  

  **复杂度分析**

  - 时间复杂度：O(L)，其中 L 是链表的长度。
  - 空间复杂度：O(L)，其中 L 是链表的长度。主要为栈的开销。

  #### 方法三：双指针

  **思路与算法**

  我们也可以在不预处理出链表的长度，以及使用常数空间的前提下解决本题。

  由于我们需要找到倒数第 n 个节点，因此我们可以使用两个指针first 和 second 同时对链表进行遍历，并且 first 比 second 超前 n 个节点。当 first 遍历到链表的末尾时，second 就恰好处于倒数第 n 个节点。

  具体地，初始时 first 和second 均指向头节点。我们首先使用first 对链表进行遍历，遍历的次数为 n。此时，first 和 second 之间间隔了 n-1个节点，即first 比 second 超前了 n个节点。

  在这之后，我们同时使用 first和 second 对链表进行遍历。当first 遍历到链表的末尾（即first 为空指针）时，second 恰好指向倒数第 n 个节点。

  根据方法一和方法二，如果我们能够得到的是倒数第 n 个节点的前驱节点而不是倒数第 n 个节点的话，删除操作会更加方便。因此我们可以考虑在初始时将 second 指向哑节点，其余的操作步骤不变。这样一来，当 first 遍历到链表的末尾时，second 的**下一个节点**就是我们需要删除的节点。

  ![p3](https://assets.leetcode-cn.com/solution-static/19/p3.png)

  **代码**

  ```java
  class Solution {
      public ListNode removeNthFromEnd(ListNode head, int n) {
          ListNode dummy = new ListNode(0, head);
          ListNode first = head;
          ListNode second = dummy;
          for (int i = 0; i < n; ++i) {
              first = first.next;
          }
          while (first != null) {
              first = first.next;
              second = second.next;
          }
          second.next = second.next.next;
          ListNode ans = dummy.next;
          return ans;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(L)，其中 L 是链表的长度。
  - 空间复杂度：O(1)。

## day08

#### [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)

- 题目

  给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2020/10/03/swap_ex1.jpg)

  ```
  输入：head = [1,2,3,4]
  输出：[2,1,4,3]
  ```

  **示例 2：**

  ```
  输入：head = []
  输出：[]
  ```

  **示例 3：**

  ```
  输入：head = [1]
  输出：[1]
  ```

   

  **提示：**

  - 链表中节点的数目在范围 `[0, 100]` 内
  - `0 <= Node.val <= 100`

- 方法

  方法一：递归

  **思路与算法**

  可以通过递归的方式实现两两交换链表中的节点。

  递归的终止条件是链表中没有节点，或者链表中只有一个节点，此时无法进行交换。

  如果链表中至少有两个节点，则在两两交换链表中的节点之后，原始链表的头节点变成新的链表的第二个节点，原始链表的第二个节点变成新的链表的头节点。链表中的其余节点的两两交换可以递归地实现。在对链表中的其余节点递归地两两交换之后，更新节点之间的指针关系，即可完成整个链表的两两交换。

  用 `head` 表示原始链表的头节点，新的链表的第二个节点，用 `newHead` 表示新的链表的头节点，原始链表的第二个节点，则原始链表中的其余节点的头节点是 `newHead.next`。令 `head.next = swapPairs(newHead.next)`，表示将其余节点进行两两交换，交换后的新的头节点为 `head` 的下一个节点。然后令 `newHead.next = head`，即完成了所有节点的交换。最后返回新的链表的头节点 `newHead`。

  **代码**

  ```java
  class Solution {
      public ListNode swapPairs(ListNode head) {
          if (head == null || head.next == null) {
              return head;
          }
          ListNode newHead = head.next;
          head.next = swapPairs(newHead.next);
          newHead.next = head;
          return newHead;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是链表的节点数量。需要对每个节点进行更新指针的操作。
  - 空间复杂度：O(n)，其中 n 是链表的节点数量。空间复杂度主要取决于递归调用的栈空间。

  方法二：迭代

  **思路与算法**

  也可以通过迭代的方式实现两两交换链表中的节点。

  创建哑结点 `dummyHead`，令 `dummyHead.next = head`。令 `temp` 表示当前到达的节点，初始时 `temp = dummyHead`。每次需要交换 `temp` 后面的两个节点。

  如果 `temp` 的后面没有节点或者只有一个节点，则没有更多的节点需要交换，因此结束交换。否则，获得 `temp` 后面的两个节点 `node1` 和 `node2`，通过更新节点的指针关系实现两两交换节点。

  具体而言，交换之前的节点关系是 `temp -> node1 -> node2`，交换之后的节点关系要变成 `temp -> node2 -> node1`，因此需要进行如下操作。

  ```java
  temp.next = node2
  node1.next = node2.next
  node2.next = node1
  ```

  完成上述操作之后，节点关系即变成 `temp -> node2 -> node1`。再令 `temp = node1`，对链表中的其余节点进行两两交换，直到全部节点都被两两交换。

  两两交换链表中的节点之后，新的链表的头节点是 `dummyHead.next`，返回新的链表的头节点即可。

  

  **代码**

  ```java
  class Solution {
      public ListNode swapPairs(ListNode head) {
          ListNode dummyHead = new ListNode(0);
          dummyHead.next = head;
          ListNode temp = dummyHead;
          while (temp.next != null && temp.next.next != null) {
              ListNode node1 = temp.next;
              ListNode node2 = temp.next.next;
              temp.next = node2;
              node1.next = node2.next;
              node2.next = node1;
              temp = node1;
          }
          return dummyHead.next;
      }
  }
  //the other
  /**
   * Definition for singly-linked list.
   * public class ListNode {
   *     int val;
   *     ListNode next;
   *     ListNode() {}
   *     ListNode(int val) { this.val = val; }
   *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
   * }
   */
  class Solution {
      public ListNode swapPairs(ListNode head) {
          //哑节点
          ListNode dummyNode = new ListNode(-1,head);
          //每两个节点的遍历链表，当不足两个时则不交换
          ListNode temp=dummyNode;
          ListNode pre=temp.next;
          if(temp.next==null){
                  return dummyNode.next;
          }
          ListNode post=temp.next.next;
          while(pre!=null&&post!=null){
              temp.next=post;
              pre.next=post.next;
              post.next=pre;
              //移动指针
              temp=pre;
              pre=temp.next;
              if(temp.next!=null){
                  post=temp.next.next;
              }
          }
          return dummyNode.next;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是链表的节点数量。需要对每个节点进行更新指针的操作。
  - 空间复杂度：O(1)。

## day09

#### [25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)

- 题目

  给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。

  `k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。

  你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex1.jpg)

  ```
  输入：head = [1,2,3,4,5], k = 2
  输出：[2,1,4,3,5]
  ```

  **示例 2：**

  ![img](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex2.jpg)

  ```
  输入：head = [1,2,3,4,5], k = 3
  输出：[3,2,1,4,5]
  ```

   

  **提示：**

  - 链表中的节点数目为 `n`
  - `1 <= k <= n <= 5000`
  - `0 <= Node.val <= 1000`

   

  **进阶：**你可以设计一个只用 `O(1)` 额外内存空间的算法解决此问题吗？

- 方法

  #### 方法一：模拟

  **思路与算法**

  本题的目标非常清晰易懂，不涉及复杂的算法，但是实现过程中需要考虑的细节比较多，容易写出冗长的代码。主要考查面试者设计的能力。

  我们需要把链表节点按照 `k` 个一组分组，所以可以使用一个指针 `head` 依次指向每组的头节点。这个指针每次向前移动 `k` 步，直至链表结尾。对于每个分组，我们先判断它的长度是否大于等于 `k`。若是，我们就翻转这部分链表，否则不需要翻转。

  接下来的问题就是如何翻转一个分组内的子链表。翻转一个链表并不难，过程可以参考「[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)」。但是对于一个子链表，除了翻转其本身之外，还需要将子链表的头部与上一个子链表连接，以及子链表的尾部与下一个子链表连接。如下图所示：

  ![fig1](https://assets.leetcode-cn.com/solution-static/25/25_fig1.png)

  因此，在翻转子链表的时候，我们不仅需要子链表头节点 `head`，还需要有 `head` 的上一个节点 `pre`，以便翻转完后把子链表再接回 `pre`。

  但是对于第一个子链表，它的头节点 `head` 前面是没有节点 `pre` 的。太麻烦了！难道只能特判了吗？答案是否定的。没有条件，我们就创造条件；没有节点，我们就创建一个节点。我们新建一个节点（**哑节点**），把它接到链表的头部，让它作为 `pre` 的初始值，这样 `head` 前面就有了一个节点，我们就可以避开链表头部的边界条件。这么做还有一个好处，下面我们会看到。

  反复移动指针 `head` 与 `pre`，对 `head` 所指向的子链表进行翻转，直到结尾，我们就得到了答案。下面我们该返回函数值了。

  有的同学可能发现这又是一件麻烦事：链表翻转之后，链表的头节点发生了变化，那么应该返回哪个节点呢？照理来说，前 `k` 个节点翻转之后，链表的头节点应该是第 `k` 个节点。那么要在遍历过程中记录第 `k` 个节点吗？但是如果链表里面没有 `k` 个节点，答案又还是原来的头节点。我们又多了一大堆循环和判断要写，太崩溃了！

  等等！还记得我们创建了节点 `pre` 吗？这个节点一开始被连接到了头节点的前面，而无论之后链表有没有翻转，它的 `next` 指针都会指向正确的头节点。那么我们只要返回它的下一个节点就好了。至此，问题解决。

  ```java
  class Solution {
      public ListNode reverseKGroup(ListNode head, int k) {
          ListNode hair = new ListNode(0);
          hair.next = head;
          ListNode pre = hair;
  
          while (head != null) {
              ListNode tail = pre;
              // 查看剩余部分长度是否大于等于 k
              for (int i = 0; i < k; ++i) {
                  tail = tail.next;
                  if (tail == null) {
                      return hair.next;
                  }
              }
              ListNode nex = tail.next;
              ListNode[] reverse = myReverse(head, tail);
              head = reverse[0];
              tail = reverse[1];
              // 把子链表重新接回原链表
              pre.next = head;
              tail.next = nex;
              pre = tail;
              head = tail.next;
          }
  
          return hair.next;
      }
  
      public ListNode[] myReverse(ListNode head, ListNode tail) {
          ListNode prev = tail.next;
          ListNode p = head;
          while (prev != tail) {
              ListNode nex = p.next;
              p.next = prev;
              prev = p;
              p = nex;
          }
          return new ListNode[]{tail, head};
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 为链表的长度。`head` 指针会在 $O(\lfloor \frac{n}{k} \rfloor) $个节点上停留，每次停留需要进行一次 O(k) 的翻转操作。
  - 空间复杂度：O(1)，我们只需要建立常数个变量。

## day10

#### [141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)

- 题目

  给你一个链表的头节点 `head` ，判断链表中是否有环。

  如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递** 。仅仅是为了标识链表的实际情况。

  *如果链表中存在环* ，则返回 `true` 。 否则，返回 `false` 。

   

  **示例 1：**

  ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist.png)

  ```
  输入：head = [3,2,0,-4], pos = 1
  输出：true
  解释：链表中有一个环，其尾部连接到第二个节点。
  ```

  **示例 2：**

  ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test2.png)

  ```
  输入：head = [1,2], pos = 0
  输出：true
  解释：链表中有一个环，其尾部连接到第一个节点。
  ```

  **示例 3：**

  ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test3.png)

  ```
  输入：head = [1], pos = -1
  输出：false
  解释：链表中没有环。
  ```

   

  **提示：**

  - 链表中节点的数目范围是 `[0, 104]`
  - `-105 <= Node.val <= 105`
  - `pos` 为 `-1` 或者链表中的一个 **有效索引** 。

   

  **进阶：**你能用 `O(1)`（即，常量）内存解决此问题吗？

- 解法

  #### 方法一：哈希表

  **思路及算法**

  最容易想到的方法是遍历所有节点，每次遍历到一个节点时，判断该节点此前是否被访问过。

  具体地，我们可以使用哈希表来存储所有已经访问过的节点。每次我们到达一个节点，如果该节点已经存在于哈希表中，则说明该链表是环形链表，否则就将该节点加入哈希表中。重复这一过程，直到我们遍历完整个链表即可。

  

  **代码**

  ```java
  public class Solution {
      public boolean hasCycle(ListNode head) {
          Set<ListNode> seen = new HashSet<ListNode>();
          while (head != null) {
              if (!seen.add(head)) {
                  return true;
              }
              head = head.next;
          }
          return false;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N)，其中 N 是链表中的节点数。最坏情况下我们需要遍历每个节点一次。
  - 空间复杂度：O(N)，其中 N 是链表中的节点数。主要为哈希表的开销，最坏情况下我们需要将每个节点插入到哈希表中一次。

  

  #### 方法二：快慢指针

  **思路及算法**

  本方法需要读者对「Floyd 判圈算法」（又称龟兔赛跑算法）有所了解。

  假想「乌龟」和「兔子」在链表上移动，「兔子」跑得快，「乌龟」跑得慢。当「乌龟」和「兔子」从链表上的同一个节点开始移动时，如果该链表中没有环，那么「兔子」将一直处于「乌龟」的前方；如果该链表中有环，那么「兔子」会先于「乌龟」进入环，并且一直在环内移动。等到「乌龟」进入环时，由于「兔子」的速度快，它一定会在某个时刻与乌龟相遇，即套了「乌龟」若干圈。

  我们可以根据上述思路来解决本题。具体地，我们定义两个指针，一快一满。慢指针每次只移动一步，而快指针每次移动两步。初始时，慢指针在位置 `head`，而快指针在位置 `head.next`。这样一来，如果在移动的过程中，快指针反过来追上慢指针，就说明该链表为环形链表。否则快指针将到达链表尾部，该链表不为环形链表。

  

  **细节**

  为什么我们要规定初始时慢指针在位置 `head`，快指针在位置 `head.next`，而不是两个指针都在位置 `head`（即与「乌龟」和「兔子」中的叙述相同）？

  - 观察下面的代码，我们使用的是 `while` 循环，循环条件先于循环体。由于循环条件一定是判断快慢指针是否重合，如果我们将两个指针初始都置于 `head`，那么 `while` 循环就不会执行。因此，我们可以假想一个在 `head` 之前的虚拟节点，慢指针从虚拟节点移动一步到达 `head`，快指针从虚拟节点移动两步到达 `head.next`，这样我们就可以使用 `while` 循环了。
  - 当然，我们也可以使用 `do-while` 循环。此时，我们就可以把快慢指针的初始值都置为 `head`。

  

  **代码**

  ```java
  public class Solution {
      public boolean hasCycle(ListNode head) {
          if (head == null || head.next == null) {
              return false;
          }
          ListNode slow = head;
          ListNode fast = head.next;
          while (slow != fast) {
              if (fast == null || fast.next == null) {
                  return false;
              }
              slow = slow.next;
              fast = fast.next.next;
          }
          return true;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N)，其中 N 是链表中的节点数。
    - 当链表中不存在环时，快指针将先于慢指针到达链表尾部，链表中每个节点至多被访问两次。
    - 当链表中存在环时，每一轮移动后，快慢指针的距离将减小一。而初始距离为环的长度，因此至多移动 N 轮。
  - 空间复杂度：O(1)。我们只使用了两个指针的额外空间。


## day11

#### [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)

- 题目

  给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 *如果链表无环，则返回 `null`。*

  如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

  **不允许修改** 链表。

  

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist.png)

  ```
  输入：head = [3,2,0,-4], pos = 1
  输出：返回索引为 1 的链表节点
  解释：链表中有一个环，其尾部连接到第二个节点。
  ```

  **示例 2：**

  ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test2.png)

  ```
  输入：head = [1,2], pos = 0
  输出：返回索引为 0 的链表节点
  解释：链表中有一个环，其尾部连接到第一个节点。
  ```

  **示例 3：**

  ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test3.png)

  ```
  输入：head = [1], pos = -1
  输出：返回 null
  解释：链表中没有环。
  ```

   

  **提示：**

  - 链表中节点的数目范围在范围 `[0, 104]` 内
  - `-105 <= Node.val <= 105`
  - `pos` 的值为 `-1` 或者链表中的一个有效索引

   

  **进阶：**你是否可以使用 `O(1)` 空间解决此题？

- 方法

  **方法一：哈希表**

  **思路与算法**

  一个非常直观的思路是：我们遍历链表中的每个节点，并将它记录下来；一旦遇到了此前遍历过的节点，就可以判定链表中存在环。借助哈希表可以很方便地实现。

  **代码**

  ```java
  public class Solution {
      public ListNode detectCycle(ListNode head) {
          ListNode pos = head;
          Set<ListNode> visited = new HashSet<ListNode>();
          while (pos != null) {
              if (visited.contains(pos)) {
                  return pos;
              } else {
                  visited.add(pos);
              }
              pos = pos.next;
          }
          return null;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N)，其中 N 为链表中节点的数目。我们恰好需要访问链表中的每一个节点。

    空间复杂度：O(N)，其中 N 为链表中节点的数目。我们需要将链表中的每个节点都保存在哈希表当中。
  
  **方法二：快慢指针**
  
    **思路与算法**
  
    我们使用两个指针，fast 与 slow。它们起始都位于链表的头部。随后，slow 指针每次向后移动一个位置，而 fast 指针向后移动两个位置。如果链表中存在环，则 fast 指针最终将再次与 slow 指针在环中相遇。
  
    如下图所示，设链表中环外部分的长度为 a。slow 指针进入环后，又走了 b 的距离与 fast 相遇。此时，fast 指针已经走完了环的 n 圈，因此它走过的总距离为 `a+n*(b+c)+b = a+(n+1)*b+n*c`。
  
    ![fig1](https://assets.leetcode-cn.com/solution-static/142/142_fig1.png)
  
    根据题意，任意时刻，fast 指针走过的距离都为 slow 指针的 2 倍。因此，我们有
  
    ​							`a+(n+1)*b+n*c = 2*(a+b) ==> a = c+(n-1)*(b+c)`
  
    有了 a=c+(n-1)(b+c) 的等量关系，我们会发现：从相遇点到入环点的距离加上 n-1 圈的环长，恰好等于从链表头部到入环点的距离。
  
    因此，当发现 slow 与 fast 相遇时，我们再额外使用一个指针 ptr。起始，它指向链表头部；随后，它和 slow 每次向后移动一个位置。最终，它们会在入环点相遇。
  
    ```java
  public class Solution {
      public ListNode detectCycle(ListNode head) {
          if (head == null) {
              return null;
          }
          ListNode slow = head, fast = head;
          while (fast != null) {
              slow = slow.next;
              if (fast.next != null) {
                  fast = fast.next.next;
              } else {
                  return null;
              }
              if (fast == slow) {
                  ListNode ptr = head;
                  while (ptr != slow) {
                      ptr = ptr.next;
                      slow = slow.next;
                  }
                  return ptr;
              }
          }
          return null;
      }
  }
    ```
  
    **复杂度分析**
  
    - 时间复杂度：O(N)，其中 N 为链表中节点的数目。在最初判断快慢指针是否相遇时，slow 指针走过的距离不会超过链表的总长度；随后寻找入环点时，走过的距离也不会超过链表的总长度。因此，总的执行时间为 O(N)+O(N)=O(N)。
    - 空间复杂度：O(1)。我们只使用了 slow,fast,ptr 三个指针。

## day12

#### [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)

- 题目

  给你两个单链表的头节点 `headA` 和 `headB` ，请你找出并返回两个单链表相交的起始节点。如果两个链表不存在相交节点，返回 `null` 。

  图示两个链表在节点 `c1` 开始相交**：**

  [![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_statement.png)](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_statement.png)

  题目数据 **保证** 整个链式结构中不存在环。

  **注意**，函数返回结果后，链表必须 **保持其原始结构** 。

  **自定义评测：**

  **评测系统** 的输入如下（你设计的程序 **不适用** 此输入）：

  - `intersectVal` - 相交的起始节点的值。如果不存在相交节点，这一值为 `0`
  - `listA` - 第一个链表
  - `listB` - 第二个链表
  - `skipA` - 在 `listA` 中（从头节点开始）跳到交叉节点的节点数
  - `skipB` - 在 `listB` 中（从头节点开始）跳到交叉节点的节点数

  评测系统将根据这些输入创建链式数据结构，并将两个头节点 `headA` 和 `headB` 传递给你的程序。如果程序能够正确返回相交节点，那么你的解决方案将被 **视作正确答案** 。

   

  **示例 1：**

  [![img](https://assets.leetcode.com/uploads/2021/03/05/160_example_1_1.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_1.png)

  ```
  输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,6,1,8,4,5], skipA = 2, skipB = 3
  输出：Intersected at '8'
  解释：相交节点的值为 8 （注意，如果两个链表相交则不能为 0）。
  从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,6,1,8,4,5]。
  在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。
  ```

  **示例 2：**

  [![img](https://assets.leetcode.com/uploads/2021/03/05/160_example_2.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_2.png)

  ```
  输入：intersectVal = 2, listA = [1,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
  输出：Intersected at '2'
  解释：相交节点的值为 2 （注意，如果两个链表相交则不能为 0）。
  从各自的表头开始算起，链表 A 为 [1,9,1,2,4]，链表 B 为 [3,2,4]。
  在 A 中，相交节点前有 3 个节点；在 B 中，相交节点前有 1 个节点。
  ```

  **示例 3：**

  [![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_example_3.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_3.png)

  ```
  输入：intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
  输出：null
  解释：从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。
  由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
  这两个链表不相交，因此返回 null 。
  ```

   

  **提示：**

  - `listA` 中节点数目为 `m`
  - `listB` 中节点数目为 `n`
  - `1 <= m, n <= 3 * 104`
  - `1 <= Node.val <= 105`
  - `0 <= skipA <= m`
  - `0 <= skipB <= n`
  - 如果 `listA` 和 `listB` 没有交点，`intersectVal` 为 `0`
  - 如果 `listA` 和 `listB` 有交点，`intersectVal == listA[skipA] == listB[skipB]`

   

  **进阶：**你能否设计一个时间复杂度 `O(m + n)` 、仅用 `O(1)` 内存的解决方案？

- 方法

  #### 方法一：哈希集合

  **思路和算法**

  判断两个链表是否相交，可以使用哈希集合存储链表节点。

  首先遍历链表 headA，并将链表 headA 中的每个节点加入哈希集合中。然后遍历链表 headB，对于遍历到的每个节点，判断该节点是否在哈希集合中：

  - 如果当前节点不在哈希集合中，则继续遍历下一个节点；
  - 如果当前节点在哈希集合中，则后面的节点都在哈希集合中，即从当前节点开始的所有节点都在两个链表的相交部分，因此在链表 headB 中遍历到的第一个在哈希集合中的节点就是两个链表相交的节点，返回该节点。

  如果链表 headB 中的所有节点都不在哈希集合中，则两个链表不相交，返回 null。

  **代码**

  ```java
  public class Solution {
      public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
          Set<ListNode> visited = new HashSet<ListNode>();
          ListNode temp = headA;
          while (temp != null) {
              visited.add(temp);
              temp = temp.next;
          }
          temp = headB;
          while (temp != null) {
              if (visited.contains(temp)) {
                  return temp;
              }
              temp = temp.next;
          }
          return null;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(m+n)，其中 m 和 n 是分别是链表 headA 和 headB 的长度。需要遍历两个链表各一次。
  - 空间复杂度：O(m)，其中 m 是链表 headA 的长度。需要使用哈希集合存储链表 headA 中的全部节点。

  #### 方法二：双指针

  **思路和算法**

  使用双指针的方法，可以将空间复杂度降至 O(1)*O*(1)。

  只有当链表 headA 和 headB 都不为空时，两个链表才可能相交。因此首先判断链表 headA 和 headB 是否为空，如果其中至少有一个链表为空，则两个链表一定不相交，返回 null。

  当链表 headA 和 headB 都不为空时，创建两个指针 pA 和 pB*，初始时分别指向两个链表的头节点 headA 和 headB，然后将两个指针依次遍历两个链表的每个节点。具体做法如下：

  - 每步操作需要同时更新指针 pA 和 pB。
  - 如果指针 pA 不为空，则将指针 pA 移到下一个节点；如果指针 pB 不为空，则将指针 pB 移到下一个节点。
  - 如果指针 pA 为空，则将指针 pA 移到链表 headB 的头节点；如果指针 pB 为空，则将指针 pB 移到链表 headA 的头节点。
  - 当指针 pA 和 pB 指向同一个节点或者都为空时，返回它们指向的节点或者 null。

  **证明**

  下面提供双指针方法的正确性证明。考虑两种情况，第一种情况是两个链表相交，第二种情况是两个链表不相交。

  情况一：两个链表相交

  链表 headA 和 headB 的长度分别是 m 和 n*。假设链表 headA 的不相交部分有 a 个节点，链表 headB 的不相交部分有 b 个节点，两个链表相交的部分有 c 个节点，则有 a+c=m，b+c=n。

  - 如果 a=b，则两个指针会同时到达两个链表相交的节点，此时返回相交的节点；
  - 如果 a!=*b*，则指针 pA 会遍历完链表 headA，指针 pB 会遍历完链表 headB，两个指针不会同时到达链表的尾节点，然后指针 pA 移到链表 headB 的头节点，指针 pB 移到链表 headA 的头节点，然后两个指针继续移动，在指针 pA 移动了 `a+c+b `次、指针 pB 移动了 `b+c+a` 次之后，两个指针会同时到达两个链表相交的节点，该节点也是两个指针第一次同时指向的节点，此时返回相交的节点。

  情况二：两个链表不相交

  链表 headA 和 headB 的长度分别是 m 和 n。考虑当 m=n 和 m != n 时，两个指针分别会如何移动：

  - 如果 m=n，则两个指针会同时到达两个链表的尾节点，然后同时变成空值 null，此时返回 null；
  - 如果 m != n，则由于两个链表没有公共节点，两个指针也不会同时到达两个链表的尾节点，因此两个指针都会遍历完两个链表，在指针 pA 移动了 `m+n` 次、指针 pB 移动了 `n+m` 次之后，两个指针会同时变成空值 null，此时返回 null。

  **代码**

  ```java
  public class Solution {
      public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
          if (headA == null || headB == null) {
              return null;
          }
          ListNode pA = headA, pB = headB;
          while (pA != pB) {
              pA = pA == null ? headB : pA.next;
              pB = pB == null ? headA : pB.next;
          }
          return pA;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(m+n)，其中 m 和 n 是分别是链表 headA 和 headB 的长度。两个指针同时遍历两个链表，每个指针遍历两个链表各一次。
  - 空间复杂度：O(1)。

## day13

#### [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)

- 题目

  给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2021/02/19/rev1ex1.jpg)

  ```
  输入：head = [1,2,3,4,5]
  输出：[5,4,3,2,1]
  ```

  **示例 2：**

  ![img](https://assets.leetcode.com/uploads/2021/02/19/rev1ex2.jpg)

  ```
  输入：head = [1,2]
  输出：[2,1]
  ```

  **示例 3：**

  ```
  输入：head = []
  输出：[]
  ```

   

  **提示：**

  - 链表中节点的数目范围是 `[0, 5000]`
  - `-5000 <= Node.val <= 5000`

- 解法

  #### 方法一：迭代

  假设存在链表 1-->2-->3-->∅，我们想要把它改成 ∅<--1<--2<--3。

  在遍历列表时，将当前节点的 next 指针改为指向前一个元素。由于节点没有引用其上一个节点，因此必须事先存储其前一个元素。在更改引用之前，还需要另一个指针来存储下一个节点。不要忘记在最后返回新的头引用！

  ```java
  class Solution {
      public ListNode reverseList(ListNode head) {
          ListNode prev = null;
          ListNode curr = head;
          while (curr != null) {
              ListNode nextTemp = curr.next;
              curr.next = prev;
              prev = curr;
              curr = nextTemp;
          }
          return prev;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，假设 n 是列表的长度，时间复杂度是 O(n)。
  - 空间复杂度：O(1)。

  #### 方法二：递归

  递归版本稍微复杂一些，其关键在于反向工作。假设列表的其余部分已经被反转，现在我们应该如何反转它前面的部分？

  假设列表为：

  ​							n~1~-->…-->n~k−1~-->n~k~-->n~k+1~-->…-->n~m~-->∅

  若从节点 n~k+1~ 到 n~k~

  ​							n~1~-->…-->n~k−1~-->n~k~-->n~k+1~<--…<--n~m~

  我们希望 n~k+1~ 的下一个节点指向 n~k~

  所以，n~k~.next.next*=*n~k~

  要小心的是 n~1~ 的下一个必须指向 ∅ 。如果你忽略了这一点，你的链表中可能会产生循环。如果使用大小为 22 的链表测试代码，则可能会捕获此错误。

  ```java
  class Solution {
      public ListNode reverseList(ListNode head) {
          if (head == null || head.next == null) {
              return head;
          }
          ListNode p = reverseList(head.next);
          head.next.next = head;
          head.next = null;
          return p;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，假设 n 是列表的长度，那么时间复杂度为 O(n)。
  - 空间复杂度：O(n)，由于使用递归，将会使用隐式栈空间。递归深度可能会达到 n 层。

## day14

#### [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)

- 题目

  给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。

  有效字符串需满足：

  1. 左括号必须用相同类型的右括号闭合。
  2. 左括号必须以正确的顺序闭合。

   

  **示例 1：**

  ```
  输入：s = "()"
  输出：true
  ```

  **示例 2：**

  ```
  输入：s = "()[]{}"
  输出：true
  ```

  **示例 3：**

  ```
  输入：s = "(]"
  输出：false
  ```

  **示例 4：**

  ```
  输入：s = "([)]"
  输出：false
  ```

  **示例 5：**

  ```
  输入：s = "{[]}"
  输出：true
  ```

   

  **提示：**

  - `1 <= s.length <= 104`
  - `s` 仅由括号 `'()[]{}'` 组成

- 题解

  #### 方法一：栈

  判断括号的有效性可以使用「栈」这一数据结构来解决。

  我们遍历给定的字符串 s。当我们遇到一个左括号时，我们会期望在后续的遍历中，有一个相同类型的右括号将其闭合。由于**后遇到的左括号要先闭合**，因此我们可以将这个左括号放入栈顶。

  当我们遇到一个右括号时，我们需要将一个相同类型的左括号闭合。此时，我们可以取出栈顶的左括号并判断它们是否是相同类型的括号。如果不是相同的类型，或者栈中并没有左括号，那么字符串 s 无效，返回 False。为了快速判断括号的类型，我们可以使用哈希表存储每一种括号。哈希表的键为右括号，值为相同类型的左括号。

  在遍历结束后，如果栈中没有左括号，说明我们将字符串 s 中的所有左括号闭合，返回 True，否则返回 False。

  注意到有效字符串的长度一定为偶数，因此如果字符串的长度为奇数，我们可以直接返回 False，省去后续的遍历判断过程。

  ```java
  class Solution {
      public boolean isValid(String s) {
          int n = s.length();
          if (n % 2 == 1) {
              return false;
          }
  
          Map<Character, Character> pairs = new HashMap<Character, Character>() {{
              put(')', '(');
              put(']', '[');
              put('}', '{');
          }};
          Deque<Character> stack = new LinkedList<Character>();
          for (int i = 0; i < n; i++) {
              char ch = s.charAt(i);
              if (pairs.containsKey(ch)) {
                  if (stack.isEmpty() || stack.peek() != pairs.get(ch)) {
                      return false;
                  }
                  stack.pop();
              } else {
                  stack.push(ch);
              }
          }
          return stack.isEmpty();
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是字符串 s 的长度。
  - 空间复杂度：O*(*n+∣Σ∣)，其中 Σ 表示字符集，本题中字符串只包含 66 种括号，∣Σ∣=6。栈中的字符数量为 O(n)，而哈希表使用的空间为 O(∣Σ∣)，相加即可得到总空间复杂度。

## day15

#### [225. 用队列实现栈](https://leetcode.cn/problems/implement-stack-using-queues/)

- 题目

  请你仅使用两个队列实现一个后入先出（LIFO）的栈，并支持普通栈的全部四种操作（`push`、`top`、`pop` 和 `empty`）。

  实现 `MyStack` 类：

  - `void push(int x)` 将元素 x 压入栈顶。
  - `int pop()` 移除并返回栈顶元素。
  - `int top()` 返回栈顶元素。
  - `boolean empty()` 如果栈是空的，返回 `true` ；否则，返回 `false` 。

   

  **注意：**

  - 你只能使用队列的基本操作 —— 也就是 `push to back`、`peek/pop from front`、`size` 和 `is empty` 这些操作。
  - 你所使用的语言也许不支持队列。 你可以使用 list （列表）或者 deque（双端队列）来模拟一个队列 , 只要是标准的队列操作即可。

   

  **示例：**

  ```
  输入：
  ["MyStack", "push", "push", "top", "pop", "empty"]
  [[], [1], [2], [], [], []]
  输出：
  [null, null, null, 2, 2, false]
  
  解释：
  MyStack myStack = new MyStack();
  myStack.push(1);
  myStack.push(2);
  myStack.top(); // 返回 2
  myStack.pop(); // 返回 2
  myStack.empty(); // 返回 False
  ```

   

  **提示：**

  - `1 <= x <= 9`
  - 最多调用`100` 次 `push`、`pop`、`top` 和 `empty`
  - 每次调用 `pop` 和 `top` 都保证栈不为空

   

  **进阶：**你能否仅用一个队列来实现栈。

- 方法

  #### 方法一：两个队列

  为了满足栈的特性，即最后入栈的元素最先出栈，在使用队列实现栈时，应满足队列前端的元素是最后入栈的元素。可以使用两个队列实现栈的操作，其中 queue1 用于存储栈内的元素，queue2 作为入栈操作的辅助队列。

  入栈操作时，首先将元素入队到 queue2，然后将 queue1 的全部元素依次出队并入队到 queue2，此时 queue2 的前端的元素即为新入栈的元素，再将 queue1 和 queue2 互换，则 queue1 的元素即为栈内的元素，queue1 的前端和后端分别对应栈顶和栈底。

  由于每次入栈操作都确保 queue1 的前端元素为栈顶元素，因此出栈操作和获得栈顶元素操作都可以简单实现。出栈操作只需要移除 queue1 的前端元素并返回即可，获得栈顶元素操作只需要获得 queue1 的前端元素并返回即可（不移除元素）。

  由于 queue1 用于存储栈内的元素，判断栈是否为空时，只需要判断 queue1 是否为空即可。

  ![fig1](https://assets.leetcode-cn.com/solution-static/225/225_fig1.gif)

  

  ```java
  class MyStack {
      Queue<Integer> queue1;
      Queue<Integer> queue2;
  
      /** Initialize your data structure here. */
      public MyStack() {
          queue1 = new LinkedList<Integer>();
          queue2 = new LinkedList<Integer>();
      }
      
      /** Push element x onto stack. */
      public void push(int x) {
          queue2.offer(x);
          while (!queue1.isEmpty()) {
              queue2.offer(queue1.poll());
          }
          Queue<Integer> temp = queue1;
          queue1 = queue2;
          queue2 = temp;
      }
      
      /** Removes the element on top of the stack and returns that element. */
      public int pop() {
          return queue1.poll();
      }
      
      /** Get the top element. */
      public int top() {
          return queue1.peek();
      }
      
      /** Returns whether the stack is empty. */
      public boolean empty() {
          return queue1.isEmpty();
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：入栈操作 O(n)，其余操作都是 O(1)，其中 n 是栈内的元素个数。
    入栈操作需要将 queue1​ 中的 n 个元素出队，并入队 n+1 个元素到 queue2​，共有 2n+1 次操作，每次出队和入队操作的时间复杂度都是 O(1)，因此入栈操作的时间复杂度是 O(n)。
    出栈操作对应将 queue1​ 的前端元素出队，时间复杂度是 O(1)。
    获得栈顶元素操作对应获得 queue1​ 的前端元素，时间复杂度是 O(1)。
    判断栈是否为空操作只需要判断 queue1​ 是否为空，时间复杂度是 O(1)。
  - 空间复杂度：O(n)，其中 n 是栈内的元素个数。需要使用两个队列存储栈内的元素。

  #### 方法二：一个队列

  方法一使用了两个队列实现栈的操作，也可以使用一个队列实现栈的操作。

  使用一个队列时，为了满足栈的特性，即最后入栈的元素最先出栈，同样需要满足队列前端的元素是最后入栈的元素。

  入栈操作时，首先获得入栈前的元素个数 n，然后将元素入队到队列，再将队列中的前 n 个元素（即除了新入栈的元素之外的全部元素）依次出队并入队到队列，此时队列的前端的元素即为新入栈的元素，且队列的前端和后端分别对应栈顶和栈底。

  由于每次入栈操作都确保队列的前端元素为栈顶元素，因此出栈操作和获得栈顶元素操作都可以简单实现。出栈操作只需要移除队列的前端元素并返回即可，获得栈顶元素操作只需要获得队列的前端元素并返回即可（不移除元素）。

  由于队列用于存储栈内的元素，判断栈是否为空时，只需要判断队列是否为空即可。

  ![fig2](https://assets.leetcode-cn.com/solution-static/225/225_fig2.gif)

  

  ```java
  class MyStack {
      Queue<Integer> queue;
  
      /** Initialize your data structure here. */
      public MyStack() {
          queue = new LinkedList<Integer>();
      }
      
      /** Push element x onto stack. */
      public void push(int x) {
          int n = queue.size();
          queue.offer(x);
          for (int i = 0; i < n; i++) {
              queue.offer(queue.poll());
          }
      }
      
      /** Removes the element on top of the stack and returns that element. */
      public int pop() {
          return queue.poll();
      }
      
      /** Get the top element. */
      public int top() {
          return queue.peek();
      }
      
      /** Returns whether the stack is empty. */
      public boolean empty() {
          return queue.isEmpty();
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：入栈操作 O(n)，其余操作都是 O(1)，其中 n 是栈内的元素个数。
    入栈操作需要将队列中的 n 个元素出队，并入队 n+1 个元素到队列，共有 2n+1 次操作，每次出队和入队操作的时间复杂度都是 O(1)，因此入栈操作的时间复杂度是 O(n)。
    出栈操作对应将队列的前端元素出队，时间复杂度是 O(1)。
    获得栈顶元素操作对应获得队列的前端元素，时间复杂度是 O(1)。
    判断栈是否为空操作只需要判断队列是否为空，时间复杂度是 O(1)。
  - 空间复杂度：O(n)，其中 n 是栈内的元素个数。需要使用一个队列存储栈内的元素。

## day16

#### [232. 用栈实现队列](https://leetcode.cn/problems/implement-queue-using-stacks/)

- 题目

  请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（`push`、`pop`、`peek`、`empty`）：

  实现 `MyQueue` 类：

  - `void push(int x)` 将元素 x 推到队列的末尾
  - `int pop()` 从队列的开头移除并返回元素
  - `int peek()` 返回队列开头的元素
  - `boolean empty()` 如果队列为空，返回 `true` ；否则，返回 `false`

  **说明：**

  - 你 **只能** 使用标准的栈操作 —— 也就是只有 `push to top`, `peek/pop from top`, `size`, 和 `is empty` 操作是合法的。
  - 你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。

   

  **示例 1：**

  ```
  输入：
  ["MyQueue", "push", "push", "peek", "pop", "empty"]
  [[], [1], [2], [], [], []]
  输出：
  [null, null, null, 1, 1, false]
  
  解释：
  MyQueue myQueue = new MyQueue();
  myQueue.push(1); // queue is: [1]
  myQueue.push(2); // queue is: [1, 2] (leftmost is front of the queue)
  myQueue.peek(); // return 1
  myQueue.pop(); // return 1, queue is [2]
  myQueue.empty(); // return false
  ```

  

   

  **提示：**

  - `1 <= x <= 9`
  - 最多调用 `100` 次 `push`、`pop`、`peek` 和 `empty`
  - 假设所有操作都是有效的 （例如，一个空的队列不会调用 `pop` 或者 `peek` 操作）

   

  **进阶：**

  - 你能否实现每个操作均摊时间复杂度为 `O(1)` 的队列？换句话说，执行 `n` 个操作的总时间复杂度为 `O(n)` ，即使其中一个操作可能花费较长时间。

- 方法

  https://leetcode.cn/problems/implement-queue-using-stacks/solution/yong-zhan-shi-xian-dui-lie-by-leetcode/

- 代码

  ```java
  class MyQueue {
      Stack<Integer> s1;
      Stack<Integer> s2;
      public MyQueue() {
          this.s1=new Stack<>();
          this.s2=new Stack<>();
      }
      
      public void push(int x) {
          s1.push(x);
      }
      
      public int pop() {
          while(s1.size()!=0){
              s2.push(s1.pop());
          }
          int res=s2.pop();
          while(s2.size()!=0){
              s1.push(s2.pop());
          }
          return res;
      }
      
      public int peek() {
          while(s1.size()!=0){
              s2.push(s1.pop());
          }
          int res=s2.peek();
          while(s2.size()!=0){
              s1.push(s2.pop());
          }
          return res;
      }
      
      public boolean empty() {
          return s1.empty();
      }
  }
  ```


## day17

#### [239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)

- 题目

  给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

  返回 *滑动窗口中的最大值* 。

   

  **示例 1：**

  ```
  输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
  输出：[3,3,5,5,6,7]
  解释：
  滑动窗口的位置                最大值
  ---------------               -----
  [1  3  -1] -3  5  3  6  7       3
   1 [3  -1  -3] 5  3  6  7       3
   1  3 [-1  -3  5] 3  6  7       5
   1  3  -1 [-3  5  3] 6  7       5
   1  3  -1  -3 [5  3  6] 7       6
   1  3  -1  -3  5 [3  6  7]      7
  ```

  **示例 2：**

  ```
  输入：nums = [1], k = 1
  输出：[1]
  ```

   

  **提示：**

  - `1 <= nums.length <= 105`
  - `-104 <= nums[i] <= 104`
  - `1 <= k <= nums.length`

- 方法一（超时）

  暴力解法，依次移动窗口，不断寻找最大值，当然可以取巧：每一次只滑动一个索引位置，因此，每一次滑动窗口内的数已经知道了最大的数。如果最大的数还在下一次滑动窗口内，那么就可以利用这个属性，只需要比较上次滑动窗口的最大值和这次滑动窗口的末尾索引处的值，就可以得出下一次滑动窗口的最大值了，而不用傻乎乎的重复比较。当然，还是超时了...

  ```java
  class Solution {
      public int[] maxSlidingWindow(int[] nums, int k) {
          int len=nums.length;
          int max=Integer.MIN_VALUE;
          int index=-1;
          int[] arr = new int[len-k+1];
          for(int i=0;i<len-k+1;i++){
              if(index!=0){
                  if(max<nums[i+k-1]){
                      max=nums[i+k-1];
                  }
              }
              max=Integer.MIN_VALUE;
              index=-1;
              for(int j=i;j<i+k;j++){
                  if(max<nums[j]){
                      max=nums[j];
                      index=j;
                  }
              }
              arr[i]=max;
          }
          return arr;
      }
  }
  
  /**
  已经知道上一个最大的数了，
  记录这个最大的数的位置如果不是窗口的第一位，显然具有利用价值：
  无需比较上次窗口的最后一位，只需要比较窗口的下一位即可。
   */
  ```

- 其他方法

  优先队列、单调队列（双端队列的特殊情况）

  https://leetcode.cn/problems/sliding-window-maximum/solution/hua-dong-chuang-kou-zui-da-zhi-by-leetco-ki6m/

## day18

#### [1047. 删除字符串中的所有相邻重复项](https://leetcode.cn/problems/remove-all-adjacent-duplicates-in-string/)

- 题目

  给出由小写字母组成的字符串 `S`，**重复项删除操作**会选择两个相邻且相同的字母，并删除它们。

  在 S 上反复执行重复项删除操作，直到无法继续删除。

  在完成所有重复项删除操作后返回最终的字符串。答案保证唯一。

   

  **示例：**

  ```
  输入："abbaca"
  输出："ca"
  解释：
  例如，在 "abbaca" 中，我们可以删除 "bb" 由于两字母相邻且相同，这是此时唯一可以执行删除操作的重复项。之后我们得到字符串 "aaca"，其中又只有 "aa" 可以执行重复项删除操作，所以最后的字符串为 "ca"。
  ```

   

  **提示：**

  1. `1 <= S.length <= 20000`
  2. `S` 仅由小写英文字母组成。

- 解法

  **方法一：栈**

  充分理解题意后，我们可以发现，当字符串中同时有多组相邻重复项时，我们无论是先删除哪一个，都不会影响最终的结果。因此我们可以从左向右顺次处理该字符串。

  而消除一对相邻重复项可能会导致新的相邻重复项出现，如从字符串 abba 中删除 bb 会导致出现新的相邻重复项 aa 出现。因此我们需要保存当前还未被删除的字符。一种显而易见的数据结构呼之欲出：栈。我们只需要遍历该字符串，如果当前字符和栈顶字符相同，我们就**贪心**地将其消去，否则就将其入栈即可。

  **代码**

  ```java
  class Solution {
      public String removeDuplicates(String s) {
          StringBuffer stack = new StringBuffer();
          int top = -1;
          for (int i = 0; i < s.length(); ++i) {
              char ch = s.charAt(i);
              if (top >= 0 && stack.charAt(top) == ch) {
                  stack.deleteCharAt(top);
                  --top;
              } else {
                  stack.append(ch);
                  ++top;
              }
          }
          return stack.toString();
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是字符串的长度。我们只需要遍历该字符串一次。
  - 空间复杂度：O(n) 或 O(1))，取决于使用的语言提供的字符串类是否提供了类似「入栈」和「出栈」的接口。注意返回值不计入空间复杂度。

## day19

#### [151. 颠倒字符串中的单词](https://leetcode.cn/problems/reverse-words-in-a-string/)

- 题目

  给你一个字符串 `s` ，颠倒字符串中 **单词** 的顺序。

  **单词** 是由非空格字符组成的字符串。`s` 中使用至少一个空格将字符串中的 **单词** 分隔开。

  返回 **单词** 顺序颠倒且 **单词** 之间用单个空格连接的结果字符串。

  **注意：**输入字符串 `s`中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。

   

  **示例 1：**

  ```
  输入：s = "the sky is blue"
  输出："blue is sky the"
  ```

  **示例 2：**

  ```
  输入：s = "  hello world  "
  输出："world hello"
  解释：颠倒后的字符串中不能存在前导空格和尾随空格。
  ```

  **示例 3：**

  ```
  输入：s = "a good   example"
  输出："example good a"
  解释：如果两个单词间有多余的空格，颠倒后的字符串需要将单词间的空格减少到仅有一个。
  ```

   

  **提示：**

  - `1 <= s.length <= 104`
  - `s` 包含英文大小写字母、数字和空格 `' '`
  - `s` 中 **至少存在一个** 单词

- 方法一

  **思路：**

  1. 去除首部空格
  2. 将空格之间的子串追加到字符串累加器并附带一个空格
  3. 追加完过后，再以空格分隔字符串
  4. 倒序数组，追加空格即可

  **代码：（空间复杂度较高）**

  ```java
  class Solution {
      public String reverseWords(String s) {
          char[] chs = s.toCharArray();
          char prev = 0;
          StringBuilder stringBuilder = new StringBuilder();
  
          int index=-1;
          for (int i = 0; i < s.length(); i++) {
              if (chs[i]!=' '){
                  index=i;
                  break;
              }
          }
  
          for (int i = index; i < chs.length; i++) {
              if (chs[i] == ' ' && prev == ' ') {//如果当前字符和上一个字符为' ',
                  continue;
              }
  
              stringBuilder.append(chs[i]);
              prev = chs[i];
          }
          String str = stringBuilder.toString();
          String[] split = str.split(" ");
          stringBuilder = new StringBuilder();
          for (int i = split.length - 1; i > 0; i--) {
              stringBuilder.append(split[i]).append(' ');
          }
          stringBuilder.append(split[0]);
          return stringBuilder.toString();
      }
  }
  ```

- 方法二

  https://leetcode.cn/problems/reverse-words-in-a-string/solution/fan-zhuan-zi-fu-chuan-li-de-dan-ci-by-leetcode-sol/

## day20

#### [541. 反转字符串 II](https://leetcode.cn/problems/reverse-string-ii/)

- 题目

  给定一个字符串 `s` 和一个整数 `k`，从字符串开头算起，每计数至 `2k` 个字符，就反转这 `2k` 字符中的前 `k` 个字符。

  - 如果剩余字符少于 `k` 个，则将剩余字符全部反转。
  - 如果剩余字符小于 `2k` 但大于或等于 `k` 个，则反转前 `k` 个字符，其余字符保持原样。

   

  **示例 1：**

  ```
  输入：s = "abcdefg", k = 2
  输出："bacdfeg"
  ```

  **示例 2：**

  ```
  输入：s = "abcd", k = 2
  输出："bacd"
  ```

   

  **提示：**

  - `1 <= s.length <= 104`
  - `s` 仅由小写英文组成
  - `1 <= k <= 104`

- 方法

  #### 方法一：模拟

  我们直接按题意进行模拟：反转每个下标从 2k2*k* 的倍数开始的，长度为 k*k* 的子串。若该子串长度不足 k*k*，则反转整个子串。

  ```java
  class Solution {
      public String reverseStr(String s, int k) {
          int n = s.length();
          char[] arr = s.toCharArray();
          for (int i = 0; i < n; i += 2 * k) {
              reverse(arr, i, Math.min(i + k, n) - 1);
          }
          return new String(arr);
      }
  
      public void reverse(char[] arr, int left, int right) {
          while (left < right) {
              char temp = arr[left];
              arr[left] = arr[right];
              arr[right] = temp;
              left++;
              right--;
          }
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是字符串 s 的长度。
  - 空间复杂度：O(1) 或 O(n)，取决于使用的语言中字符串类型的性质。如果字符串是可修改的，那么我们可以直接在原字符串上修改，空间复杂度为 O(1)，否则需要使用 O(n) 的空间将字符串临时转换为可以修改的数据结构（例如数组），空间复杂度为 O(n)。

## day21

#### [1. 两数之和](https://leetcode.cn/problems/two-sum/)

- 题目

  给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。

  你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

  你可以按任意顺序返回答案。

   

  **示例 1：**

  ```
  输入：nums = [2,7,11,15], target = 9
  输出：[0,1]
  解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
  ```

  **示例 2：**

  ```
  输入：nums = [3,2,4], target = 6
  输出：[1,2]
  ```

  **示例 3：**

  ```
  输入：nums = [3,3], target = 6
  输出：[0,1]
  ```

   

  **提示：**

  - `2 <= nums.length <= 104`
  - `-109 <= nums[i] <= 109`
  - `-109 <= target <= 109`
  - **只会存在一个有效答案**

  **进阶：**你可以想出一个时间复杂度小于 `O(n2)` 的算法吗？

- 方法

  **方法一：暴力枚举**

  **思路及算法**

  最容易想到的方法是枚举数组中的每一个数 `x`，寻找数组中是否存在 `target - x`。当我们使用遍历整个数组的方式寻找 `target - x` 时，需要注意到每一个位于 `x` 之前的元素都已经和 `x` 匹配过，因此不需要再进行匹配。而每一个元素不能被使用两次，所以我们只需要在 `x` 后面的元素中寻找 `target - x`。

  **代码**

  

  ```java
  class Solution {
      public int[] twoSum(int[] nums, int target) {
          int n = nums.length;
          for (int i = 0; i < n; ++i) {
              for (int j = i + 1; j < n; ++j) {
                  if (nums[i] + nums[j] == target) {
                      return new int[]{i, j};
                  }
              }
          }
          return new int[0];
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N^2^)，其中 N 是数组中的元素数量。最坏情况下数组中任意两个数都要被匹配一次。
  - 空间复杂度：O(1)。

  **方法二：哈希表**

  **思路及算法**

  **此题，如果使用方法一，显然时间复杂度过高。结合题意：在一堆值里找一个值，应该考虑转化为hash的数据结构，hash的时间复杂度为O(1)**

  注意到方法一的时间复杂度较高的原因是寻找 `target - x` 的时间复杂度过高。因此，我们需要一种更优秀的方法，能够快速寻找数组中是否存在目标元素。如果存在，我们需要找出它的索引。使用哈希表，可以将寻找 `target - x` 的时间复杂度降低到从 O(N) 降低到 O(1)。这样我们创建一个哈希表，对于每一个 `x`，我们首先查询哈希表中是否存在 `target - x`，然后将 `x` 插入到哈希表中，即可保证不会让 `x` 和自己匹配。

  **代码**

  ```java
  class Solution {
      public int[] twoSum(int[] nums, int target) {
          Map<Integer, Integer> hashtable = new HashMap<Integer, Integer>();
          for (int i = 0; i < nums.length; ++i) {
              // 将此处的n次循环变为1次即可得到结果，这就是hash的好处
              if (hashtable.containsKey(target - nums[i])) {
                  return new int[]{hashtable.get(target - nums[i]), i};
              }
              hashtable.put(nums[i], i);
          }
          return new int[0];
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N)，其中 N 是数组中的元素数量。对于每一个元素 `x`，我们可以 O(1) 地寻找 `target - x`。
  - 空间复杂度：O(N)，其中 N 是数组中的元素数量。主要为哈希表的开销。

## day22

#### [15. 三数之和](https://leetcode.cn/problems/3sum/)

- 题目

  给你一个包含 `n` 个整数的数组 `nums`，判断 `nums` 中是否存在三个元素 *a，b，c ，*使得 *a + b + c =* 0 ？请你找出所有和为 `0` 且不重复的三元组。

  **注意：**答案中不可以包含重复的三元组。

   

  **示例 1：**

  ```
  输入：nums = [-1,0,1,2,-1,-4]
  输出：[[-1,-1,2],[-1,0,1]]
  ```

  **示例 2：**

  ```
  输入：nums = []
  输出：[]
  ```

  **示例 3：**

  ```
  输入：nums = [0]
  输出：[]
  ```

   

  **提示：**

  - `0 <= nums.length <= 3000`
  - `-105 <= nums[i] <= 105`

- 解法

  **方法一：排序 + 双指针**

  题目中要求找到所有「不重复」且和为 0 的三元组，这个「不重复」的要求使得我们无法简单地使用三重循环枚举所有的三元组。这是因为在最坏的情况下，数组中的元素全部为 0，即

  ```
  [0, 0, 0, 0, 0, ..., 0, 0, 0]
  ```

  任意一个三元组的和都为 00。如果我直接使用三重循环枚举三元组，会得到 O(N^3^) 个满足题目要求的三元组（其中 N 是数组的长度）时间复杂度至少为 O(N^3^)。在这之后，我们还需要使用哈希表进行去重操作，得到不包含重复三元组的最终答案，又消耗了大量的空间。这个做法的时间复杂度和空间复杂度都很高，因此我们要换一种思路来考虑这个问题。

  「不重复」的本质是什么？我们保持三重循环的大框架不变，只需要保证：

  - 第二重循环枚举到的元素不小于当前第一重循环枚举到的元素；
  - 第三重循环枚举到的元素不小于当前第二重循环枚举到的元素。

  也就是说，我们枚举的三元组 (a, b, c) 满足 a*≤*b*≤*c，保证了只有 (a, b, c) 这个顺序会被枚举到，而 (b, a, c)、(c, b, a) 等等这些不会，这样就减少了重复。要实现这一点，我们可以将数组中的元素从小到大进行排序，随后使用普通的三重循环就可以满足上面的要求。

  同时，对于每一重循环而言，相邻两次枚举的元素不能相同，否则也会造成重复。举个例子，如果排完序的数组为

  ```
  [0, 1, 2, 2, 2, 3]
   ^  ^  ^
  ```

  我们使用三重循环枚举到的第一个三元组为 (0, 1, 2)，如果第三重循环继续枚举下一个元素，那么仍然是三元组 (0, 1, 2)，产生了重复。因此我们需要将第三重循环「跳到」下一个不相同的元素，即数组中的最后一个元素 3，枚举三元组 (0, 1, 3)。

  下面给出了改进的方法的伪代码实现：

  ```
  nums.sort()
  for first = 0 .. n-1
      // 只有和上一次枚举的元素不相同，我们才会进行枚举
      if first == 0 or nums[first] != nums[first-1] then
          for second = first+1 .. n-1
              if second == first+1 or nums[second] != nums[second-1] then
                  for third = second+1 .. n-1
                      if third == second+1 or nums[third] != nums[third-1] then
                          // 判断是否有 a+b+c==0
                          check(first, second, third)
  ```

  这种方法的时间复杂度仍然为 O(N^3^)，毕竟我们还是没有跳出三重循环的大框架。然而它是很容易继续优化的，可以发现，如果我们固定了前两重循环枚举到的元素 a 和 b，那么只有唯一的 c 满足 a+b+c=0。当第二重循环往后枚举一个元素 b' 时，由于 b' > b，那么满足 a+b'+c'=0 的 c' 一定有 c' < c，即 c' 在数组中一定出现在 c 的左侧。也就是说，我们可以从小到大枚举 b，**同时**从大到小枚举 c，即**第二重循环和第三重循环实际上是并列的关系**。

  有了这样的发现，我们就可以保持第二重循环不变，而将**第三重循环变成一个从数组最右端开始向左移动的指针**，从而得到下面的伪代码：

  ```
  nums.sort()
  for first = 0 .. n-1
      if first == 0 or nums[first] != nums[first-1] then
          // 第三重循环对应的指针
          third = n-1
          for second = first+1 .. n-1
              if second == first+1 or nums[second] != nums[second-1] then
                  // 向左移动指针，直到 a+b+c 不大于 0
                  while nums[first]+nums[second]+nums[third] > 0
                      third = third-1
                  // 判断是否有 a+b+c==0
                  check(first, second, third)
  ```

  这个方法就是我们常说的「双指针」，当我们需要枚举数组中的两个元素时，如果我们发现随着第一个元素的递增，第二个元素是递减的，那么就可以使用双指针的方法，将枚举的时间复杂度从 O(N^2)*O*(*N*2) 减少至 O(N)。为什么是 O(N) 呢？这是因为在枚举的过程每一步中，「左指针」会向右移动一个位置（也就是题目中的 b），而「右指针」会向左移动若干个位置，这个与数组的元素有关，但我们知道它一共会移动的位置数为 O(N)，**均摊下来**，每次也向左移动一个位置，因此时间复杂度为 O(N)。

  注意到我们的伪代码中还有第一重循环，时间复杂度为 O(N)*O*(*N*)，因此枚举的总时间复杂度为 O(N^2^)。由于排序的时间复杂度为 O(NlogN)，在渐进意义下小于前者，因此算法的总时间复杂度为 O(N^2^)。

  上述的伪代码中还有一些细节需要补充，例如我们需要保持左指针一直在右指针的左侧（即满足 b*≤*c），具体可以参考下面的代码，均给出了详细的注释。

  **代码：**

  ```java
  class Solution {
      public List<List<Integer>> threeSum(int[] nums) {
          int n = nums.length;
          Arrays.sort(nums);
          List<List<Integer>> ans = new ArrayList<List<Integer>>();
          // 枚举 a
          for (int first = 0; first < n; ++first) {
              // 需要和上一次枚举的数不相同
              if (first > 0 && nums[first] == nums[first - 1]) {
                  continue;
              }
              // c 对应的指针初始指向数组的最右端
              int third = n - 1;
              int target = -nums[first];
              // 枚举 b
              for (int second = first + 1; second < n; ++second) {
                  // 需要和上一次枚举的数不相同
                  if (second > first + 1 && nums[second] == nums[second - 1]) {
                      continue;
                  }
                  // 需要保证 b 的指针在 c 的指针的左侧
                  while (second < third && nums[second] + nums[third] > target) {
                      --third;
                  }
                  // 如果指针重合，随着 b 后续的增加
                  // 就不会有满足 a+b+c=0 并且 b<c 的 c 了，可以退出循环
                  if (second == third) {
                      break;
                  }
                  if (nums[second] + nums[third] == target) {
                      List<Integer> list = new ArrayList<Integer>();
                      list.add(nums[first]);
                      list.add(nums[second]);
                      list.add(nums[third]);
                      ans.add(list);
                  }
              }
          }
          return ans;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N^2^)，其中 N 是数组 nums 的长度。
  - 空间复杂度：O(logN)。我们忽略存储答案的空间，额外的排序的空间复杂度为 O(logN)。然而我们修改了输入的数组 nums，在实际情况下不一定允许，因此也可以看成使用了一个额外的数组存储了 nums 的副本并进行排序，空间复杂度为 O(N)。

## day23

#### [202. 快乐数](https://leetcode.cn/problems/happy-number/)

- 题目

  编写一个算法来判断一个数 `n` 是不是快乐数。

  **「快乐数」** 定义为：

  - 对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和。
  - 然后重复这个过程直到这个数变为 1，也可能是 **无限循环** 但始终变不到 1。
  - 如果这个过程 **结果为** 1，那么这个数就是快乐数。

  如果 `n` 是 *快乐数* 就返回 `true` ；不是，则返回 `false` 。

   

  **示例 1：**

  ```
  输入：n = 19
  输出：true
  解释：
  12 + 92 = 82
  82 + 22 = 68
  62 + 82 = 100
  12 + 02 + 02 = 1
  ```

  **示例 2：**

  ```
  输入：n = 2
  输出：false
  ```

   

  **提示：**

  - `1 <= n <= 231 - 1`

- 解法

  **暴力错误且超时**

  ```java
  class Solution {
      public boolean isHappy(int n) {
          int tmp=n;
          while(true){
              tmp=get(tmp);
              if(tmp==n) return false;
              else if(tmp==1) return true;
          }
      }
  
      public int get(int n) {
          int tmp=n;
          int sum=0;
          int r=0;
          while(tmp!=0){
              r = tmp%10;
              sum = r*r+sum;
              tmp = tmp/10;
          }
          return sum;
      }
  }
  ```

  超时的原因在于：条件判断错误，不是一定要出现到初始数字才会陷入死循环，有可能是中途的某个节点出现循环，例如：数字`116`

  ![fig2](https://assets.leetcode-cn.com/solution-static/202/202_fig2.png)

  显然，需要使用弗洛伊德判圈算法（龟兔赛跑算法）--> 快慢指针法：一个快指针和一个慢指针同时出发，如果快指针追上了慢指针说明是一个死循环，就返回`false`

  **快慢指针法**

  ```java
  class Solution {
      public boolean isHappy(int n) {
      int slowRunner = n;
          int fastRunner = getNext(n);
          while (fastRunner != 1 && slowRunner != fastRunner) {
              slowRunner = getNext(slowRunner);
              fastRunner = getNext(getNext(fastRunner));
          }
          return fastRunner == 1;
      }
  
      public int getNext(int n) {
          int tmp=n;
          int sum=0;
          int r=0;
          while(tmp!=0){
              r = tmp%10;
              sum = r*r+sum;
              tmp = tmp/10;
          }
          return sum;
      }
  }
  ```

  

## day24

#### [454. 四数相加 II](https://leetcode.cn/problems/4sum-ii/)

- 题目

  给你四个整数数组 `nums1`、`nums2`、`nums3` 和 `nums4` ，数组长度都是 `n` ，请你计算有多少个元组 `(i, j, k, l)` 能满足：

  - `0 <= i, j, k, l < n`
  - `nums1[i] + nums2[j] + nums3[k] + nums4[l] == 0`

   

  **示例 1：**

  ```
  输入：nums1 = [1,2], nums2 = [-2,-1], nums3 = [-1,2], nums4 = [0,2]
  输出：2
  解释：
  两个元组如下：
  1. (0, 0, 0, 1) -> nums1[0] + nums2[0] + nums3[0] + nums4[1] = 1 + (-2) + (-1) + 2 = 0
  2. (1, 1, 0, 0) -> nums1[1] + nums2[1] + nums3[0] + nums4[0] = 2 + (-1) + (-1) + 0 = 0
  ```

  **示例 2：**

  ```
  输入：nums1 = [0], nums2 = [0], nums3 = [0], nums4 = [0]
  输出：1
  ```

   

   **提示：**

  - `n == nums1.length`
  - `n == nums2.length`
  - `n == nums3.length`
  - `n == nums4.length`
  - `1 <= n <= 200`
  - `-228 <= nums1[i], nums2[i], nums3[i], nums4[i] <= 228`

- 解法：分组+hash表

  **思路与算法**

  我们可以将四个数组分成两部分，A 和 B 为一组，C 和 D 为另外一组。

  对于 A 和 B，我们使用二重循环对它们进行遍历，得到所有 A[i]+B[j] 的值并存入哈希映射中。对于哈希映射中的每个键值对，每个键表示一种 A[i]+B[j]，对应的值为 A[i]+B[j] 出现的次数。

  对于 C*C* 和 D*D*，我们同样使用二重循环对它们进行遍历。当遍历到 C[k]+D[l] 时，如果 -(C[k]+D[l]) 出现在哈希映射中，那么将 -(C[k]+D[l]) 对应的值累加进答案中。

  最终即可得到满足 A[i]+B[j]+C[k]+D[l]=0 的四元组数目。

  **代码**

  ```java
  class Solution {
      public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
          Map<Integer, Integer> countAB = new HashMap<Integer, Integer>();
          for (int u : A) {
              for (int v : B) {
                  countAB.put(u + v, countAB.getOrDefault(u + v, 0) + 1);
              }
          }
          int ans = 0;
          for (int u : C) {
              for (int v : D) {
                  if (countAB.containsKey(-u - v)) {
                      ans += countAB.get(-u - v);
                  }
              }
          }
          return ans;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n^2^)。我们使用了两次二重循环，时间复杂度均为 O(n^2^)。在循环中对哈希映射进行的修改以及查询操作的期望时间复杂度均为 O(1)，因此总时间复杂度为 O(n^2^)。
  - 空间复杂度：O(n^2^)，即为哈希映射需要使用的空间。在最坏的情况下，A[i]+B[j] 的值均不相同，因此值的个数为 n^2^，也就需要 O(n^2^) 的空间。

## day25

#### [100. 相同的树](https://leetcode.cn/problems/same-tree/)

- 题目

  给你两棵二叉树的根节点 `p` 和 `q` ，编写一个函数来检验这两棵树是否相同。

  如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2020/12/20/ex1.jpg)

  ```
  输入：p = [1,2,3], q = [1,2,3]
  输出：true
  ```

  **示例 2：**

  ![img](https://assets.leetcode.com/uploads/2020/12/20/ex2.jpg)

  ```
  输入：p = [1,2], q = [1,null,2]
  输出：false
  ```

  **示例 3：**

  ![img](https://assets.leetcode.com/uploads/2020/12/20/ex3.jpg)

  ```
  输入：p = [1,2,1], q = [1,1,2]
  输出：false
  ```

   

  **提示：**

  - 两棵树上的节点数目都在范围 `[0, 100]` 内
  - `-104 <= Node.val <= 104`

- 解法

  两个二叉树相同，当且仅当两个二叉树的结构完全相同，且所有对应节点的值相同。因此，可以通过搜索的方式判断两个二叉树是否相同。

  **方法一：深度优先搜索**

  如果两个二叉树都为空，则两个二叉树相同。如果两个二叉树中有且只有一个为空，则两个二叉树一定不相同。

  如果两个二叉树都不为空，那么首先判断它们的根节点的值是否相同，若不相同则两个二叉树一定不同，若相同，再分别判断两个二叉树的左子树是否相同以及右子树是否相同。这是一个递归的过程，因此可以使用深度优先搜索，递归地判断两个二叉树是否相同。

  ```java
  class Solution {
      public boolean isSameTree(TreeNode p, TreeNode q) {
          if (p == null && q == null) {
              return true;
          } else if (p == null || q == null) {
              return false;
          } else if (p.val != q.val) {
              return false;
          } else {
              return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
          }
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(min(m,n))，其中 m 和 n 分别是两个二叉树的节点数。对两个二叉树同时进行深度优先搜索，只有当两个二叉树中的对应节点都不为空时才会访问到该节点，因此被访问到的节点数不会超过较小的二叉树的节点数。
  - 空间复杂度：O(min(m,n))，其中 m 和 n 分别是两个二叉树的节点数。空间复杂度取决于递归调用的层数，递归调用的层数不会超过较小的二叉树的最大高度，最坏情况下，二叉树的高度等于节点数。

  **方法二：广度优先搜索**

  也可以通过广度优先搜索判断两个二叉树是否相同。同样首先判断两个二叉树是否为空，如果两个二叉树都不为空，则从两个二叉树的根节点开始广度优先搜索。

  使用两个队列分别存储两个二叉树的节点。初始时将两个二叉树的根节点分别加入两个队列。每次从两个队列各取出一个节点，进行如下比较操作。

  1. 比较两个节点的值，如果两个节点的值不相同则两个二叉树一定不同；
  2. 如果两个节点的值相同，则判断两个节点的子节点是否为空，如果只有一个节点的左子节点为空，或者只有一个节点的右子节点为空，则两个二叉树的结构不同，因此两个二叉树一定不同；
  3. 如果两个节点的子节点的结构相同，则将两个节点的非空子节点分别加入两个队列，子节点加入队列时需要注意顺序，如果左右子节点都不为空，则先加入左子节点，后加入右子节点。

  如果搜索结束时两个队列同时为空，则两个二叉树相同。如果只有一个队列为空，则两个二叉树的结构不同，因此两个二叉树不同。

  ```java
  class Solution {
      public boolean isSameTree(TreeNode p, TreeNode q) {
          if (p == null && q == null) {
              return true;
          } else if (p == null || q == null) {
              return false;
          }
          Queue<TreeNode> queue1 = new LinkedList<TreeNode>();
          Queue<TreeNode> queue2 = new LinkedList<TreeNode>();
          queue1.offer(p);
          queue2.offer(q);
          while (!queue1.isEmpty() && !queue2.isEmpty()) {
              TreeNode node1 = queue1.poll();
              TreeNode node2 = queue2.poll();
              if (node1.val != node2.val) {
                  return false;
              }
              TreeNode left1 = node1.left, right1 = node1.right, left2 = node2.left, right2 = node2.right;
              if (left1 == null ^ left2 == null) {
                  return false;
              }
              if (right1 == null ^ right2 == null) {
                  return false;
              }
              if (left1 != null) {
                  queue1.offer(left1);
              }
              if (right1 != null) {
                  queue1.offer(right1);
              }
              if (left2 != null) {
                  queue2.offer(left2);
              }
              if (right2 != null) {
                  queue2.offer(right2);
              }
          }
          return queue1.isEmpty() && queue2.isEmpty();
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(min(m,n))，其中 m 和 n 分别是两个二叉树的节点数。对两个二叉树同时进行广度优先搜索，只有当两个二叉树中的对应节点都不为空时才会访问到该节点，因此被访问到的节点数不会超过较小的二叉树的节点数。
  - 空间复杂度：O(min(m,n))，其中 m 和 n 分别是两个二叉树的节点数。空间复杂度取决于队列中的元素个数，队列中的元素个数不会超过较小的二叉树的节点数。

## day26

#### [101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/)

- 题目

  给你一个二叉树的根节点 `root` ， 检查它是否轴对称。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2021/02/19/symtree1.jpg)

  ```
  输入：root = [1,2,2,3,4,4,3]
  输出：true
  ```

  **示例 2：**

  ![img](https://assets.leetcode.com/uploads/2021/02/19/symtree2.jpg)

  ```
  输入：root = [1,2,2,null,3,null,3]
  输出：false
  ```

   

  **提示：**

  - 树中节点数目在范围 `[1, 1000]` 内
  - `-100 <= Node.val <= 100`

   

  **进阶：**你可以运用递归和迭代两种方法解决这个问题吗？

- 解法

  **方法一：递归**

  **思路和算法**

  如果一个树的左子树与右子树镜像对称，那么这个树是对称的。

  ![fig1](https://assets.leetcode-cn.com/solution-static/101/101_fig1.PNG)

  因此，该问题可以转化为：两个树在什么情况下互为镜像？

  如果同时满足下面的条件，两个树互为镜像：

  - 它们的两个根结点具有相同的值
  - 每个树的右子树都与另一个树的左子树镜像对称

  ![fig2](https://assets.leetcode-cn.com/solution-static/101/101_fig2.PNG)

  我们可以实现这样一个递归函数，通过「同步移动」两个指针的方法来遍历这棵树，p 指针和 q 指针一开始都指向这棵树的根，随后 p 右移时，q 左移，p 左移时，q 右移。每次检查当前 p 和 q 节点的值是否相等，如果相等再判断左右子树是否对称。

  **代码如下：**

  ```java
  class Solution {
      public boolean isSymmetric(TreeNode root) {
          return check(root.left, root.right);
      }
  
      public boolean check(TreeNode p, TreeNode q) {
          if (p == null && q == null) {
              return true;
          }else if (p == null || q == null) {
              return false;
          }else if (p.val != q.val ){
              return false; 
          }
          return check(p.left, q.right) && check(p.right, q.left); // 就是看左子树和右子树节点是否相同
      }
  }
  ```

  **复杂度分析**

  假设树上一共 n 个节点。

  - 时间复杂度：这里遍历了这棵树，渐进时间复杂度为 O(n)。
  - 空间复杂度：这里的空间复杂度和递归使用的栈空间有关，这里递归层数不超过 n，故渐进空间复杂度为 O(n)。

  **方法二：迭代**

  **思路和算法**

  「方法一」中我们用递归的方法实现了对称性的判断，那么如何用迭代的方法实现呢？首先我们引入一个队列，这是把递归程序改写成迭代程序的常用方法。初始化时我们把根节点入队两次。每次提取两个结点并比较它们的值（队列中每两个连续的结点应该是相等的，而且它们的子树互为镜像），然后将两个结点的左右子结点按相反的顺序插入队列中。当队列为空时，或者我们检测到树不对称（即从队列中取出两个不相等的连续结点）时，该算法结束。

  ```java
  class Solution {
      public boolean isSymmetric(TreeNode root) {
          return check(root, root);
      }
  
      public boolean check(TreeNode u, TreeNode v) {
          Queue<TreeNode> q = new LinkedList<TreeNode>();
          q.offer(u);
          q.offer(v);
          while (!q.isEmpty()) {
              u = q.poll();
              v = q.poll();
              if (u == null && v == null) {
                  continue;
              }
              if ((u == null || v == null) || (u.val != v.val)) {
                  return false;
              }
              // 核心代码
              q.offer(u.left);
              q.offer(v.right);
              q.offer(u.right);
              q.offer(v.left);
          }
          return true;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，同「方法一」。
  - 空间复杂度：这里需要用一个队列来维护节点，每个节点最多进队一次，出队一次，队列中最多不会超过 n 个点，故渐进空间复杂度为 O(n)。



## day27

#### [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)

- 题目

  给定一个二叉树，找出其最大深度。

  二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

  **说明:** 叶子节点是指没有子节点的节点。

  **示例：**
  给定二叉树 `[3,9,20,null,null,15,7]`，

  ```txt
      3
     / \
    9  20
      /  \
     15   7
  ```

  返回它的最大深度 3 。

- 解法

  **方法一：深度优先搜索**

  **思路与算法**

  如果我们知道了左子树和右子树的最大深度 l 和 r，那么该二叉树的最大深度即为`max(l,r)+1`

  而左子树和右子树的最大深度又可以以同样的方式进行计算。因此我们可以用「深度优先搜索」的方法来计算二叉树的最大深度。具体而言，在计算当前二叉树的最大深度时，可以先递归计算出其左子树和右子树的最大深度，然后在 O(1) 时间内计算出当前二叉树的最大深度。递归在访问到空节点时退出。

  ```java
  class Solution {
      public int maxDepth(TreeNode root) {
          if (root == null) {
              return 0;
          } else {
              int leftHeight = maxDepth(root.left);
              int rightHeight = maxDepth(root.right);
              return Math.max(leftHeight, rightHeight) + 1;
          }
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 为二叉树节点的个数。每个节点在递归中只被遍历一次。
  - 空间复杂度：O(height)，其中 height 表示二叉树的高度。递归函数需要栈空间，而栈空间取决于递归的深度，因此空间复杂度等价于二叉树的高度。

  **方法二：广度优先搜索**

  **思路与算法**

  我们也可以用「广度优先搜索」的方法来解决这道题目，但我们需要对其进行一些修改，此时我们广度优先搜索的队列里存放的是「当前层的所有节点」。每次拓展下一层的时候，不同于广度优先搜索的每次只从队列里拿出一个节点，我们需要将队列里的所有节点都拿出来进行拓展，这样能保证每次拓展完的时候队列里存放的是当前层的所有节点，即我们是一层一层地进行拓展，最后我们用一个变量 ans 来维护拓展的次数，该二叉树的最大深度即为 ans。

  ```java
  class Solution {
      public int maxDepth(TreeNode root) {
          if (root == null) {
              return 0;
          }
          Queue<TreeNode> queue = new LinkedList<TreeNode>();
          queue.offer(root);
          int ans = 0;
          while (!queue.isEmpty()) {
              int size = queue.size();
              while (size > 0) {
                  TreeNode node = queue.poll();
                  if (node.left != null) {
                      queue.offer(node.left);
                  }
                  if (node.right != null) {
                      queue.offer(node.right);
                  }
                  size--;
              }
              ans++;
          }
          return ans;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 为二叉树的节点个数。与方法一同样的分析，每个节点只会被访问一次。
  - 空间复杂度：此方法空间的消耗取决于队列存储的元素数量，其在最坏情况下会达到 O(n)。
