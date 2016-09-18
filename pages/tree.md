#树和二叉树遍历

**树：**是一种抽象的数据类型，用来模拟具有树状结构性质的数据集合。树由n（n>=1）个有限节点，有层次的组成。可以看做是一个倒挂的树。根据图论来说的话，树是一种无向图，任意两点间存在唯一的一个连接路径。<br />
树的特点：

* 每个节点有零个或多个子节点；
* 没有父节点的节点称为根节点；
* 每一个非根节点有且只有一个父节点；
* 除了根节点外，每个子节点可以分为多个不相交的子树；

树分类：

* 无序树 ：子节点之间没有顺序关系
* 有序树 ：子节点之间有顺序关系
	* 二叉树	：每个节点最多含有两个子树的树
		* 二叉搜索树	：左子树小于父节点，右子树大于父节点
		* 完全二叉树	：除了最后一层以外，每层的节点都是满节点，当最后一层是满节点时候也叫满二叉树
		* 平衡二叉树 ：叶节点深度差不超过1
	* 霍夫曼树 ：带权路径最短的二叉树
	* B树 ：多余两个子树的有序查找树
	
	
常见问题：**树的遍历**

二叉树的遍历（前、中、后序）
![tree](http://ocaya4boy.bkt.clouddn.com/image/tree.jpeg?imageView2/0/w/200/h/250)

前序遍历：先根节点，然后做子树，最后右子树<br />
中序遍历：先左子树，然后根节点，最后右子树<br />
后序遍历：先左子树，然后右子树，最后根节点<br />

递归实现的方法：以前序遍历为示例，其它可以调换一下输出的位置来实现打印顺序。

```
void preorder(bintree t){
    if(t){
        printf("%c ",t->data);
        preorder(t->lchild);
        preorder(t->rchild);
    }
}
```

二叉树的**插入、查找**<br />
先看一下搜索二叉树的结构
![search_tree](http://ocaya4boy.bkt.clouddn.com/image/2/search_tree.jpeg?imageView2/0/w/300)

插入步骤：

* 如果没有根节点，那么插入的数就是根节点。
* 如果有根节点和根节点对比
	* 相等返回
	* 不等继续对比
		* 小于往左子树方向继续寻找
		* 大于往右子树方向继续寻找
		* 如果方向节点为空，那么该位置就是插入位置
		  
现在要插入16，先与根节点15对比，大于右子树，所以继续和18对比；和右子树对比发现小于该节点，所以和18的左子树对比，没有这个节点，所以16插入这个位置。
![search_tree](http://ocaya4boy.bkt.clouddn.com/image/2/search_tree_1.jpeg?imageView2/0/w/300)

查找步骤：<br />
查找步骤和插入方式一样，按照搜索二叉树的结构，只是在相等的时候返回就可以了；

删除：<br />
根据节点的位置不同删除也分下面三种情况：<br />

* 删除的节点下面没有其他节点
* 删除的节点下面有一个子节点
* 删除的节点下面有两个子节点

看一下第一种：删除26（节点下面没有其他节点直接删除就可以）<br />
![search_tree](http://ocaya4boy.bkt.clouddn.com/image/2/search_tree_2.jpeg?imageView2/0/w/300)

第二种：删除10（节点下面有一个子节点，那么可以让8直接之下10下面的节点）<br />
![search_tree](http://ocaya4boy.bkt.clouddn.com/image/2/search_tree_3.jpeg?imageView2/0/w/300)

第三种：删除7（节点下面有两个子节点，先删除7，找到右节点中的最小的值，替换7，然后删除，子节点比较长的可能要递归）<br />
![search_tree](http://ocaya4boy.bkt.clouddn.com/image/2/search_tree_4.jpeg?imageView2/0/w/300)

下面是伪代码：

```
delete(tree, node){
	node f;
	//	先查找节点
	!find
		return null;
		
	find(f.left == node || f.right == node)	// 这里标记是左分子还是右分子查找到
	//	判断节点的左右子节点
	//	当没有子节点时候直接删除
	if(node.left == null && node.left == null)
		remove(node.element);
	
	//	当有子节点时候，
	if(node.left != null || node.right != null)
		f.left || f.reght = node.left || node.right; // 这里把非空的子分支赋值给上面的标记的查找节点
		
	if(node.left != null && node.right != null)
		node s = find(node.right.small); //	查找右分子最小值(右分子的最深层的左分子值)
		node.element = s.element;
		delete(s);	//	这里还需要执行一个删除最小值的操作
}
```

学习中的总结，可能会有错误的地方。:-D
