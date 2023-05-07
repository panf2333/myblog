# 二分
## 循环不变量
数据在哪一个区间还未确定
1. 定义区间，这个区间内的数据还没有确认过，是未确定的 
2. 当值不在区间的时候跳出循环（当不满足区间定义的时候跳出循环，此时所有项都已明确） 
3. mid 定义   
    a. mid = left + (right - left + 1) / 2; 
    > <font color="red">会靠近 right, right 是开区间不要用这个，因为right = mid 可能会导致死循环</font>
    -------------
    b. mid = left + (right - left) / 2; 
    > <font color="red">会靠近left, left 是开区间不要用这个，因为left = mid 可能会导致死循环</font>
    -------------

## 区间定义
### 左闭右闭  <font color="red">[first, left) false</font> [left, right] <font color="33FFFF"> true (right, last]</font>
1. 确定mid不满足时删去[origin left, mid]	left = mid + 1
2. 确定mid 满足时删去 [mid, origin ]		right = mid - 1
全部确定后（right < left） 因为left的左侧肯定是不满足的  <font color="33FFFF">返回left 即可。</font>
```java
    // 循环不变式，当条件成立时进行循环，不成立时跳出循环
    // 闭区间
    private int lowBound(int[] nums, int target) {
        // 闭区间[left, right]
        int left = 0;
        int right = nums.length - 1;
        int mid = 0;
        //循环不变式 区间中没有值的时候跳出循环
        while (left <= right) {
            mid = left + (right - left) / 2;
            // 删去[origin left, mid]未确定区间 [mid+1, right]
            if (nums[mid] < target) {
                left = mid + 1;
            } else {
                // 删去[mid, origin right]未确定区间[left, mid - 1]
                right = mid - 1;
            }
        }
        return left;
    }
```

### 左闭右开  <font color="red"> [first, left) false</font>  [left, right)  <font color="33FFFF"> true [right, last]</font>

1. 确定mid不满足时删去[origin left, mid]    left = mid + 1
2. 确定mid 满足时删去 [mid, origin right] ， 但是因为右开,所以未确定里不包括mid. right = mid,  
全部确定后left == right <font color="33FFFF">所以left 和 right 都可以</font>
```java
	// 左闭右开区间 未确定区间 [left, right)
    private int lowBound2(int[] nums, int target) {
        // 左闭右开区间[left, right)
        int left = 0;
        int right = nums.length;
        int mid = 0;
        //循环不变式 区间中没有值的时候跳出循环
        while (left < right) {
            mid = left + (right - left) / 2;
            // 删去[origin left, mid]  未确定 [mid + 1, right]
            if (nums[mid] < target) {
                left = mid + 1; //[mid + 1, right)
            } else {
                // 删去[mid, origin right]  未确定 [left, mid)  所以right = mid
                right = mid; //[left, mid)
            }
        }
        return left;
    }
```
### 左开右开  <font color="red"> [first, left] false</font> (left, right) <font color="33FFFF"> true [right, last]</font>
1. 确定mid不满足时删去[origin left, mid]  但是因为左开,所以未确定里不包括mid.   left = mid
2. 确定mid 满足时删去 [mid, origin right] ， 但是因为右开,所以未确定里不包括mid. right = mid,  
全部确定后left + 1 == right 时， 已经明确left 是不满足的，<font color="33FFFF">所以返回right</font>
```java
// 左开右开区间 (left, right)
    private int lowBound3(int[] nums, int target) {
        // 左开右开区间（left, right)
        int left = -1;
        int right = nums.length;
        int mid = 0;
        //循环不变式 区间中没有值的时候跳出循环
        while (left < right - 1) {
            mid = left + (right - left) / 2;
            // 删去[origin left, mid]  未确定 (mid, right)
            if (nums[mid] < target) {
                left = mid; //(mid, right)
            } else {
                // 删去[mid, origin right]  未确定 (left, mid)  所以right = mid
                right = mid;  //(left, mid)
            }
        }
        return right;
    }
```

### 左开右闭  <font color="red"> [first, left] false</font> (left, right] <font color="33FFFF"> true (right, last]</font>
1. 确定mid不满足时删去[origin left, mid]  但是因为左开,所以未确定里不包括mid.   left = mid
2. 确定mid 满足时删去 [mid, origin right] ， 但是因为右闭,所以未确定里不包括mid. right = mid - 1,  
全部确定后left == right 时， 已经明确left 是不满足的，<font color="33FFFF">所以返回left + 1 或者right + 1</font>
```java
    // 左开右闭区间 (left, right]
    private int lowBound4(int[] nums, int target) {
        // 左开右闭区间（left, right]
        int left = -1;
        int right = nums.length - 1;
        int mid = 0;
        //循环不变式 区间中没有值的时候跳出循环
        while (left < right) {
            mid = left + (right - left + 1) / 2;
            // 删去[origin left, mid]  未确定 (mid, right]
            if (nums[mid] < target) {
                left = mid; //(mid, right]
            } else {
                // 删去[mid, origin right]  未确定 (left, mid - 1]  所以right = mid - 1
                right = mid - 1;  //(left, mid)
            }
        }
        // return right + 1;
        return left + 1;
    }

```
