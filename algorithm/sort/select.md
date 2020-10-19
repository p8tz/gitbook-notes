```java
void selectSort(int[] nums) {
    for (int i = 0; i < nums.length - 1; i++) {
        int min = i;	// 记录最小值的index
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[j] < nums[min]) {
                min = j;
            }
        }
        int t = nums[i];
        nums[i] = nums[min];
        nums[min] = t;
    }
}
```
