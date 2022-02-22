# LeetCode28：字符串匹配算法

## 简介

假设有个字符串(也叫做主串)S：“BBC ABCDAB ABCDABCDABDE”，模式串P：“ABCDABD”，判断模式串P在主串S中是否存在？如存在，返回模式串在主串匹配的起始位置，如不存在，返回-1。

这类字符串匹配问题，常见匹配算法有：

- 暴力BF匹配算法
- KMP匹配算法
- Sunday匹配算法

## 暴力匹配算法

### 原理

1. 主串S定义下标i，初始化为0；模式串定义下标j，初始化为0；

![image-20220218174243424](_media\image-20220218174243424-bf01.png)

2. 比较S[i = 0]  和  P[j=0] ，在这里 S[0] = 'B'，P[0] = 'A'，'B' != 'A'，则主串S后移1位，模式串从头开始匹配

![image-20220218174355311](_media\image-20220218174355311-bf02.png)

3. 比较S[i=1] 和 P[j=0] ，在这里 S[1] = 'B'，P[0] = 'A'，'B' != 'A'，则主串S后移1位，模式串从头开始匹配

![image-20220218174602383](_media\image-20220218174602383-bf03.png)

4. 比较S[i=2] 和 P[j=0] ，在这里 S[2] = 'C'，P[0] = 'A'，'C' != 'A'，则主串S后移1位，模式串从头开始匹配；依次类推，直到i = 4

![image-20220218174745744](_media\image-20220218174745744-bf04.png)

5. 比较S[i=4] 和 P[j=0]，在这里 S[4] = 'A'，P[0] = 'A'，‘A’ == ‘A’，**此时主串S和模式串P均后移1位**

![image-20220218174833743](_media\image-20220218174833743-bf05.png)

6. 比较S[i=5] 和 P[j=1]，在这里 S[5] = 'B'，P[1] = 'B'，‘B’ == ‘B’，此时主串S和模式串P均后移1位

![image-20220218174951790](_media\image-20220218174951790-bf06.png)

7. 依此类推，直到i = 10, j =  6

![image-20220218175100383](_media\image-20220218175100383-bf07.png)

8. 此时 S[i] = ' '，P[j] = 'D' ,S[i]  !=  P[j]。**这种情况我们从主串S当前匹配起始下标(4)的下一个位置(5)，模式串P重新从0开始接着匹配**

![image-20220218175910333](_media\image-20220218175910333-bf08.png)

9. 比较S[i]和P[j]，这里S[5] = 'B'，P[0] = 'A'，S[4] != P[0] ，只需要主串S后移1位，模式串从头开始匹配

![image-20220218180026836](_media\image-20220218180026836-bf09.png)

10. 依次类推，直到 i = 11 ,j = 0 

![image-20220218180134333](_media\image-20220218180134333-bf10.png)

11. 此时回到与第5步同样的情况，主串和模式串均后移1位

![image-20220218180443627](_media\image-20220218180443627-bf11.png)

12. 依次类推，直到 i = 17 , j = 6；

![image-20220218180550412](_media\image-20220218180550412-bf12.png)

13. 此时与第8步的情况相同，采用同样的方法，则 i = 12,j = 0

![image-20220218180715788](_media\image-20220218180715788-bf13.png)

14. S[i=12] != P[j=0] ，主串S后移1位，模式串从头开始匹配

![image-20220218181029003](_media\image-20220218181029003-bf14.png)

15. 依次类推，直到i = 15

![image-20220218181131835](_media\image-20220218181131835-bf15.png)

16. 此时S[i=15] == P[j=0] ，则主串和模式串均后移1位，直到 i = 21 ，j = 6

![image-20220218181321450](_media\image-20220218181321450-bf16.png)

  模式串P的所有字符已经完全匹配，说明主串S中存在模式串P，其在主串中的起始下标为 i - j = 15。



###  伪代码 

  从上面的推导过程来看，主串S和模式串P比较的过程中，只有两种情况

- S[i] == P[j]  那么下标均后移1位
- S[i] != P[j] 那么主串S的下标 i = i - j  + 1，模式串P的下标 j = 0

