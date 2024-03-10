# Red-Black Tree

## Definition

1. A node is either red or black
2. The root is black
3. All leaves (NULL) are black
4. Both children of every red node are black
5. Every simple path from root to leaves contains the same number of black nodes.

## Insertion

### Core

  For red child and black parent, exchanging their color and rotate parent tree, the red child will transfer to opposite subtree.

### Cases

  First of all, insert node is red. We have cases as follows:

#### Parent is **black**

  No rule is broken. **Easy**.

#### Parent is **red**.

  Cause of rule 4 is broken, we have to rebalance the tree.

  The key to rebalance is looking at **uncle node**.

##### Uncle node is red.

  The grandparent must be **black**, we change parent & uncle tobe black, and grandparent to red.

  Every node keep the rules except grandparent, cause we don't know what is his/her parent's color.

  But for now, we can transfer insert node to grandparent, then, we can solve it recursively upward.

##### Uncle node is **black**.

  Grandparent must be black.

  Cause three is symmetrical, inserting right node is equivalent to inserting left node.

  Next, we introduce how to rebalance inserting node in left subtree.

  First, if parent is left child and i am right child.

<img src="https://raw.githubusercontent.com/lawlient/notes/master/c/date%20structure/tree/rbtree/data/insert_lr.png" alt="insert right child in left subtree">


  If we do right rotation directly, my new parent is still red.

  We can rotate subtree that root is parent first, then the situation is as follow.

![insert_lr_left_rotate](https://raw.githubusercontent.com/lawlient/notes/master/c/date%20structure/tree/rbtree/data/insert_lr_left_rotate.png)

  I am left child and my parent is left child now. It is same as inserting left node with left parent.

![insert_ll](https://raw.githubusercontent.com/lawlient/notes/master/c/date%20structure/tree/rbtree/data/insert_ll.png)

  Rotating root of grandparent, the tree will be balance.

![insert_ll_right_rotate](https://raw.githubusercontent.com/lawlient/notes/master/c/date%20structure/tree/rbtree/data/insert_ll_right_rotate.png)

## Deletion


### Core 

Look at brother node. Make situation as :
  1. brother is black
  2. the child of brother nearly me is black
  3. the child of brother far away from me is red
Then, make parent black and rotate at parent, the less of height is compensated, replace brother's color with parent. 
For the child of brother nearly me, the color of it's new parent is still black, height keep still.
For the child of brother far away from me, it's height is reduce 1 cause of rotate, making it black compensate for reducing.


####

Brother is red. Rotate at parent, new brother will be black.


####

Both child of brother is black. Making brother red to transfer less of black node to parent, solving it recursively.

####

The child of brother nearly me is red. Rotate at brother，it is going to be next.

####

The child of brother nearly me is black. It will be the situation descript in ![Core Chapter](#### Core)


## Reference

- [2-3-4 trees &amp; rbtree.pdf](https://www.cs.purdue.edu/homes/ayg/CS251/slides/chap13b.pdf)
- [ngx_rbtree.c](https://github.com/nginx/nginx/blob/master/src/core/ngx_rbtree.c)
- [ttdot: Drawing Trees in Dot](http://www.math.bas.bg/bantchev/ttdot/ttdot.html)
