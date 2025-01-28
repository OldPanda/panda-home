---
title: 如何将递归转成迭代
pubDate: 2021-10-28 22:00 PST
categories: ["聊聊技术"]
tags: iteration, recursion, LeetCode, Python, 迭代, 递归, 算法
heroImage: /images/blog/stone-stack-scaled.jpeg
heroImageDescription: Photo by Sean Stratton on Unsplash
---

> 要理解递归，先得理解递归

# 发现问题

函数的递归调用是码农在日常工作中不可或缺的利器，在某些问题上，函数递归可以提供更为简洁的代码实现和更为直观的阅读理解，比如说我们很熟悉的树形结构的遍历。

然而，当函数调用的层数过多的时候，就可能导致著名的 [Stack Overflow](https://en.wikipedia.org/wiki/Stack_buffer_overflow) 错误，而栈空间一般是由编译器（ C/C++ 等）或者 Java 虚拟机（ Java 系语言）来管理，对程序员是不可见的，当然我们可以通过配置参数来调整程序的栈空间大小，但不免麻烦，并且递归层数一增加，栈很可能又要溢出。

[尾递归](https://en.wikipedia.org/wiki/Tail_call)是一个常用的优化方法，很多语言在执行的时候会识别这种优化，因为递归函数调用是一个函数的最后一步，所以在递归调用之前，就可以把当前函数调用从栈中弹出，再把新的函数调用压栈，这样就不会因为递归深度的增加吃掉栈空间了。

但尾递归并不是很容易就能实现的，用二叉树的中序遍历举例，它的递归版本非常简单直观，

```python
# 树节点的定义
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def inorder_traversal(node):
    if node is None:
        return []
    left = inorder_traversal(node.left)
    right = inorder_traversal(node.right)
    return left + [node.val] + right
```

但试图将其转成尾递归则非常麻烦，因为它包含两个递归调用，并且两个递归调用的结果在最终要拼接在一起返回。这时如果还想避免栈溢出的问题，就需要将它转成迭代的形式，乍一听也是一个不太容易的工作，虽然[某刷题网站](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/solution/er-cha-shu-de-zhong-xu-bian-li-by-leetcode-solutio/)给出了迭代形式的代码，但我个人认为这个答案写得并不易于理解，读完之后与其说是我在尝试理解迭代的过程，倒不如说是在死记硬背，我们需要一种更直接的方式将递归代码转成迭代代码。

# 解决问题

函数递归本质上是函数调用入栈出栈的过程，并且在其中隐式地含有每一个函数调用的状态，函数通过查看这个状态来决定是否已满足停止条件，所以我们自然而然地想到用[栈数据结构](https://en.wikipedia.org/wiki/Stack_\(abstract_data_type\))来模拟函数递归的过程。

同样地，以二叉树的中序遍历为例，每一层函数调用处理一个节点，同时还需要一个变量来记录当前节点的左子树是否已经遍历完毕，同时这个状态变量还担负起避免重复遍历同一条路径的作用，为简单起见，用整数 `0` ， `1` ， `2` 分别表示“第一次访问该节点”，“该节点的左子树已经遍历”，“该节点本身已经被遍历”，所以初始状态为，

```python
stack = [[node, 0]]
```

然后用一个 `while` 循环来检查栈的长度，只要不为空，说明模拟的函数递归调用还没有结束，在每一次循环中，查看状态变量，如果

- 为 0 ，说明第一次访问该节点，如果它的左子树不为空，将左子树节点压栈，并将状态置为 1 ，这是因为下一次访问到这个节点的时候，它的左子树节点必然已经全部出栈，这时就该遍历该节点本身了

- 为 1 ，说明该节点的左子树遍历已经完成，将当前节点的值存到结果列表中，并将状态置为 2

- 为 2 ，说明该节点已经被遍历，此时应该将该节点出栈，如果它的右子树不为空的话，将其右节点入栈

这样完整的二叉树中序遍历迭代版代码为，

```python
def inorder_traversal(node):
    if not node:
        # 边缘情况检查
        return []
    output = []
    stack = [[node, 0]]
    while stack:
        node, status = stack[-1]
        if status == 0:
            stack[-1][1] = 1
            if node.left:
                stack.append([node.left, 0])
        elif status == 1:
            stack[-1][1] = 2
            output.append(node.val)
        else:
            stack.pop()
            if node.right:
                stack.append([node.right, 0])
    return output
```

这样看起来是不是更加简单易懂了呢？

# 更多的例子

## 二叉树的后序遍历

树的后序遍历的迭代版本一向以令人头大著称，配合上[某刷题网站的题解](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/solution/er-cha-shu-de-hou-xu-bian-li-by-leetcode-solution/)，就更加扑朔迷离了，那么这个问题能不能套用上述转换“公式”呢？当然可以，但首先要确定不同状态的含义。

与中序遍历略有不同，在这里用数字 `0` ， `1` ， `2` 来分别表示“初次访问该节点”，“左子树已遍历完成”，“右子树已遍历完成”的状态。在每一次循环中，查看当栈中的节点状态

- 为 0 ，初次访问该节点，将状态置为 1 ，如果该节点的左子树存在的话，将左子树入栈

- 为 1 ，该节点的左子树遍历完成，将状态置为 2 ， 如果该节点的右子树存在的话，将右子树入栈

- 为 2 ，该节点的右子树遍历完成，将当前节点的值存入结果列表中，然后该节点出栈

逻辑顺序理清了，就不难写出如下代码，

```python
def postorder_traversal(node):
    if not node:
        # 边缘情况检查
        return []
    output = []
    stack = [[node, 0]]
    while stack:
        node, status = stack[-1]
        if status == 0:
            stack[-1][1] = 1
            if node.left:
                stack.append([node.left, 0])
        elif status == 1:
            stack[-1][1] = 2
            if node.right:
                stack.append([node.right, 0])
        else:
            output.append(node.val)
            stack.pop()
    return output
```

## 将嵌套列表扁平化

为了说明简单，这里使用与[某刷题网站](https://leetcode-cn.com/problems/flatten-nested-list-iterator/)上同样的类定义来表示嵌套列表，即列表中的每个元素既可以是一个整数，也可以是一个列表，要求将其中所有的整数都存在列表的第一层并输出。比如说对于一个嵌套列表 `[1,2,[3,4],[[5],6,[7]]]` ，希望得到 `[1,2,3,4,5,6,7]` 。

```python
class NestedInteger:
    def isInteger(self) -> bool:
        """
        @return True if this NestedInteger holds a single integer, rather than a nested list.
        """
    def getInteger(self) -> int:
        """
        @return the single integer that this NestedInteger holds, if it holds a single integer
        Return None if this NestedInteger holds a nested list
        """
    def getList(self) -> [NestedInteger]:
        """
        @return the nested list that this NestedInteger holds, if it holds a nested list
        Return None if this NestedInteger holds a single integer
        """
```

它的递归版本也很简单，遇到整数就直接输出，遇到列表就递归调用自己并返回结果，

```python
def flatten_list(nested_list: [NestedInteger]):
    result = []
    def flatten(nested_list):
        for item in nested_list:
            if item.isInteger():
                result.append(item.getInteger())
            else:
                result.extend(flatten(item.getList()))
        return result
    return flatten(result)
```

现在，让我们尝试套用上述“公式”将其转换为迭代版本。首先，要明确状态变量的含义，因为每一次的递归调用是作用在一个列表上的，所以当用栈模拟函数调用时，状态变量可以规定为当前正在处理数组的下标，这样初始状态则为

```python
stack = [[nested_list, 0]]
```

在每一次循环中，若当前元素为整数，将该整数存到结果数组中，并将状态变量加一，代表着当前元素已经处理完毕；如果当前元素为列表，则将该列表与其初始状态（ `0` ）入栈，同时将状态加一；如果状态超出了列表的范围，则代表该列表已经被处理完毕，可以出栈。因此，最终的代码为

```python
def flatten_list(nested_list: [NestedInteger]):
    result = []
    stack = [[nested_list, 0]]
    while stack:
        l, status = stack[-1]
        if status >= len(l):
            stack.pop()
            continue
        if l[status].isInteger():
            result.append(l[status].getInteger())
            stack[-1][1] += 1
        else:
            stack[-1][1] += 1
            stack.append([l[status].getList(), 0])
    return result
```

# 2021/10/29 更新

昨晚写完本文之后，今早才发现原来网上已经有了很多文章介绍类似的技巧，其中示例问题、编程语言、具体实现有所不同，但思想是一致的，为了更全面的记录，我把这些文章都加进了参考链接里👇🏻。

# 参考链接

- [LeetCode 二叉树中序遍历题解](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/solution/er-cha-shu-de-zhong-xu-bian-li-by-leetcode-solutio/)

- [LeetCode 二叉树后续遍历题解](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/solution/er-cha-shu-de-hou-xu-bian-li-by-leetcode-solution/)

- [Converting Recursion to Iteration](https://www.cs.odu.edu/~zeil/cs361/latest/Public/recursionConversion/index.html)
