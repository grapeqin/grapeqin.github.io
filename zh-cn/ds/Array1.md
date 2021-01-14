# 两数之和

> 原题：[LeetCode1](https://leetcode-cn.com/problems/two-sum/)



## 初始思路

暴力法：针对数组中的每个元素，依次遍历该元素后面的元素，判断和是否为target，如是，直接返回，如不是依次迭代，直到遍历完所有的元素。

## 代码实现1

```java
public int[] twoSum(int[] nums, int target) {
    if(null == nums){
        return null;
    }
    for(int i =0;i<nums.length-1;i++){
        for(int j=i+1;j<nums.length;j++){
            if(target == nums[i]+nums[j]){
                return new int[]{i,j};
            }
        }
    }
    return null;
}
```

**时间复杂度** ：用了两层循环，时间复杂度为O(N^2^)

**空间复杂度**：没使用其他内存空间，空间复杂度为O(1)

## 优化思路

第一层遍历是少不了的，那第二层循环，我们如果引入一个哈希表来存储每个元素与它的下标之间的映射关系，是不是可以省略掉这层循环，这样时间复杂度就可以提升为O(N)。具体做法就是在遍历过程中，先判断target-nums[i]元素是否位于哈希表中，如果存在，说明已经找到了匹配的目标元素，直接return，否则将当前元素和它的下标放入哈希表。

## 代码实现2

```java
public int[] twoSum(int[] nums, int target) {
    if(null == nums){
        return null;
    }
    Map<Integer,Integer> map = new HashMap<>();
    for(int i =0;i<nums.length;i++){
        if(map.containsKey(target-nums[i])){
            return new int[]{map.get(target-nums[i]),i};
        }
        map.put(nums[i],i);
    }
    return null;
}
```

**时间复杂度**：由于只有一层循环，时间复杂度为O(N)

**空间复杂度**：引入了外部结构哈希表，空间复杂度O(N)