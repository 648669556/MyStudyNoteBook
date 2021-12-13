# 我的leetcode刷题记录



#### 11、盛最多水的容器🧡

给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0) 。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。

说明：你不能倾斜容器。

```java
class Solution {
    public int maxArea(int[] height) {
      int left=0;
      int right=height.length-1;
      int max=0;
      while(left<right){ 
           max=Math.max(max,(right-left)*Math.min(height[left],height[right]));
        if(height[left]>height[right]){ 
            right--;
        }else{
            left++;
        }
      }
    return max;
    }
}
```

> 解题思路

这题关键词 是 从`两端`找，我们比较容易想到使用双指针的方法来写。

本题的难点是，怎么去移动我们的指针。

分以下情况：

- 容器的宽度要比较大
- 容器的高度要大

以此我们可以得出，我们移动指针的时候要考虑每次将**比较矮的那一端**进行移动。

--------------



#### 15、三数之和

给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。

注意：答案中不可以包含重复的三元组。

```java
 static List<List<Integer>> towSum(int[] nums, int left){
        List<List<Integer>> res= new ArrayList<>();
        int target=-nums[left-1];
        int i=left+1;
        int right=nums.length-1;
        while(left<right){
            int sum=nums[left]+nums[right];
            if(sum==target){
                ArrayList<Integer> list=new ArrayList<Integer>();
                list.add(nums[left]);
                list.add(nums[right]);
                list.add(-target);
                res.add(list);
                int r=right;
                do {
                    right--;
                }while (right>0&&nums[right]==nums[r]);
                int l=left;
                do {
                    left++;
                }while (left<nums.length&&nums[left]==nums[l]);
            }
            if(sum>target){
                int r=right;
                do {
                    right--;
                }while (right>0&&nums[right]==nums[r]);
            }else if(sum<target){
                int l=left;
                do {
                    left++;
                }while (left<nums.length&&nums[left]==nums[l]);
            }
        }
        return res;
    }
    public static List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> res =new ArrayList<List<Integer>>();
        for(int i=0;i<nums.length;++i){
            if(nums[i]>0){
                return res;
            }
            if(i!=0&&nums[i]==nums[i-1]){
                continue;
            }
            List<List<Integer>> list = towSum(nums,i+1);
            res.addAll(list);
        }
        return res;
    }
```

> 解题思路

可以这样想，每次取一个数字进来，然后让我们在数组中求另外两个数。题目就变成了towSum问题，本题的难点在于怎么去除找到的数组中重复的项。

首先我们可以通过对数组的排序然后遍历，将那个值作为另外两个数字的目标值。

这样就已经去除了很多的重复的。

如果值>0则就可以不继续进行遍历了，后面的数字不可能让和为0了。

几个重点：

- 遍历放入的值不能和上一个数字重复
- 双指针查找数字的时候也要跳过重复的数字

-----------------

#### 39、组合总数

给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的数字可以无限制重复被选取。

说明：

    所有数字（包括 target）都是正整数。
    解集不能包含重复的组合。 

示例 1：

输入：candidates = [2,3,6,7], target = 7,
所求解集为：
[
  [7],
  [2,2,3]
]

示例 2：

输入：candidates = [2,3,5], target = 8,
所求解集为：
[
  [2,2,2,2],
  [2,3,3],
  [3,5]
]

 

提示：

    1 <= candidates.length <= 30
    1 <= candidates[i] <= 200
    candidate 中的每个元素都是独一无二的。
    1 <= target <= 500

```java
 static private void dfs(int[] nums, List<List<Integer>> res, int target, List<Integer> list,int index) {
        if (target < 0) {
            return;
        }
        if (target == 0) {
            res.add(new LinkedList<Integer>(list));
            return;
        }
        for (int i = index; i < nums.length; i++) {
            if (target < nums[i]) {
                break;
            }
            list.add(nums[i]);
            dfs(nums, res, target - nums[i], list,i);
            list.remove(list.size()-1);
        }
    }
   static public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> res = new ArrayList<>();
        Arrays.sort(candidates);
        dfs(candidates, res, target, new LinkedList<>(),0);
        return res;
    }
```

> 解题思路

一开始就可以看到这是一个经典的回溯的题目，先写出回溯的模板。

然后就是剪枝的问题了；

怎么去剪枝呢？

- 我们可以通过对数组进行排序，然后在target大于当前数字的时候结束
- 我们可以不用每次都从最小的开始寻找，我们可以从当前数字开始寻找。因为之前比较小的数字已经寻找过了。

------------



#### 41、缺失的第一个正数

给你一个未排序的整数数组，请你找出其中没有出现的最小的正整数。

 

示例 1:

输入: [1,2,0]
输出: 3

示例 2:

输入: [3,4,-1,1]
输出: 2

示例 3:

输入: [7,8,9,11,12]
输出: 1

 

提示：

你的算法的时间复杂度应为O(n)，并且只能使用常数级别的额外空间。

