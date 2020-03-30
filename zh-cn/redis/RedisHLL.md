# Redis基本数据结构 - 基数统计(HyperLogLog)

## 基本用法

基本用法请参考[Redis HyperLogLog命令用法](http://www.redis.cn/commands.html#hyperloglog)

## 实现原理

HyperLogLog作为大数据量下基数统计的一种算法，能够用较少的内存量来得出在一定误差范围内的基数统计值，它并不是Redis独有的，在基数统计的场景下，并不实际存储元素，而是运用一种算法来近似的计算基数值，无法应用于需要准确计数的场景。如果你想了解更多有关HyperLogLog的实现原理请参考

[神奇的HyperLogLog算法]([http://www.rainybowe.com/blog/2017/07/13/%E7%A5%9E%E5%A5%87%E7%9A%84HyperLogLog%E7%AE%97%E6%B3%95/index.html?utm_source=tuicool&utm_medium=referral](http://www.rainybowe.com/blog/2017/07/13/神奇的HyperLogLog算法/index.html?utm_source=tuicool&utm_medium=referral))

[一种Redis新的数据结构-HyperLogLog](http://antirez.com/news/75)

[HyperLogLog: the analysis of a near-optimal cardinality estimation algorithm](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)

[HyperLogLog算法演示](http://content.research.neustar.biz/blog/hll.html)

## 运用场景

-   大型互联网应用中的基数统计：比如PV数统计、UV数统计、用户搜索网站的关键词数量等。

