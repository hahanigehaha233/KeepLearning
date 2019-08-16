##<center>排序算法注意事项 </center>

## 链表排序
链表进行插入排序时需要存储的是当前比较节点的前一个节点。

## 快排
快排在进行前后指针移动时，需要后面的指针先移动，**需要先找到一个比他小的数，再找比它大的数，如果在左边没有比它大的数了，说明右边比它小的数就是它的位置**

## 桶排序
最好的时间为`O(N+M)`，但是最差的情况是所有元素在一个桶中，这样的会时间复杂度为 $O(n^{2})$

- 桶长度 = 区间长度 - 区间个数 = (Max - Min) / (len - 1)
- 桶个数 = 区间长度 / 桶长度 + 1

- [桶排序动图](https://blog.csdn.net/developer1024/article/details/79770240)


## 计数排序
时间复杂度 $O(N+K)$

空间复杂度 $O(N+K)$，其中，K为待排序数组中最大值。需要在最大值不大于数组长度太多的情况下比较节约空间。

```
public int[] countsort(int[] nums){
    int len = nums.length;
    int num_max = Integer.MIN_VALUE;
    int num_min = Integer.MAX_VALUE;
    for(int i = 0;i < len;++i){
        if(num_max < nums[i]) num_max = nums[i];
        if(num_min > nums[i]) num_min = nums[i];
    }
    int[] counts = new int[num_max - num_min + 1];
    for(int i = 0 ;i < len;++i){
        counts[nums[i]] ++;
    }
    for(int i = 1;i < counts.length;++i){
        counts[i] += counts[i-1];
    }
    int[] res = new int[len];
    for(int i = len;i >=0;--i){
        res[--counts[nums[i]]] = nums[i];
    }
    return res;
}
```
