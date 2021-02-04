伪快排，这种写法简单，但是排序过程没有严格遵循快速的思想

```c++
void quick_sort(vector<int>& nums, int L, int R) {
    if (L >= R) return;
    int l = L;
    int r = R;
    int pivot = nums[(r - l) / 2 + l];
    while (l <= r) {
        while (nums[l] < pivot) l++;
        while (nums[r] > pivot) r--;
        if (l <= r) {
            swap(nums[l], nums[r]);
            l++;
            r--;
        }
    }
    quick_sort(nums, L, r);
    quick_sort(nums, l, R);
}
```

真正的快排

```java
void quick_sort(vector<int>& nums, int L, int R) {
    if (L >= R) return;
    // 每次选取最后一个元素作为pivot, 如果任意选取则要把它放在首或尾
    int pivot = nums[R];
    int l = L;
    // 移动元素过程中不涉及到pivot, 所以是right - 1
    int r = R - 1;
    while (l <= r) { // 对于这种情况: [0, 1], 指针l必须移动到1处
        while (l < R && nums[l] < pivot) l++;
        while (r > l && nums[r] > pivot) r--;
        // 上面l可能越过r, 需要额外一次判断
        if (l >= r) break;
        swap(nums[l], nums[r]);
        l++;
        r--;    
    }
    // 把pivot移到最终位置
    swap(nums[l], nums[R]);
    // 以pivot所在的索引为分界点左右递归
    quick_sort(nums, L, l - 1);
    quick_sort(nums, l + 1, R);
}
```

