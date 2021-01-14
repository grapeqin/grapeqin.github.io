# 二维数组的查找(数组)

[剑指Offer 04. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

## 解题思路

通过观察n * m的数组matrix我们可以发现，二维数组中对于任意一个数字，小于它的值一定在左边，大于它的值一定在下边。

由此，我们可以采用这样的办法来搜索目标数据target，沿着左下到右上的对角线开始查找。假设行坐标为r,列坐标为c，当

target > matrix[r] [c] ，说明目标值位于当前值的下边，行数加1；当target < matrix[r] [c]，说明目标值位于当前位置的左边，列数减1；当 target == matrix[r] [c]，说明目标值找到，直接返回。

基于上面的分析，整理得到如下所示代码实现

```java
public boolean Find(int target, int[][] array) {
        // 参数合法性校验
        if (null == array || array.length == 0 || array[0].length == 0) {
            return false;
        }
        //对角线从右上往左下方向查找
        int r = 0;
        int c = array[0].length - 1;
        while (r < array.length && c >= 0) {
            if (target == array[r][c]) {
                return true;
            } else if (target < array[r][c]) {
                c--;
            } else {
                r++;
            }
        }
        return false;
}
```

以上实现的平均时间复杂度为O(M+N)，空间复杂度为O(1)