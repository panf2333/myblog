- [Over view of Sort](#over-view-of-sort)
  - [Bubble sort](#bubble-sort)
    - [Time and space complexity](#time-and-space-complexity)
    - [Template Code](#template-code)
  - [Selection sort](#selection-sort)
    - [Time and space complexity](#time-and-space-complexity-1)
    - [Template Code](#template-code-1)
  - [Insertion sort](#insertion-sort)
    - [Time and space complexity](#time-and-space-complexity-2)
    - [Template Code](#template-code-2)
  - [Shell sort](#shell-sort)
    - [Time and space complexity](#time-and-space-complexity-3)
    - [Template Code](#template-code-3)
  - [Quick sort](#quick-sort)
  - [Merge sort](#merge-sort)
  - [Heap sort](#heap-sort)
  - [Count sort](#count-sort)
  - [Bucket sort](#bucket-sort)
  - [Base sort](#base-sort)

# Over view of Sort
![Sort Compare](assets/SortCompare.png)
> Stable means that we can keep the original relative order of the equal elements after sorting.
> 
## Bubble sort

1. Compare the adjacent two element[i - 1, 1], if the nums[i - 1] > nums[i], then swap them.
2. We compare all of the adjacent number pair. Until we don't need to swap.
3. Because of swap may move the [4, 8, 3] to [4, 3, 8]. So, we need to loop to ensure after swap the [i, i + 1] are also order.
4. In fact, the kth largest number will be moved to the correct position in k times. So, we only need to loop nums.length times. If no swap is needed, we can skip out.

### Time and space complexity
We suppose the array length is n.
Time : O(n ^ 2)
Space: O(1)

### Template Code
```java
    public static void BubbleSort(int[] nums) {
        // we max need to loop nums.length times
        for (int i = 0; i < nums.length; i++) {
            int swapCount = 0;
            // in each loop we only change the adjacent pair. In fact the max value will move to the end in first time.
            for (int j = 1; j < nums.length; j++) {
                if (nums[j - 1] > nums[j]) {
                    swap(nums, j - 1, j);
                    swapCount++;
                }
            }
            //  If no swap is needed, the array is ordered.
            if (swapCount == 0) {
                break;
            }
        }
    }

    public static void swap(int[] nums, int i, int j) {
        nums[i] ^= nums[j]; // i1 = i ^ j
        nums[j] ^= nums[i]; // j1 = j ^ i1 = j ^ i ^ j = i
        nums[i] ^= nums[j]; // i2 = i1 ^ j1 = i ^ j ^ i = j
    }

```
## Selection sort
1. First, find the smallest (largest) element in the unsorted sequence and store it to the beginning of the sorted sequence.
2. Then continue to find the smallest (large) element from the remaining unsorted elements and put it at the end of the sorted sequence.
3. Repeat the second step until all elements are sorted.
> not stable example: [<font color="red">**2**</font>, 3, 2, 1, 4] -> [1, 3, 2, <font color="red">**2**</font>, 4]


### Time and space complexity
We suppose the array length is n.
Time : O(n ^ 2)
Space: O(1)

### Template Code
```java
    public static void selectionSort(int[] nums) {
        for (int i = 0; i < nums.length - 1; i++) {
            int minimumIndex = i;
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[minimumIndex] > nums[j]) {
                    minimumIndex = j;
                }
            }
            if (minimumIndex > i)
                swap(nums, i, minimumIndex);
        }
    }

    public static void swap(int[] nums, int i, int j) {
        nums[i] ^= nums[j]; // i1 = i ^ j
        nums[j] ^= nums[i]; // j1 = j ^ i1 = j ^ i ^ j = i
        nums[i] ^= nums[j]; // i2 = i1 ^ j1 = i ^ j ^ i = j
    }

```

## Insertion sort
1. start with the first element, which can be considered to have been sorted;

2. taking out the next element and scanning from backwards to forwards in the sequence of already sorted elements;

3. if the element (already sorted) is larger than the new element, move the element to the next position;

4. repeating step 3 until a position is found where the sorted element is less than or equal to the new element;

5. inserting the new element after that position;

6. Repeat steps 2~5.

### Time and space complexity
We suppose the array length is n.
Time : O(n ^ 2)
Space: O(1)

### Template Code
```java
    public static void InsertionSort(int[] nums) {
        for (int i = 1; i < nums.length; i++) {
            int key = nums[i];
            int j = i - 1;
            for (; j >= 0 && nums[j] > key; j--) {
                nums[j + 1] = nums[j];
            }
            nums[j + 1] = key;
        }
    }
```

## Shell sort
The sequence of records to be sorted is first divided into several sub-sequences, and then the entire sequence is sorted by direct insertion when the records in the sequence are "basically ordered". The time complexity of this algorithm is O(n log n).

1. select an incremental sequence t1, t2, ......, tk, where ti > tj, tk = 1;

2. sort the sequence k times by the number of incremental sequences k;

3. For each sort, according to the corresponding increment ti, the sequence to be sorted is partitioned into a number of subsequences of length m, and each subsequences is sorted by direct insertion respectively. When the increment factor is 1, the whole sequence can be sorted.


### Time and space complexity
We suppose the array length is n.
Time : O(n * log n)
> Actually each increment step will scan array once. And the increment is divide k every time, So we will loop (log n) times;
Space: O(1)

### Template Code
```java
    public static void shellSort(int[] nums) {
        int increment = nums.length;
        // execute al least once
        do {
            // 3 is up to you, + 1 is ensure we can sorted the array.
            increment = increment / 3 + 1;
            // start from [0, increment - 1]
            for (int i = 0; i < increment; i++) {
                // Insertion Sort subsequence
                for (int j = i + increment; j < nums.length; j += increment) {
                    int key = nums[j];
                    int k = j;
                    while (k > i && nums[k] < nums[k - increment]) {
                        nums[k] = nums[k - increment];
                        k -= increment;
                    }
                    if (k != j) {
                        nums[k] = key;
                    }
                }
            }
        }
        while (increment > 1);
    }
```

## Quick sort
## Merge sort
## Heap sort
## Count sort
## Bucket sort
## Base sort