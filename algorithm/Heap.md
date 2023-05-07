## Heap
A heap is a structure that approximates a complete binary tree and satisfies both the properties of a heap: The key or index of a child node is always less (or greater) than or equal its parent node.


### from down layer to top layer to heapify
1. put the new element in the end
2. compare to its parent node and to swap until don't need to swap or arrive root node

### from up layer to down layer to heapify
When remove the top node
1. put the last element in the top node
2. compare to its children node and to swap until don't need to swap or arrive leaf node


### Time and space complexity
We suppose the array length is n.
Time : O(log n)
> beacuse of the heap is approximate a complete binary tree. So the tree height is only need log n to find and reheapify the heap. 
Space: O(1)

### Template Code
```java
// heapify

public class Heap {
    private int[] a; // start store data from index 1
    private int n;  // the max store data size
    private int count; // the counts already stored in the heap

    public Heap(int capacity) {
        a = new int[capacity];
        Arrays.fill(a, -1);
        n = capacity;
        count = 0;
    }

    // i: left child i * 2 + 1,  right child i * 2 + 2
    // left child k: k / 2 == > i
    // right child k : (k - 1) / 2 ==> i
    public void insert(int data) {
        if (count >= n) return; // heap is get to capability
        a[count] = data;
        int i = count++;
        while ((i - 1) / 2 >= 0 && a[i] > a[(i - 1) / 2]) { // from down layer to top layer to heapify
            swap(a, i, (i - 1) / 2); // swap() swap these index data
            i = (i - 1) / 2;
        }
    }

    public void removeMax() {
        if (count == 0) return -1; // the heap is empty
        a[0] = a[--count];
        heapify(a, count - 1, 0);
    }

    /*
     * from top layer to down layer to heapify
     * nums is the array
     * n is the end index
     * i is the start heapify index
     */
    private static void heapify(int[] nums, int n, int i) {
        while (true) {
            int maxPos = i;
            if (i * 2 + 1 <= n && nums[i] < nums[i * 2 + 1]) maxPos = i * 2 + 1;
            if (i * 2 + 2 <= n && nums[maxPos] < nums[i * 2 + 2]) maxPos = i * 2 + 2;
            if (maxPos == i) break;
            swap(nums, i, maxPos);
            i = maxPos;
        }
    }

    private static void swap(int[] nums, int i, int j) {
        nums[i] ^= nums[j]; // i1 = i ^ j
        nums[j] ^= nums[i]; // j1 = j ^ i1 = j ^ i ^ j = i
        nums[i] ^= nums[j]; // i2 = i1 ^ j1 = i ^ j ^ i = j
    }
}

```