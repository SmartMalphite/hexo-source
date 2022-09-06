---
title: LeetCode.919 | 完全二叉树插入器
date: 2022-08-09 22:25:49
categories: 
    - LeetCode
tags: 
    - 完全二叉树
---

## [题目描述](https://leetcode.cn/problems/complete-binary-tree-inserter/)
![](https://raw.githubusercontent.com/SmartMalphite/PicBed/master/img-hexo/20220809230841.png)

## 题目翻译
实现一个算法，支持初始化、数据插入和返回根节点。其中初始化阶段的入参是一个已有的完全二叉树。

## 心路历程
### 第一问: 怎样的节点可以插入数据?
该节点的左右节点任一节点为空

### 第二问: 怎样找到这样的节点?
自然而然的想法就是遍历这颗完全二叉树。但是遍历一颗树的方法有很多，在本题中我们应该采用哪种方法呢?
其实很容易想到我们在自己动手插入数据的时候，一定是逐层、从左向右的插入。即先填充最后一行非叶子结点中尚未饱和的节点，再在叶子结点的行从左到右依次添加子节点。
**(注意: 以上说法的前提是这是一颗完全二叉树)**

#### 落地: 怎么横向遍历一颗完全二叉树?
1. 将根节点放到一个list中。
2. 将其左右节点(如果存在)也放到list末尾
3. 将根节点从list中踢出，list中的原2号位作为新的根节点
4. 重复2-3，直到list为空

```go
nodeList := []*TreeNode{root} //step 1
for len(nodeList) > 0{
    currentRoot := nodeList[0]

    //step 2
    if currentRoot.Left != nil{
        nodeList = append(nodeList, currentRoot.Left)
    }
    if currentRoot.Right != nil{
        nodeList = append(nodeList, currentRoot.Right)
    }

    nodeList = nodeList[1:] // step 3
}
```

### 第三问: 找到这些节点后怎么用?
组织一个候选list，按照遍历的顺序将满足条件的节点依次插入其中。Insert函数调用时，只需要取第一个节点判断向左插还是向右插:
1. 向左插: 完成后无需其他操作
2. 向右插: 完成后list第一个节点已经饱和，将其移出list
最后，不论向左还是向右插入数据后，都需要在候选list的最后加入本次插入的child节点
**因为遍历得到的候选list是逐层、从左到右的顺序。直接向后追加child依旧能保证此顺序**

### 第四问: 初始化时数据结构如何?
综上所述，为了实现根节点返回函数，显然我们需要在数据结构中记录根节点root;
为了实现Insert方法，我们需要记录候选节点的list
```go
type CBTInserter struct {
    root *TreeNode
    candidate []*TreeNode
}
```

---

## 整体代码实现
```go
type CBTInserter struct {
    root *TreeNode
    candidate []*TreeNode
}

// 初始化函数需要做的就是将root记录下来，并计算出第一版候选人名单
// 如何计算候选人名单: 根据上述第二步的思路，在遍历的过程中找到符合第一步要求的节点，并将其记录下来
func Constructor(root *TreeNode) CBTInserter {
    nodeList := []*TreeNode{root}
    candidate := []*TreeNode{}
    for len(nodeList) > 0{
        currentRoot := nodeList[0]
        if currentRoot.Left != nil{
            nodeList = append(nodeList, currentRoot.Left)
        }
        if currentRoot.Right != nil{
            nodeList = append(nodeList, currentRoot.Right)
        }
        nodeList = nodeList[1:]
        if currentRoot.Left == nil || currentRoot.Right == nil{ // 找到目标节点，记录在候选名单末尾
            candidate = append(candidate, currentRoot)
        }
    }
    return CBTInserter{root, candidate}
}

// 插入数据，按照第三步实现即可
func (this *CBTInserter) Insert(val int) int {
    child := &TreeNode{val, nil, nil}
    node := this.candidate[0]
    if node.Left == nil{
        node.Left = child
    }else {
        node.Right = child
        this.candidate = this.candidate[1:]
    }
    this.candidate = append(this.candidate, child)
    return node.Val
}


func (this *CBTInserter) Get_root() *TreeNode {
    return this.root
}
```