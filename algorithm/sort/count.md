基于下标的排序方式

创建一个能包含待排序数据范围的数组, 把每一个元素按下标的方式映射到新数组, 然后遍历即可

因此, 计数排序不适用于数据上下限差过大或有小数这种不方便做映射的场景

```java
void countSort(int[] nums) {
    int min = nums[0], max = nums[0];
    for (int num : nums) {
        max = Math.max(max, num);
        min = Math.min(min, num);
    }
    int[] map = new int[max - min + 1];

    for (int num : nums) {
        map[num - min]++;
    }

    int index = 0;
    for (int i = 0; i < map.length; i++) {
        if (map[i] == 0) continue;

        while (map[i] != 0) {
            nums[index++] = min + i;
            map[i]--;
        }
    }
}
```

