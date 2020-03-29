# LeetCode3：无重复字符的最长子串

> 原题：[LeetCode3](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)



## 初始思路

提到处理无重复字符，首先可以想到运用set集合来存储字符。我们定义两个指针p1，p2，初始指向字符串第一个字符，p2往后移动，同时判断p2指向的元素是否位于set集合中，如果不存在，则将该元素添加到集合中，同时比较并记录子串的长度；如果存在，说明[p1,p2)的子串已经存在p2指向的字符，此时就从set集合中移除p1指针的字符，然后p1指针后移，直到set集合中不包含p2指向的字符为止，反复迭代整个过程，具体流程请参考如下动画演示：

<video width="100%" height="400" controls>
  <source src="/zh-cn/ds/_media/Leetcode3.mp4" type="video/mp4">
  通过set和双指针思想来获取无重复字符的最长子串长度
</video>

## 代码实现1

```java
    public int lengthOfLongestSubstring(String s) {
        if(null == s || s.length() == 0){
            return 0;
        }
        int first_pointer = 0;
        int second_pointer = 0;
        int max = 0;
        Set<Character> hash_set = new HashSet<>();
        while(second_pointer < s.length()){
            //当前元素不存在,追加到set集合中,同时比较子串的长度
            if(!hash_set.contains(s.charAt(second_pointer))){
                hash_set.add(s.charAt(second_pointer));
                second_pointer++;
                max = Math.max(max,hash_set.size());
            }else{
                //当前元素存在,移除first_pointer指向的元素
                hash_set.remove(s.charAt(first_pointer));
                first_pointer++;
            }
        }
        return max;
    }
```

**时间复杂度** ：最坏情况下，指针p1、p2都会遍历完整个数组，共遍历2N次，平均时间复杂度为O(N)

**空间复杂度**：由于引入了set集合，最坏情况下整个字符串不重复，占用空间N，平均空间复杂度为O(N)

## 优化思路

在前面的思路中，当发现有重复元素时，我们是让p1依次往后移动，来一步一步排除重复元素，那么如果我在保存重复元素时，也能保存它在字符串中的位置，就意味着我能直接定位到重复元素所在的位置，直接让p1指针移动到已重复元素的下一位，加速p1的移动。具体效果如下图动画所示：

<video width="100%" height="400" controls>
  <source src="/zh-cn/ds/_media/Leetcode3_02.mp4" type="video/mp4">
  通过hash和双指针思想来获取无重复字符的最长子串长度
</video>

## 代码实现2

```java
    public int lengthOfLongestSubstring(String s) {
        if(null == s || s.length() == 0){
            return 0;
        }
        int first_pointer = 0;
        int second_pointer = 0;
        int max = 0;
        Map<Character,Integer> map = new HashMap<>();
        while(second_pointer < s.length()){
            if(map.containsKey(s.charAt(second_pointer))){
                first_pointer = Math.max(first_pointer,map.get(s.charAt(second_pointer))+1);
            }
            map.put(s.charAt(second_pointer),second_pointer);
            max = Math.max(max,second_pointer - first_pointer + 1);
            second_pointer++;
        }
        return max;
    }

```

**时间复杂度**：这种思路下,p2指针依然需要完整遍历整个字符串，p1指针只会结合重复元素的位置直接定位，所以时间复杂度为O(N)

**空间复杂度**：该思路依然需要申请Map来保存元素和其索引，时间复杂度为O(N)