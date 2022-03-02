# 二叉搜索树

## 定义

二叉搜索树是一种特殊的二叉树。每个节点都大于左子树节点，小于右子树节点。二叉搜索树有三种常见的操作：

- 查找
- 插入
- 删除

## 查找

对于下面这棵二叉搜索树：

![bst01](\_media\bst01.png)

如何查找**19**这个节点是否存在？

1. 声明node = root；
2. 循环判断node != null
   1. 如果 node.val == target，直接返回
   2. 如果 node.val > target，node = node.left
   3. 如果 node.val < target，node = node.right
3. 遍历结束还未找到，程序直接返回未找到

将以上查找思路转换成代码如下所示：

```java
/**
 * 二叉搜索树节点定义
 */
class BstNode {
    int val;
    BstNode left;
    BstNode right;

    public BstNode() {
    }

    public BstNode(int val) {
        this.val = val;
    }

    public BstNode(int val, BstNode left, BstNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }

    public int getVal() {
        return val;
    }

    public BstNode getLeft() {
        return left;
    }

    public BstNode getRight() {
        return right;
    }

    @Override
    public String toString() {
        return "BstNode{" +
                "val=" + val +
                ", left=" + left +
                ", right=" + right +
                '}';
    }
}

public class Bst {

    /**
     * 二叉搜索树的根节点
     */
    BstNode root;

    /**
     * 查找指定值在二叉搜索树中是否存在
     *
     * @param data
     * @return
     */
    public BstNode search(int data) {
        BstNode node = root;
        while (null != node) {
            if(data == node.getVal()){
                return node;
            }else if(data < node.getVal()){
                node = node.left;
            }else{
                node = node.right;
            }
        }
        return null;
    }

    public static void main(String[] args) {
        Bst bst = new Bst();
        bst.createBstTree();
        int target = 19;
        BstNode node = bst.search(target);
        System.out.println("查询target = " + target +" 在BstTree中的node = " + node);
        
        target = 55;
        node = bst.search(target);
        System.out.println("查询target = " + target +" 在BstTree中的node = " + node);        
    }

    private void createBstTree(){
        BstNode node19 = new BstNode(19);
        BstNode node16 = new BstNode(16);

        BstNode node27 = new BstNode(27);
        BstNode node25 = new BstNode(25);
        node25.left = node19;
        node25.right = node27;

        BstNode node18 = new BstNode(18);
        node18.left = node16;
        node18.right = node25;

        BstNode node13 = new BstNode(13);
        BstNode node17 = new BstNode(17);
        node17.left = node13;
        node17.right = node18;

        BstNode node66 = new BstNode(66);
        BstNode node51 = new BstNode(51);
        BstNode node58 = new BstNode(58);
        node58.left = node51;
        node58.right = node66;

        BstNode node34 = new BstNode(34);
        BstNode node50 = new BstNode(50);
        node50.left = node34;
        node50.right = node58;

        BstNode node33 = new BstNode(33);
        node33.left = node17;
        node33.right = node50;

        this.root = node33;
    }
}
```

运行结果如下所示：

```shell
查询target = 19 在BstTree中的node = BstNode{val=19, left=null, right=null}
查询target = 55 在BstTree中的node = null
```

## 插入

利用 33,17,50,13,18,34,58,16,25,51,66,19和27 这些数字构建一棵二叉搜索树

参考下面的思路来处理：

1. 定义node = root
2. 如果 null == node , 构建新节点并设置给root，程序返回
3. 循环node != null
   1. 如果target < node.val
      1. 如果 node.left == null ，则 node.left = Node(target)
      2. 否则 node = node.left
   2. 如果 target > node.val 
      1. 如果 node.right == null，则 node.right = Node(target)
      2. 否则 node = node.right

将以上思路转换成代码实现如下：

```java
// 插入新节点
/**
 * 二叉查找树的插入
 *
 * @param target
 */
public void insert(int target) {
    BstNode node = root;
    //根节点为空,直接返回
    if (null == node) {
        root = new BstNode(target);
        return;
    }
    while (null != node) {
        // 待插入节点值比当前节点值小,说明要插入的节点位置在当前节点的左子树
        if (target < node.getVal()) {
            // 左子树为空,可以直接插入
            if (null == node.left) {
                node.left = new BstNode(target);
                break;
            } else {
                node = node.left;
            }
        } else {
            if (null == node.right) {
                node.right = new BstNode(target);
                break;
            } else {
                node = node.right;
            }
        }
    }
}
```

上面代码生成的二叉搜索树如下所示：

![bst-insert-02](_media\bst02.png)



## 删除

二叉搜索树的删除操作相比较而言就比较复杂了，根据**待删除节点子节点**的个数分为3种情况：

- 无子节点：将待删除节点的父节点指向该节点的指针指向空即可，如下图删除节点55

![bst03](_media\bst03.png)



- 有1个子节点：将待删除节点的父节点指向该节点的指针指向该节点的子节点即可，如下图删除节点13

![bst04](_media\bst04.png)

- 有2个子节点，如下图删除节点18
  - 找到待删除节点右子树中最小的节点
  - 将待删除节点用右子树中最小的节点替换掉
  - 在删除这个最小节点

![bst05](_media\bst05.png)

根据上面的思路，给出二叉搜索树删除节点的源代码

```java
// 删除二叉搜索树给定的节点
/**
 * 二叉查找树的删除
 *
 * @param target
 */
public void delete(int target) {
    //声明保存父节点变量
    BstNode pp = null;
    //声明保存当前节点变量
    BstNode p = root;
    
    //查找待删除的节点
    while (null != p){
        pp = p;
        if(target == p.getVal()){
            break;
        }else if(target < p.getVal()){
            p = p.left;
        }else{
            p = p.right;
        }
    }
    //未找到待删除的节点,直接返回
    if(null == p){
        return;
    }

    // 有2个子节点
    if(null != p.left && null != p.right){
        BstNode minPP = p;
        BstNode minP = p.right;
        while (null != minP && minP.left != null){
            minPP = minP;
            minP = minP.left;
        }
        //待删除节点右子树中最小的节点覆盖待删除节点
        p.val = minP.getVal();
        //这两次赋值主要是为了后面统一操作
        p = minP;
        pp = minPP;
    }

    //找到待删除节点的子节点
    BstNode child = null;
    if (null != p.left) {
        child = p.left;
    } else {
        child = p.right;
    }

    if(pp == null){
        //删除树根
        root = child;
    }else if(p != pp.left){
        pp.left = child;
    }else{
        pp.right = child;
    }
}
```

