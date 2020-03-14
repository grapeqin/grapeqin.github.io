# LeetCode442：数组中重复的数据

> 原题：[LeetCode442](https://leetcode-cn.com/problems/find-all-duplicates-in-an-array/)

## 初始思路

抛开该题对空间复杂度的限制，我们能想到借助Set集合来帮助判断是否有重复元素。申请一个set集合，然后依次遍历数组，对于每一个元素，首先判断该元素是否在set中存在，如果不存在，将该元素添加到set集合中；如果存在，表示当前这个元素是重复的，将它添加到存放目标结果的list中，接着继续遍历下一个元素，直到遍历完整个数组，就能得到所有存在重复的元素。

## 代码实现1

```java
class Solution {
    public List<Integer> findDuplicates(int[] nums) {
        List<Integer> result = new ArrayList<>();
        if(null==nums || nums.length < 2){
            return result;
        }
        Set<Integer> hash_set = new HashSet<>();
        for(int i=0; i<nums.length;i++){
            //如果set集合中存在当前元素，表明当前元素是重复数据
            if(hash_set.contains(nums[i])){
                result.add(nums[i]);
            }
            hash_set.add(nums[i]);
        }
        return result;
    }
}
```

**时间复杂度** ：整个过程只需要遍历一次数组，时间复杂度为O(N) 

**空间复杂度**：根据题意，存在出现两次的元素，那对于数组长度为N的数组来说，最多会存在N/2个重复的元素，set占用空间大小为O(N/2) ，即空间复杂度为O(N)。 

## 优化思路

现在我们考虑题目的要求，空间复杂度为O(1)，这种要求要么申请一个单独的变量，要么直接利用源数组。由于题目强调有些元素出现两次，意味着一个变量不能标记多个元素的重复情况，剩下就是直接利用源数组。这里引入一个比较巧妙的思路，我们这样来考虑：对于长度为N的数组，其中的元素满足1<=a[i]<=N，如果我们对数组元素减去1就得到0<=a[i]-1<=N-1，这个关系是不是很熟悉，这不就是长度为N的数组的下标范围么？对于数组内的任一元素a[i]，我们令index=|a[i]| - 1，index一定满足0<=index<=N-1这个关系，接着将index处的元素取反即a[index]=-a[index]，如果一个元素存在重复，经过上面的步骤后，当前a[index]处的值会再次取反回到正数，即平常我们所说的负负得正，这样描述可能还不是很清晰，我们借助如下所示的动画来理解这一过程：

**图1 采用负负得正的思想实现从数组中找出重复的数据**





## 代码实现2

```java
class Solution {
    public List<Integer> findDuplicates(int[] nums) {
        List<Integer> result = new ArrayList<>();
        if(null==nums || nums.length < 2){
            return result;
        }
        for(int i=0;i<nums.length;i++){
            //数组的元素值，可能为负，但原始值都是正数，这里直接取绝对值
            int val = Math.abs(nums[i]);
            //数组nums的下标
            int index = val - 1;
            //对数组元素取反
            nums[index] = -nums[index];
            if(nums[index] > 0){
                //负负得正，保存结果
                result.add(val);
            }
        }
        return result;
    }
}
```

**时间复杂度**：整个过程只对数组进行了一次遍历，时间复杂度为O(N)

**空间复杂度**：整个过程并为开辟新空间，空间复杂度为O(1)

## 扩展题型

题目中说1<=a[i]<=N，如果改成0<=a[i]<=N-1，对思路的提示意义会更明显，但如果参考上面的思路来实现的话就有些困难了，因为这里a[i]可能等于0，负负得正没有办法判断0这个特殊情况。我们首先把这个数组的元素整体加1，这样就得到了与原题一样的条件，然后采用同样的办法进行重复元素判断，只是在添加重复元素时记得要减1。参考代码如下：

```java
public List<Integer> findDuplicates(int[] nums) {
    List<Integer> result = new ArrayList<>();
    if (null == nums || nums.length < 2) {
      return result;
    }
    // 将数组转换成1<=a[i]<=N
    for (int i = 0; i < nums.length; i++) {
      nums[i] += 1;
    }
    for (int i = 0; i < nums.length; i++) {
      int val = Math.abs(nums[i]);
      int index = val - 1;
      nums[index] = -nums[index];
      if (nums[index] > 0) {
        // 这里记得-1,因为源数据迭代之前都+1了
        result.add(val - 1);
      }
    }
    return result;
  }
```

**时间复杂度** ：源数组提前遍历一遍执行累加操作，然后需要再次遍历查找重复元素，整体运行时间为O(2N)，所以时间复杂度为O(N) 

**空间复杂度**：整个过程并未单独开辟空间，所以空间复杂度为O(1)。