```java
static public int firstMissingPositive(int[] nums) {
        int n = nums.length;
        for (int i = 0; i < n; ++i) {
            if (nums[i] <= 0) {
                nums[i] = n + 1;
            }
        }
        for (int i = 0; i < n; ++i) {
            int num = Math.abs(nums[i]);
            if (num <= n) {
                nums[num - 1] = -Math.abs(nums[num - 1]);
            }
        }
        for (int i = 0; i < n; ++i) {
            if (nums[i] > 0) {
                return i + 1;
            }
        }
        return n + 1;
    }
```

> 解题思路

这题难点在与，解题思路。如果不管时间复杂度的话，可以简单的用一个快排然后遍历一遍就可以了。

如果不在乎空间复杂度。我们可以用一个set，先遍历一遍数组，存储值。然后从1开始向后遍历返回第一个没有在set里面找到的那个就可以了。

O(n)的时间复杂度，空间为1的话。

我们需要仿照set那个。在原来的数组上进行改造。

首先我们把干扰项去除掉。

把数组中`<=0`的数字都变为`数组的length+1`。

然后我们再遍历一边，这时候我们的策略是将出现的数字对应在数组中的下标设置为负数。

最后遍历的时候我们返回从1开始第一个不是负数的就可以了。

----------------

#### 42、接雨水😘【双指针】【单调栈】

给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

 

示例 1：![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/rainwatertrap.png)

输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
		输出：6
		解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。 

示例 2：

输入：height = [4,2,0,3,2,5]
输出：9

 

提示：

    n == height.length
    0 <= n <= 3 * 104
    0 <= height[i] <= 105

> 单调栈

```java
static public int trap(int[] height) {
        int len = height.length;
        int res = 0;
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i < len; ++i) {
            while (!stack.isEmpty() && height[i] > height[stack.peekLast()]) {
                int low = height[stack.removeLast()];
                int width;
                while (!stack.isEmpty() && height[stack.peekLast()] == low) {
                    stack.removeLast();
                }
                int area = 0;
                if (stack.isEmpty()) {
                    width = i;
                    area = 0;
                } else {
                    width = i - stack.peekLast() - 1;
                    area = width * (Math.min(height[i], height[stack.peekLast()])-low);
                }
                res+=area;
            }
            stack.addLast(i);
        }
        return res;
    }
```

单调栈，首先我们将单调递减的柱体放入，当遇到比前面的高的柱子的时候，我们就会去计算一次蓄水量，这种先进后出的形式，我们可以使用我们栈的数据结构来帮助我们计算。当我们遍历的时候，就去放入栈。

> 双指针法

```java
static public int trap(int[] height) {
        int length = height.length;
        int left = 0, right = length-1;
        int res = 0;
        int max_left=0,max_right=0;

        while(left<right){
            if(height[left]<height[right]){
             if(height[left]>max_left){
                 max_left=height[left];
             }else {
                 res+=max_left-height[left];
             }
             left++;
            }else {
                if(height[right]>max_right){
                    max_right=height[right];
                }else {
                    res+=max_right-height[right];
                }
                right--;
            }
        }
        return res;
    }
```

双指针法比较难想，就是我们选择左右两个端点，作为我们的左边或者右边的最高的面，当我们去遍历左边的时候，寻找一个单调递减的并计算蓄水量。如果左边的边界比右边的大了，我们就去计算右边的。

--------------------

#### 49、异位词分词

给定一个字符串数组，将字母异位词组合在一起。字母异位词指字母相同，但排列不同的字符串。

示例:

输入: ["eat", "tea", "tan", "ate", "nat", "bat"]
输出:
[
  ["ate","eat","tea"],
  ["nat","tan"],
  ["bat"]
]

说明：

    所有输入均为小写字母。
    不考虑答案输出的顺序。

```java
 public static List<List<String>> groupAnagrams(String[] strs) {
            List<List<String>> lists = new ArrayList<>();
            HashMap<String,List<String>> map =new HashMap<>();
            for (String str : strs) {
                char[] chars = str.toCharArray();
                Arrays.sort(chars);
                String s = new String(chars);
                if(map.containsKey(s)){
                    List<String> strings = map.get(s);
                    strings.add(str);
                }else {
                    ArrayList<String> strings = new ArrayList<>();
                    strings.add(str);
                    map.put(s,strings);
                }
            }
            Set<String> strings = map.keySet();
            for (String string : strings) {
                lists.add(map.get(string));
            }
            return lists;
        }
```

> 解题思路

模式识别：如果是需要分类的题目，一般需要用到散列表。

这题最主要的问题是， 我们怎么找到散列表的key？

有两种方法：

- 排序
- 记数

排序我们可以通过api的方式方便快捷的实现。

记数我们可以这样。

#1#2#0#0##...#0

类似这样的字符串结构，需要我们自己去写一个函数来完成。

#### 62、不同路径

