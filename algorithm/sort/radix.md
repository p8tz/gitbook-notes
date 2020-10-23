不适用于负数和小数

先根据个位分到不同的桶, 然后回写数组. 再根据十位分, 以此类推

```java
void radixSort(int[] nums) {
    int max = nums[0];
    for (int num : nums) {
        max = Math.max(max, num);
    }
    // 执行轮次
    int round = (max + "").length();

    // 10个桶, 一个桶最坏情况下有nums.length个元素
    int[][] bucket = new int[10][nums.length];
    // 记录每个桶有多少元素
    // bucketElCount[0]代表第0个桶元素数量
    int[] bucketElCount = new int[10];

    for (int r = 0; r < round; r++) {
        // 把每个元素放入对应的桶中
        for (int num : nums) {
            int radix = (int) (num / Math.pow(10, r) % 10);
            // 放到第radix个桶中, 从索引0处开始放
            bucket[radix][bucketElCount[radix]++] = num;
        }

        // 每次排序后放回nums
        int index = 0;
        for (int i = 0; i < bucket.length; i++) {
            if (bucketElCount[i] == 0) continue;
			
            // 一个桶有3个元素, 对应在bucket中的位置就是0, 1, 2
            for (int j = 0; j < bucketElCount[i]; j++) {
                nums[index++] = bucket[i][j];
            }
            // 元素数量归零
            bucketElCount[i] = 0;
        }
    }
}
```

