# LeetCode26：删除排序数组中的重复项

> 原题：[LeetCode26](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)



## 初始思路

首先想到的一种思路是从数组nums末尾往数组开头去遍历，假设当前遍历的下标为j，当nums[j]==nums[j-1]，说明当前下标的元素nums[j]为重复元素，可以把下标j之后的所有元素往前挪动一个位置，假设j后的坐标为i,即i = j+1，将[i,nums.length-1]范围内的元素依次向前移动一次；完成第一个重复元素的删除，此时需要记录当前已发现的重复元素的个数，记为counter++

## 代码实现

```java
public int removeDuplicates(int[] nums) {
  int counter = 0;
  for(int j = nums.length-1;j>0;j--){
    if(nums[j] == nums[j-1]){
      counter++;
      for(int i = j;i<nums.length-1;i++){
        nums[i] = nums[i+1];
      }
    }
  }
  return nums.length - counter;
}
```

**时间复杂度** ：最坏情况下，所有元素都是重复的，时间复杂度为O(N^2^)，重复元素个数越多，内存循环次数越多

**空间复杂度**：O(1)

## 优化思路1

在前面的思路中，我们在内层循环中每次都是将下标[i,nums.length-1]范围内的元素都挪动一遍，但实际上，由于发现了重复元素，从末尾开始倒数第counter个元素即[nums.length-counter,nums.length-1]范围内的元素是无效的，我们不会再去使用它，所以只需要挪动[i,nums.length-counter]范围内的有效元素即可，这样重复元素越多，内层循环迭代的次数相比越少。

## 代码实现1

```java
public int removeDuplicates(int[] nums) {
  int counter = 0;
  for(int j = nums.length-1;j>0;j--){
    if(nums[j] == nums[j-1]){
      counter++;
      for(int i = j;i<nums.length-counter;i++){
        nums[i] = nums[i+1];
      }
    }
  }
  return nums.length - counter;
}
```

**时间复杂度**：平均时间复杂度依然没有改变太多，仍然为O(N^2^)

**空间复杂度**：O(1)

## 优化思路2

在前面的思路中，我们在内层循环中去依次挪动重复的元素，时间复杂度因为使用了双层循环一直保持在O( N^2^) 。这里我们引入双指针解法。声明两个指针i = 0;j=i+1;当nums[i]==nums[j]时，说明j位置的元素与i位置的元素重复，j继续往后迭代，直到找到第一个nums[j] != nums[i]，这时可以将nums[j]的元素赋值给nums[i+1]，然后i++;最后[0,i]范围内的数组即为删除重复元素后的数组，新数组的长度为i+1。

<video width="100%" height="400" controls>
  <source src="/zh-cn/ds/_media/Leetcode26.mp4" type="video/mp4">
	通过双指针删除排序数组中的重复项
</video>

## 代码实现2

```java
public int removeDuplicates(int[] nums) {
	int i = 0;
  for(int j=i+1;j<nums.length;j++){
    if(nums[j]!=nums[i]){
      nums[i+1] = nums[j];
      i++;
    }
  }
  return i+1;  
}
```

**时间复杂度**：最差情况下，所有元素都不相等，i和j下标都要走完整个数组，时间复杂度为O(2N)，最好情况下，所有元素都相等，只有j走完整个数组，i一直未移动，时间复杂度为O(N)；综上平均时间复杂度为O(N)

**空间复杂度**：O(1)
