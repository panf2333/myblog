# Binary
https://leetcode.cn/circle/discuss/CaOJ45/


## Subset
```java
// 枚举 i1 的子集 j
for (int j = i1; j > 0; j = (j - 1) & i1) 

//如果j是i1的子集，那么这样就是去掉该子集
i1=i1^j
```
>![](assets/Binary_子集枚举.png)