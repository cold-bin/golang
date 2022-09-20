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

## day28

#### [111. 二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)

- 题目

  给定一个二叉树，找出其最小深度。

  最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

  **说明：**叶子节点是指没有子节点的节点。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2020/10/12/ex_depth.jpg)

  ```
  输入：root = [3,9,20,null,null,15,7]
  输出：2
  ```

  **示例 2：**

  ```
  输入：root = [2,null,3,null,4,null,5,null,6]
  输出：5
  ```

   

  **提示：**

  - 树中节点数的范围在 `[0, 105]` 内
  - `-1000 <= Node.val <= 1000`

- 解法

  **方法一：深度优先搜索**

  **思路及解法**

  首先可以想到使用深度优先搜索的方法，遍历整棵树，记录最小深度。

  对于每一个非叶子节点，我们只需要分别计算其左右子树的最小叶子节点深度。这样就将一个大问题转化为了小问题，可以递归地解决该问题。

  **代码**

  ```java
  class Solution {
      public int minDepth(TreeNode root) {
          if (root==null) return 0;
          int l=Integer.MAX_VALUE;
          int r=Integer.MAX_VALUE;
          if (root.left==null&&root.right==null) return 1;
          if (root.left!=null){
              l=minDepth(root.left)+1;
          }
          if(root.right!=null){
              // System.out.println(root.val);
              r=minDepth(root.right)+1;
          }
          return l>r?r:l;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N)，其中 N 是树的节点数。对每个节点访问一次。
  - 空间复杂度：O(H)，其中 H 是树的高度。空间复杂度主要取决于递归时栈空间的开销，最坏情况下，树呈现链状，空间复杂度为 O(N)。平均情况下树的高度与节点数的对数正相关，空间复杂度为 O(log N)。

  **方法二：广度优先搜索**

  **思路及解法**

  同样，我们可以想到使用广度优先搜索的方法，遍历整棵树。

  当我们找到一个叶子节点时，直接返回这个叶子节点的深度。广度优先搜索的性质保证了最先搜索到的叶子节点的深度一定最小。

  **代码**

  ```java
  class Solution {
      public int minDepth(TreeNode root) {
          if (root==null) return 0;
          if (root.left==null&&root.right==null) return 1;
  
          int layerNum=1;
          Queue<TreeNode> queue=new LinkedList<>();
          queue.offer(root);
          while(!queue.isEmpty()){
              int size = queue.size();
              while(size>0){
                  TreeNode node = queue.poll();
                  if(node.left==null&&node.right==null) return layerNum;
                  if(node.left!=null) queue.offer(node.left);
                  if(node.right!=null) queue.offer(node.right);
                  size--;
              }
              layerNum++;
          }
          return layerNum;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N)，其中 N 是树的节点数。对每个节点访问一次。
  - 空间复杂度：O(N)，其中 N 是树的节点数。空间复杂度主要取决于队列的开销，队列中的元素个数不会超过树的节点数。

## day29

#### [112. 路径总和](https://leetcode.cn/problems/path-sum/)

- 题目

  给你二叉树的根节点 `root` 和一个表示目标和的整数 `targetSum` 。判断该树中是否存在 **根节点到叶子节点** 的路径，这条路径上所有节点值相加等于目标和 `targetSum` 。如果存在，返回 `true` ；否则，返回 `false` 。

  **叶子节点** 是指没有子节点的节点。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2021/01/18/pathsum1.jpg)

  ```
  输入：root = [5,4,8,11,null,13,4,7,2,null,null,null,1], targetSum = 22
  输出：true
  解释：等于目标和的根节点到叶节点路径如上图所示。
  ```

  **示例 2：**

  ![img](https://assets.leetcode.com/uploads/2021/01/18/pathsum2.jpg)

  ```
  输入：root = [1,2,3], targetSum = 5
  输出：false
  解释：树中存在两条根节点到叶子节点的路径：
  (1 --> 2): 和为 3
  (1 --> 3): 和为 4
  不存在 sum = 5 的根节点到叶子节点的路径。
  ```

  **示例 3：**

  ```
  输入：root = [], targetSum = 0
  输出：false
  解释：由于树是空的，所以不存在根节点到叶子节点的路径。
  ```

   

  **提示：**

  - 树中节点的数目在范围 `[0, 5000]` 内
  - `-1000 <= Node.val <= 1000`
  - `-1000 <= targetSum <= 1000`

- 解法

  **写在前面**

  注意到本题的要求是，询问是否有从「根节点」到某个「叶子节点」经过的路径上的节点之和等于目标和。核心思想是对树进行一次遍历，在遍历时记录从根节点到当前节点的路径和，以防止重复计算。

  > 需要特别注意的是，给定的 `root` 可能为空。

  **方法一：广度优先搜索**

  **思路及算法**

  首先我们可以想到使用广度优先搜索的方式，记录从根节点到当前节点的路径和，以防止重复计算。

  这样我们使用两个队列，分别存储将要遍历的节点，以及根节点到这些节点的路径和即可。

  **代码**

  ```java
  class Solution {
      public boolean hasPathSum(TreeNode root, int sum) {
          if (root == null) {
              return false;
          }
          Queue<TreeNode> queNode = new LinkedList<>();
          Queue<Integer> queVal = new LinkedList<>();
          queNode.offer(root);
          queVal.offer(root.val);
          while (!queNode.isEmpty()) {
              TreeNode now = queNode.poll();
              int temp = queVal.poll();
              if (now.left == null && now.right == null) {
                  if (temp == sum) {
                      return true;
                  }
                  continue;
              }
              if (now.left != null) {
                  queNode.offer(now.left);
                  queVal.offer(now.left.val + temp);
              }
              if (now.right != null) {
                  queNode.offer(now.right);
                  queVal.offer(now.right.val + temp);
              }
          }
          return false;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N)，其中 N 是树的节点数。对每个节点访问一次。
  - 空间复杂度：O(N)，其中 N 是树的节点数。空间复杂度主要取决于队列的开销，队列中的元素个数不会超过树的节点数。

  ##### **方法二：递归**

  **思路及算法**

  观察要求我们完成的函数，我们可以归纳出它的功能：询问是否存在从当前节点 `root` 到叶子节点的路径，满足其路径和为 `sum`。

  假定从根节点到当前节点的值之和为 `val`，我们可以将这个大问题转化为一个小问题：是否存在从当前节点的子节点到叶子的路径，满足其路径和为 `sum - val`。

  不难发现这满足递归的性质，若当前节点就是叶子节点，那么我们直接判断 `sum` 是否等于 `val` 即可（因为路径和已经确定，就是当前节点的值，我们只需要判断该路径和是否满足条件）。若当前节点不是叶子节点，我们只需要递归地询问它的子节点是否能满足条件即可。

  **代码**

  ```java
  class Solution {
      public boolean hasPathSum(TreeNode root, int sum) {
          if (root == null) {
              return false;
          }
          if (root.left == null && root.right == null) {
              return sum == root.val;
          }
          return hasPathSum(root.left, sum - root.val) || hasPathSum(root.right, sum - root.val);
      }
  }
  // 或者：正向思维，遍历二叉树，将到达叶子节点的路径和累加与 targetSum 比较，如果相等就返回true
  class Solution {
      public boolean hasPathSum(TreeNode root, int targetSum) {
          return doSome(root,targetSum,0);
      }
  
      public boolean doSome(TreeNode root,int targetSum,int tmp){
          if (root==null) return false;
          // 叶子节点
          if(root.left==null&&root.right==null) {
              if(tmp+root.val==targetSum){
                  return true;
              } else {
                  tmp=0;
                  return false;
              }
          }
          if(root.left!=null) {
              boolean l =doSome(root.left,targetSum,tmp+root.val);
              if (l) return true;
          }
          if(root.right!=null) {
              boolean r = doSome(root.right,targetSum,tmp+root.val);
              if (r) return true;
          }
          return false;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N)，其中 N 是树的节点数。对每个节点访问一次。
  - 空间复杂度：O(H)，其中 H 是树的高度。空间复杂度主要取决于递归时栈空间的开销，最坏情况下，树呈现链状，空间复杂度为 O(N)。平均情况下树的高度与节点数的对数正相关，空间复杂度为 O(log N)。

## day30

#### [222. 完全二叉树的节点个数](https://leetcode.cn/problems/count-complete-tree-nodes/)

- 题目

  给你一棵 **完全二叉树** 的根节点 `root` ，求出该树的节点个数。

  [完全二叉树](https://baike.baidu.com/item/完全二叉树/7773232?fr=aladdin) 的定义如下：在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，并且最下面一层的节点都集中在该层最左边的若干位置。若最底层为第 `h` 层，则该层包含 `1~ 2h` 个节点。

   

  **示例 1：**

  ![fig1](https://assets.leetcode-cn.com/solution-static/222/1.png)

  ```
  输入：root = [1,2,3,4,5,6]
  输出：6
  ```

  **示例 2：**

  ```
  输入：root = []
  输出：0
  ```

  **示例 3：**

  ```
  输入：root = [1]
  输出：1
  ```

   

  **提示：**

  - 树中节点的数目范围是`[0, 5 * 104]`
  - `0 <= Node.val <= 5 * 104`
  - 题目数据保证输入的树是 **完全二叉树**

   

  **进阶：**遍历树来统计节点是一种时间复杂度为 `O(n)` 的简单解决方案。你可以设计一个更快的算法吗？

- 解法

  **方法一：深度优先遍历**

  ```java
  class Solution {
      public int countNodes(TreeNode root) {
      	if(root == null) {
      		return 0;
      	}
      	int left = countNodes(root.left);
      	int right = countNodes(root.right);
      	
      	return left+right+1;
      	
      }
  }
  ```

  **方法二：广度优先遍历**

  一如既往，广度优先貌似硬是要比深度优先慢一些

  ```java
  class Solution {
      public int countNodes(TreeNode root) {
          if (root==null) return 0;
          Queue<TreeNode> queue=new LinkedList<>();
          int count=0;
          queue.offer(root);
  
          while(!queue.isEmpty()){
              int size = queue.size();
              while(size>0){
                  TreeNode n = queue.poll();
                  if (n!=null) count++;
                  if (n.left!=null) queue.offer(n.left);
                  if (n.right!=null) queue.offer(n.right);
                  size--;
              }
          }
          return count;
      }
  }
  ```

  **方法三：二分查找 + 位运算**

  对于任意二叉树，都可以通过**广度优先搜索**或**深度优先搜索**计算节点个数，时间复杂度和空间复杂度都是 O(n)，其中 n 是二叉树的节点个数。这道题规定了给出的是完全二叉树，因此可以利用完全二叉树的特性计算节点个数。

  规定根节点位于第 0 层，完全二叉树的最大层数为 h。根据完全二叉树的特性可知，完全二叉树的最左边的节点一定位于最底层，因此从根节点出发，每次访问左子节点，直到遇到叶子节点，该叶子节点即为完全二叉树的最左边的节点，经过的路径长度即为最大层数 h。

  当 0≤i<h 时，第 i 层包含 2^i^ 个节点，最底层包含的节点数最少为 1，最多为 2^h^。

  当最底层包含 1 个节点时，完全二叉树的节点个数是
  $$
  \sum_{i=0}^{h-1}2^i+1=2^h
  $$
  当最底层包含 2^h^ 个节点时，完全二叉树的节点个数是
  $$
  \sum_{i=0}^{h}2^i=2^{h+1}-1
  $$
  因此对于最大层数为 h 的完全二叉树，节点个数一定在 [2^h^,2^h+1^-1] 的范围内，可以在该范围内通过二分查找的方式得到完全二叉树的节点个数。

  具体做法是，根据节点个数范围的上下界得到当前需要判断的节点个数 k，如果第 k 个节点存在，则节点个数一定**大于或等于** k，如果第 k 个节点不存在，则节点个数一定**小于** k，由此可以将查找的范围缩小一半，直到得到节点个数。

  如何判断第 k 个节点是否存在呢？如果第 k 个节点位于第 h 层，则 k 的二进制表示包含 h+1 位，其中最高位是 1，其余各位从高到低表示从根节点到第 k 个节点的路径，0 表示移动到左子节点，1 表示移动到右子节点。通过位运算得到第 k 个节点对应的路径，判断该路径对应的节点是否存在，即可判断第 k 个节点是否存在。

  ![fig1](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202208292012217.png)

  ```java
  class Solution {
      public int countNodes(TreeNode root) {
          if (root == null) {
              return 0;
          }
          int level = 0;
          TreeNode node = root;
          while (node.left != null) {
              level++;
              node = node.left;
          }
          int low = 1 << level, high = (1 << (level + 1)) - 1;
          while (low < high) {
              int mid = (high - low + 1) / 2 + low;
              if (exists(root, level, mid)) {
                  low = mid;
              } else {
                  high = mid - 1;
              }
          }
          return low;
      }
  
      public boolean exists(TreeNode root, int level, int k) {
          int bits = 1 << (level - 1);
          TreeNode node = root;
          while (node != null && bits > 0) {
              if ((bits & k) == 0) {
                  node = node.left;
              } else {
                  node = node.right;
              }
              bits >>= 1;
          }
          return node != null;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(log^2^n)，其中 n 是完全二叉树的节点数。
    首先需要 O(h) 的时间得到完全二叉树的最大层数，其中 h 是完全二叉树的最大层数。
    使用二分查找确定节点个数时，需要查找的次数为 O(log 2^h^)=O(h)，每次查找需要遍历从根节点开始的一条长度为 h 的路径，需要 O(h) 的时间，因此二分查找的总时间复杂度是 O(h^2^)。
    因此总时间复杂度是 O(h^2^)。由于完全二叉树满足 2^h^≤n<2^h+1^，因此有 O(h)=O(log n)，O(h^2^)=O(log^2^ n)。
  - 空间复杂度：O(1)。只需要维护有限的额外空间。

## day31

#### [226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)

- 题目

  给你一棵二叉树的根节点 `root` ，翻转这棵二叉树，并返回其根节点。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2021/03/14/invert1-tree.jpg)

  ```
  输入：root = [4,2,7,1,3,6,9]
  输出：[4,7,2,9,6,3,1]
  ```

  **示例 2：**

  ![img](https://assets.leetcode.com/uploads/2021/03/14/invert2-tree.jpg)

  ```
  输入：root = [2,1,3]
  输出：[2,3,1]
  ```

  **示例 3：**

  ```
  输入：root = []
  输出：[]
  ```

   

  **提示：**

  - 树中节点数目范围在 `[0, 100]` 内
  - `-100 <= Node.val <= 100`

- 解法

  **方法一：深度优先遍历**

  **思路与算法**

  这是一道很经典的二叉树问题。显然，我们从根节点开始，递归地对树进行遍历，并从叶子节点先开始翻转。如果当前遍历到的节点 root 的左右两棵子树都已经翻转，那么我们只需要交换两棵子树的位置，即可完成以 root 为根节点的整棵子树的翻转。

  **代码**

  ```java
  class Solution {
      public TreeNode invertTree(TreeNode root) {
          if (root == null) {
              return null;
          }
          TreeNode left = invertTree(root.left);
          TreeNode right = invertTree(root.right);
          root.left = right;
          root.right = left;
          return root;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N)，其中 N 为二叉树节点的数目。我们会遍历二叉树中的每一个节点，对每个节点而言，我们在常数时间内交换其两棵子树。
  - 空间复杂度：O(N)。使用的空间由递归栈的深度决定，它等于当前节点在二叉树中的高度。在平均情况下，二叉树的高度与节点个数为对数关系，即 O*(logN)。而在最坏情况下，树形成链状，空间复杂度为 O(N)。

  **方法二：广度优先遍历**

  ```java
      public TreeNode invertTree(TreeNode root) {
          if (root == null)
              return root;
          Queue<TreeNode> queue = new LinkedList<>();
          queue.add(root);//相当于把数据加入到队列尾部
          while (!queue.isEmpty()) {
              //poll方法相当于移除队列头部的元素
              TreeNode node = queue.poll();
              //先交换子节点
              TreeNode left = node.left;
              node.left = node.right;
              node.right = left;
              if (node.left != null)
                  queue.add(node.left);
              if (node.right != null)
                  queue.add(node.right);
          }
          return root;
      }
  ```

  

## day32

#### [236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)

- 题目

  给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

  [百度百科](https://baike.baidu.com/item/最近公共祖先/8918834?fr=aladdin)中最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2018/12/14/binarytree.png)

  ```
  输入：root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
  输出：3
  解释：节点 5 和节点 1 的最近公共祖先是节点 3 。
  ```

  **示例 2：**

  ![img](https://assets.leetcode.com/uploads/2018/12/14/binarytree.png)

  ```
  输入：root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
  输出：5
  解释：节点 5 和节点 4 的最近公共祖先是节点 5 。因为根据定义最近公共祖先节点可以为节点本身。
  ```

  **示例 3：**

  ```
  输入：root = [1,2], p = 1, q = 2
  输出：1
  ```

   

  **提示：**

  - 树中节点数目在范围 `[2, 105]` 内。
  - `-109 <= Node.val <= 109`
  - 所有 `Node.val` `互不相同` 。
  - `p != q`
  - `p` 和 `q` 均存在于给定的二叉树中。

- 解法

  **方法零：两层递归**

  ```java
  class Solution {
      public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
          if (root==null) return null;
          
          //左子树是否含有这两个节点，如果有，就继续递归，直到左子树不再同时含有两个节点
          if (doSome(root.left,p)&&doSome(root.left,q)) {
              return lowestCommonAncestor(root.left,p,q);
          }
          if (doSome(root.right,p)&&doSome(root.right,q)){
              return lowestCommonAncestor(root.right,p,q);
          }
          return root;
      }
      // 检查 node子树里是否含有n节点
      public boolean doSome(TreeNode node,TreeNode n){
          if (node==null)return false;
          if (node.val==n.val){
              return true;
          }
          boolean l = doSome(node.left,n);
          boolean r = doSome(node.right,n);
          return l||r;
      }
  }
  ```

  **方法一：递归**

  **思路和算法**

  我们递归遍历整棵二叉树，定义 f~x~ 表示 x 节点的子树中是否包含 p 节点或 q 节点，如果包含为 `true`，否则为 `false`。那么符合条件的最近公共祖先 x 一定满足如下条件：**(f^lson^ && f^rson^) ∣∣ ((x = p ∣∣ x = q) && (f^lson^ ∣∣ f^rson^))**

  其中 `lson` 和 `rson` 分别代表 x 节点的左孩子和右孩子。初看可能会感觉条件判断有点复杂，我们来一条条看，**f^lson^ && f^rson^`** 说明左子树和右子树均包含 p 节点或 q  节点，如果左子树包含的是 p 节点，那么右子树只能包含 q 节点，反之亦然，因为 p 节点和 q 节点都是不同且唯一的节点，因此如果满足这个判断条件即可说明 x 就是我们要找的最近公共祖先。再来看第二条判断条件，这个判断条件即是考虑了 x 恰好是 p 节点或 q 节点且它的左子树或右子树有一个包含了另一个节点的情况，因此如果满足这个判断条件亦可说明 x 就是我们要找的最近公共祖先。

  你可能会疑惑这样找出来的公共祖先深度是否是最大的。其实是最大的，因为我们是自底向上从叶子节点开始更新的，所以在所有满足条件的公共祖先中一定是深度最大的祖先先被访问到，且由于 f~x~ 本身的定义很巧妙，在找到最近公共祖先 x 以后，f~x~ 按定义被设置为 `true` ，即假定了这个子树中只有一个 p 节点或 q 节点，因此其他公共祖先不会再被判断为符合条件。

  ```java
  class Solution {
  
      private TreeNode ans;
  
      public Solution() {
          this.ans = null;
      }
  
      private boolean dfs(TreeNode root, TreeNode p, TreeNode q) {
          if (root == null) return false;
          boolean lson = dfs(root.left, p, q);
          boolean rson = dfs(root.right, p, q);
          if ((lson && rson) || ((root.val == p.val || root.val == q.val) && (lson || rson))) {
              ans = root;
          } 
          return lson || rson || (root.val == p.val || root.val == q.val);
      }
  
      public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
          this.dfs(root, p, q);
          return this.ans;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N) ，其中 N 是二叉树的节点数。二叉树的所有节点有且只会被访问一次，因此时间复杂度为 O(N)。
  - 空间复杂度：O(N) ，其中 N 是二叉树的节点数。递归调用的栈深度取决于二叉树的高度，二叉树最坏情况下为一条链，此时高度为 N ，因此空间复杂度为 O(N)。

  **方法二：存储父节点**

  **思路**

  我们可以用哈希表存储所有节点的父节点，然后我们就可以利用节点的父节点信息从 `p` 结点开始不断往上跳，并记录已经访问过的节点，再从 `q` 节点开始不断往上跳，如果碰到已经访问过的节点，那么这个节点就是我们要找的最近公共祖先。

  **算法**

  1. 从根节点开始遍历整棵二叉树，用哈希表记录每个节点的父节点指针。
  2. 从 `p` 节点开始不断往它的祖先移动，并用数据结构记录已经访问过的祖先节点。
  3. 同样，我们再从 `q` 节点开始不断往它的祖先移动，如果有祖先已经被访问过，即意味着这是 `p` 和 `q` 的深度最深的公共祖先，即 LCA 节点。

  ```java
  class Solution {
      Map<Integer, TreeNode> parent = new HashMap<Integer, TreeNode>();
      Set<Integer> visited = new HashSet<Integer>();
  
      public void dfs(TreeNode root) {
          if (root.left != null) {
              parent.put(root.left.val, root);
              dfs(root.left);
          }
          if (root.right != null) {
              parent.put(root.right.val, root);
              dfs(root.right);
          }
      }
  
      public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
          dfs(root);
          while (p != null) {
              visited.add(p.val);
              p = parent.get(p.val);
          }
          while (q != null) {
              if (visited.contains(q.val)) {
                  return q;
              }
              q = parent.get(q.val);
          }
          return null;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N)，其中 N 是二叉树的节点数。二叉树的所有节点有且只会被访问一次，从 `p` 和 `q` 节点往上跳经过的祖先节点个数不会超过 N，因此总的时间复杂度为 O(N)。
  - 空间复杂度：O(N) ，其中 N 是二叉树的节点数。递归调用的栈深度取决于二叉树的高度，二叉树最坏情况下为一条链，此时高度为 N，因此空间复杂度为 O(N)，哈希表存储每个节点的父节点也需要 O(N) 的空间复杂度，因此最后总的空间复杂度为 O(N)。

## day33

#### [257. 二叉树的所有路径](https://leetcode.cn/problems/binary-tree-paths/)

- 题目

  给你一个二叉树的根节点 `root` ，按 **任意顺序** ，返回所有从根节点到叶子节点的路径。

  **叶子节点** 是指没有子节点的节点。

  

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2021/03/12/paths-tree.jpg)

  ```
  输入：root = [1,2,3,null,5]
  输出：["1->2->5","1->3"]
  ```

  **示例 2：**

  ```
  输入：root = [1]
  输出：["1"]
  ```

   

  **提示：**

  - 树中节点的数目在范围 `[1, 100]` 内
  - `-100 <= Node.val <= 100`

- 解法

  **方法一：深度优先搜索**

  **思路与算法**

  最直观的方法是使用深度优先搜索。在深度优先搜索遍历二叉树时，我们需要考虑当前的节点以及它的孩子节点。

  - 如果当前节点**不是叶子节点**，则在当前的路径末尾添加该节点，并继续递归遍历该节点的每一个孩子节点。
  - 如果当前节点**是叶子节点**，则在当前路径末尾添加该节点后我们就得到了一条从根节点到叶子节点的路径，将该路径加入到答案即可。

  如此，当遍历完整棵二叉树以后我们就得到了所有从根节点到叶子节点的路径。当然，深度优先搜索也可以使用非递归的方式实现，这里不再赘述。

  **代码**

  ```java
  class Solution {
      public List<String> binaryTreePaths(TreeNode root) {
          List<String> paths = new ArrayList<String>();
          constructPaths(root, "", paths);
          return paths;
      }
  
      public void constructPaths(TreeNode root, String path, List<String> paths) {
          if (root != null) {
              StringBuffer pathSB = new StringBuffer(path);
              pathSB.append(Integer.toString(root.val));
              if (root.left == null && root.right == null) {  // 当前节点是叶子节点
                  paths.add(pathSB.toString());  // 把路径加入到答案中
              } else {
                  pathSB.append("->");  // 当前节点不是叶子节点，继续递归遍历
                  constructPaths(root.left, pathSB.toString(), paths);
                  constructPaths(root.right, pathSB.toString(), paths);
              }
          }
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N^2^)，其中 N 表示节点数目。在深度优先搜索中每个节点会被访问一次且只会被访问一次，每一次会对 `path` 变量进行拷贝构造，时间代价为 O(N) ，故时间复杂度为 O(N^2^)。

  - 空间复杂度：O(N^2^)，其中 N 表示节点数目。除答案数组外我们需要考虑递归调用的栈空间。在最坏情况下，当二叉树中每个节点只有一个孩子节点时，即整棵二叉树呈一个链状，此时递归的层数为 N，此时每一层的 `path` 变量的空间代价的总和为
    $$
    O(\sum_{i = 1}^{N} i) = O(N^2)
    $$
     空间复杂度为 O(N^2^)。最好情况下，当二叉树为平衡二叉树时，它的高度为 log N ，此时空间复杂度为 
    $$
    O((\log {N})^2)
    $$
    。

  **方法二：广度优先搜索**

  **思路与算法**

  我们也可以用广度优先搜索来实现。我们维护一个队列，存储节点以及根到该节点的路径。一开始这个队列里只有根节点。在每一步迭代中，我们取出队列中的首节点，如果它**是叶子节点**，则将它对应的路径加入到答案中。如果它**不是叶子节点**，则将它的所有孩子节点加入到队列的末尾。当队列为空时广度优先搜索结束，我们即能得到答案。

  **代码**

  ```java
  class Solution {
      public List<String> binaryTreePaths(TreeNode root) {
          List<String> paths = new ArrayList<String>();
          if (root == null) {
              return paths;
          }
          Queue<TreeNode> nodeQueue = new LinkedList<TreeNode>();
          Queue<String> pathQueue = new LinkedList<String>();
  
          nodeQueue.offer(root);
          pathQueue.offer(Integer.toString(root.val));
  
          while (!nodeQueue.isEmpty()) {
              TreeNode node = nodeQueue.poll(); 
              String path = pathQueue.poll();
  
              if (node.left == null && node.right == null) {
                  paths.add(path);
              } else {
                  if (node.left != null) {
                      nodeQueue.offer(node.left);
                      pathQueue.offer(new StringBuffer(path).append("->").append(node.left.val).toString());
                  }
  
                  if (node.right != null) {
                      nodeQueue.offer(node.right);
                      pathQueue.offer(new StringBuffer(path).append("->").append(node.right.val).toString());
                  }
              }
          }
          return paths;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(N^2^)，其中 N 表示节点数目。分析同方法一。
  - 空间复杂度：O(N^2^)，其中 N 表示节点数目。在最坏情况下，队列中会存在 N 个节点，保存字符串的队列中每个节点的最大长度为 N，故空间复杂度为 O(N^2^)。

## day34

#### [404. 左叶子之和](https://leetcode.cn/problems/sum-of-left-leaves/)

- 题目

  给定二叉树的根节点 `root` ，返回所有左叶子之和。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2021/04/08/leftsum-tree.jpg)

  ```
  输入: root = [3,9,20,null,null,15,7] 
  输出: 24 
  解释: 在这个二叉树中，有两个左叶子，分别是 9 和 15，所以返回 24
  ```

  **示例 2:**

  ```
  输入: root = [1]
  输出: 0
  ```

   

  **提示:**

  - 节点数在 `[1, 1000]` 范围内
  - `-1000 <= Node.val <= 1000`

- 解法

  一个节点为「左叶子」节点，当且仅当它是某个节点的左子节点，并且它是一个叶子结点。因此我们可以考虑对整棵树进行遍历，当我们遍历到节点 node 时，如果它的左子节点是一个叶子结点，那么就将它的左子节点的值累加计入答案。

  遍历整棵树的方法有深度优先搜索和广度优先搜索，下面分别给出了实现代码。

  **方法一：深度优先搜索**

  ```java
  class Solution {
      public int sumOfLeftLeaves(TreeNode root) {
          return root != null ? dfs(root) : 0;
      }
  
      public int dfs(TreeNode node) {
          int ans = 0;
          if (node.left != null) {
              ans += isLeafNode(node.left) ? node.left.val : dfs(node.left);
          }
          if (node.right != null && !isLeafNode(node.right)) {
              ans += dfs(node.right);
          }
          return ans;
      }
  
      public boolean isLeafNode(TreeNode node) {
          return node.left == null && node.right == null;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是树中的节点个数。
  - 空间复杂度：O(n)。空间复杂度与深度优先搜索使用的栈的最大深度相关。在最坏的情况下，树呈现链式结构，深度为 O(n)，对应的空间复杂度也为 O(n)。

  **方法二：广度优先搜索**

  ```java
  class Solution {
      public int sumOfLeftLeaves(TreeNode root) {
          if (root == null) {
              return 0;
          }
  
          Queue<TreeNode> queue = new LinkedList<TreeNode>();
          queue.offer(root);
          int ans = 0;
          while (!queue.isEmpty()) {
              TreeNode node = queue.poll();
              if (node.left != null) {
                  if (isLeafNode(node.left)) {
                      ans += node.left.val;
                  } else {
                      queue.offer(node.left);
                  }
              }
              if (node.right != null) {
                  if (!isLeafNode(node.right)) {
                      queue.offer(node.right);
                  }
              }
          }
          return ans;
      }
  
      public boolean isLeafNode(TreeNode node) {
          return node.left == null && node.right == null;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n) ，其中 n 是树中的节点个数。
  - 空间复杂度：O(n) 。空间复杂度与广度优先搜索使用的队列需要的容量相关，为 O(n)。

## day35

#### [513. 找树左下角的值](https://leetcode.cn/problems/find-bottom-left-tree-value/)

- 题目

  给定一个二叉树的 **根节点** `root`，请找出该二叉树的 **最底层 最左边** 节点的值。

  假设二叉树中至少有一个节点。

   

  **示例 1:**

  ![img](https://assets.leetcode.com/uploads/2020/12/14/tree1.jpg)

  ```
  输入: root = [2,1,3]
  输出: 1
  ```

  **示例 2:**

  ![img](https://assets.leetcode.com/uploads/2020/12/14/tree2.jpg)

  ```
  输入: [1,2,3,4,null,5,6,null,null,7]
  输出: 7
  ```

   

  **提示:**

  - 二叉树的节点个数的范围是 `[1,104]`
  - `-231 <= Node.val <= 231 - 1` 

- 解法

  **方法一：深度优先搜索**

  使用 `height` 记录遍历到的节点的高度，`curVal` 记录高度在 `curHeight` 的最左节点的值。在深度优先搜索时，我们先搜索当前节点的左子节点，再搜索当前节点的右子节点，然后判断当前节点的高度 `height` 是否大于 `curHeight`，如果是，那么将 `curVal` 设置为当前结点的值,`curHeight` 设置为 `height`。

  > 因为我们先遍历左子树，然后再遍历右子树，所以对同一高度的所有节点，最左节点肯定是最先被遍历到的。

  ```java
  class Solution {
      int curVal = 0;
      int curHeight = 0;
  
      public int findBottomLeftValue(TreeNode root) {
          int curHeight = 0;
          dfs(root, 0);
          return curVal;
      }
  
      public void dfs(TreeNode root, int height) {
          if (root == null) {
              return;
          }
          height++;
          dfs(root.left, height);
          dfs(root.right, height);
          // 相当于后续遍历麻，最先遍历的一定是左子节点，然后是右、根子节点
          if (height > curHeight) {
              curHeight = height;
              curVal = root.val;
          }
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是二叉树的节点数目。需要遍历 n 个节点。
  - 空间复杂度：O(n)。递归栈需要占用 O(n) 的空间。

  **方法二：广度优先搜索**

  使用广度优先搜索遍历每一层的节点。在遍历一个节点时，需要先把它的非空右子节点放入队列，然后再把它的非空左子节点放入队列，这样才能保证从右到左遍历每一层的节点。广度优先搜索所遍历的最后一个节点的值就是最底层最左边节点的值。

  ```java
  class Solution {
      public int findBottomLeftValue(TreeNode root) {
          int ret = 0;
          Queue<TreeNode> queue = new ArrayDeque<TreeNode>();
          queue.offer(root);
          while (!queue.isEmpty()) {
              TreeNode p = queue.poll();
              if (p.right != null) {
                  queue.offer(p.right);
              }
              if (p.left != null) {
                  queue.offer(p.left);
              }
              ret = p.val;
          }
          return ret;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是二叉树的节点数目。
  - 空间复杂度：O(n)。如果二叉树是满完全二叉树，那么队列 q 最多保存 `2/n` 个节点。

## day36

#### [559. N 叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-n-ary-tree/)

- 题目

  给定一个 N 叉树，找到其最大深度。

  最大深度是指从根节点到最远叶子节点的最长路径上的节点总数。

  N 叉树输入按层序遍历序列化表示，每组子节点由空值分隔（请参见示例）。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2018/10/12/narytreeexample.png)

  ```
  输入：root = [1,null,3,2,4,null,5,6]
  输出：3
  ```

  **示例 2：**

  ![img](https://assets.leetcode.com/uploads/2019/11/08/sample_4_964.png)

  ```
  输入：root = [1,null,2,3,4,5,null,null,6,7,null,8,null,9,10,null,null,11,null,12,null,13,null,null,14]
  输出：5
  ```

   

  **提示：**

  - 树的深度不会超过 `1000` 。
  - 树的节点数目位于 `[0, 104]` 之间。

- 解法

  **方法一：深度优先搜索**

  如果根节点有 N 个子节点，则这 N 个子节点对应 N 个子树。记 N 个子树的最大深度中的最大值为 maxChildDepth，则该 N 叉树的最大深度为 maxChildDepth+1。

  每个子树的最大深度又可以以同样的方式进行计算。因此我们可以用「深度优先搜索」的方法计算 N 叉树的最大深度。具体而言，在计算当前 N 叉树的最大深度时，可以先递归计算出其每个子树的最大深度，然后在 O(1) 的时间内计算出当前 N 叉树的最大深度。递归在访问到空节点时退出。

  ```java
  class Solution {
      public int maxDepth(Node root) {
          if (root == null) {
              return 0;
          }
          int maxChildDepth = 0;
          List<Node> children = root.children;
          for (Node child : children) {
              int childDepth = maxDepth(child);
              maxChildDepth = Math.max(maxChildDepth, childDepth);
          }
          return maxChildDepth + 1;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 为 N 叉树节点的个数。每个节点在递归中只被遍历一次。
  - 空间复杂度：O(height)，其中 height 表示 N 叉树的高度。递归函数需要栈空间，而栈空间取决于递归的深度，因此空间复杂度等价于 N 叉树的高度。

  **方法二：广度优先搜索**

  我们也可以用「广度优先搜索」的方法来解决这道题目，但我们需要对其进行一些修改，此时我们广度优先搜索的队列里存放的是「当前层的所有节点」。每次拓展下一层的时候，不同于广度优先搜索的每次只从队列里拿出一个节点，我们需要将队列里的所有节点都拿出来进行拓展，这样能保证每次拓展完的时候队列里存放的是当前层的所有节点，即我们是一层一层地进行拓展。最后我们用一个变量 ans 来维护拓展的次数，该 N 叉树的最大深度即为 ans。

  ```java
  class Solution {
      public int maxDepth(Node root) {
          if (root == null) {
              return 0;
          }
          Queue<Node> queue = new LinkedList<Node>();
          queue.offer(root);
          int ans = 0;
          while (!queue.isEmpty()) {
              int size = queue.size();
              while (size > 0) {
                  Node node = queue.poll();
                  List<Node> children = node.children;
                  for (Node child : children) {
                      queue.offer(child);
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

  - 时间复杂度：O(n)，其中 n 为 N 叉树的节点个数。与方法一同样的分析，每个节点只会被访问一次。
  - 空间复杂度：此方法空间的消耗取决于队列存储的元素数量，其在最坏情况下会达到 O(n)。

## day37

#### [617. 合并二叉树](https://leetcode.cn/problems/merge-two-binary-trees/)

- 题目

  给你两棵二叉树： `root1` 和 `root2` 。

  想象一下，当你将其中一棵覆盖到另一棵之上时，两棵树上的一些节点将会重叠（而另一些不会）。你需要将这两棵树合并成一棵新二叉树。合并的规则是：如果两个节点重叠，那么将这两个节点的值相加作为合并后节点的新值；否则，**不为** null 的节点将直接作为新二叉树的节点。

  返回合并后的二叉树。

  **注意:** 合并过程必须从两个树的根节点开始。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2021/02/05/merge.jpg)

  ```
  输入：root1 = [1,3,2,5], root2 = [2,1,3,null,4,null,7]
  输出：[3,4,5,5,4,null,7]
  ```

  **示例 2：**

  ```
  输入：root1 = [1], root2 = [1,2]
  输出：[2,2]
  ```

   

  **提示：**

  - 两棵树中的节点数目在范围 `[0, 2000]` 内
  - `-104 <= Node.val <= 104`

- 解法

  **方法一：深度优先搜索**

  可以使用深度优先搜索合并两个二叉树。从根节点开始同时遍历两个二叉树，并将对应的节点进行合并。

  两个二叉树的对应节点可能存在以下三种情况，对于每种情况使用不同的合并方式。

  - 如果两个二叉树的对应节点都为空，则合并后的二叉树的对应节点也为空；
  - 如果两个二叉树的对应节点只有一个为空，则合并后的二叉树的对应节点为其中的非空节点；
  - 如果两个二叉树的对应节点都不为空，则合并后的二叉树的对应节点的值为两个二叉树的对应节点的值之和，此时需要显性合并两个节点。

  对一个节点进行合并之后，还要对该节点的左右子树分别进行合并。这是一个递归的过程。

  ```java
  class Solution {
      public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
          TreeNode node; // 定义一个局部变量用于递归时，本层节点记录
          if(root1==null&&root2==null){// 当两个根节点都是null，直接返回null
              return null;
          }else if(root1==null){// 当两个根节点有一个为空时，也直接返回非空节点，没必要加
              return root2;
          }else if(root2==null){// 当两个根节点有一个为空时，也直接返回非空节点，没必要加
              return root1;
          }else{// 当两个根节点都不为空的时候，先记录合并后的根节点
              node = new TreeNode(root1.val+root2.val);
          }
          // 然后再递归的遍历左右子树
          TreeNode left = mergeTrees(root1.left,root2.left);
          TreeNode right = mergeTrees(root1.right,root2.right);
          // 将左右子树的结果挂在新的根节点上
          node.left = left;
          node.right = right;
          return node;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(min(m,n))，其中 m 和 n 分别是两个二叉树的节点个数。对两个二叉树同时进行深度优先搜索，只有当两个二叉树中的对应节点都不为空时才会对该节点进行显性合并操作，因此被访问到的节点数不会超过较小的二叉树的节点数。
  - 空间复杂度：O(min(m,n))，其中 m 和 n 分别是两个二叉树的节点个数。空间复杂度取决于递归调用的层数，递归调用的层数不会超过较小的二叉树的最大高度，最坏情况下，二叉树的高度等于节点数。

  **方法二：广度优先搜索**

  也可以使用广度优先搜索合并两个二叉树。首先判断两个二叉树是否为空，如果两个二叉树都为空，则合并后的二叉树也为空，如果只有一个二叉树为空，则合并后的二叉树为另一个非空的二叉树。

  如果两个二叉树都不为空，则首先计算合并后的根节点的值，然后从合并后的二叉树与两个原始二叉树的根节点开始广度优先搜索，从根节点开始同时遍历每个二叉树，并将对应的节点进行合并。

  使用三个队列分别存储合并后的二叉树的节点以及两个原始二叉树的节点。初始时将每个二叉树的根节点分别加入相应的队列。每次从每个队列中取出一个节点，判断两个原始二叉树的节点的左右子节点是否为空。如果两个原始二叉树的当前节点中至少有一个节点的左子节点不为空，则合并后的二叉树的对应节点的左子节点也不为空。对于右子节点同理。

  如果合并后的二叉树的左子节点不为空，则需要根据两个原始二叉树的左子节点计算合并后的二叉树的左子节点以及整个左子树。考虑以下两种情况：

  - 如果两个原始二叉树的左子节点都不为空，则合并后的二叉树的左子节点的值为两个原始二叉树的左子节点的值之和，在创建合并后的二叉树的左子节点之后，将每个二叉树中的左子节点都加入相应的队列；
  - 如果两个原始二叉树的左子节点有一个为空，即有一个原始二叉树的左子树为空，则合并后的二叉树的左子树即为另一个原始二叉树的左子树，此时也不需要对非空左子树继续遍历，因此不需要将左子节点加入队列。

  对于右子节点和右子树，处理方法与左子节点和左子树相同。

  ```java
  class Solution {
      public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
          if (t1 == null) {
              return t2;
          }
          if (t2 == null) {
              return t1;
          }
          TreeNode merged = new TreeNode(t1.val + t2.val);
          Queue<TreeNode> queue = new LinkedList<TreeNode>();
          Queue<TreeNode> queue1 = new LinkedList<TreeNode>();
          Queue<TreeNode> queue2 = new LinkedList<TreeNode>();
          queue.offer(merged);
          queue1.offer(t1);
          queue2.offer(t2);
          while (!queue1.isEmpty() && !queue2.isEmpty()) {
              TreeNode node = queue.poll(), node1 = queue1.poll(), node2 = queue2.poll();
              TreeNode left1 = node1.left, left2 = node2.left, right1 = node1.right, right2 = node2.right;
              if (left1 != null || left2 != null) {
                  if (left1 != null && left2 != null) {
                      TreeNode left = new TreeNode(left1.val + left2.val);
                      node.left = left;
                      queue.offer(left);
                      queue1.offer(left1);
                      queue2.offer(left2);
                  } else if (left1 != null) {
                      node.left = left1;
                  } else if (left2 != null) {
                      node.left = left2;
                  }
              }
              if (right1 != null || right2 != null) {
                  if (right1 != null && right2 != null) {
                      TreeNode right = new TreeNode(right1.val + right2.val);
                      node.right = right;
                      queue.offer(right);
                      queue1.offer(right1);
                      queue2.offer(right2);
                  } else if (right1 != null) {
                      node.right = right1;
                  } else {
                      node.right = right2;
                  }
              }
          }
          return merged;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(min(m,n))，其中 m 和 n 分别是两个二叉树的节点个数。对两个二叉树同时进行广度优先搜索，只有当两个二叉树中的对应节点都不为空时才会访问到该节点，因此被访问到的节点数不会超过较小的二叉树的节点数。
  - 空间复杂度：O(min(m,n))，其中 m 和 n 分别是两个二叉树的节点个数。空间复杂度取决于队列中的元素个数，队列中的元素个数不会超过较小的二叉树的节点数。

## day38

#### [654. 最大二叉树](https://leetcode.cn/problems/maximum-binary-tree/)

- 题目

  给定一个不重复的整数数组 `nums` 。 **最大二叉树** 可以用下面的算法从 `nums` 递归地构建:

  1. 创建一个根节点，其值为 `nums` 中的最大值。
  2. 递归地在最大值 **左边** 的 **子数组前缀上** 构建左子树。
  3. 递归地在最大值 **右边** 的 **子数组后缀上** 构建右子树。

  返回 *`nums` 构建的* ***最大二叉树\*** 。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2020/12/24/tree1.jpg)

  ```
  输入：nums = [3,2,1,6,0,5]
  输出：[6,3,5,null,2,0,null,null,1]
  解释：递归调用如下所示：
  - [3,2,1,6,0,5] 中的最大值是 6 ，左边部分是 [3,2,1] ，右边部分是 [0,5] 。
      - [3,2,1] 中的最大值是 3 ，左边部分是 [] ，右边部分是 [2,1] 。
          - 空数组，无子节点。
          - [2,1] 中的最大值是 2 ，左边部分是 [] ，右边部分是 [1] 。
              - 空数组，无子节点。
              - 只有一个元素，所以子节点是一个值为 1 的节点。
      - [0,5] 中的最大值是 5 ，左边部分是 [0] ，右边部分是 [] 。
          - 只有一个元素，所以子节点是一个值为 0 的节点。
          - 空数组，无子节点。
  ```

  **示例 2：**

  ![img](https://assets.leetcode.com/uploads/2020/12/24/tree2.jpg)

  ```
  输入：nums = [3,2,1]
  输出：[3,null,2,null,1]
  ```

   

  **提示：**

  - `1 <= nums.length <= 1000`
  - `0 <= nums[i] <= 1000`
  - `nums` 中的所有整数 **互不相同**

- 解法

  **方法一：递归**

  **思路与算法**

  最简单的方法是直接按照题目描述进行模拟。

  我们用递归函数 construct(nums,left,right)  表示对数组 nums 中从 nums 到  的元素构建一棵树。我们首先找到这一区间中的最大值，记为 nums 中从 nums[best]，这样就确定了根节点的值。随后我们就可以进行递归：

  - 左子树为 construct(nums, left, best-1)；
  - 右子树为 construct(nums,best+1,right)。

  当递归到一个无效的区间（即 left*>*right）时，便可以返回一棵空的树。

  **代码**

  ```java
  class Solution {
      public TreeNode constructMaximumBinaryTree(int[] nums) {
          return construct(nums, 0, nums.length - 1);
      }
  
      public TreeNode construct(int[] nums, int left, int right) {
          if (left > right) {
              return null;
          }
          int best = left;
          for (int i = left + 1; i <= right; ++i) {
              if (nums[i] > nums[best]) {
                  best = i;
              }
          }
          TreeNode node = new TreeNode(nums[best]);
          node.left = construct(nums, left, best - 1);
          node.right = construct(nums, best + 1, right);
          return node;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n^2^)，其中 n 是数组 nums 的长度。在最坏的情况下，数组严格递增或递减，需要递归 n 层，第 i (0≤i<n) 层需要遍历 n-i 个元素以找出最大值，总时间复杂度为 O(n^2^)。
  - 空间复杂度：O(n)，即为最坏情况下需要使用的栈空间。

  **方法二：单调栈**

  **思路与算法**

  我们可以将题目中构造树的过程等价转换为下面的构造过程：

  - 初始时，我们只有一个根节点，其中存储了整个数组；
  - 在每一步操作中，我们可以「任选」一个存储了超过一个数的节点，找出其中的最大值并存储在该节点。最大值左侧的数组部分下放到该节点的左子节点，右侧的数组部分下放到该节点的右子节点；
  - 如果所有的节点都恰好存储了一个数，那么构造结束。

  由于最终构造出的是一棵树，因此无需按照题目的要求「递归」地进行构造，而是每次可以「任选」一个节点进行构造。这里可以类比一棵树的「深度优先搜索」和「广度优先搜索」，二者都可以起到遍历整棵树的效果。

  既然可以任意进行选择，那么我们不妨每次选择数组中最大值**最大**的那个节点进行构造。这样一来，我们就可以保证按照数组中元素降序排序的顺序依次构造每个节点。因此：

  > 当我们选择的节点中数组的最大值为 nums[i] 时，所有大于 nums[i] 的元素已经被构造过（即被单独放入某一个节点中），所有小于 nums[i] 的元素还没有被构造过。

  这就说明：

  > 在最终构造出的树上，以 nums[i] 为根节点的子树，在原数组中对应的区间，左边界为 nums[i] 左侧第一个比它大的元素所在的位置，右边界为 nums[i] 右侧第一个比它大的元素所在的位置。左右边界均为开边界。
  >
  > 如果某一侧边界不存在，则那一侧边界为数组的边界。如果两侧边界均不存在，说明其为最大值，即根节点。

  并且：

  > nums[i] 的父结点是两个边界中较小的那个元素对应的节点。

  因此，我们的任务变为：找出每一个元素左侧和右侧第一个比它大的元素所在的位置。这就是一个经典的单调栈问题了，可以参考 [503. 下一个更大元素 II](https://leetcode.cn/problems/next-greater-element-ii/)。如果左侧的元素较小，那么该元素就是左侧元素的右子节点；如果右侧的元素较小，那么该元素就是右侧元素的左子节点。

  **代码**

  ```java
  class Solution {
      public TreeNode constructMaximumBinaryTree(int[] nums) {
          int n = nums.length;
          Deque<Integer> stack = new ArrayDeque<Integer>();
          int[] left = new int[n];
          int[] right = new int[n];
          Arrays.fill(left, -1);
          Arrays.fill(right, -1);
          TreeNode[] tree = new TreeNode[n];
          for (int i = 0; i < n; ++i) {
              tree[i] = new TreeNode(nums[i]);
              while (!stack.isEmpty() && nums[i] > nums[stack.peek()]) {
                  right[stack.pop()] = i;
              }
              if (!stack.isEmpty()) {
                  left[i] = stack.peek();
              }
              stack.push(i);
          }
  
          TreeNode root = null;
          for (int i = 0; i < n; ++i) {
              if (left[i] == -1 && right[i] == -1) {
                  root = tree[i];
              } else if (right[i] == -1 || (left[i] != -1 && nums[left[i]] < nums[right[i]])) {
                  tree[left[i]].right = tree[i];
              } else {
                  tree[right[i]].left = tree[i];
              }
          }
          return root;
      }
  }
  ```

  我们还可以把最后构造树的过程放进单调栈求解的步骤中，省去用来存储左右边界的数组。下面的代码理解起来较为困难，同一个节点的左右子树会被多次赋值，读者可以仔细品味其妙处所在。

  ```java
  class Solution {
      public TreeNode constructMaximumBinaryTree(int[] nums) {
          int n = nums.length;
          List<Integer> stack = new ArrayList<Integer>();
          TreeNode[] tree = new TreeNode[n];
          for (int i = 0; i < n; ++i) {
              tree[i] = new TreeNode(nums[i]);
              while (!stack.isEmpty() && nums[i] > nums[stack.get(stack.size() - 1)]) {
                  tree[i].left = tree[stack.get(stack.size() - 1)];
                  stack.remove(stack.size() - 1);
              }
              if (!stack.isEmpty()) {
                  tree[stack.get(stack.size() - 1)].right = tree[i];
              }
              stack.add(i);
          }
          return tree[stack.get(0)];
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是数组 nums 的长度。单调栈求解左右边界和构造树均需要 O(n) 的时间。
  - 空间复杂度：O(n)，即为单调栈和数组 tree 需要使用的空间。

## day39

#### [144. 二叉树的前序遍历](https://leetcode.cn/problems/binary-tree-preorder-traversal/)

- 题目

  给你二叉树的根节点 `root` ，返回它节点值的 **前序** 遍历。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2020/09/15/inorder_1.jpg)

  ```
  输入：root = [1,null,2,3]
  输出：[1,2,3]
  ```

  **示例 2：**

  ```
  输入：root = []
  输出：[]
  ```

  **示例 3：**

  ```
  输入：root = [1]
  输出：[1]
  ```

  **示例 4：**

  ![img](https://assets.leetcode.com/uploads/2020/09/15/inorder_5.jpg)

  ```
  输入：root = [1,2]
  输出：[1,2]
  ```

  **示例 5：**

  ![img](https://assets.leetcode.com/uploads/2020/09/15/inorder_4.jpg)

  ```
  输入：root = [1,null,2]
  输出：[1,2]
  ```

   

  **提示：**

  - 树中节点数目在范围 `[0, 100]` 内
  - `-100 <= Node.val <= 100`

   

  **进阶：**递归算法很简单，你可以通过迭代算法完成吗？

- 解法

  **方法一：递归**

  **思路与算法**

  首先我们需要了解什么是二叉树的前序遍历：按照访问根节点——左子树——右子树的方式遍历这棵树，而在访问左子树或者右子树的时候，我们按照同样的方式遍历，直到遍历完整棵树。因此整个遍历过程天然具有递归的性质，我们可以直接用递归函数来模拟这一过程。

  定义 `preorder(root)` 表示当前遍历到 `root` 节点的答案。按照定义，我们只要首先将 `root` 节点的值加入答案，然后递归调用 `preorder(root.left)` 来遍历 `root` 节点的左子树，最后递归调用 `preorder(root.right)` 来遍历 `root` 节点的右子树即可，递归终止的条件为碰到空节点。

  **代码**

  ```java
  class Solution {
      public List<Integer> preorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<Integer>();
          preorder(root, res);
          return res;
      }
  
      public void preorder(TreeNode root, List<Integer> res) {
          if (root == null) {
              return;
          }
          res.add(root.val);
          preorder(root.left, res);
          preorder(root.right, res);
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是二叉树的节点数。每一个节点恰好被遍历一次。
  - 空间复杂度：O(n)，为递归过程中栈的开销，平均情况下为 O(log n)，最坏情况下树呈现链状，为 O(n)。

  **方法二：迭代**

  **思路与算法**

  我们也可以用迭代的方式实现方法一的递归函数，两种方式是等价的，区别在于递归的时候隐式地维护了一个栈，而我们在迭代的时候需要显式地将这个栈模拟出来，其余的实现与细节都相同，具体可以参考下面的代码。

  - 初始化维护一个栈，将根节点入栈。

  - 当栈不为空时
  
  - - 弹出栈顶元素 node，将节点值加入结果数组中。
    - 若 node 的右子树不为空，右子树入栈。
    - 若 node 的左子树不为空，左子树入栈。
  
  **代码**
  
  ```java
  class Solution {
      public List<Integer> preorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<Integer>();
          if (root == null) {
              return res;
          }
  
          Deque<TreeNode> stack = new LinkedList<TreeNode>();
          TreeNode node = root;
          while (!stack.isEmpty() || node != null) {
              while (node != null) {
                  res.add(node.val);
                  stack.push(node);
                  node = node.left;
              }
              node = stack.pop();
              node = node.right;
          }
          return res;
      }
  }
  ```
  
  **复杂度分析**
  
  - 时间复杂度：O(n)，其中 n 是二叉树的节点数。每一个节点恰好被遍历一次。
  - 空间复杂度：O(n)，为迭代过程中显式栈的开销，平均情况下为 O(log n)，最坏情况下树呈现链状，为 O(n)。

  **方法三：Morris 遍历**
  
  **思路与算法**
  
  有一种巧妙的方法可以在线性时间内，只占用常数空间来实现前序遍历。这种方法由 J. H. Morris 在 1979 年的论文「Traversing Binary Trees Simply and Cheaply」中首次提出，因此被称为 Morris 遍历。
  
  Morris 遍历的核心思想是利用树的大量空闲指针，实现空间开销的极限缩减。其前序遍历规则总结如下：
  
  1. 新建临时节点，令该节点为 `root`；
  2. 如果当前节点的左子节点为空，将当前节点加入答案，并遍历当前节点的右子节点；
  3. 如果当前节点的左子节点不为空，在当前节点的左子树中找到当前节点在中序遍历下的前驱节点：
     - 如果前驱节点的右子节点为空，将前驱节点的右子节点设置为当前节点。然后将当前节点加入答案，并将前驱节点的右子节点更新为当前节点。当前节点更新为当前节点的左子节点。
     - 如果前驱节点的右子节点为当前节点，将它的右子节点重新设为空。当前节点更新为当前节点的右子节点。
  4. 重复步骤 2 和步骤 3，直到遍历结束。
  
  这样我们利用 Morris 遍历的方法，前序遍历该二叉树，即可实现线性时间与常数空间的遍历。
  
  **代码**
  
  ```java
  class Solution {
      public List<Integer> preorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<Integer>();
          if (root == null) {
              return res;
          }
  
          TreeNode p1 = root, p2 = null;
  
          while (p1 != null) {
              p2 = p1.left;
              if (p2 != null) {
                  while (p2.right != null && p2.right != p1) {
                      p2 = p2.right;
                  }
                  if (p2.right == null) {
                      res.add(p1.val);
                      p2.right = p1;
                      p1 = p1.left;
                      continue;
                  } else {
                      p2.right = null;
                  }
              } else {
                  res.add(p1.val);
              }
              p1 = p1.right;
          }
          return res;
      }
  }
  ```
  
  **复杂度分析**
  
  - 时间复杂度：O(n)，其中 n 是二叉树的节点数。没有左子树的节点只被访问一次，有左子树的节点被访问两次。
  - 空间复杂度：O(1)。只操作已经存在的指针（树的空闲指针），因此只需要常数的额外空间。

## day40

#### [94. 二叉树的中序遍历](https://leetcode.cn/problems/binary-tree-inorder-traversal/)

- 题目

  给定一个二叉树的根节点 `root` ，返回 *它的 **中序** 遍历* 。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2020/09/15/inorder_1.jpg)

  ```
  输入：root = [1,null,2,3]
  输出：[1,3,2]
  ```

  **示例 2：**

  ```
  输入：root = []
  输出：[]
  ```

  **示例 3：**

  ```
  输入：root = [1]
  输出：[1]
  ```

   

  **提示：**

  - 树中节点数目在范围 `[0, 100]` 内
  - `-100 <= Node.val <= 100`

   

  **进阶:** 递归算法很简单，你可以通过迭代算法完成吗？

- 解法

  **方法一：递归**

  **思路与算法**

  首先我们需要了解什么是二叉树的中序遍历：按照访问左子树——根节点——右子树的方式遍历这棵树，而在访问左子树或者右子树的时候我们按照同样的方式遍历，直到遍历完整棵树。因此整个遍历过程天然具有递归的性质，我们可以直接用递归函数来模拟这一过程。

  定义 `inorder(root)` 表示当前遍历到 root 节点的答案，那么按照定义，我们只要递归调用 `inorder(root.left)` 来遍历 root 节点的左子树，然后将 root 节点的值加入答案，再递归调用`inorder(root.right)` 来遍历 root 节点的右子树即可，递归终止的条件为碰到空节点。

  **代码**

  ```java
  class Solution {
      public List<Integer> inorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<Integer>();
          inorder(root, res);
          return res;
      }
  
      public void inorder(TreeNode root, List<Integer> res) {
          if (root == null) {
              return;
          }
          inorder(root.left, res);
          res.add(root.val);
          inorder(root.right, res);
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 为二叉树节点的个数。二叉树的遍历中每个节点会被访问一次且只会被访问一次。
  - 空间复杂度：O(n)。空间复杂度取决于递归的栈深度，而栈深度在二叉树为一条链的情况下会达到 O(n) 的级别。

  **方法二：迭代**

  **思路与算法**

  方法一的递归函数我们也可以用迭代的方式实现，两种方式是等价的，区别在于递归的时候隐式地维护了一个栈，而我们在迭代的时候需要显式地将这个栈模拟出来，其他都相同，具体实现可以看下面的代码。

  - 初始化一个空栈。

  - 当【根节点不为空】或者【栈不为空】时，从根节点开始
  
  - - 若当前节点有左子树，一直遍历左子树，每次将当前节点压入栈中。
    - 若当前节点无左子树，从栈中弹出该节点，尝试访问该节点的右子树。
  
  **代码**
  
  ```java
  class Solution {
      public List<Integer> inorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<Integer>();
          Deque<TreeNode> stk = new LinkedList<TreeNode>();
          while (root != null || !stk.isEmpty()) {
              while (root != null) {
                  stk.push(root);
                  root = root.left;
              }
              root = stk.pop();
              res.add(root.val);
              root = root.right;
          }
          return res;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 为二叉树节点的个数。二叉树的遍历中每个节点会被访问一次且只会被访问一次。
  - 空间复杂度：O(n)。空间复杂度取决于栈深度，而栈深度在二叉树为一条链的情况下会达到 O(n) 的级别。
  
  **方法三：Morris 中序遍历**
  
  **思路与算法**
  
  **Morris 遍历算法**是另一种遍历二叉树的方法，它能将非递归的中序遍历空间复杂度降为 O(1)。
  
  **Morris 遍历算法**整体步骤如下（假设当前遍历到的节点为 x）：
  
  1. 如果 x 无左孩子，先将 x 的值加入答案数组，再访问 x 的右孩子，即x*=*x.right。
  2. 如果 x 有左孩子，则找到 x 左子树上最右的节点（即左子树中序遍历的最后一个节点，x 在中序遍历中的前驱节点），我们记为predecessor. 根据predecessor的右孩子是否为空，进行如下操作。
     - 如果 predecessor 的右孩子为空，则将其右孩子指向 x，然后访问 x 的左孩子，即 x = x.left。
     - 如果 predecessor 的右孩子不为空，则此时其右孩子指向 x，说明我们已经遍历完 x 的左子树，我们将 predecessor 的右孩子置空，将 x 的值加入答案数组，然后访问 x 的右孩子，即 x*=*x.right。
  3. 重复上述操作，直至访问完整棵树。
  
  
  
  其实整个过程我们就多做一步：假设当前遍历到的节点为 x，将 x 的左子树中最右边的节点的右孩子指向 x，这样在左子树遍历完成后我们通过这个指向走回了 x，且能通过这个指向知晓我们已经遍历完成了左子树，而不用再通过栈来维护，省去了栈的空间复杂度。
  
  **代码**
  
  ```java
  class Solution {
      public List<Integer> inorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<Integer>();
          TreeNode predecessor = null;
  
          while (root != null) {
              if (root.left != null) {
                  // predecessor 节点就是当前 root 节点向左走一步，然后一直向右走至无法走为止
                  predecessor = root.left;
                  while (predecessor.right != null && predecessor.right != root) {
                      predecessor = predecessor.right;
                  }
                  
                  // 让 predecessor 的右指针指向 root，继续遍历左子树
                  if (predecessor.right == null) {
                      predecessor.right = root;
                      root = root.left;
                  }
                  // 说明左子树已经访问完了，我们需要断开链接
                  else {
                      res.add(root.val);
                      predecessor.right = null;
                      root = root.right;
                  }
              }
              // 如果没有左孩子，则直接访问右孩子
              else {
                  res.add(root.val);
                  root = root.right;
              }
          }
          return res;
      }
  }
  ```
  
  **复杂度分析**
  
  - 时间复杂度：O(n)，其中 n 为二叉搜索树的节点个数。Morris 遍历中每个节点会被访问两次，因此总时间复杂度为 O(2n)=O(n)。
  - 空间复杂度：O(1)。

## day41

#### [105. 从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

- 题目

  给定两个整数数组 `preorder` 和 `inorder` ，其中 `preorder` 是二叉树的**先序遍历**， `inorder` 是同一棵树的**中序遍历**，请构造二叉树并返回其根节点。

   

  **示例 1:**

  ![img](https://assets.leetcode.com/uploads/2021/02/19/tree.jpg)

  ```
  输入: preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
  输出: [3,9,20,null,null,15,7]
  ```

  **示例 2:**

  ```
  输入: preorder = [-1], inorder = [-1]
  输出: [-1]
  ```

   

  **提示:**

  - `1 <= preorder.length <= 3000`
  - `inorder.length == preorder.length`
  - `-3000 <= preorder[i], inorder[i] <= 3000`
  - `preorder` 和 `inorder` 均 **无重复** 元素
  - `inorder` 均出现在 `preorder`
  - `preorder` **保证** 为二叉树的前序遍历序列
  - `inorder` **保证** 为二叉树的中序遍历序列

- 解法

  二叉树前序遍历的顺序为：

  - 先遍历根节点；
  - 随后递归地遍历左子树；
  - 最后递归地遍历右子树。

  二叉树中序遍历的顺序为：

  - 先递归地遍历左子树；
  - 随后遍历根节点；
  - 最后递归地遍历右子树。

  在「递归」地遍历某个子树的过程中，我们也是将这颗子树看成一颗全新的树，按照上述的顺序进行遍历。挖掘「前序遍历」和「中序遍历」的性质，我们就可以得出本题的做法。

  **方法一：递归**

  **思路**

  对于任意一颗树而言，前序遍历的形式总是

  ```
  [ 根节点, [左子树的前序遍历结果], [右子树的前序遍历结果] ]
  ```

  即根节点总是前序遍历中的第一个节点。而中序遍历的形式总是

  ```
  [ [左子树的中序遍历结果], 根节点, [右子树的中序遍历结果] ]
  ```

  只要我们在中序遍历中**定位**到根节点，那么我们就可以分别知道左子树和右子树中的节点数目。由于同一颗子树的前序遍历和中序遍历的长度显然是相同的，因此我们就可以对应到前序遍历的结果中，对上述形式中的所有**左右括号**进行定位。

  这样以来，我们就知道了左子树的前序遍历和中序遍历结果，以及右子树的前序遍历和中序遍历结果，我们就可以递归地对构造出左子树和右子树，再将这两颗子树接到根节点的左右位置。

  **细节**

  在中序遍历中对根节点进行定位时，一种简单的方法是直接扫描整个中序遍历的结果并找出根节点，但这样做的时间复杂度较高。我们可以考虑使用哈希表来帮助我们快速地定位根节点。对于哈希映射中的每个键值对，键表示一个元素（节点的值），值表示其在中序遍历中的出现位置。在构造二叉树的过程之前，我们可以对中序遍历的列表进行一遍扫描，就可以构造出这个哈希映射。在此后构造二叉树的过程中，我们就只需要 O(1) 的时间对根节点进行定位了。

  下面的代码给出了详细的注释。

  ```java
  class Solution {
      private Map<Integer, Integer> indexMap;
  
      public TreeNode myBuildTree(int[] preorder, int[] inorder, int preorder_left, int preorder_right, int inorder_left, int inorder_right) {
          if (preorder_left > preorder_right) {
              return null;
          }
  
          // 前序遍历中的第一个节点就是根节点
          int preorder_root = preorder_left;
          // 在中序遍历中定位根节点
          int inorder_root = indexMap.get(preorder[preorder_root]);
          
          // 先把根节点建立出来
          TreeNode root = new TreeNode(preorder[preorder_root]);
          // 得到左子树中的节点数目
          int size_left_subtree = inorder_root - inorder_left;
          // 递归地构造左子树，并连接到根节点
          // 先序遍历中「从 左边界+1 开始的 size_left_subtree」个元素就对应了中序遍历中「从 左边界 开始到 根节点定位-1」的元素
          root.left = myBuildTree(preorder, inorder, preorder_left + 1, preorder_left + size_left_subtree, inorder_left, inorder_root - 1);
          // 递归地构造右子树，并连接到根节点
          // 先序遍历中「从 左边界+1+左子树节点数目 开始到 右边界」的元素就对应了中序遍历中「从 根节点定位+1 到 右边界」的元素
          root.right = myBuildTree(preorder, inorder, preorder_left + size_left_subtree + 1, preorder_right, inorder_root + 1, inorder_right);
          return root;
      }
  
      public TreeNode buildTree(int[] preorder, int[] inorder) {
          int n = preorder.length;
          // 构造哈希映射，帮助我们快速定位根节点
          indexMap = new HashMap<Integer, Integer>();
          for (int i = 0; i < n; i++) {
              indexMap.put(inorder[i], i);
          }
          return myBuildTree(preorder, inorder, 0, n - 1, 0, n - 1);
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是树中的节点个数。
  - 空间复杂度：O(n)，除去返回的答案需要的 O(n) 空间之外，我们还需要使用 O(n) 的空间存储哈希映射，以及 O(h)（其中 h 是树的高度）的空间表示递归时栈空间。这里 h < n，所以总空间复杂度为 O(n)。

  **方法二：迭代**

  **思路**

  迭代法是一种非常巧妙的实现方法。

  对于前序遍历中的任意两个连续节点 u 和 v，根据前序遍历的流程，我们可以知道 u 和 v 只有两种可能的关系：

  - v 是 u 的左儿子。这是因为在遍历到 u 之后，下一个遍历的节点就是 u 的左儿子，即 v；
  - u 没有左儿子，并且 v 是 u 的某个祖先节点（或者 u 本身）的右儿子。如果 u 没有左儿子，那么下一个遍历的节点就是 u 的右儿子。如果 u 没有右儿子，我们就会向上回溯，直到遇到第一个有右儿子（且 u 不在它的右儿子的子树中）的节点 u_a，那么 v 就是 u_a 的右儿子。

  第二种关系看上去有些复杂。我们举一个例子来说明其正确性，并在例子中给出我们的迭代算法。

  **例子**

  我们以树

  ```
          3
         / \
        9  20
       /  /  \
      8  15   7
     / \
    5  10
   /
  4
  ```

  为例，它的前序遍历和中序遍历分别为

  ```
  preorder = [3, 9, 8, 5, 4, 10, 20, 15, 7]
  inorder = [4, 5, 8, 10, 9, 3, 15, 20, 7]
  ```

  我们用一个栈 `stack` 来维护「当前节点的所有还没有考虑过右儿子的祖先节点」，栈顶就是当前节点。也就是说，只有在栈中的节点才可能连接一个新的右儿子。同时，我们用一个指针 `index` 指向中序遍历的某个位置，初始值为 `0`。`index` 对应的节点是「当前节点不断往左走达到的最终节点」，这也是符合中序遍历的，它的作用在下面的过程中会有所体现。

  首先我们将根节点 `3` 入栈，再初始化 `index` 所指向的节点为 `4`，随后对于前序遍历中的每个节点，我们依次判断它是栈顶节点的左儿子，还是栈中某个节点的右儿子。

  - 我们遍历 `9`。`9` 一定是栈顶节点 `3` 的左儿子。我们使用反证法，假设 `9` 是 `3` 的右儿子，那么 `3` 没有左儿子，`index` 应该恰好指向 `3`，但实际上为 `4`，因此产生了矛盾。所以我们将 `9` 作为 `3` 的左儿子，并将 `9` 入栈。

    - `stack = [3, 9]`
    - `index -> inorder[0] = 4`

  - 我们遍历 `8`，`5` 和 `4`。同理可得它们都是上一个节点（栈顶节点）的左儿子，所以它们会依次入栈。

    - `stack = [3, 9, 8, 5, 4]`
    - `index -> inorder[0] = 4`

  - 我们遍历 `10`，这时情况就不一样了。我们发现 `index` 恰好指向当前的栈顶节点 `4`，也就是说 `4` 没有左儿子，那么 `10` 必须为栈中某个节点的右儿子。那么如何找到这个节点呢？栈中的节点的顺序和它们在前序遍历中出现的顺序是一致的，而且每一个节点的右儿子都还没有被遍历过，**那么这些节点的顺序和它们在中序遍历中出现的顺序一定是相反的**。

    > 这是因为栈中的任意两个相邻的节点，前者都是后者的某个祖先。并且我们知道，栈中的任意一个节点的右儿子还没有被遍历过，说明后者一定是前者左儿子的子树中的节点，那么后者就先于前者出现在中序遍历中。

    因此我们可以把 `index` 不断向右移动，并与栈顶节点进行比较。如果 `index` 对应的元素恰好等于栈顶节点，那么说明我们在中序遍历中找到了栈顶节点，所以将 `index` 增加 `1` 并弹出栈顶节点，直到 `index` 对应的元素不等于栈顶节点。按照这样的过程，我们弹出的最后一个节点 `x` 就是 `10` 的双亲节点，**这是因为 `10` 出现在了 `x` 与 `x` 在栈中的下一个节点的中序遍历之间**，因此 `10` 就是 `x` 的右儿子。

    回到我们的例子，我们会依次从栈顶弹出 `4`，`5` 和 `8`，并且将 `index` 向右移动了三次。我们将 `10` 作为最后弹出的节点 `8` 的右儿子，并将 `10` 入栈。

    - `stack = [3, 9, 10]`
    - `index -> inorder[3] = 10`

  - 我们遍历 `20`。同理，`index` 恰好指向当前栈顶节点 `10`，那么我们会依次从栈顶弹出 `10`，`9` 和 `3`，并且将 `index` 向右移动了三次。我们将 `20` 作为最后弹出的节点 `3` 的右儿子，并将 `20` 入栈。

    - `stack = [20]`
    - `index -> inorder[6] = 15`

  - 我们遍历 `15`，将 `15` 作为栈顶节点 `20` 的左儿子，并将 `15` 入栈。

    - `stack = [20, 15]`
    - `index -> inorder[6] = 15`

  - 我们遍历 `7`。`index` 恰好指向当前栈顶节点 `15`，那么我们会依次从栈顶弹出 `15` 和 `20`，并且将 `index` 向右移动了两次。我们将 `7` 作为最后弹出的节点 `20` 的右儿子，并将 `7` 入栈。

    - `stack = [7]`
    - `index -> inorder[8] = 7`

  此时遍历结束，我们就构造出了正确的二叉树。

  **算法**

  我们归纳出上述例子中的算法流程：

  - 我们用一个栈和一个指针辅助进行二叉树的构造。初始时栈中存放了根节点（前序遍历的第一个节点），指针指向中序遍历的第一个节点；
  - 我们依次枚举前序遍历中除了第一个节点以外的每个节点。如果 `index` 恰好指向栈顶节点，那么我们不断地弹出栈顶节点并向右移动 `index`，并将当前节点作为最后一个弹出的节点的右儿子；如果 `index` 和栈顶节点不同，我们将当前节点作为栈顶节点的左儿子；
  - 无论是哪一种情况，我们最后都将当前的节点入栈。

  最后得到的二叉树即为答案。

  ```java
  class Solution {
      public TreeNode buildTree(int[] preorder, int[] inorder) {
          if (preorder == null || preorder.length == 0) {
              return null;
          }
          TreeNode root = new TreeNode(preorder[0]);
          Deque<TreeNode> stack = new LinkedList<TreeNode>();
          stack.push(root);
          int inorderIndex = 0;
          for (int i = 1; i < preorder.length; i++) {
              int preorderVal = preorder[i];
              TreeNode node = stack.peek();
              if (node.val != inorder[inorderIndex]) {
                  node.left = new TreeNode(preorderVal);
                  stack.push(node.left);
              } else {
                  while (!stack.isEmpty() && stack.peek().val == inorder[inorderIndex]) {
                      node = stack.pop();
                      inorderIndex++;
                  }
                  node.right = new TreeNode(preorderVal);
                  stack.push(node.right);
              }
          }
          return root;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是树中的节点个数。
  - 空间复杂度：O(n)，除去返回的答案需要的 O(n) 空间之外，我们还需要使用 O(h)（其中 h 是树的高度）的空间存储栈。这里 h < n，所以（在最坏情况下）总空间复杂度为 O(n)。

## day42

#### [106. 从中序与后序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

- 题目

  给定两个整数数组 `inorder` 和 `postorder` ，其中 `inorder` 是二叉树的中序遍历， `postorder` 是同一棵树的后序遍历，请你构造并返回这颗 *二叉树* 。

   

  **示例 1:**

  ![img](https://assets.leetcode.com/uploads/2021/02/19/tree.jpg)

  ```
  输入：inorder = [9,3,15,20,7], postorder = [9,15,7,20,3]
  输出：[3,9,20,null,null,15,7]
  ```

  **示例 2:**

  ```
  输入：inorder = [-1], postorder = [-1]
  输出：[-1]
  ```

   

  **提示:**

  - `1 <= inorder.length <= 3000`
  - `postorder.length == inorder.length`
  - `-3000 <= inorder[i], postorder[i] <= 3000`
  - `inorder` 和 `postorder` 都由 **不同** 的值组成
  - `postorder` 中每一个值都在 `inorder` 中
  - `inorder` **保证**是树的中序遍历
  - `postorder` **保证**是树的后序遍历

- 解法

  **方法一：递归**

  我们可以发现后序遍历的数组最后一个元素代表的即为根节点。知道这个性质后，我们可以利用已知的根节点信息在中序遍历的数组中找到根节点所在的下标，然后根据其将中序遍历的数组分成左右两部分，左边部分即左子树，右边部分为右子树，针对每个部分可以用同样的方法继续递归下去构造。

  ![fig1](https://assets.leetcode-cn.com/solution-static/106/6.png)

  **算法**

  - 为了高效查找根节点元素在中序遍历数组中的下标，我们选择创建哈希表来存储中序序列，即建立一个（元素，下标）键值对的哈希表。
  - 定义递归函数 `helper(in_left, in_right)` 表示当前递归到中序序列中当前子树的左右边界，递归入口为`helper(0, n - 1)` ：
    - 如果 `in_left > in_right`，说明子树为空，返回空节点。
    - 选择后序遍历的最后一个节点作为根节点。
    - 利用哈希表 O(1) 查询当根节点在中序遍历中下标为 `index`。从 `in_left` 到 `index - 1` 属于左子树，从 `index + 1` 到 `in_right` 属于右子树。
    - 根据后序遍历逻辑，递归创建右子树 `helper(index + 1, in_right)` 和左子树 `helper(in_left, index - 1)`。注意这里有需要先创建右子树，再创建左子树的依赖关系。可以理解为在后序遍历的数组中整个数组是先存储左子树的节点，再存储右子树的节点，最后存储根节点，如果按每次选择「后序遍历的最后一个节点」为根节点，则先被构造出来的应该为右子树。
    - 返回根节点 `root`

  **代码**

  ```java
  class Solution {
      int post_idx;
      int[] postorder;
      int[] inorder;
      Map<Integer, Integer> idx_map = new HashMap<Integer, Integer>();
  
      public TreeNode helper(int in_left, int in_right) {
          // 如果这里没有节点构造二叉树了，就结束
          if (in_left > in_right) {
              return null;
          }
  
          // 选择 post_idx 位置的元素作为当前子树根节点
          int root_val = postorder[post_idx];
          TreeNode root = new TreeNode(root_val);
  
          // 根据 root 所在位置分成左右两棵子树
          int index = idx_map.get(root_val);
  
          // 下标减一
          post_idx--;
          // 构造右子树
          root.right = helper(index + 1, in_right);
          // 构造左子树
          root.left = helper(in_left, index - 1);
          return root;
      }
  
      public TreeNode buildTree(int[] inorder, int[] postorder) {
          this.postorder = postorder;
          this.inorder = inorder;
          // 从后序遍历的最后一个元素开始
          post_idx = postorder.length - 1;
  
          // 建立（元素，下标）键值对的哈希表
          int idx = 0;
          for (Integer val : inorder) {
              idx_map.put(val, idx++);
          }
          
          return helper(0, inorder.length - 1);
      }
  }
  
  // 或者
  
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是树中的节点个数。
  - 空间复杂度：O(n)。我们需要使用 O(n) 的空间存储哈希表，以及 O(h)（其中 h 是树的高度）的空间表示递归时栈空间。这里 h < n，所以总空间复杂度为 O(n)。

  **方法二：迭代**

  **思路**

  迭代法是一种非常巧妙的实现方法。迭代法的实现基于以下两点发现。

  - 如果将中序遍历反序，则得到反向的中序遍历，即每次遍历右孩子，再遍历根节点，最后遍历左孩子。
  - 如果将后序遍历反序，则得到反向的前序遍历，即每次遍历根节点，再遍历右孩子，最后遍历左孩子。

  「反向」的意思是交换遍历左孩子和右孩子的顺序，即反向的遍历中，右孩子在左孩子之前被遍历。

  因此可以使用和「[105. 从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)」的迭代方法类似的方法构造二叉树。

  对于后序遍历中的任意两个连续节点 u 和 v（在后序遍历中，u 在 v 的前面），根据后序遍历的流程，我们可以知道 u 和 v 只有两种可能的关系：

  - u 是 v 的右儿子。这是因为在遍历到 u 之后，下一个遍历的节点就是 u 的双亲节点，即 v；
  - v 没有右儿子，并且 u 是 v 的某个祖先节点（或者 v 本身）的左儿子。如果 v 没有右儿子，那么上一个遍历的节点就是 v 的左儿子。如果 v 没有左儿子，则从 v 开始向上遍历 v 的祖先节点，直到遇到一个有左儿子（且 v 不在它的左儿子的子树中）的节点 v_a，那么 u 就是 v_a 的左儿子。

  第二种关系看上去有些复杂。我们举一个例子来说明其正确性，并在例子中给出我们的迭代算法。

  **例子**

  我们以树

  ```
          3
         / \
        9  20
       / \   \
      15 10   7
             / \
            5   8
                 \
                  4
  ```

  为例，它的中序遍历和后序遍历分别为

  ```
  inorder = [15, 9, 10, 3, 20, 5, 7, 8, 4]
  postorder = [15, 10, 9, 5, 4, 8, 7, 20, 3]
  ```

  我们用一个栈 `stack` 来维护「当前节点的所有还没有考虑过左儿子的祖先节点」，栈顶就是当前节点。也就是说，只有在栈中的节点才可能连接一个新的左儿子。同时，我们用一个指针 `index` 指向中序遍历的某个位置，初始值为 `n - 1`，其中 `n` 为数组的长度。`index` 对应的节点是「当前节点不断往右走达到的最终节点」，这也是符合反向中序遍历的，它的作用在下面的过程中会有所体现。

  首先我们将根节点 `3` 入栈，再初始化 `index` 所指向的节点为 `4`，随后对于后序遍历中的每个节点，我们依次判断它是栈顶节点的右儿子，还是栈中某个节点的左儿子。

  - 我们遍历 `20`。`20` 一定是栈顶节点 `3` 的右儿子。我们使用反证法，假设 `20` 是 `3` 的左儿子，因为 `20` 和 `3` 中间不存在其他的节点，那么 `3` 没有右儿子，`index` 应该恰好指向 `3`，但实际上为 `4`，因此产生了矛盾。所以我们将 `20` 作为 `3` 的右儿子，并将 `20` 入栈。

    - `stack = [3, 20]`
    - `index -> inorder[8] = 4`

  - 我们遍历 `7`，`8` 和 `4`。同理可得它们都是上一个节点（栈顶节点）的右儿子，所以它们会依次入栈。

    - `stack = [3, 20, 7, 8, 4]`
    - `index -> inorder[8] = 4`

  - 我们遍历 `5`，这时情况就不一样了。我们发现 `index` 恰好指向当前的栈顶节点 `4`，也就是说 `4` 没有右儿子，那么 `5` 必须为栈中某个节点的左儿子。那么如何找到这个节点呢？栈中的节点的顺序和它们在反向前序遍历中出现的顺序是一致的，而且每一个节点的左儿子都还没有被遍历过，**那么这些节点的顺序和它们在反向中序遍历中出现的顺序一定是相反的**。

    > 这是因为栈中的任意两个相邻的节点，前者都是后者的某个祖先。并且我们知道，栈中的任意一个节点的左儿子还没有被遍历过，说明后者一定是前者右儿子的子树中的节点，那么后者就先于前者出现在反向中序遍历中。

    因此我们可以把 `index` 不断向左移动，并与栈顶节点进行比较。如果 `index` 对应的元素恰好等于栈顶节点，那么说明我们在反向中序遍历中找到了栈顶节点，所以将 `index` 减少 `1` 并弹出栈顶节点，直到 `index` 对应的元素不等于栈顶节点。按照这样的过程，我们弹出的最后一个节点 `x` 就是 `5` 的双亲节点，**这是因为 `5` 出现在了 `x` 与 `x` 在栈中的下一个节点的反向中序遍历之间**，因此 `5` 就是 `x` 的左儿子。

    回到我们的例子，我们会依次从栈顶弹出 `4`，`8` 和 `7`，并且将 `index` 向左移动了三次。我们将 `5` 作为最后弹出的节点 `7` 的左儿子，并将 `5` 入栈。

    - `stack = [3, 20, 5]`
    - `index -> inorder[5] = 5`

  - 我们遍历 `9`。同理，`index` 恰好指向当前栈顶节点 `5`，那么我们会依次从栈顶弹出 `5`，`20` 和 `3`，并且将 `index` 向左移动了三次。我们将 `9` 作为最后弹出的节点 `3` 的左儿子，并将 `9` 入栈。

    - `stack = [9]`
    - `index -> inorder[2] = 10`

  - 我们遍历 `10`，将 `10` 作为栈顶节点 `9` 的右儿子，并将 `10` 入栈。

    - `stack = [9, 10]`
    - `index -> inorder[2] = 10`

  - 我们遍历 `15`。`index` 恰好指向当前栈顶节点 `10`，那么我们会依次从栈顶弹出 `10` 和 `9`，并且将 `index` 向左移动了两次。我们将 `15` 作为最后弹出的节点 `9` 的左儿子，并将 `15` 入栈。

    - `stack = [15]`
    - `index -> inorder[0] = 15`

  此时遍历结束，我们就构造出了正确的二叉树。

  **算法**

  我们归纳出上述例子中的算法流程：

  - 我们用一个栈和一个指针辅助进行二叉树的构造。初始时栈中存放了根节点（后序遍历的最后一个节点），指针指向中序遍历的最后一个节点；
  - 我们依次枚举后序遍历中除了第一个节点以外的每个节点。如果 `index` 恰好指向栈顶节点，那么我们不断地弹出栈顶节点并向左移动 `index`，并将当前节点作为最后一个弹出的节点的左儿子；如果 `index` 和栈顶节点不同，我们将当前节点作为栈顶节点的右儿子；
  - 无论是哪一种情况，我们最后都将当前的节点入栈。

  最后得到的二叉树即为答案。

  ```java
  class Solution {
      public TreeNode buildTree(int[] inorder, int[] postorder) {
          if (postorder == null || postorder.length == 0) {
              return null;
          }
          TreeNode root = new TreeNode(postorder[postorder.length - 1]);
          Deque<TreeNode> stack = new LinkedList<TreeNode>();
          stack.push(root);
          int inorderIndex = inorder.length - 1;
          for (int i = postorder.length - 2; i >= 0; i--) {
              int postorderVal = postorder[i];
              TreeNode node = stack.peek();
              if (node.val != inorder[inorderIndex]) {
                  node.right = new TreeNode(postorderVal);
                  stack.push(node.right);
              } else {
                  while (!stack.isEmpty() && stack.peek().val == inorder[inorderIndex]) {
                      node = stack.pop();
                      inorderIndex--;
                  }
                  node.left = new TreeNode(postorderVal);
                  stack.push(node.left);
              }
          }
          return root;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是树中的节点个数。
  - 空间复杂度：O(n)，我们需要使用 O(h)（其中 h 是树的高度）的空间存储栈。这里 h < n，所以（在最坏情况下）总空间复杂度为 O(n)。

## day43

#### [145. 二叉树的后序遍历](https://leetcode.cn/problems/binary-tree-postorder-traversal/)

- 题目

  给你一棵二叉树的根节点 `root` ，返回其节点值的 **后序遍历** 。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2020/08/28/pre1.jpg)

  ```
  输入：root = [1,null,2,3]
  输出：[3,2,1]
  ```

  **示例 2：**

  ```
  输入：root = []
  输出：[]
  ```

  **示例 3：**

  ```
  输入：root = [1]
  输出：[1]
  ```

   

  **提示：**

  - 树中节点的数目在范围 `[0, 100]` 内
  - `-100 <= Node.val <= 100`

   

  **进阶：**递归算法很简单，你可以通过迭代算法完成吗？

- 解法

  **方法一：递归**

  **思路与算法**

  首先我们需要了解什么是二叉树的后序遍历：按照访问左子树——右子树——根节点的方式遍历这棵树，而在访问左子树或者右子树的时候，我们按照同样的方式遍历，直到遍历完整棵树。因此整个遍历过程天然具有递归的性质，我们可以直接用递归函数来模拟这一过程。

  定义 `postorder(root)` 表示当前遍历到 `root` 节点的答案。按照定义，我们只要递归调用 `postorder(root->left)` 来遍历 `root` 节点的左子树，然后递归调用 `postorder(root->right)` 来遍历 `root` 节点的右子树，最后将 `root` 节点的值加入答案即可，递归终止的条件为碰到空节点。

  **代码**

  ```java
  class Solution {
      public List<Integer> postorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<Integer>();
          postorder(root, res);
          return res;
      }
  
      public void postorder(TreeNode root, List<Integer> res) {
          if (root == null) {
              return;
          }
          postorder(root.left, res);
          postorder(root.right, res);
          res.add(root.val);
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是二叉搜索树的节点数。每一个节点恰好被遍历一次。
  - 空间复杂度：O(n)，为递归过程中栈的开销，平均情况下为 O(log n)，最坏情况下树呈现链状，为 O(n)。

  **方法二：迭代**

  **思路与算法**

  我们也可以用迭代的方式实现方法一的递归函数，两种方式是等价的，区别在于递归的时候隐式地维护了一个栈，而我们在迭代的时候需要显式地将这个栈模拟出来，其余的实现与细节都相同，具体可以参考下面的代码。

  **代码**

  ```java
  class Solution {
      public List<Integer> postorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<Integer>();
          if (root == null) {
              return res;
          }
  
          Deque<TreeNode> stack = new LinkedList<TreeNode>();
          TreeNode prev = null;
          while (root != null || !stack.isEmpty()) {
              while (root != null) {
                  stack.push(root);
                  root = root.left;
              }
              root = stack.pop();
              if (root.right == null || root.right == prev) {
                  res.add(root.val);
                  prev = root;
                  root = null;
              } else {
                  stack.push(root);
                  root = root.right;
              }
          }
          return res;
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是二叉搜索树的节点数。每一个节点恰好被遍历一次。
  - 空间复杂度：O(n)，为迭代过程中显式栈的开销，平均情况下为 O(log n)，最坏情况下树呈现链状，为 O(n)。

  **方法三：Morris 遍历**

  **思路与算法**

  有一种巧妙的方法可以在线性时间内，只占用常数空间来实现后序遍历。这种方法由 J. H. Morris 在 1979 年的论文「Traversing Binary Trees Simply and Cheaply」中首次提出，因此被称为 Morris 遍历。

  Morris 遍历的核心思想是利用树的大量空闲指针，实现空间开销的极限缩减。其后序遍历规则总结如下：

  1. 新建临时节点，令该节点为 `root`；
  2. 如果当前节点的左子节点为空，则遍历当前节点的右子节点；
  3. 如果当前节点的左子节点不为空，在当前节点的左子树中找到当前节点在中序遍历下的前驱节点；
     - 如果前驱节点的右子节点为空，将前驱节点的右子节点设置为当前节点，当前节点更新为当前节点的左子节点。
     - 如果前驱节点的右子节点为当前节点，将它的右子节点重新设为空。倒序输出从当前节点的左子节点到该前驱节点这条路径上的所有节点。当前节点更新为当前节点的右子节点。
  4. 重复步骤 2 和步骤 3，直到遍历结束。

  这样我们利用 Morris 遍历的方法，后序遍历该二叉搜索树，即可实现线性时间与常数空间的遍历。

  **代码**

  ```java
  class Solution {
      public List<Integer> postorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<Integer>();
          if (root == null) {
              return res;
          }
  
          TreeNode p1 = root, p2 = null;
  
          while (p1 != null) {
              p2 = p1.left;
              if (p2 != null) {
                  while (p2.right != null && p2.right != p1) {
                      p2 = p2.right;
                  }
                  if (p2.right == null) {
                      p2.right = p1;
                      p1 = p1.left;
                      continue;
                  } else {
                      p2.right = null;
                      addPath(res, p1.left);
                  }
              }
              p1 = p1.right;
          }
          addPath(res, root);
          return res;
      }
  
      public void addPath(List<Integer> res, TreeNode node) {
          int count = 0;
          while (node != null) {
              ++count;
              res.add(node.val);
              node = node.right;
          }
          int left = res.size() - count, right = res.size() - 1;
          while (left < right) {
              int temp = res.get(left);
              res.set(left, res.get(right));
              res.set(right, temp);
              left++;
              right--;
          }
      }
  }
  ```

  **复杂度分析**

  - 时间复杂度：O(n)，其中 n 是二叉树的节点数。没有左子树的节点只被访问一次，有左子树的节点被访问两次。
  - 空间复杂度：O(1)。只操作已经存在的指针（树的空闲指针），因此只需要常数的额外空间。

## day44

#### [102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)

- 题目

  给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2021/02/19/tree1.jpg)

  ```
  输入：root = [3,9,20,null,null,15,7]
  输出：[[3],[9,20],[15,7]]
  ```

  **示例 2：**

  ```
  输入：root = [1]
  输出：[[1]]
  ```

  **示例 3：**

  ```
  输入：root = []
  输出：[]
  ```

   

  **提示：**

  - 树中节点数目在范围 `[0, 2000]` 内
  - `-1000 <= Node.val <= 1000`

- 解法

  **解法一：深度优先**

  采用递归方法来找出层序序列。思路：我们知道，遍历递归地遍历二叉树是往深处走，我们只需要在往深处走的时候，将相同深度的节点放到一起即可。

  ```java
  class Solution {
      public List<List<Integer>> levelOrder(TreeNode root) {
          if (root==null) return new ArrayList<>();
          List<List<Integer>> lls =new ArrayList<>();
          doSome(root,1,lls);
          return lls;
      }
      // 递归层序遍历的秘诀：深度相同的节点放到同一个list
      public void doSome(TreeNode node,int depth,List<List<Integer>> lls){
          // 当前深度下没有对应的“层list”，就创建
          if(lls.size()<depth) {
              lls.add(new ArrayList<Integer>());
          }
          //在同一深度，把节点加上去
          lls.get(depth-1).add(node.val);
  
          if(node.left!=null) doSome(node.left,depth+1,lls);
          if(node.right!=null) doSome(node.right,depth+1,lls);
      }
  }
  ```

**解法二：广度优先搜索**

**解题思路**

本文将会讲解为什么这道题适合用广度优先搜索（BFS），以及 BFS 适用于什么样的场景。

DFS（深度优先搜索）和 BFS（广度优先搜索）就像孪生兄弟，提到一个总是想起另一个。然而在实际使用中，我们用 DFS 的时候远远多于 BFS。那么，是不是 BFS 就没有什么用呢？

如果我们使用 DFS/BFS 只是为了遍历一棵树、一张图上的所有结点的话，那么 DFS 和 BFS 的能力没什么差别，我们当然更倾向于更方便写、空间复杂度更低的 DFS 遍历。不过，某些使用场景是 DFS 做不到的，只能使用 BFS 遍历。这就是本文要介绍的两个场景：「层序遍历」、「最短路径」。

本文包括以下内容：

- DFS 与 BFS 的特点比较
- BFS 的适用场景
- 如何用 BFS 进行层序遍历
- 如何用 BFS 求解最短路径问题

**DFS 与 BFS**

让我们先看看在二叉树上进行 DFS 遍历和 BFS 遍历的代码比较。

DFS 遍历使用**递归**：

```java
void dfs(TreeNode root) {
    if (root == null) {
        return;
    }
    dfs(root.left);
    dfs(root.right);
}
```

BFS 遍历使用**队列**数据结构：

```java
void bfs(TreeNode root) {
    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.add(root);
    while (!queue.isEmpty()) {
        TreeNode node = queue.poll(); // Java 的 pop 写作 poll()
        if (node.left != null) {
            queue.add(node.left);
        }
        if (node.right != null) {
            queue.add(node.right);
        }
    }
}
```

只是比较两段代码的话，最直观的感受就是：DFS 遍历的代码比 BFS 简洁太多了！这是因为递归的方式隐含地使用了系统的 **栈**，我们不需要自己维护一个数据结构。如果只是简单地将二叉树遍历一遍，那么 DFS 显然是更方便的选择。

虽然 DFS 与 BFS 都是将二叉树的所有结点遍历了一遍，但它们遍历结点的顺序不同。

![DFS 与 BFS 对比](https://pic.leetcode-cn.com/fdcd3bd27f4008948084f6ec86b58535e71f66862bd89a34bd6fe4cc42d68e89.gif)

这个遍历顺序也是 BFS 能够用来解「层序遍历」、「最短路径」问题的根本原因。下面，我们结合几道例题来讲讲 BFS 是如何求解层序遍历和最短路径问题的。

**BFS 的应用一：层序遍历**

BFS 的层序遍历应用就是本题了：

[LeetCode 102. Binary Tree Level Order Traversal](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/) 二叉树的层序遍历（Medium）

> 给定一个二叉树，返回其按层序遍历得到的节点值。层序遍历即**逐层地、从左到右访问**所有结点。

什么是层序遍历呢？简单来说，层序遍历就是把二叉树分层，然后每一层从左到右遍历：

![二叉树的层序遍历](https://pic.leetcode-cn.com/ce41cf1cabfa7a56387f63d927c8819fe1479ecf6f193a2a1b47964f5a8d1c8e.jpg)

乍一看来，这个遍历顺序和 BFS 是一样的，我们可以直接用 BFS 得出层序遍历结果。然而，层序遍历要求的输入结果和 BFS 是不同的。层序遍历要求我们区分每一层，也就是返回一个二维数组。而 BFS 的遍历结果是一个一维数组，无法区分每一层。

![BFS 遍历与层序遍历的输出结果不同](https://pic.leetcode-cn.com/fd1d63037d0e2f787d2140fee406e109094a4f66ab0837a7273f8b371eef8096.jpg)

那么，怎么给 BFS 遍历的结果分层呢？我们首先来观察一下 BFS 遍历的过程中，结点进队列和出队列的过程：

![BFS 遍历的过程（动图）](https://pic.leetcode-cn.com/4529bf559c6a2d84d550eebaee027c3b7ae25069e4ec91f27b29a4c6358d6662.gif)

截取 BFS 遍历过程中的某个时刻：

![BFS 遍历中某个时刻队列的状态](https://pic.leetcode-cn.com/9f178b56ff1c94388d893f2fb48e9e77e186aba7cfd7483637776359062d68b8.jpg)

可以看到，此时队列中的结点是 3、4、5，分别来自第 1 层和第 2 层。这个时候，第 1 层的结点还没出完，第 2 层的结点就进来了，而且两层的结点在队列中紧挨在一起，我们**无法区分队列中的结点来自哪一层**。

因此，我们需要稍微修改一下代码，在每一层遍历开始前，先记录队列中的结点数量 n（也就是这一层的结点数量），然后一口气处理完这一层的 n 个结点。

```java
// 二叉树的层序遍历
void bfs(TreeNode root) {
    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.add(root);
    while (!queue.isEmpty()) {
        int n = queue.size();
        for (int i = 0; i < n; i++) { 
            // 变量 i 无实际意义，只是为了循环 n 次
            TreeNode node = queue.poll();
            if (node.left != null) {
                queue.add(node.left);
            }
            if (node.right != null) {
                queue.add(node.right);
            }
        }
    }
}
```

这样，我们就将 BFS 遍历改造成了层序遍历。在遍历的过程中，结点进队列和出队列的过程为：

![img](https://pic.leetcode-cn.com/94cd1fa999df0276f1dae77a9cca83f4cabda9e2e0b8571cd9550a8ee3545f56.gif)

可以看到，在 while 循环的每一轮中，都是将当前层的所有结点出队列，再将下一层的所有结点入队列，这样就实现了层序遍历。

最终我们得到的题解代码为：

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();

    Queue<TreeNode> queue = new ArrayDeque<>();
    if (root != null) {
        queue.add(root);
    }
    while (!queue.isEmpty()) {
        int n = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < n; i++) { 
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) {
                queue.add(node.left);
            }
            if (node.right != null) {
                queue.add(node.right);
            }
        }
        res.add(level);
    }

    return res;
}
```

**BFS 的应用二：最短路径**

在一棵树中，一个结点到另一个结点的路径是唯一的，但在图中，结点之间可能有多条路径，其中哪条路最近呢？这一类问题称为**最短路径问题**。最短路径问题也是 BFS 的典型应用，而且其方法与层序遍历关系密切。

在二叉树中，BFS 可以实现一层一层的遍历。在图中同样如此。从源点出发，BFS 首先遍历到第一层结点，到源点的距离为 1，然后遍历到第二层结点，到源点的距离为 2…… 可以看到，用 BFS 的话，距离源点更近的点会先被遍历到，这样就能找到到某个点的最短路径了。

![层序遍历与最短路径](https://pic.leetcode-cn.com/01a3617511b1070216582ae59136888072116ccba360ab7c2aa60fc273351b85.jpg)

> **小贴士：**
>
> 很多同学一看到「最短路径」，就条件反射地想到「Dijkstra 算法」。为什么 BFS 遍历也能找到最短路径呢？
>
> 这是因为，Dijkstra 算法解决的是**带权最短路径问题**，而我们这里关注的是**无权最短路径问题**。也可以看成每条边的权重都是 1。这样的最短路径问题，用 BFS 求解就行了。
>
> 在面试中，你可能更希望写 BFS 而不是 Dijkstra 。毕竟，敢保证自己能写对 Dijkstra 算法的人不多。

最短路径问题属于图算法。由于图的表示和描述比较复杂，本文用比较简单的网格结构代替。网格结构是一种特殊的图，它的表示和遍历都比较简单，适合作为练习题。在 LeetCode 中，最短路径问题也以网格结构为主。

**最短路径例题讲解**

[LeetCode 1162. As Far from Land as Possible](https://leetcode-cn.com/problems/as-far-from-land-as-possible/) 离开陆地的最远距离（Medium）

> 你现在手里有一份大小为 n*×*n 的地图网格 `grid`，上面的每个单元格都标记为 0 或者 1，其中 0 代表海洋，1 代表陆地，请你找出一个海洋区域，这个海洋区域到离它最近的陆地区域的距离是最大的。
>
> 我们这里说的距离是「曼哈顿距离」。(x_0, y_0) 和 (x_1, y_1) 这两个区域之间的距离是 |x_0 - x_1| + |y_0 - y_1| 。
>
> 如果我们的地图上只有陆地或者海洋，请返回 -1。

这道题就是一个在网格结构中求最短路径的问题。同时，它也是一个「岛屿问题」，即用网格中的 1 和 0 表示陆地和海洋，模拟出若干个岛屿。

在上一篇文章中，我们介绍了网格结构的基本概念，以及网格结构中的 DFS 遍历。其中一些概念和技巧也可以用在 BFS 遍历中：

- 格子 `(r, c)` 的相邻四个格子为：`(r-1, c)`、`(r+1, c)`、`(r, c-1)` 和 `(r, c+1)`；
- 使用函数 `inArea` 判断当前格子的坐标是否在网格范围内；
- 将遍历过的格子标记为 2，避免重复遍历。

对于网格结构的性质、网格结构的 DFS 遍历技巧不是很了解的同学，可以复习一下上一篇文章：[LeetCode 例题精讲 | 12 岛屿问题：网格结构中的 DFS](https://leetcode.cn/link/?target=https://mp.weixin.qq.com/s?__biz=MzA5ODk3ODA4OQ==&mid=2648167208&idx=1&sn=d8118c7c0e0f57ea2bdd8aa4d6ac7ab7&chksm=88aa236ebfddaa78a6183cf6dcf88f82c5ff5efb7f5c55d6844d9104b307862869eb9032bd1f&token=1064083695&lang=zh_CN#rd)。

上一篇文章讲过了网格结构 DFS 遍历，这篇文章正好讲解一下网格结构的 BFS 遍历。要解最短路径问题，我们首先要写出层序遍历的代码，仿照上面的二叉树层序遍历代码，类似地可以写出网格层序遍历：

```java
// 网格结构的层序遍历
// 从格子 (i, j) 开始遍历
void bfs(int[][] grid, int i, int j) {
    Queue<int[]> queue = new ArrayDeque<>();
    queue.add(new int[]{r, c});
    while (!queue.isEmpty()) {
        int n = queue.size();
        // 这里是取一个“圈”内的所有节点，层层递进
        for (int i = 0; i < n; i++) { 
            int[] node = queue.poll();
            int r = node[0];
            int c = node[1];
            // 广度优先搜索，往四面扩散遍历
            if (r-1 >= 0 && grid[r-1][c] == 0) {
                grid[r-1][c] = 2;
                queue.add(new int[]{r-1, c});
            }
            if (r+1 < N && grid[r+1][c] == 0) {
                grid[r+1][c] = 2;
                queue.add(new int[]{r+1, c});
            }
            if (c-1 >= 0 && grid[r][c-1] == 0) {
                grid[r][c-1] = 2;
                queue.add(new int[]{r, c-1});
            }
            if (c+1 < N && grid[r][c+1] == 0) {
                grid[r][c+1] = 2;
                queue.add(new int[]{r, c+1});
            }
        }
    }
}
```

以上的层序遍历代码有几个注意点：

- 队列中的元素类型是 `int[]` 数组，每个数组的长度为 2，包含格子的行坐标和列坐标。
- 为了避免重复遍历，这里使用到了和 DFS 遍历一样的技巧：把已遍历的格子标记为 2。注意：我们在将格子放入队列之前就将其标记为 2。想一想，这是为什么？
- 在将格子放入队列之前就检查其坐标是否在网格范围内，避免将「不存在」的格子放入队列。

这段网格遍历代码还有一些可以优化的地方。由于一个格子有四个相邻的格子，代码中判断了四遍格子坐标的合法性，代码稍微有点啰嗦。我们可以用一个 `moves` 数组存储相邻格子的四个方向：

```java
int[][] moves = {
    {-1, 0}, {1, 0}, {0, -1}, {0, 1},
};
```

然后把四个 if 判断变成一个循环：

```java
for (int[][] move : moves) {
    int r2 = r + move[0];
    int c2 = c + move[1];
    if (inArea(grid, r2, c2) && grid[r2][c2] == 0) {
        grid[r2][c2] = 2;
        queue.add(new int[]{r2, c2});
    }
}
```

写好了层序遍历的代码，接下来我们看看如何来解决本题中的最短路径问题。

这道题要找的是距离陆地最远的海洋格子。假设网格中只有一个陆地格子，我们可以从这个陆地格子出发做层序遍历，直到所有格子都遍历完。最终遍历了几层，海洋格子的最远距离就是几。

![从单个陆地格子出发的距离（动图）](https://pic.leetcode-cn.com/8e108e43731bd61a225f79cde11783ae0df17f171974896dc631bbdcda637aa9.gif)

那么有多个陆地格子的时候怎么办呢？一种方法是将每个陆地格子都作为起点做一次层序遍历，但是这样的时间开销太大。

**BFS 完全可以以多个格子同时作为起点**。我们可以把所有的陆地格子同时放入初始队列，然后开始层序遍历，这样遍历的效果如下图所示：

![从多个陆地格子出发的距离](https://pic.leetcode-cn.com/0a7e2f150e95617c19ff7eddb9f2a8d795c23d02b8c1e51a1ff45920d493047b.gif)

这种遍历方法实际上叫做「**多源 BFS**」。多源 BFS 的定义不是今天讨论的重点，你只需要记住多源 BFS 很方便，只需要把多个源点同时放入初始队列即可。

需要注意的是，虽然上面的图示用 1、2、3、4 表示层序遍历的层数，但是在代码中，我们不需要给每个遍历到的格子标记层数，只需要用一个 `distance` 变量记录当前的遍历的层数（也就是到陆地格子的距离）即可。

最终，我们得到的题解代码为：

```java
public int maxDistance(int[][] grid) {
    int N = grid.length;

    Queue<int[]> queue = new ArrayDeque<>();
    // 将所有的陆地格子加入队列
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            if (grid[i][j] == 1) {
                queue.add(new int[]{i, j});
            }
        }
    }
    
    // 如果地图上只有陆地或者海洋，返回 -1
    if (queue.isEmpty() || queue.size() == N * N) {
        return -1;
    }

    int[][] moves = {
        {-1, 0}, {1, 0}, {0, -1}, {0, 1},
    };

    int distance = -1; // 记录当前遍历的层数（距离）
    while (!queue.isEmpty()) {
        distance++;
        int n = queue.size();
        for (int i = 0; i < n; i++) { 
            int[] node = queue.poll();
            int r = node[0];
            int c = node[1];
            for (int[] move : moves) {
                int r2 = r + move[0];
                int c2 = c + move[1];
                if (inArea(grid, r2, c2) && grid[r2][c2] == 0) {
                    grid[r2][c2] = 2;
                    queue.add(new int[]{r2, c2});
                }
            }
        }
    }

    return distance;
}

// 判断坐标 (r, c) 是否在网格中
boolean inArea(int[][] grid, int r, int c) {
    return 0 <= r && r < grid.length 
        && 0 <= c && c < grid[0].length;
}
```

**总结**

可以看到，「BFS 遍历」、「层序遍历」、「最短路径」实际上是递进的关系。在 BFS 遍历的基础上区分遍历的每一层，就得到了层序遍历。在层序遍历的基础上记录层数，就得到了最短路径。

BFS 遍历是一类很值得反复体会和练习的题目。一方面，BFS 遍历是一个经典的基础算法，需要重点掌握。另一方面，我们需要能根据题意分析出题目是要求最短路径，知道是要做 BFS 遍历。

本文讲解的只是两道非常典型的例题。LeetCode 中还有许多层序遍历和最短路径的题目

层序遍历的一些变种题目：

- [LeetCode 103. Binary Tree Zigzag Level Order Traversal](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/) 之字形层序遍历
- [LeetCode 199. Binary Tree Right Side View](https://leetcode-cn.com/problems/binary-tree-right-side-view/) 找每一层的最右结点
- [LeetCode 515. Find Largest Value in Each Tree Row](https://leetcode-cn.com/problems/find-largest-value-in-each-tree-row/) 计算每一层的最大值
- [LeetCode 637. Average of Levels in Binary Tree](https://leetcode-cn.com/problems/average-of-levels-in-binary-tree/) 计算每一层的平均值

对于最短路径问题，还有两道题目也是求网格结构中的最短路径，和我们讲解的距离岛屿的最远距离非常类似：

- [LeetCode 542. 01 Matrix](https://leetcode-cn.com/problems/01-matrix/)
- [LeetCode 994. Rotting Oranges](https://leetcode-cn.com/problems/rotting-oranges/)

还有一道在真正的图结构中求最短路径的问题：

- [LeetCode 310. Minimum Height Trees](https://leetcode-cn.com/problems/minimum-height-trees/)

经过了本文的讲解，相信解决这些题目也不是难事。

## day45

#### [107. 二叉树的层序遍历 II](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/)

- 题目

  给你二叉树的根节点 `root` ，返回其节点值 **自底向上的层序遍历** 。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）

   

  **示例 1：**

  ![img](https://assets.leetcode.com/uploads/2021/02/19/tree1.jpg)

  ```
  输入：root = [3,9,20,null,null,15,7]
  输出：[[15,7],[9,20],[3]]
  ```

  **示例 2：**

  ```
  输入：root = [1]
  输出：[[1]]
  ```

  **示例 3：**

  ```
  输入：root = []
  输出：[]
  ```

   

  **提示：**

  - 树中节点数目在范围 `[0, 2000]` 内
  - `-1000 <= Node.val <= 1000`

- 解法

  **解法一：无脑广度优先**

  在前面一题的思路上，转换一下：只需要在添加当层遍历list时，插入头部即可。

  [题解](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/solution/er-cha-shu-de-ceng-ci-bian-li-ii-by-leetcode-solut/)

  **解法二：不优雅的深度优先遍历**

  将上题设深度优先产生的序列反序即可。

  ```java
  class Solution {
      void level(TreeNode root, int index, List<List<Integer>> res) {
          // 当前行对应的列表不存在，加一个空列表
          if(res.size() < index) {
              res.add(new ArrayList<Integer>());
          }
          // 将当前节点的值加入当前行的 res 中
          res.get(index-1).add(root.val);
          // 递归处理左子树
          if(root.left != null) {
              level(root.left, index+1, res);
          }
          // 递归处理右子树
          if(root.right != null) {
              level(root.right, index+1, res);
          }
      }
      public List<List<Integer>> levelOrderBottom(TreeNode root) {
          if(root == null) {
              return new ArrayList<List<Integer>>();
          }
  
          List<List<Integer>> list = new ArrayList<List<Integer>>();
          List<List<Integer>> res = new ArrayList<List<Integer>>();
          level(root, 1, list);
          for(int i = list.size()-1; i >= 0; i--){
              res.add(list.get(i));
          }
          return res;
      }
  }
  ```

  
