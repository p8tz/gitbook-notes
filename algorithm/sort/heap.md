```java
void heapSort(int[] nums) {
    int len = nums.length;
	
    // 第一次建堆是一个整体的建堆过程
    // 从最后一个有子节点的节点开始建堆
    for (int i = len / 2 - 1; i >= 0; i--) {
        heapify(nums, i, len);
    }

    // 第一次建堆完成后，最大值就在首位，移到最后即可
    // 然后继续建堆，这里的每一次建堆只需要局部就能完成
    for (int i = len - 1; i >= 0; i--) {
        // 把最大值移到后面去
        int t = nums[0];
        nums[0] = nums[i];
        nums[i] = t;

        // 建堆
        heapify(nums, 0, i);
    }
}

void heapify(int[] nums, int i, int len) {
    int max = i;
    // 左子节点
    int l = 2 * i + 1;
    // 右子节点
    int r = 2 * i + 2;
    
    // 找到这三个节点中最大数的索引，记为max
    if (l < len && nums[max] < nums[l]) {
        max = l;
    }
    if (r < len && nums[max] < nums[r]) {
        max = r;
    }
    
    // 调整
    if (max != i) {
        int t = nums[i];
        nums[i] = nums[max];
        nums[max] = t;

        // 递归调整子节点
        heapify(nums, max, len);
    }
}
```