一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

**示例 1：**

![img](https://assets.leetcode.com/uploads/2018/10/22/robot_maze.png)

```
输入：m = 3, n = 7
输出：28
```

**示例 2：**

```
输入：m = 3, n = 2
输出：3
解释：
从左上角开始，总共有 3 条路径可以到达右下角。
1. 向右 -> 向右 -> 向下
2. 向右 -> 向下 -> 向右
3. 向下 -> 向右 -> 向右
```

**示例 3：**

```
输入：m = 7, n = 3
输出：28
```

**示例 4：**

```
输入：m = 3, n = 3
输出：6
```

 

**提示：**

- `1 <= m, n <= 100`
- 题目数据保证答案小于等于 `2 * 109`



```java
  static public int uniquePaths(int m, int n) {
        int[][] dp=new int[m][n];
        for(int i=0;i<m;++i){
            dp[i][0]=1;
        }
        for(int i=0;i<n;++i){
            dp[0][i]=1;
        }
        for(int i=1;i<m;i++){
            for(int j=1;j<n;++j){
                dp[i][j]=dp[i-1][j]+dp[i][j-1];
            }
        }
        return dp[m-1][n-1];
    }
```

> 解题思路

非常基础的dp题目，把左边和上边的可以走的次数加起来就是了。

------------

#### 75、颜色分类

给定一个包含红色、白色和蓝色，一共 n 个元素的数组，原地对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

此题中，我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。

 

进阶：

    你可以不使用代码库中的排序函数来解决这道题吗？
    你能想出一个仅使用常数空间的一趟扫描算法吗？

 










示例 1：

输入：nums = [2,0,2,1,1,0]
输出：[0,0,1,1,2,2]

示例 2：

输入：nums = [2,0,1]
输出：[0,1,2]

示例 3：

输入：nums = [0]
输出：[0]

示例 4：

输入：nums = [1]
输出：[1]

 

提示：

    n == nums.length
    1 <= n <= 300
    nums[i] 为 0、1 或 2

```java
 public void sortColors(int[] nums) {
        int zero=-1;
        int one =0;
        int two=nums.length;
        while(one<two){
            if(nums[one]==0){
                swap(nums,one++,++zero);
            }else if(nums[one]==2){
                swap(nums,one,--two);
            }else {
                one++;
            }
        }
    }

    public void swap(int[] nums,int a,int b){
        int temp=nums[a];
        nums[a]=nums[b];
        nums[b]=temp;
    }
```

> 解题思路

可以利用三指针的思路，最左边代表的是0，中间代表的是1，最右边代表的是2.

把0号指针指向数组-1，把2号指针指向数组length，中间指针从0开始向后遍历，如果遇到0，就和0号指针交换，并且+1，如果遇到了2号指针就2号指针减一。

#### 78、子集🧡

给定一组**不含重复元素**的整数数组 *nums*，返回该数组所有可能的子集（幂集）。

**说明：**解集不能包含重复的子集。

**示例:**

```
输入: nums = [1,2,3]
输出:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```

```java
 static public void dfs(int[] nums,List<List<Integer>> res,List<Integer> list,int index){
        res.add(new ArrayList<>(list));
        for (int i = index; i < nums.length; i++) {
            list.add(nums[i]);
            dfs(nums,res,list,i+1);
            list.remove(list.size()-1);
        }
    }
    static public List<List<Integer>> subsets(int[] nums) {
        int[] dp=new int[nums.length];
        List<List<Integer>> res = new ArrayList<>();
        dfs(nums,res,new ArrayList<>(),0);
        return res;
    }
```

> 解题思路

看到不含重复元素的整数数组，然后还是不能包含重复的子集。我们可以想到使用回溯法，然后剪枝。

这题的难点在于剪枝。怎么去剪枝呢？

因为题目里面已经说明了是`不含重复元素`所以我们只用把一个数与他之后的数字进行连接就好了，他之前的数字已经是使用过了才会到他的。

-----------------

#### 79、单词搜索🧡

给定一个二维网格和一个单词，找出该单词是否存在于网格中。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

 

示例:

board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

给定 word = "ABCCED", 返回 true
给定 word = "SEE", 返回 true
给定 word = "ABCB", 返回 false

 

提示：

    board 和 word 中只包含大写和小写英文字母。
    1 <= board.length <= 200
    1 <= board[i].length <= 200
    1 <= word.length <= 10^3

```java
 static public boolean dfs(int[][] dp, char[][] nums, char[] target, int index, int y, int x) {
        if (y < 0 || y == nums.length || x < 0 || x == nums[0].length || dp[y][x] == 1) {
            return false;
        }
        if (nums[y][x] != target[index]) {
            return false;
        }
        if (index == target.length - 1) {
            return true;
        }
        int[][] direction = {
                {0, 1},
                {1, 0},
                {-1, 0},
                {0, -1}
        };
        dp[y][x] = 1;
        for (int i = 0; i < 4; i++) {
            int nextX = x + direction[i][0];
            int nextY = y + direction[i][1];
            boolean dfs = dfs(dp, nums, target, index + 1, nextY ,nextX);
            if(dfs){
                return true;
            }
        }
        dp[y][x] = 0;
        return false;
    }

    static public boolean exist(char[][] board, String word) {
        int n = board.length;
        int m = board[0].length;
        int[][] dp = new int[n][m];
        char[] chars = word.toCharArray();
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (board[i][j] == chars[0]) {
                    boolean dfs = dfs(dp, board, chars, 0, i, j);
                    if (dfs) {
                        return dfs;
                    }
                }
            }
        }
        return false;
    }
```

> 解题思路

这题，就是回溯查找的题目，首先需要我们在图中寻找“蛇头”,然后我们找到了头部后就继续去找后面的身体部分，在四个方向上。如果有一个不一样则立即返回去找另个方向的。就是深度优先搜索加上剪枝的题目。经典

#### 80、删除排序数组中的重复项Ⅱ

给定一个增序排列数组 nums ，你需要在 原地 删除重复出现的元素，使得每个元素最多出现两次，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。

 

说明：

为什么返回数值是整数，但输出的答案是数组呢？

请注意，输入数组是以“引用”方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下：

// nums 是以“引用”方式传递的。也就是说，不对实参做任何拷贝
int len = removeDuplicates(nums);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中该长度范围内的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}

 

