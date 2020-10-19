```java
void insertSort(int[] nums) {
    for (int i = 1; i < nums.length; i++) {
        // 记录当前要插入的值
        int val = nums[i];
        // 从已排好序的最后一位开始比较
        int idx = i - 1;
        while (idx >= 0 && val < nums[idx]) {
            nums[idx + 1] = nums[idx];
            idx--;
        }
        nums[idx + 1] = val;
    }
}
```

