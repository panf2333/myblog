# Dynamic
state * operations in each iteration.
every state is calculated by before iteration, and these states need operation ciybts to get the final count;
For example:
state is [deep, count] deep * 100; (1...deep,  1...100)
every iteration operation count is 100;
deep - 1 is calculated by before dfs.
so we only need to calculate the end iteration. So，we multiple 100.
```
private int dfs(int deep, int count) {
    for () //100
        dfs(deep + 1, i);
}
```