---
title: rust-move-borrow
date: 2022-10-16 13:07:44
tags: newbie
category: rust
index_img: /img/rust.png
---

# 一题看所有权问题

## 写文原因

众所周知，在rust中所有权问题是一个非常重要的问题，他直接影响了一个程序的性能。另外配合上rust的检查器+rust的生命周期，他能在很多情况下及时反馈出程序的潜在问题，这篇文章将会用一个leetcode题目分析一下如何在编写程序的时候进行合理的思考。  

https://leetcode.cn/problems/merge-two-sorted-lists

**【注】：**本文并不是算的讲解，最好在能够思考出算法的实现方案下阅读。

## move与borrow

关于变量表达式的问题：https://doc.rust-lang.org/reference/expressions.html?highlight=move#place-expressions-and-value-expressions

**关于这页相关理论，下面这句话尤为需要注意，这句话在循环中能起到非常大的作用。**

> After moving out of a place expression that evaluates to a local variable, the location is deinitialized and cannot be read from again until it is reinitialized

关于所有权的理论在这个链接中有详细的说明：https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html

下面我们简单的讲一下，接下来需要的理论

#### 1.所有权

对于一个临时且不是reference的变量我们可以认为我们拥有了它的所有权，此时对于我们声明的变量我们可以认为我们拥有了它的所有权，变量内存地址位置就会开辟出装载该变量所需要的内存空间。例如：

```rust
let node = ListNode::new(1);
```

#### 2.可变性

默认情况下rust视为我们不能对变量的内存空间进行操作，即使拥有所有权依旧需要`mut`pattern来提供可操作该区域内存空间的能力。即使你是房子的主人，也需要先用钥匙打开房子的门。嗯！没错，就是这种感觉。

```rust
let mut node = ListNode::new(1);
node.next = Some(Box::new(ListNode::new(2)));
//下面是没有mut编译器的报错
//cannot assign to `node.next`, as `node` is not declared as mutable cannot assign
```

#### 3.不可变借用

借用这个概念真的非常贴近我们日常的表述了，就是东西我借给你了，但是你要原模原样得还给我。

在代码中，我们可以理解为将数据内存空间的读取能力分发给了其他变量。

```rust
let node = ListNode::new(1);
let node2 = &node;
```

#### 4.可变借用

与不可变借用相似，提供了可修改原有数据内存空间的能力。东西仍然在我这里，但是你修改之后，我也这边也产生了变化。

但是在这种借用前，这个东西首先是能够修改的，也就是绑定了`mut` pattern。另外可变借用的操作符号是`&mut`。

这篇官文也说明了使用可变借用的其他性质：https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#mutable-references

```rust
let mut node = ListNode::new(1);
println!("{}", &node.val);
let node2 = &mut node;
node2.val = 2;
println("{}", node.val);
```

## 具体过程

优秀的代码在产出前一定有很多的思考，但是在产出之前rust会让你增加对于代码结构编织的思考，不然你就可能只能将伪代码保存在你的脑子里了。对于我们这样的新手如果不认真思考的话就不是项目规模化后的问题了，而是能不能让编译器爸爸放你过关😂。

#### 算法简析：对于两条链合并情况的分析

1. 两条空链
2. 只有一条 链/子链 存在
3. 两条 链/子链 都存在

#### 将思路具体化

1.创建防止第一个节点就是NULL而无法正确执行算法的辅助节点

```rust
let mut head = ListNode::new(-101);
let mut cur = &mut head;					 //cur变量的内存地址中应该要记录当前连接到的最后一个节点，另外最后一个点就是cur记录的是一个地址，但是我们在拼接时需要修改这个地址，而不单单是修改地址所指的数据空间。
...
head.next
```

2.在两条子链都存在的情况下进行的拼接操作

While let expression:https://doc.rust-lang.org/reference/expressions/loop-expr.html#predicate-pattern-loops

Match expression: https://doc.rust-lang.org/reference/expressions/match-expr.html

> If the scrutinee expression is a [value expression](https://doc.rust-lang.org/reference/expressions.html#place-expressions-and-value-expressions), it is first evaluated into a temporary location, and the resulting value is sequentially compared to the patterns in the arms until a match is found.

针对这个问题我们可以做出以下两种思考

思考一：

```rust
while let (Some(n1), Some(n2)) = (&list1, &list2) {//在这里使用借用，这样不会导致list的所有权转移到这里，然后后面就会无法访问list1，list2
}
```

思考二：

```rust
while let (Some(n1), Some(n2)) = (list1, list2) {//这就是转移所有权并使用tuple pattern进行结构赋值，这样n1，n2所有权也在当前作用域下。但是使用这种方案我们需要进行一点修改，因为这样的所有权解构会造成list1，list2无法在后续访问
}
```

3.取小的节点进行拼接

思考一的body

```rust
while let (Some(n1), Some(n2)) = (&list1, &list2) {//list1在未重分配前会飘红
	if n1.val < n2.val {
      cur.next = list1; //1.这里move进去，当我们写到这里在IDE环境我们会发现 &list1 这里出现 borrow after move的飘红报错，这时请用好上面说到的 重分配理论，请自信使用为list1分配一个所有权的value的代码
      cur = cur.next.as_mut().unwrap();  //因为cur是一个可变借用，所以需要一个可变借用
      list1 = cur.next.take(); //2.从cur.next这里move出来，并未list1重分配
  } else {
      //相同道理
  }
}
cur.next = if list1.is_some() { list1 } else { list2 };
```

思考二的body

```rust
loop {
    if list1.is_some() && list2.is_some() {//这么做是为了防止while let最后一次解构引起后续list1，list2无法borrow的问题
        let (n1, n2) = (list1.unwrap(), list2.unwrap());
        if &n1.val < &n2.val {
            cur.next = Some(n1);
            cur = cur.next.as_mut().unwrap();
            list1 = cur.next.take();
            list2 = Some(n2)
        } else {
            //相同道理
        }
    } else {
        break;
    }
}
cur.next = if list1.is_some() { list1 } else { list2 };
```

## 总结

在rust代码的编写中我们需要更多地去思考，而不是过度依赖于编译器，要做到有理有据。过度依赖编译器很容易不自信啦。
