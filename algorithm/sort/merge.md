归并排序分为两步

1. divide
2. merge

```java
void mergeSort(int[] nums) {
    int[] tmp = new int[nums.length];
    divide(nums, 0, nums.length - 1, tmp);
}

void divide(int[] nums, int l, int r, int[] tmp) {
    if (l >= r)
        return;

    // 从中点分开
    int mid = (r - l) / 2 + l;
    divide(nums, l, mid, tmp);
    divide(nums, mid + 1, r, tmp);

    // 合并 [l, mid] 和 [mid + 1, r]
    merge(nums, l, mid, r, tmp);
}

void merge(int[] nums, int l, int mid, int r, int[] tmp) {
    int i = l, j = mid + 1;
    int m = mid, n = r;

    int index = 0;
    while (i <= m && j <= n) {
        if (nums[i] < nums[j]) {
            tmp[index++] = nums[i++];
        } else {
            tmp[index++] = nums[j++];
        }
    }

    // 处理剩余元素
    while (i <= m) {
        tmp[index++] = nums[i++];
    }
    while (j <= n) {
        tmp[index++] = nums[j++];
    }

    // 回写原数组
    for (int k = 0; k < index; k++) {
        nums[l + k] = tmp[k];
    }
    // System.arraycopy(tmp, 0, nums, l, index);
}
```

