```java
void quickSort(int[] nums, int left, int right) {
    if (left >= right) return;

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