示例 1：

输入：nums = [1,1,1,2,2,3]
输出：5, nums = [1,1,2,2,3]
解释：函数应返回新长度 length = 5, 并且原数组的前五个元素被修改为 1, 1, 2, 2, 3 。 你不需要考虑数组中超出新长度后面的元素。

示例 2：

输入：nums = [0,0,1,1,1,1,2,3,3]
输出：7, nums = [0,0,1,1,2,3,3]
解释：函数应返回新长度 length = 7, 并且原数组的前五个元素被修改为 0, 0, 1, 1, 2, 3, 3 。 你不需要考虑数组中超出新长度后面的元素。

 

提示：

    0 <= nums.length <= 3 * 104
    -104 <= nums[i] <= 104
    nums 按递增顺序排列

```java
static public int removeDuplicates(int[] nums) {
        int i = 0;
        int j = 0;
        int n = 0;
        int current = nums[i];
        int count = 0;
        while (j < nums.length) {
            if (current == nums[j]) {
                if (count < 2) {
                    nums[i++] = nums[j++];
                    count++;
                    n++;
                } else {
                    while (j < nums.length && nums[j] == current) {
                        j++;
                    }
                    if (j == nums.length) {
                        break;
                    }
                    current = nums[j];
                    count = 0;
                }
            } else {
                current = nums[j];
                nums[i++] = nums[j++];
                count = 1;
                n++;
            }
        }
        return n;
    }
```

> 解题思路

这个题目还是蛮好想的，就是一个快慢指针，一个去维护数组，一个去向后遍历然后判断是否可以插入，如果可以插入就插入到第一个指针的位置，不可以就继续向后寻找。

------------

#### 84、柱状图中最大的矩形💗

给定 n 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1 。

求在该柱状图中，能够勾勒出来的矩形的最大面积。

 ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/histogram.png)

以上是柱状图的示例，其中每个柱子的宽度为 1，给定的高度为 [2,1,5,6,2,3]。

 ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/histogram_area.png)

图中阴影部分为所能勾勒出的最大矩形面积，其面积为 10 个单位。

 

示例:

输入: [2,1,5,6,2,3]
输出: 10

暴力解法：

```java
static public int largestRectangleArea(int[] heights) {
        int len = heights.length;
        if (len == 0) {
            return 0;
        }
        if (len == 1) {
            return heights[0];
        }
        int area=0;
        for(int i=0;i<len;++i){
            int height=heights[i];
            int left=i,right=i;
            while(left>=1&&heights[left-1]>=height){
                left--;
            }
            while(right<len-1&&heights[right+1]>=height){
                right++;
            }
            int width=right-left+1;
            area=Math.max(area,height*width);
        }
        return area;
    }
```

> 暴力写法

暴力写法比较容易想到，在java中也可以通过测试用例。

面积 = 长度 * 高度

我们可以固定高度去求长度。

我们从每一个柱子开始，向左右两边查找大于等于他的高度柱子，最后我们就可以求得长度。

每一个都去遍历。

>  单调栈写法

```java
 static public int largestRectangleArea(int[] heights) {
        int len = heights.length;
        if (len == 0) {
            return 0;
        }
        if (len == 1) {
            return heights[0];
        }
        int area = 0;
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i < len; ++i) {
            while (!stack.isEmpty()&&heights[stack.peekLast()] > heights[i]) {
                int height = heights[stack.removeLast()];
                int width;
                if(stack.isEmpty()){
                    width=i;
                }else {
                    width=i-stack.peekLast()-1;
                }
                area = Math.max(area, height * width);
            }
            stack.addLast(i);
        }
        while (!stack.isEmpty()) {
            int height = heights[stack.removeLast()];
            int width;
            if(stack.isEmpty()){
                width=len;
            }else {
                width=len-stack.peekLast()-1;
            }
            area = Math.max(area, height * width);
        }
        return area;
    }
```

