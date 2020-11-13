## 树状数组

### 应用

- 区间求和
- 单点更新

### 实现

```java
class BIT {
    int[] tree;
    int n;

    public BIT(int n) {
        this.n = n;
        this.tree = new int[n + 1];
    }

    // 单点更新
    void update(int i, int val) {
        while (i <= n) {
            tree[i] += val;
            i += lowbit(i);
        }
    }

    // 前缀和查询
    int query(int i) {
        int sum = 0;
        while (i > 0) {
            sum += tree[i];
            i -= lowbit(i);
        }
        return sum;
    }

    // 只保留最后一个1
    int lowbit(int x) {
        return x & -x;
        // return x & (~x + 1);
    }
}
```

## 例题

### leetcode 315

**计算右侧小于当前元素的个数**

给定一个整数数组 nums，按要求返回一个新数组 counts。数组 counts 有该性质： counts[i] 的值是  nums[i] 右侧小于 nums[i] 的元素的数量。

 ```
示例：

输入：nums = [5,2,6,1]
输出：[2,1,1,0] 
解释：
5 的右侧有 2 个更小的元素 (2 和 1)
2 的右侧仅有 1 个更小的元素 (1)
6 的右侧有 1 个更小的元素 (1)
1 的右侧有 0 个更小的元素
 ```

- `0 <= nums.length <= 10^5`
- `-10^4 <= nums[i] <= 10^4`

**题解**

```java
class Solution {
    public List<Integer> countSmaller(int[] nums) {
        if (nums.length == 0) return new ArrayList<>();
        // 1. 去重
        Set<Integer> set = new TreeSet<>();
        for (int num : nums) {
            set.add(num);
        }
        // 2. 离散化
        Map<Integer, Integer> map = new HashMap<>();
        int rank = 1;
        for (int num : set) {
            map.put(num, rank++);
        }
        // 3. 树状数组
        BIT bit = new BIT(rank);
        List<Integer> ans = new ArrayList<>();
        for (int i = nums.length - 1; i >= 0; i--) {
            rank = map.get(nums[i]);
            ans.add(bit.query(rank - 1));
            bit.update(rank, 1);
        }
        Collections.reverse(ans);
        return ans;
    }
}
```

### leetcode第214场周赛 t4

**通过指令创建有序数组**

给你一个整数数组 `instructions `，你需要根据 `instructions `中的元素创建一个有序数组。一开始你有一个空的数组 `nums `，你需要 从左到右 遍历 `instructions `中的元素，将它们依次插入 `nums `数组中。每一次插入操作的 代价 是以下两者的 较小值 ：

`nums `中 严格小于  `instructions[i]` 的数字数目。
`nums `中 严格大于 ` instructions[i]` 的数字数目。
比方说，如果要将 3 插入到 `nums = [1,2,3,5]` ，那么插入操作的 代价 为 `min(2, 1)` (元素 1 和  2 小于 3 ，元素 5 大于 3 ），插入后 `nums `变成 `[1,2,3,3,5]` 。

请你返回将 `instructions `中所有元素依次插入 `nums `后的 总最小代价 。由于答案会很大，请将它对 109 + 7 取余 后返回。

```
示例 ：

输入：instructions = [1,5,6,2]
输出：1
解释：一开始 nums = [] 。
插入 1 ，代价为 min(0, 0) = 0 ，现在 nums = [1] 。
插入 5 ，代价为 min(1, 0) = 0 ，现在 nums = [1,5] 。
插入 6 ，代价为 min(2, 0) = 0 ，现在 nums = [1,5,6] 。
插入 2 ，代价为 min(1, 2) = 1 ，现在 nums = [1,2,5,6] 。
总代价为 0 + 0 + 0 + 1 = 1 。
```

- `1 <= instructions.length <= 105`
- `1 <= instructions[i] <= 105`

**题解**

```java
class Solution {
    public int createSortedArray(int[] instructions) {
        int n = Arrays.stream(instructions).max().getAsInt();
        BIT bit = new BIT(n);
        long ans = 0;
        int cnt = 0;
        for (int num : instructions) {
            int low = bit.query(num - 1);
            int high = cnt - bit.query(num);
            ans += Math.min(low, high);
            bit.update(num, 1);
            cnt++;
        }
        return (int) (ans % 1_000_000_007);
    }
}
```

