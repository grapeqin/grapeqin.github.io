# LeetCode27： 移除元素

> 原题：[LeetCode27](https://leetcode-cn.com/problems/remove-element/)



## 初始思路

采用LeetCode26优化思路2类似的思想。引入两个指针，分别初始化为i=j=0；j依次迭代，当nums[j] != val时，将nums[j]赋值给nums[i]，同时i++，最后在j++；当nums[j]==val时，直接j++即可。

## 代码实现

```java
public int removeElement(int[] nums, int val) {
    int i = 0;
    for(int j = 0;j<nums.length;j++){
        if(nums[j] != val){
            nums[i] = nums[j];
            i++;
        }
    }
    return i;
}
```

**时间复杂度** ：当少量元素与目标元素相等，下标i因为要复制下标j的元素，导致下标i,j都会完整遍历整个数组，花费时间为2N，时间复杂度为O(N^2^)

**空间复杂度**：O(1)

## 优化思路1

在前面的思路中，我们发现当数组中的元素与目标元素不相等时，所有不相等的元素都需要完成一次拷贝动作，题目中提到不限制目标数组中元素的顺序，我们可以采用下面的思路。

从数组下标i=0开始遍历，同时令n=nums.length，当i<n时，如果发现nums[i] == val，表示当前这个元素可以被移除，我们将它与当前数组最后一个元素互换位置，即将nums[n-1]赋值给nums[i]  (实现上可以将数组的长度缩短1)，当nums[i]!=val，表示当前元素是目标数组的元素，将i++；重复整个步骤，最后返回i就是目标数组的长度。

<video width="100%" height="400" controls>
  <source src="/zh-cn/ds/_media/Leetcode27_02.mp4" type="video/mp4">
	移除元素
</video>

## 代码实现1

```java
public int removeElement(int[] nums, int val) {
    int i = 0;
    int n = nums.length;
    while(i < n){
        if(nums[i] == val){
            nums[i] = nums[n-1];
            n--;
        }else{
            i++;
        }
    }
    return i;
}
```

**时间复杂度**：这种思路每次发现元素与目标值相等时，只会将数组长度缩短，最多只需要遍历N次就可以达成目标，时间复杂度为O(N)

**空间复杂度**：O(1)