从暴力写法我们可以发现，每次我们去求一个柱子可以产生的最大矩形的时候，如果柱子的右边的高度大于等于他那么就无法确定他的最大面积。只有遇到了比他矮的柱子才可以确定他的最大面积。这符合我们先进后出的思想，我们可以考虑使用栈来辅助我们计算。

哨兵改进写法

```java
static public int largestRectangleArea(int[] heights) {
        int len = heights.length;
        if (len == 0) {
            return 0;
        }
        if (len == 1) {
            return heights[0];
        }
        int area = 0;
        int[] temp = new int[len + 2];
        for (int i = 0; i < len; i++) {
            temp[i + 1] = heights[i];
        }
        len += 2;
        heights = temp;
        Deque<Integer> stack = new ArrayDeque<>();
        stack.addLast(0);
        for (int i = 1; i < len; ++i) {
            while (heights[stack.peekLast()] > heights[i]) {
                int height = heights[stack.removeLast()];
                int width = i - stack.peekLast() - 1;
                area = Math.max(area, height * width);
            }
            stack.addLast(i);
        }
        return area;
    }
```

上面的那种写法我们对于边界的判断是通过人为的去判断的，显得有些繁琐。我们可以使用设置哨兵的方式进一步优化我们的代码。

----------

#### 209、长度最小的子数组😮

给定一个含有 n 个正整数的数组和一个正整数 target 。

找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。

示例 1：

输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。

示例 2：

输入：target = 4, nums = [1,4,4]
输出：1

示例 3：

输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0

 

提示：

    1 <= target <= 109
    1 <= nums.length <= 105
    1 <= nums[i] <= 105

 








进阶：

    如果你已经实现时间复杂度O(n log(n)) 的解法, 请尝试设计一个 O(n)  时间复杂度的解法。

> 前缀和数组加 二分查找

首先我们可以先把数组逐个加在一起，这样我们可以得到一个单调递增的数组，然后根据有序性我们可以在 起始点范围 `l`到 我们找到的点之间二分寻找我们的目标，然后每次判断是否是最小并更新。

代码如下：

```java
 public int minSubArrayLen(int target, int[] nums) {
        int [] dp = new int [nums.length+1];
        for(int i =1;i<=nums.length;++i){
            dp[i]=dp[i-1]+nums[i-1];
        }
        int res = Integer.MAX_VALUE;
        for(int i=1;i<dp.length;i++){
            int l =i;
            int r = dp.length-1;
            while(l<r){
                int mid = l+(r-l)/2;
                if(dp[mid]-dp[i-1]>=target){
                    r = mid;
                }else if(dp[mid]-dp[i-1]<target){
                    l = mid+1;
                }
            } 
            if(dp[l]-dp[i-1]>=target){
                res = Math.min(res,l-i+1);
            }
        }
        return res==Integer.MAX_VALUE?0:res;
    }
```

这里有个细节就是我们需要安放一个哨兵在头部，来防止 超出数组范围。

> 滑动窗口

我们可以在一边遍历的时候一边维护一个滑动窗口，当加入当前 `i`的元素的时候，如果去除队头的元素依然符合我们的要求，则我们循环将队头出队。并同时更新最小长度。

```java
 public int minSubArrayLen(int target, int[] nums) {
      int l = 0;
      int len = nums.length;
      int sum = 0;
      int res = Integer.MAX_VALUE;
      for(int i = 0 ; i < len ; i++){
            while(sum+nums[i]-nums[l]>=target){
                  sum-=nums[l];
                  l++;
              }
            sum+=nums[i];
          if(sum>=target){
              res = Math.min(res,i-l+1);
          }
      }
      return res==Integer.MAX_VALUE?0:res;
    }
```

----------

#### 316、去除重复字母😡

给你一个字符串 s ，请你去除字符串中重复的字母，使得每个字母只出现一次。需保证 返回结果的字典序最小（要求不能打乱其他字符的相对位置）。

注意：该题与 1081 https://leetcode-cn.com/problems/smallest-subsequence-of-distinct-characters 相同

 

示例 1：

输入：s = "bcabc"
输出："abc"

示例 2：

输入：s = "cbacdcbc"
输出："acdb"

 

提示：

    1 <= s.length <= 104
    s 由小写英文字母组成

