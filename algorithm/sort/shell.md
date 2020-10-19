```java
void shellSort(int[] nums) {
    // 在插入排序的基础上，增加了gap
    // 间距相同gap的元素进行插入排序
    // 每次缩小gap，直到缩为1就变为了标准的插入排序
    
    // 外层控制gap
    for (int gap = nums.length / 2; gap > 0; gap /= 2) {
        // 增量由1变为gap
        for (int i = gap; i < nums.length; i++) {
            int val = nums[i];
            int idx = i - gap;
            while (idx >= 0 && val < nums[idx]) {
                nums[idx + gap] = nums[idx];
                idx -= gap;
            }
            nums[idx + gap] = val;
        }
    }
}
```

