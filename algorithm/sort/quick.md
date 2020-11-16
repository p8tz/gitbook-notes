这种写法简单，但是排序过程没有严格遵循快速的思想

```java
void quickSort(int[] nums, int left, int right) {
    if (left >= right)
        return;

    int l = left;
    int r = right;
    int pivot = nums[(right - left) / 2 + left];
    while (l <= r) {
        while (nums[l] < pivot) {
            l++;
        }
        while (nums[r] > pivot) {
            r--;
        }
        if (l <= r) {
            int t = nums[l];
            nums[l] = nums[r];
            nums[r] = t;
            l++;
            r--;
        }
    }
    
    quickSort(nums, left, r);
    quickSort(nums, l, right);
}
```

真正的快排

```java
void quickSort(int[] nums, int left, int right) {
    if (left >= right)
        return;

    // 每次选取最后一个元素作为pivot, 如果任意选取则要把它放在首或尾
    int pivot = nums[right];
    int l = left;
    // 移动元素过程中不涉及到pivot, 所以是right - 1
    int r = right - 1; 
    
    while (l <= r) { // 对于这种情况: [0, 1], 指针l必须移动到1处
        while (l < right && nums[l] < pivot) {
            l++;
        }
        while (l < r && nums[r] > pivot) {
            r--;
        }
        if (l == r)
            break;
        if (l < r) {
            swap(nums, l, r);
            l++;
            r--;
        }
    }
    // 把pivot移到最终位置
    swap(nums, l, right);
    
    // 以pivot所在的索引为分界点左右递归
    quickSort(nums, left, l - 1);
    quickSort(nums, l + 1, right);
}

void swap(int[] a, int i, int j) {
    int t = a[i];
    a[i] = a[j];
    a[j] = t;
}
```