```java
 static public String removeDuplicateLetters(String s) {
        char[] chars = s.toCharArray();
        int length = s.length();
        boolean[] visted=new boolean[26];
        int[] lastindex=new int[26];
        for(int i=0;i<length;++i){
            lastindex[chars[i]-'a']=i;
        }
        Deque<Character> stack=new ArrayDeque<>();
        for(int i=0;i<length;++i){
            if(visted[chars[i]-'a']){
                continue;
            }
            while(!stack.isEmpty()&&stack.peekLast()>chars[i]&&lastindex[stack.peekLast()-'a']>i){
                visted[stack.peekLast()-'a']=false;
                stack.removeLast();
            }
                stack.addLast(chars[i]);
                visted[chars[i]-'a']=true;
        }
        StringBuilder sb=new StringBuilder();
        for (Character character : stack) {
            sb.append(character);
        }
        return sb.toString();
    }
```

> 解题思路

我们先读题目，然后观察，发现如果每次去比较当前的字母和前面的字母大小，如果当前的字母大小小于前面的大小的时候，我们看看这个前面的字母是否是在当前的最后一个了，如果不是最后一个说明我们可以放在当前这个字母的后面，我们就把前面的字母弹出，将当前的字母放入，当然如果遇到了前面已经放入了的字母我们可以直接跳过。

-------

#### 363、矩形区域不超过K的最大数值和

给你一个 m x n 的矩阵 matrix 和一个整数 k ，找出并返回矩阵内部矩形区域的不超过 k 的最大数值和。

题目数据保证总会存在一个数值和不超过 k 的矩形区域。

 ![img](https://assets.leetcode.com/uploads/2021/03/18/sum-grid.jpg)

示例 1：

输入：matrix = [[1,0,1],[0,-2,3]], k = 2
输出：2
解释：蓝色边框圈出来的矩形区域 [[0, 1], [-2, 3]] 的数值和是 2，且 2 是不超过 k 的最大数字（k = 2）。

示例 2：

输入：matrix = [[2,2,-1]], k = 3
输出：3

提示：

    m == matrix.length
    n == matrix[i].length
    1 <= m, n <= 100
    -100 <= matrix[i][j] <= 100
    -105 <= k <= 105

> 二维前缀和

```java
 public int maxSumSubmatrix(int[][] matrix, int target) {
        int n=matrix.length;
        int m=matrix[0].length;
        int res = -Integer.MAX_VALUE;
        int[][] dp=new int[n+1][m+1];
        for(int i=1;i<=n;i++){
            for(int j=1;j<=m;++j){
                dp[i][j]=dp[i-1][j]+dp[i][j-1]+matrix[i-1][j-1]-dp[i-1][j-1];
            }
        }
        for(int i = 1;i<=n;++i){
            for(int j =1;j<=m;++j){
                for(int k=i;k<=n;++k){
                    for(int l=j;l<=m;l++){
                        int cur = dp[k][l] + dp[i-1][j-1] - dp[k][j-1] - dp[i-1][l];
                        if(cur<=target){
                            res = Math.max(res,cur);
                        }
                    }
                }
            }
        }
        return res;
    }
```

这个的思路很简单，就是我们用数组存储从0号位置到当前这个位置的这个矩形的大小。

然后就可以通过计算获得矩阵的大小。之后就是判断是否符合条件然后替换就可以了。

后面优化的话可以这样，我们可以先确定矩形的3条边，最后那个边用二分去查找。

----------

#### 368、最大整除子集

给你一个由 无重复 正整数组成的集合 nums ，请你找出并返回其中最大的整除子集 answer ，子集中每一元素对 (answer[i], answer[j]) 都应当满足：

    answer[i] % answer[j] == 0 ，或
    answer[j] % answer[i] == 0

如果存在多个有效解子集，返回其中任何一个均可。

 

示例 1：

输入：nums = [1,2,3]
输出：[1,2]
解释：[1,3] 也会被视为正确答案。

示例 2：

输入：nums = [1,2,4,8]
输出：[1,2,4,8]

 

提示：

    1 <= nums.length <= 1000
    1 <= nums[i] <= 2 * 109
    nums 中的所有整数 互不相同

```java
static public List<Integer> largestDivisibleSubset(int[] nums) {
        Arrays.sort(nums);
        int[] dp = new int[nums.length];
        int[] g = new int[nums.length];
        Arrays.fill(g,-1);
        int res = 0;
        for (int i = 0; i < nums.length; ++i) {
            int j = i;
            while (j > 0) {
                j--;
                if ((nums[i] % nums[j]) == 0) {
                    if (dp[j] + 1 > dp[i]) {
                        dp[i] = dp[j] + 1;
                        g[i] = j;
                    }
                }
            }
            if (dp[i] > dp[res]) {
                res = i;
            }
        }
        List<Integer> list = new ArrayList<>();
        while (res >= 0) {
            list.add(0,nums[res]);
            res = g[res];
        }
        return list;
    }
```

> 解题思路

我们可以将数组先排序，然后寻找当前的这个数前面最长的一个列加一个。题目就变成了经典的`最长递增子序列` 然后我们就可以这样判断：

- ​	如果前面没有可以整除的数字，那么我们就把状态设置为1。
- 如果前面有可以整除的数字，则我们将当前的状态设置为可以整除的数字中最长的。

然后就是我们需要记录一下状态的序列。

-----------

#### 377、组合总和4

给你一个由 不同 整数组成的数组 nums ，和一个目标整数 target 。请你从 nums 中找出并返回总和为 target 的元素组合的个数。

题目数据保证答案符合 32 位整数范围。

 

示例 1：

输入：nums = [1,2,3], target = 4
输出：7
解释：
所有可能的组合为：
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)
请注意，顺序不同的序列被视作不同的组合。