直到主串或者模式串下标越界即程序终止。基于以上分析，我们给出如下暴力字符串匹配的伪代码：



1. 定义主串S，下标为i=0；模式串P，下标为j=0；

2. 以 j < P.length() 作为条件进行循环
   1. 判断 i >=  S.length()，直接返回 -1
   2. 接着比较S[i] 和 P[j] 
   3. S[i] == P[j] 则 i++,j++
   4. S[i] != P[j] 则 i = i - j + 1, j = 0
3. 返回 i - j 作为结果



### 源代码

```java
/**
 * 字符串匹配算法：时间复杂度O(M*N)
 * @param s 主串
 * @param p 模式串
 * @return 如果模式串p存在于主串s中，则返回其实下标；否则返回-1
 */
private int strStr(String s, String p) {
    // 参数合法性检查
    if (null == s || null == p) {
        return -1;
    }
    if (p.length() > s.length()) {
        return -1;
    }
    int sIndex = 0;
    int pIndex = 0;
    while (pIndex < p.length()) {
        //主串下标越界,直接返回
        if (sIndex >= s.length()) {
            return -1;
        }
        // s[sIndex] == p[pIndex] 继续比较下一个字符
        if (s.charAt(sIndex) == p.charAt(pIndex)) {
            sIndex++;
            pIndex++;
        } else {
            sIndex = sIndex - pIndex + 1;
            pIndex = 0;
        }
    }
    return sIndex - pIndex;
}
```



## KMP匹配算法

### 原理

上文的暴力匹配算法在第7步主串S的下标已经匹配到 i = 10 ，模式串P的下标匹配到 j = 6的情况下，由于 S[i] != P[j] ，让主串和模式串回溯，i = 5，j = 0以继续完成字符串匹配。

