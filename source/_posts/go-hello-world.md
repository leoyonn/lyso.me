title: 初识go语言，写了个递归题目作为helloworld
date: 2012-11-24 23:22:11
tags: 
- go
- golang
categories: 
---
![image](https://lh6.googleusercontent.com/-mSbXs4DHQeg/TlVL0OUc49I/AAAAAAAAIkw/1kUwgCeYLLA/w800-h800/IMAG0217.jpg)
　　上周D总在公司TTT上普及了go，当时只是概念上知道这门语言，D总出的小题目我用Java写了类似于DP的算法，被宣布太耗内存^^。这两天在公交上看了会reference（http://golang.org ），最喜欢的特性是goroutines、多返回值和并列赋值/声明。觉得光看没用，还是写个helloworld吧，想到之前看到过一个这样的题目：

> 一颗二叉树，其节点上不均匀的分布了若干石头，石头数跟二叉树总节点数相同，石头只能在边上（即父子节点之间）进行搬运，每次只能搬运一颗石头。求使每个节点石头皆为一的最少搬运次数。
，甚至可能我的解法不是最优，这里主要是作为golang练手，在代码里详述。由于连机器上都还没装go环境，直接在http://play.golang.org 上写的：

```


package main
 
 import (
     "fmt"
 )
 
 var N int = 9
 var moves int = 0
 
 type Node struct {
     V int        // number of stones the node keep
     Feel int    // number of stones the tree have, can be positive or negative
     L *Node        // left child
     R *Node        // right child
 }
 
 func (node *Node) String() string {
     return fmt.Sprintf("{%d|%d}", node.V, node.Feel)
 } 
 
 /**
     what the mokced tree seems like:
      0|1
     / \
    1|3 2|1
   / \   \
|0 4|1 5|0
   \     / \
    6|1 7|0 8|2
 */
 func mock() *Node {
     // make new nodes
     nodes := make([]*Node, N)
     nodes[0] = &Node{1,0,nil,nil}
     nodes[1] = &Node{3,0,nil,nil}
     nodes[2] = &Node{1,0,nil,nil}
     nodes[3] = &Node{0,0,nil,nil}
     nodes[4] = &Node{1,0,nil,nil}
     nodes[5] = &Node{0,0,nil,nil}
     nodes[6] = &Node{1,0,nil,nil}
     nodes[7] = &Node{0,0,nil,nil}
     nodes[8] = &Node{2,0,nil,nil}
 
     // construct tree    
     nodes[0].L, nodes[0].R = nodes[1], nodes[2]
     nodes[1].L, nodes[1].R = nodes[3], nodes[4]    
     nodes[2].L, nodes[2].R = nil, nodes[5]
     nodes[3].L, nodes[3].R = nil, nodes[6]
     nodes[4].L, nodes[4].R = nil, nil
     nodes[5].L, nodes[5].R = nodes[7], nodes[8]
     nodes[6].L, nodes[6].R = nil, nil
     nodes[7].L, nodes[7].R = nil, nil
     nodes[8].L, nodes[8].R = nil, nil
     return nodes[0]
 }
 
 /**
  move stones between root, root.L, root.R, recursively.
 */
 func move(root *Node) {
     moves = 0
     print("init", root)
     count(root)
     print("after count", root)
     collect(root)
     print("after collect", root)
     welfare(root)
     print("after welfare, finally got", root)
     fmt.Println("all stone moves: ", moves)
 }
 
 /**
  count feel of stones and number of nodes of root.
 */
 func count(root *Node) (feel int) {
     if root == nil {
         return 0 
     }
     feelL := count(root.L)
     feelR := count(root.R)
     root.Feel = feelL + feelR + root.V - 1
     return root.Feel
 }
 
 /**
   collect redundant stones up from rich child(ren)
 */
 func collect(root *Node) {
     if root == nil {
         return
     }
     collect(root.L)
     collect(root.R)
     if root.L != nil && root.L.Feel > 0 {
         // todo: number of stones to collect
         todo := root.L.Feel
         moves += todo
         // move upward
         root.V += todo
         root.L.Feel = 0
         root.L.V -= todo
     }
     if root.R != nil && root.R.Feel > 0 {
         todo := root.R.Feel
         moves += todo
         root.V += todo
         root.R.Feel = 0
         root.R.V -= todo
     }
 }
 
 /**
   dispatch all stones collected to poor child(ren)
 */
 func welfare(root *Node) {
     if root == nil {
         return
     }
     if root.L != nil && root.L.Feel < 0 {
         todo := -root.L.Feel
         root.L.Feel = 0
         root.L.V += todo
         root.Feel = 0 
         root.V -= todo
         moves += todo
     }
     if root.R != nil && root.R.Feel < 0 {
         todo := -root.R.Feel
         root.R.Feel = 0
         root.R.V += todo
         root.Feel = 0
         root.V -= todo
         moves += todo
     }
     welfare(root.L)
     welfare(root.R)
 }
 
 /**
  bfs print using chan as queue
 */
 func print(title string, root *Node) {
     fmt.Println("==========| ", title, " |==========");
     queue := make(chan *Node, N)
     queue <- root
     i := 0
     for node := range queue {
         fmt.Print(node, "; ")
         if node.L != nil {
             queue <- node.L
         }
         if node.R != nil {
             queue <- node.R
         }
         if i += 1; i == N {
             close(queue)
             break
         }
     }
     fmt.Println()
 }
 
 func main() {
     move(mock())
 }
```

　　其中最自我感觉良好的是，自己想到了个主意用chan作为queue把二叉树BFS打印出来^^，虽然这个在熟手们可能是常用做法。

　　代码里略写了些注释。

　　如果：
 1. 思路/逻辑不正确；
 2. 还有比这更优化的解法；
 3. 有些代码不符合go规范或约定；

　　请不吝点评：D
