# 剑指Offer06. 从尾到头打印链表

[剑指Offer6.从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

[牛客网.从尾到头打印链表](https://www.nowcoder.com/practice/d0267f7f55b3412ba93bd35cfa8e8035?tpId=13&tqId=11156&tPage=1&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking&from=cyc_github)

## 解法一:递归

从尾到头打印一个链表，我们可以想到通过递归来求解。只要没有到达链表尾，就一直递归下去，到链表尾则递归结束，然后将链表的值保存到中间集合里边。实现如下所示：

```java
/**
*    public class ListNode {
*        int val;
*        ListNode next = null;
*
*        ListNode(int val) {
*            this.val = val;
*        }
*    }
*
*/
import java.util.ArrayList;
public class Solution {
    //保存中间结果
    private ArrayList<Integer> result = new ArrayList<>();
    
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        if(null==listNode){
            return result;
        }
        resolveList(listNode);
        return result;
    }
    
  	/**
  	* 递归求解
  	*/
    private void resolveList(ListNode listNode){
        if(listNode == null){
            return;
        }
        resolveList(listNode.next);
        result.add(listNode.val);
    }
}
```

另外一种更简洁递归写法如下所示：

```java 
/**
*    public class ListNode {
*        int val;
*        ListNode next = null;
*
*        ListNode(int val) {
*            this.val = val;
*        }
*    }
*
*/
import java.util.ArrayList;
import java.util.Stack;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ArrayList<Integer> ret = new ArrayList<>();
        if(null!=listNode){
            ret.addAll(printListFromTailToHead(listNode.next));
            ret.add(listNode.val);
        }
        return ret;
    }
}
```

## 解法二:栈

逆序输出一个链表，我们也可以想到Java中有一种数据结构Stack符合这类需求，所以接下来我们采用Stack这种数据结构来求解到这道题，实现如下：

```java
/**
*    public class ListNode {
*        int val;
*        ListNode next = null;
*
*        ListNode(int val) {
*            this.val = val;
*        }
*    }
*
*/
import java.util.ArrayList;
import java.util.Stack;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
      	//先将链表依次入栈
        Stack<Integer> stack = new Stack<>();
        while(listNode!=null){
            stack.push(listNode.val);
            listNode = listNode.next;
        }
      	//然后再出栈并保存结果
        ArrayList<Integer> result = new ArrayList<>(stack.size());
        while(!stack.isEmpty()){
            result.add(stack.pop());
        }
        return result;
    }
}
```

## 解法三:链表反转

既然要为尾到头来打印链表，那我们也可以先将链表反转，然后再依次打印链表的值即可。代码如下：

```java
/**
*    public class ListNode {
*        int val;
*        ListNode next = null;
*
*        ListNode(int val) {
*            this.val = val;
*        }
*    }
*
*/
import java.util.ArrayList;
import java.util.Stack;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
				//单链表反转需要增加一个哑结点
      	ListNode dummy = new ListNode(-1);
        while(null!=listNode){
            ListNode node = new ListNode(listNode.val);
            node.next = dummy.next;
            dummy.next = node;
            listNode = listNode.next;
        }
        ArrayList<Integer> ret = new ArrayList<>();
				//依次输出链表
      	ListNode current = dummy.next;
        while(null!=current){
            ret.add(current.val);
            current = current.next;
        }
        return ret;
    }
}
```