那有没有一种算法可以不回溯主串，从而提高匹配效率呢？这就是[KMP算法](https://zh.wikipedia.org/wiki/KMP%E7%AE%97%E6%B3%95)。

开始之前，我们引入新概念：部分匹配表pmt(Partial Match Table)，其实就是与模式串P(ABCDABD)长度相等的数组，构建完成后如下图所示：

| index | 0    | 1    | 2    | 3    | 4    | 5    | 6    |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| char  | A    | B    | C    | D    | A    | B    | D    |
| value | 0    | 0    | 0    | 0    | 1    | 2    | 0    |

你一定对这个数组的value是怎么填进去的很感兴趣，请到[这里](###构建部分匹配表pmt)了解。

让我们接着上文暴力匹配算法的第7步，此时i = 10, j =  6

![image-20220218175100383](_media\image-20220218175100383-bf07.png)

8. S[i=10] != P[j=6] ，**这次不移动主串S的下标i，根据下面的公式来计算模式串j的移动**

```shell
模式串移动位数 = 已匹配的字符个数 - 最后一位已匹配字符对应在部分匹配表pmt中的值
```

在本例中，已匹配的字符为“ABCDAB”，共6个字符，最后一位已匹配字符是index=5的字符“B”，它在部分匹配表中的值为2 ，带入公式，模式串移动的位数 step = 6 - 2 = 4 ；最终，模式串待匹配的下标 j = j - step = 6 - 4 = 2；

![image-20220218182856470](_media\image-20220218182856470-bmp01.png)

9. S[i=10] != P[j=2]，采用同样的算法，模式串需要移动的位数 step = 已匹配的字符个数"AB"(2) - 最后一位已匹配字符对应在部分匹配表中的值就是pmt[1] = 0 = 2；那么模式串下标 j = j - step = 2 - 2 = 0；

![image-20220218182926517](_media\image-20220218182926517-bmp02.png)

10. S[i=10] != P[j=0] ，**j=0，模式串已经没有办法移动了，此时，我们让主串后移1位**

![image-20220218183044534](_media\image-20220218183044534-bmp03.png)

11. S[i=11] == P[j=0]，主串和模式串均往后移1位，直到i = 17 , j = 6

![image-20220218183307558](_media\image-20220218183307558-bmp04.png)

12. 回到与第8步一样的情况，根据公式计算，得到 模式串下标 j = 2  

![image-20220218183421382](_media\image-20220218183421382-bmp05.png)

13. 此时 S[i=17] == P[j=2] ，i++，j++，直到 i=21，j=6

![image-20220218183614710](_media\image-20220218183614710-bmp06.png)

到这里为止，我们已经找到模式串P在主串S中匹配的位置为 i - j = 21 - 6 = 15，算法结束。

可以看到，整个匹配过程中，主串S的下标没有回溯，仅模式串的下标发生了回溯，算法执行效率理论上得到了提升。

###  伪代码

纵观KMP算法的匹配原理，整个算法的核心主要在当主串S[i]和模式串P[j]不匹配时，如何移动模式串，找到下一次匹配下标j的值。

1. 定义主串S，下标为i=0；模式串P，下标为j=0；

2. 以 i < S.length() 作为条件进行循环
   1. 循环 当S[i] != P[j] 并且 j >0 则 j = pmt[j-1]
   2. 如果 S[i] == P[j] 则 i++,j++
   3. 否则 主串后移1位 i++
   4. 判断 j == P.length() 是 返回 i - j +1即可 



重点看下第 2.1 步，j = pmt[j-1] 如何来理解。

回到前面KMP原理分析的第8步，j = 6 , 当前已匹配字符长度 j = 6，最后一个已匹配字符下标为 j - 1 = 5对应部分匹配表的值pmt[j-1] = 2，

则模式串待移动的step=6-2 = 4。

在程序中，我们是通过重新计算模式串P的下标值j来实现的，模式串移动step步，新下标值 j = j - step = j - (j - pmt[j-1]) = j - j + pmt[j-1] = pmt[j-1]。



### 源代码

```java
/**
 * KMP字符串匹配算法 时间复杂度O(M+N)
 *
 * @param s 主串
 * @param p 模式串
 * @return 模式串p存在于主串s中，则返回其实下标；否则返回-1
 */
private int strStr(String s, String p) {
    if (null == s || null == p) {
        return -1;
    }
    //构建pmt表
    int[] pmt = createPmt(p);
    for (int i = 1, j = 0; i < s.length(); i++) {
        while (j > 0 && s.charAt(i) != p.charAt(j)){
            j = pmt[j-1];
        }
        if(s.charAt(i) == p.charAt(j)){
            j++;
        }
        if(j == p.length()){
            return i - j + 1;
        }
    }
    return -1;
}
```





### 构建部分匹配表pmt

对任意字符串，都有前缀和后缀。

所谓前缀，就是除最后一个字符外以第一个字符开头的所有字符串；所谓后缀，就是除第一个字符外以最后一个字符结尾的所有字符串。



1. 以"ABCDABD"举例来说：

前缀：“A”、"AB"、"ABC" 、"ABCD"、 "ABCDA"、 "ABCDAB"

后缀：”D“ 、”BD“、"ABD"、"DABD"、"CDABD"、"BCDABD"



2. 以"ABCDAB"举例来说：

前缀："A"、"AB"、"ABC"、"ABCD"、"ABCDA"

后缀："B"、"AB"、"DAB"、"CDAB"、"BCDAB"



一个字符串在部分匹配表的取值，是该字符串所有公共前缀和后缀中最长字符串的长度。

1. 对于"A"来说：

前缀：""

后缀：""

公共前缀和后缀：""，长度为0，表示该字符串在部分匹配表的值为0



2. 对于"AB"来说：

前缀："A"

后缀："B"

没有公共前缀和后缀，长度为0，表示该字符串在部分匹配表的值为0



3. 对于"ABC"来说：

前缀："A"、"AB"

后缀："C"、"BC"

没有公共前缀和后缀，长度为0，表示该字符串在部分匹配表的值为0



4. 对于"ABCD"来说：

前缀："A"、"AB"、"ABC"

后缀："D"、"CD"、"BCD"

没有公共前缀和后缀，长度为0，表示该字符串在部分匹配表的值为0



5. 对于"ABCDA"来说：

前缀："A"、"AB"、"ABC"、"ABCD"

后缀："A"、"DA"、"CDA"、"BCDA"

公共前缀和后缀："A" ，长度为1，表示该字符串在部分匹配表的值为1



6. 对于"ABCDAB"来说：

前缀："A"、"AB"、"ABC"、"ABCD"、"ABCDA"

后缀："B"、"AB"、"DAB"、"CDAB"、"BCDAB"

公共前缀和后缀："AB"，长度为2，表示该字符串在部分匹配表的值为2



7. 对于"ABCDABD"来说：

前缀："A"、"AB"、"ABC"、"ABCD"、"ABCDA"、"ABCDAB"

后缀："B"、"AB"、"DAB"、"CDAB"、"BCDAB"、"BCDABD"

公共前缀和后缀：""，长度为0，表示该字符串在部分匹配表的值为0



最终得到"ABCDABD"的部分匹配表如下：

| index | 0    | 1    | 2    | 3    | 4    | 5    | 6    |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| char  | A    | B    | C    | D    | A    | B    | D    |
| value | 0    | 0    | 0    | 0    | 1    | 2    | 0    |

上面的过程如何通过编程来实现呢？

定义下标i =1、 j =0 

![image-20220219165353497](_media\image-20220219165353497-pmt01.png)

P[i] != P[j]  && j = 0 所以pmt[i] = j = 0

![image-20220219165722857](_media\image-20220219165722857-pmt02.png)

同理，pmt[i] = j = 0，直到 i = 4

![image-20220219165837610](_media\image-20220219165837610-pmt03.png)

此时P[i] = P[j]，j++ ,pmt[i] = j = 1； 

![image-20220219170045226](_media\image-20220219170045226-pmt04.png)

这个时候P[i] = P[j] 继续 j++,pmt[i] = j = 2；

![image-20220219170152953](_media\image-20220219170152953-pmt05.png)

这时发现P[i] != P[j] & j >0 ，则 j = pmt[j-1] = 0；

![image-20220219170401414](_media\image-20220219170401414-pmt06.png)



**源代码：**

```java
/**
 * 根据模式串构建部分匹配表
 *
 * @param p 模式串
 * @return 部分匹配表
 */
private int[] createPmt(String p) {
    int[] pmt = new int[p.length()];
    for (int i = 1, j = 0; i < p.length(); i++) {
        while (j > 0 && p.charAt(i) != p.charAt(j)) {
            j = pmt[j - 1];
        }
        if (p.charAt(i) == p.charAt(j)) {
            j++;
        }
        pmt[i] = j;
    }
    return pmt;
}
```



## Sunday匹配算法

### 原理

1. 定义主串S下标i = 0，模式串P下标j=0

![image-20220221135620561](_media\image-20220221135620561-sunday01.png)

2. S[i] != P[j]，nextCharIndex = i - j + P.length() ，如果发现S[nextCharIndex] 在模式串P中存在，取 step = S[nextCharIndex]中最后一次出现的下标 ,那么主串S的下标i = nextCharIndex - step 。本例中 i = 0 ,j = 0 ,nextCharIndex = 0 + 7 = 7;S[7] = 'D' 在模式串"ABCDABD"中最后一次出现的下标为6，所以主串S的下标需要移动到 i = 7 - 6 = 1

![image-20220221154711007](_media\image-20220221154711007-sunday02.png)

3. S[i=1] != P[j=0]，nextCharIndex = 8, S[nextCharIndex] 仍然存在与模式串P中，最后一次出现的位置是4，所以S主串下一次移动到的位置 i = 8 - 4 = 4。

![image-20220221154940393](_media\image-20220221154940393-sunday03.png)

4. S[i=4] == P[j=0]，则i++，j++，直到 i = 10，j = 6

![image-20220221155531592](_media\image-20220221155531592-sunday04.png)

5. S[i = 10] != P[j = 6]，nextCharIndex = i - j + P.length() = 10 - 6 + 7 = 11,  S[nextCharIndex] = 'A' 在模式串P中最后一次出现的下标为 4，所以 主串S的下标 i = 11 - 4 = 7，令 j = 0

![image-20220221162803634](_media\image-20220221162803634-sunday05.png)

6. S[i = 7] ! = P[j = 0] ，同理，nextCharIndex = 14，S[nextCharIndex] 在模式串P中最后一次出现的位置是6，所以 i = 14 - 6 = 8

 	                                                 ![image-20220221163012802](_media\image-20220221163012802-sunday06.png)

7. S[i = 8] == P[j = 0] ,则 i++ ， j++，直到 i = 10, j = 2

![image-20220221163143648](_media\image-20220221163143648-sunday07.png)

8. S[i = 10] != P[j = 2] ，则取nextCharIndex = i - j + P.length() = 10 - 2 + 7 = 15，S[nextCharIndex] 在模式串P中最后一次出现的位置step为4，故 i = nextCharIndex - step = 15 - 4  = 11，同时 j = 0

![image-20220221163430993](_media\image-20220221163430993-sunday08.png)

9. S[i = 11] == P[j = 0]，直到 i = 17 ，j = 6

![image-20220221163534591](_media\image-20220221163534591-sunday09.png)

10. S[i = 17] != P[j = 6] ，nextCharIndex = i - j + P.length() = 17 - 6 + 7 = 18，S[nextCharIndex] 在模式串P最后一次出现的位置step=6，故主串S下一次移动到的位置i = nextCharIndex - step = 18 - 6 = 12，j = 0

![image-20220221163803505](_media\image-20220221163803505-sunday10.png)

11. S[i = 12] != P[j = 0] ，nextCharIndex = i -j + P.length() = 12 - 0 + 7 = 19，S[nextCharIndex] 在模式串P最后一次出现的位置step=4，故主串S下一次移动到的位置i = nextCharIndex - step = 19 - 4 = 15 ，j = 0

![image-20220221165054783](_media\image-20220221165054783-sunday11.png)

12. S[i = 15] == P[j = 0]，则i++，j++；直到 i = 21, j = 6，模式串匹配完成，故返回 模式串P在主串中的位置 i - j = 15

![image-20220221155926374](_media\image-20220221155926374-sunday12.png)

### 伪代码

上文对Sunday匹配算法的分析，每一次S[i] != P[j] 时，S[nextCharIndex] 刚好出现在模式串P中，实际上，当S[nextCharIndex]不存在于模式串P中时，主串S的下标i 可以直接移动到 nextCharIndex+1的位置，大大提高了匹配效率。 下面总结出Sunday匹配算法的伪代码

定义 主串S下标 i = 0,模式串P 下标 j = 0；

当 j <  P.length()

如果 S[i]  == P[j] 则 i++，j++

如果 S[i] != P[j] 

​		定义 nextCharIndex = i - j + P.length()；

​		判断S[nextCharIndex]在模式串P中最后一次出现的位置step

​		如果 step == -1，表示S[nextCharIndex]不曾在模式串P中，i = nextCharIndex + 1，同时 j = 0

​		如果 step >= 0 ，i = nextCharIndex - step，同时 j = 0

最后返回 i - j 即为模式串P在主串S中的起始位置



### 源码

```java
/**
 * Sunday字符串匹配算法
 * @param haystack
 * @param needle
 * @return
 */
private int strStr(String haystack, String needle) {
    //参数检查
    if (null == haystack || null == needle) {
        return -1;
    }
    if (0 == needle.length()) {
        return 0;
    }
    if (needle.length() > haystack.length()) {
        return -1;
    }
    //定义主串和模式串的下标
    int i = 0;
    int j = 0;
    while (j < needle.length()) {
        // 主串越界,直接返回匹配失败
        if (i >= haystack.length()) {
            return -1;
        }
        if (haystack.charAt(i) == needle.charAt(j)) {
            i++;
            j++;
        } else {
            int nextCharIndex = i - j + needle.length();
            if(nextCharIndex < haystack.length()){
                int step = needle.lastIndexOf(haystack.charAt(nextCharIndex));
                if (-1 == step) {
                    i = nextCharIndex + 1;
                } else {
                    i = nextCharIndex - step;
                }
                j = 0;
            }else{
                return -1;
            }
        }
    }
    return i - j;
}
```

