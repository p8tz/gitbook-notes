```java
void heap_sort(vector<int>& nums) {
    const int N = nums.size();
    // 第一次建堆是一个整体的建堆过程
    // 从最后一个有子节点的节点开始建堆
    for (int i = N / 2 - 1; i >= 0; i--) {
        siftdown(nums, i, N);
    }
    // 第一次建堆完成后，最大值就在首位，移到最后即可
    // 然后继续建堆，这里的每一次建堆只需要局部就能完成
    for (int i = N - 1; i >= 0; i--) {
        swap(nums[0], nums[i]);
        siftdown(nums, 0, i);
    }
}

void siftdown(vector<int>& nums, int i, int N) {
    int max = i;		// 当前节点
    int l = 2 * i + 1;	// 左子节点
    int r = 2 * i + 2;	// 右子节点
    // 找到最大的节点索引
    if (l < N && nums[max] < nums[l]) max = l;
    if (r < N && nums[max] < nums[r]) max = r;
    if (max != i) {
        swap(nums[max], nums[i]);
        siftdown(nums, max, N);
    }
}
```