示例 2：

输入：nums = [9], target = 3
输出：0

 

提示：

    1 <= nums.length <= 200
    1 <= nums[i] <= 1000
    nums 中的所有元素 互不相同
    1 <= target <= 1000

 







进阶：如果给定的数组中含有负数会发生什么？问题会产生何种变化？如果允许负数出现，需要向题目中添加哪些限制条件？

```java
 static public int combinationSum4(int[] nums, int target) {
        int[] f = new int[target + 1];
        f[0] = 1;
        for (int i = 1; i <= target; ++i) {
            for (int j : nums) {
                if (i >= j) {
                    f[i] += f[i - j];
                }
            }
        }
        return f[target];
    }
```

> 解题思路

dp存储的是当前 加起来总和为i 的有多少种方案。

我们可以知道 dp[0] 总和为 0 的方案只有一种，所以dp[0] = 1;天然成立，我们可以知道，

- dp[i] = [dp[target - j]],( i<=target , j <= i )

这样的一个转移方程。

----

#### 738、单调递增的数字

给定一个非负整数 N，找出小于或等于 N 的最大的整数，同时这个整数需要满足其各个位数上的数字是单调递增。

（当且仅当每个相邻位数上的数字 x 和 y 满足 x <= y 时，我们称这个整数是单调递增的。）

示例 1:

输入: N = 10
输出: 9

示例 2:

输入: N = 1234
输出: 1234

示例 3:

输入: N = 332
输出: 299

说明: N 是在 [0, 10^9] 范围内的一个整数。

```java
static public int monotoneIncreasingDigits(int n) {
        char[] chars=String.valueOf(n).toCharArray();
        int length=chars.length;
        int index=0;
        while(index>=0&&index<length-1){
            if(chars[index]>chars[index+1]){
                chars[index]--;
                Arrays.fill(chars,index+1,length,'9');
                index--;
            }
            else {
                index++;
            }
        }
        return Integer.parseInt(new String(chars));
    }
```

> 解题思路

利用贪心算法，我们可以从头开始向后遍历，如果遇到比后面的大的数字了，就将这个数字减一并将后面的数字全部写为9.然后再向前检查一下是否符合递增的规则。以此类推。

------------

#### 1011、在D天内送达包裹的能力

传送带上的包裹必须在 D 天内从一个港口运送到另一个港口。

传送带上的第 i 个包裹的重量为 weights[i]。每一天，我们都会按给出重量的顺序往传送带上装载包裹。我们装载的重量不会超过船的最大运载重量。

返回能在 D 天内将传送带上的所有包裹送达的船的最低运载能力。

 

示例 1：

输入：weights = [1,2,3,4,5,6,7,8,9,10], D = 5
输出：15
解释：
船舶最低载重 15 就能够在 5 天内送达所有包裹，如下所示：
第 1 天：1, 2, 3, 4, 5
第 2 天：6, 7
第 3 天：8
第 4 天：9
第 5 天：10

请注意，货物必须按照给定的顺序装运，因此使用载重能力为 14 的船舶并将包装分成 (2, 3, 4, 5), (1, 6, 7), (8), (9), (10) 是不允许的。 

示例 2：

输入：weights = [3,2,2,4,1,4], D = 3
输出：6
解释：
船舶最低载重 6 就能够在 3 天内送达所有包裹，如下所示：
第 1 天：3, 2
第 2 天：2, 4
第 3 天：1, 4

示例 3：

输入：weights = [1,2,3,1,1], D = 4
输出：3
解释：
第 1 天：1
第 2 天：2
第 3 天：3
第 4 天：1, 1

 

提示：

    1 <= D <= weights.length <= 50000
    1 <= weights[i] <= 500

> 解题思路

```java
boolean isSuccess(int[] weights, int target,int cur){
        int temp = 1;
        int temp_weight=0;
        for(int i:weights){
            if(i>cur)return false;
            if(temp_weight+i<=cur){
                temp_weight+=i;
            }else{
                temp++;
                temp_weight=i;
            }
        }
        return temp<=target;
    }

    public int shipWithinDays(int[] weights, int D) {
        int total = 0;
        for(int num:weights){
            total+=num;
        }
        int l=1,r=total;
        while(l<r){
            int mid = (l+r)>>>1;
            if(isSuccess(weights,D,mid)){
                r=mid;
            }else{
                l=mid+1;
            }
        }
        return l;
    }
```

