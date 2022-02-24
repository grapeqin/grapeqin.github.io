# 二叉树的搜索

二叉树的搜索分两类

- 深度优先搜索(DFS：depth first search)
- 广度优先搜索(BFS：breadth first search)

## 深度优先搜索(DFS)

沿着树的深度遍历树的节点，尽可能深的搜索树的分支

按照节点的打印顺序，又细分为三种遍历：

- 前序遍历：遍历中按照先打印当前节点，左子树和右子树的顺序
- 中序遍历：遍历中按照先打印左子树、当前节点和右子树的顺序
- 后序遍历：遍历中按照先打印左子树、右子树和当前节点的顺序

![image-20220223155917253](_media\image-20220223155917253-binary-tree01.png)

对于上图这个二叉树

- 前序遍历  A -> B -> D -> E -> C -> F -> G

![binary-tree-pre-search](_media\binary-tree-pre-search.png)



- 中序遍历 D -> B -> E -> A -> F -> C -> G

![binary-tree-mid-search](_media\binary-tree-mid-search.png)

- 后序遍历 D -> E -> B -> F -> G -> C -> A

![binary-tree-post-search](_media\binary-tree-post-search.png)



对以上三种遍历的实例分析，可以给出它们的递推公式：

- 前序遍历   print(node)，print(node.left)，print(node.right)
- 中序遍历  print(node.left)，print(node)，print(node.right)
- 后续遍历 print(node.left)，print(node.right)，print(node)

如下给出前序遍历的递归实现源代码，另外两种遍历方式源代码稍作修改即得，不再赘述。

```java
/**
 * 二叉树的定义
 */
class TreeNode {
    String val;
    TreeNode left;
    TreeNode right;

    public TreeNode() {
    }

    public TreeNode(String val) {
        this.val = val;
    }

    public TreeNode(String val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }

    public String getVal() {
        return val;
    }

    public TreeNode getLeft() {
        return left;
    }

    public TreeNode getRight() {
        return right;
    }
}

/**
 * 二叉树的递归前序遍历
 * @param treeNode
 * @param printList
 */
public void transversal(TreeNode treeNode, List<String> printList) {
    if(null==treeNode){
        return;
    }
    //所谓的前、中、后遍历，主要看这一步位于这三行代码的位置
    printList.add(treeNode.val);
    transerval(treeNode.left,printList);
    transerval(treeNode.right,printList);
}
```

递归代码大部分情况下可以改造成非递归实现，非递归实现主要借助于数据结构Stack来实现，如下是它的源代码

```java
//非递归方式实现DFS
/**
 * 二叉树的非递归DFS遍历
 * @param treeNode
 * @return
 */
public List<String> transversal(TreeNode treeNode){
    List<String> printList = new ArrayList<>();
    if(null == treeNode){
        return printList;
    }
    Stack<TreeNode> stack = new Stack<>();
    stack.push(treeNode);
    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        printList.add(node.val);
        // 注意右节点先入栈
        if (null != node.right) {
            stack.push(node.right);
        }
        if (null != node.left) {
            stack.push(node.left);
        }
    }
    return printList;
}
```

DFS在遍历过程中，会将所有的节点都扫描一遍，时间复杂度为O(N)



## 广度优先搜索(BFS)

BFS也叫做按层遍历，从树的根节点开始，从左往右遍历完第一层，接着遍历第二层，依次类推，直到遍历完所有的节点

![binary-tree-bfs](_media\binary-tree-bfs.png)

BFS非常适合借用队列queue来实现，如下所示：

```java
//迭代BFS实现
/**
 * 二叉树的非递归BFS遍历
 * @param treeNode
 * @return
 */
public List<String> bfs(TreeNode treeNode){
    List<String> printList = new ArrayList<>();
    if(null==treeNode){
        return printList;
    }
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(treeNode);
    while(!queue.isEmpty()){
        //取当前队列的大小,表示当前层的节点数目
        int size = queue.size();
        while (size-- > 0){
            //按序取队列中的元素
            TreeNode node = queue.poll();
            printList.add(node.val);
            //左子树节点不为空则入队列
            if(null!=node.left){
                queue.offer(node.left);
            }
            //右子树节点不为空则入队列
            if(null!=node.right){
                queue.offer(node.right);
            }
        }
    }
    return printList;
}
```

BFS也可以采用递归的方式来实现，采用DFS中的先序遍历，初始化一个二维数组，遍历过程中根据当前节点所在的层数，将节点保存到对应的位置

![binary-tree-bfs-of-dfs](_media\binary-tree-bfs-of-dfs.png)

```java
/**
 * 递归求解BFS
 *
 * @param treeNode 根节点
 * @param list 存放每层从左往右扫描节点的列表
 * @param level 表示当前扫描的是第几层,根节点表示第0层,依次类推
 */
public void bfs(TreeNode treeNode, List<List<String>> list, int level) {
    if (null == treeNode) {
        return;
    }
    //采用前序遍历
    if (list.size() == level) {
        List<String> subList = new ArrayList<>();
        subList.add(treeNode.val);
        list.add(level, subList);
    } else {
        list.get(level).add(treeNode.val);
    }
    bfs(treeNode.left, list, level + 1);
    bfs(treeNode.right, list, level + 1);
}
```

