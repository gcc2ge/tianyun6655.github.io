---
layout: post
title: 二叉搜索树(未完成)
subtitle: 二叉搜索树详解
date: 2017-11-09
author: Tianyun Chen (OFBank)
header-img: img/sorting1.png
catalog: true
tags:
   - Blog
   - 数据结构
   - tree
---

# 前言

今天开始数据结构的第一章，emmmm，脑海里第一个出现的数据结构是tree，竟然不是链表。既然出现了tree ，那么今天就来介绍下二叉搜索树这种数据结构。

## 二叉树

 简单的介绍树，听名字就知道是二叉搜索的退化版。简单的来说每个节点最多只有2个子节点的树。二叉树第i层最多有2^{i-1}个结点。这里还要说一下完全二叉树，所谓的完全二叉树就是深度为h的二叉树，除h层外，所有的层的结点数达到最多且第h层的结点都位于左边。

 ![完全二叉](https://github.com/tianyun6655/tianyun6655.github.io/blob/master/img/tree1.png?raw=true)
这个就是完全哒。完全二叉树主要用于堆这种数据结构，由于是完全二叉树，所以可以用数组来实现这种数据结构。堆的话以后在讲吧

## 二分搜索
开始讲二叉搜索树前，先介绍一种搜索的算法，就是大名鼎鼎的二分查找，时间复杂度为O(logN)，这种查找的前提是排好序的数组。拿target跟中间数进行对比如果等于就找到了，没有的话，则target小于中间数的话就从左半边继续查找，大的话右半边咯

![binarySearch](https://github.com/tianyun6655/tianyun6655.github.io/blob/master/img/tree2.png?raw=true)

```
func binarySearch(arr []int, n int, target int) int{

	l :=0
	r:=len(arr)-1

	for l<=r{
		//有bug
        // mid:=(l+r)>>1
        mid:=l+(r-l)/2

        if arr[mid]==target{
        	return mid
		}
		if target<arr[mid]{
			r=mid-1
		}else{
			l = mid+1
		}
	}

	return -1
}
```
这里一行代码(l+r)/2这行代码被我注释掉了，原因是这种方法找中间点其实有bug，因为如果l和r都是int的最大值那么就会造成l+r溢出了，所以改成减法会好点。

由于二分查找有具有很强烈的递归性质，每次查找都是取数组的一半。所以可以用递归来实现，就当联系下自己的递归算法：

```
func BinarySearchRecurison(arr []int, n int,target int) int{

   return  binarySearhHelper(arr,0,n-1,target)

}

func binarySearhHelper(arr []int,left,right,target int) int{

	if left < right {
        mid:=left+(right-left)/2
        if arr[mid]==target{
        	return mid
		}
	if arr[mid]>target{
		return  binarySearhHelper(arr,left,mid-1,target)
	   }else {
	     return binarySearhHelper(arr,mid+1,right,target)
	}

	}else{
		return -1
	}
}
```
j

## 二叉搜索树
### 介绍
好啦，来重点啦，为什么要使用二叉搜索树这种数据结构，不是因为这种结构很酷或者为了证明自己很腻害。是因为这种数据结构在效率上有很大提升。


|               | 查找           | 插入   | 删除   |
| ------------- |:-------------:| -----:| -----:|
| array         | O(n) | O(n) |O(n)
| BST      | O(logn)     |   O(logn)   |O(logn)  

目前只跟array来进行比较，之后介绍完其他数据结构的时候在加入其它哒。针对于这种数据结构可以发现效率不管在查找，插入，删除都高过于我们直接用数组。同时二叉搜索树在对于特殊要求的数据也有很高的效率。比如查找最大，最小数，查找特定的区间等等等等

### 特点
 - 可以不是完全二叉树(由于不是完全二叉树所以我们没有办法用array作为底层来实现)
 -  每个节点都大于他的左子结点
 -  每个节点都小于他的右子结点
 
 也就是这种要求，结点的子孩子也是一颗二叉搜索树

![bst](https://github.com/tianyun6655/tianyun6655.github.io/blob/master/img/tree3.png?raw=true)

### 代码实现
 急于树的特点我们需要建立一个node 的结构体，这个结构体里面放了，当前的数值和之后左孩子和右孩子,然后BST里面要存有根结点


 #### 基本元素

 ```
 type node struct{
	value int
	leftNode *node
	rightNode *node
}

type BinarySearchTree struct {
    rootNode *node
    size int
}

 ``` 
 #### 基本方法

 ```
 func NewBinarySearchTree() *BinarySearchTree{
	return &BinarySearchTree{size:0}
}


func(bst *BinarySearchTree)GetSize() int{
	return bst.size

}

func (bst *BinarySearchTree)IsEmpty() bool{
	return bst.size==0
}
 ```

 #### 插入
 由于每个子结点都是二叉搜索树，所以二叉搜索树具有很好递归性，我们在写 数据机构的方法的时候用递归来做的话是个很好的选择

 ##### 递归
```
func (bst *BinarySearchTree)Insert(value int){

    bst.rootNode = insert(bst.rootNode,value)

}

//返回插入节点后对应的根
func insert(rootNode *node, value int) *node{

	if rootNode==nil{

		return &node{value:value}

	}

	if value == rootNode.value{
		rootNode.value = value
	}else if value < rootNode.value{
		rootNode.leftNode = insert(rootNode.leftNode,value)
	}else if value >rootNode.value{
		rootNode.rightNode = insert(rootNode.rightNode,value)
	}

	return rootNode

}
```
插入的时候需要把 target 一直跟当前结点进行对比，如果比当前结点小的话，就跑到当前结点的左子结点那继续对比，如果大的话，就抛弃当前结点跑到结点的右孩子去，一直到碰到一样的值了，或者 没有结点对比了直接就坐下来把位置占据了，递归的话记住每次要把结点返回出来哦，可能有人会对这个递归有点蒙逼，我带着大家走一遍，我们就用上图的树好了,我们要把0插进去，那也就是最最左边:
insert(10节点, 0){
    10的左节点 = insert(5节点, 0){
        5的左节点 = insert(5节点, 0){
            2的左节点 = insert(1节点，0){
               1的左节点 = insert(Nil,0){
                   return newNode(0)
               }
               return 1节点
            }
            return 2节点
        }
        return 5节点
    }
    return 10节点
}
从 newNode ->返回1 -> 返回2 ->返回 5 ->返回10 从而就把0位置找到了

##### 非递归

```
func (bst *BinarySearchTree)InsetrWithoutRe(value int){

	if bst.rootNode==nil{
		bst.rootNode = &node{value:value}
		return
	}

	 currentNode :=bst.rootNode
     parentNode :=bst.rootNode

	 for currentNode!=nil{

	 	parentNode = currentNode

		 if value < currentNode.value{
			 currentNode = currentNode.leftNode
		 }else if value >currentNode.value{
			 currentNode= currentNode.rightNode
		 }
	 }

   if parentNode.value >value{
   	parentNode.leftNode = &node{value:value}
   }else{
   	parentNode.rightNode = &node{value:value}
   }


}
```
需要一个值保存 最后一个节点哦

### 遍历

二叉搜索树的遍历分3种
- 前序：先访问自身，在递归 访问左右(pre)
- 中序：先左，在自身最后右边(in)
- 后序：先左右后自身(post)

```
func (bst *BinarySearchTree)PreOrder(){
	preOrder(bst.rootNode)
}

func preOrder(rootNode *node){


	if rootNode!=nil{
		fmt.Println(rootNode.value)
		preOrder(rootNode.leftNode)
		preOrder(rootNode.rightNode)
	}
}

func (bst *BinarySearchTree)InOrder(){
	inOrder(bst.rootNode)
}

func inOrder(rootNode *node){

	if rootNode!=nil{
		inOrder(rootNode.leftNode)
		fmt.Println(rootNode.value)
		inOrder(rootNode.rightNode)
	}
}

func (bst *BinarySearchTree)PostOrder(){
	postOrder(bst.rootNode)
}

func postOrder(rootNode *node){

	if rootNode!=nil{
		postOrder(rootNode.leftNode)
		postOrder(rootNode.rightNode)
		fmt.Println(rootNode.value)

	}
}
```
其实这三种代码的模版一毛一样 只是输出的位置不一样，可以自己试着输出一下