题目需要我们求出N天内最小的负载。

暴力的方法我们这样想。 最小的负载不能小于1 不能大于所有货物加在一起的总和。 那么我们可以从1到 总和进行遍历，然后判断 `i`[负载] 是否可以在规定的时间内完成。但是这样的复杂度较高，看数据范围知道。所以需要我们进行优化。
			我们可以知道这个范围的两个端点，1~limit，并且是有序的 这样我们可以想到使用二分的方式来查找，这样就可以很快的找到我们答案。

----------

#### 1473、粉刷房子Ⅲ

在一个小城市里，有 m 个房子排成一排，你需要给每个房子涂上 n 种颜色之一（颜色编号为 1 到 n ）。有的房子去年夏天已经涂过颜色了，所以这些房子不需要被重新涂色。

我们将连续相同颜色尽可能多的房子称为一个街区。（比方说 houses = [1,2,2,3,3,2,1,1] ，它包含 5 个街区  [{1}, {2,2}, {3,3}, {2}, {1,1}] 。）

给你一个数组 houses ，一个 m * n 的矩阵 cost 和一个整数 target ，其中：

    houses[i]：是第 i 个房子的颜色，0 表示这个房子还没有被涂色。
    cost[i][j]：是将第 i 个房子涂成颜色 j+1 的花费。

请你返回房子涂色方案的最小总花费，使得每个房子都被涂色后，恰好组成 target 个街区。如果没有可用的涂色方案，请返回 -1 。

 

示例 1：

输入：houses = [0,0,0,0,0], cost = [[1,10],[10,1],[10,1],[1,10],[5,1]], m = 5, n = 2, target = 3
输出：9
解释：房子涂色方案为 [1,2,2,1,1]
此方案包含 target = 3 个街区，分别是 [{1}, {2,2}, {1,1}]。
涂色的总花费为 (1 + 1 + 1 + 1 + 5) = 9。

示例 2：

输入：houses = [0,2,1,2,0], cost = [[1,10],[10,1],[10,1],[1,10],[5,1]], m = 5, n = 2, target = 3
输出：11
解释：有的房子已经被涂色了，在此基础上涂色方案为 [2,2,1,2,2]
此方案包含 target = 3 个街区，分别是 [{2,2}, {1}, {2,2}]。
给第一个和最后一个房子涂色的花费为 (10 + 1) = 11。

示例 3：

输入：houses = [0,0,0,0,0], cost = [[1,10],[10,1],[1,10],[10,1],[1,10]], m = 5, n = 2, target = 5
输出：5

示例 4：

输入：houses = [3,1,2,3], cost = [[1,1,1],[1,1,1],[1,1,1],[1,1,1]], m = 4, n = 3, target = 3
输出：-1
解释：房子已经被涂色并组成了 4 个街区，分别是 [{3},{1},{2},{3}] ，无法形成 target = 3 个街区。

 

提示：

    m == houses.length == cost.length
    n == cost[i].length
    1 <= m <= 100
    1 <= n <= 20
    1 <= target <= m
    0 <= houses[i] <= n
    1 <= cost[i][j] <= 10^4

> 动态规划

```java
static public int minCost(int[] houses, int[][] cost, int m, int n, int target) {
        int inf = 0x3f3f3f;
        int[][][] dp = new int[m + 1][n + 1][target + 1];
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                dp[i][j][0] = inf;
            }
        }
        for (int i = 1; i <= m; i++) {
            int color = houses[i - 1];
            for (int j = 1; j <= n; j++) {
                for (int k = 1; k <= target; k++) {
                    if (k > i) {
                        dp[i][j][k] = inf;
                        continue;
                    }
                    if (color != 0) {
                        if (j == color) {
                            int temp = inf;
                            for (int l = 1; l <= n; l++) {
                                /*
                                寻找上一个颜色不一样的形成不同分区的最小代价
                                 */
                                if (l != j) {
                                    temp = Math.min(temp, dp[i - 1][l][k - 1]);
                                }
                            }
                            //比较如果不产生新的分区的代价比较
                            dp[i][j][k] = Math.min(temp, dp[i - 1][j][k]);
                        } else {
                            dp[i][j][k] = inf;
                        }
                    } else {
                        int u = cost[i - 1][j - 1];
                        int temp = inf;
                        for (int l = 1; l <= n; ++l) {
                            if (l != j) {
                                temp = Math.min(temp, dp[i - 1][l][k - 1]);
                            }
                        }
                        dp[i][j][k] = Math.min(dp[i - 1][j][k], temp) + u;
                    }
                }
            }
        }
        int ans = inf;
        for (int i = 1; i <= n; i++) {
            ans = Math.min(ans, dp[m][i][target]);
        }
        return ans == inf ? -1 : ans;
    }
```

