# 剑指offer03：数组中重复的数字

[LeetCode 剑指offer03](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

[牛客JZ50](https://www.nowcoder.com/practice/623a5ac0ea5b4e5f95552655361ae0a8?tpId=13&rp=1&ru=%2Fta%2Fcoding-interviews&qru=%2Fta%2Fcoding-interviews%2Fquestion-ranking)

这道题可以分两种类型来进行解答：

-   返回任意一个重复的数字
-   返回第一个重复的数字

## 解法一

要在一个集合中找重复的数字，很容易想到Java中的Set数据结构，那么第一种解法我们就使用Set集合作为重复数字的暂存空间。

遍历数组，判断set集合中是否包含当前位置的数组元素，如果包含直接返回即可，如果不包含，则将当前数组的元素add到set集合中，直到数组遍历结束为止。

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
      	// 声明一个与数组大小相同大小的HashSet集合
        Set<Integer> set = new HashSet<>(nums.length);
      	// 遍历数组
        for(int num : nums){
          	// 集合中包含数字时,说明当前数字已经重复
            if(set.contains(num)){
                return num;
            }
          	// 集合中不包含数字则将数字添加到集合
            set.add(num);
        }
        return 0;
    }
}
```

时间复杂度：整个过程需要遍历一遍数组，时间复杂度为O(N)

空间复杂度：由于引进了集合Set，最坏情况在几乎所有的数组元素都可能会添加进集合中，平均空间复杂度为O(N)

解法一 由于是依次遍历数组，所以它返回的结果是第一个重复的数字，这种解法能同时满足前文介绍的两类题型。

## 解法二

题目开头强调了在长度为n的数组中存在0~n-1的数字，解法一并没有用到这个特征。考虑到在长度为n的数组中，数组的下标范围刚好也是 0~n-1，我们可以利用数组的这个特性。

将每个元素i归位到它的下标i处，也就是让i == nums[i]。如果当前下标i!=nums[i]，那就让下标i与下标nums[i]的元素值互换，直到i == nums[i]为止。在这个过程中，只要发现下标i的元素与下标nums[i]的元素相等，那么这个元素就是重复的元素了，直接退出即可。

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
       	//数组遍历
        for(int i=0;i<nums.length;i++){
          	//第i个下标处的元素不是i，就一直交换，直到把i这个元素归位到第i个下标处为止
            while(i != nums[i]){
              	// 一旦i这个位置的元素与以nums[i]为下标的元素相等，那么这个元素就是重复的元素,直接返回
                if(nums[i] == nums[nums[i]]){
                    return nums[i];
                }
              	// 如果不相等,这两个下标处的元素互换位置
                int temp = nums[i];
                nums[i] = nums[temp];
                nums[temp] = temp;
            }
        }
        return 0;
    }
}
```

时间复杂度：进行了数组的遍历，时间复杂度为O(N)

空间复杂度：仅仅申请了一个临时变量，空间复杂度为O(1)

解法二由于会持续的交换元素的位置，所以不能保证返回的重复数字在原数组中是第一个重复的数字，这种解法只适合第一种类型，即返回数组中任何一个重复的数字。

## 解法三

题目中仅仅指出数字只要重复即可，并没有限制重复的次数，除了利用数组下标外，我们还可以考虑将元素取反来标识它是否出现过。具体来说依然是遍历数组，取下标i的元素nums[i]，当下标num[i]的元素大于0时，直接将该下标处的元素取反；反之，表明该下标处的元素已经出现过，可以直接返回。

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
      	//遍历数组
        for(int i=0;i<nums.length;i++){
          	//取元素nums[i]的下标值,可能为负值,但数组的下标必须非负,所以这里要取绝对值
            int index = Math.abs(nums[i]);
          	// 如果index下标处的元素小于0,说明index这个元素在数组中已经出现过,可以直接返回
            if(nums[index] < 0){
                return index;
            }
          	// index下标的元素取反,标识在数组中出现过
            nums[index] = -nums[index];
        }
        return 0;
    }
}
```

时间复杂度：遍历数组，时间复杂度为O(N)

空间复杂度：未引入其他数据结构，空间复杂度为O(1)

解法三并没有交换元素之间的位置，所以它能够返回第一个重复的数字，因此符合以上两种题型的解答。

以上解法三虽然能够通过测试。但它存在一个问题，就是数字0如果是重复的话，因为0取反还是0，没有办法标记0的重复出现，仅仅在最后未匹配时返回0作为程序结束。

为了更考虑到0这种情况，我们可以考虑在取反之前，对原数字先加1再取反，再还原真实下标时，当nums[i]为负数时同样先加1再取反，这样就能完美的解决0重复的问题。

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
      	//遍历数组
        for(int i=0;i<nums.length;i++){
          	//当nums[i]的值为负数时,先+1再取反还原数组的下标值
            int index = nums[i] < 0 ? -(nums[i]+1) : nums[i];
          	// index下标的值为负,表示该位置的元素重复,直接返回
            if(nums[index] < 0){
                return index;
            }
          	// 取反之前先+1,能够解决0重复的时候取反无法区分的问题
            nums[index] = -(nums[index]+1);
        }
        return 0;
    }
}
```

时间复杂度：依然是O(N)

空间复杂度：O(1)