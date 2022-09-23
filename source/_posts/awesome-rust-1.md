---
title: awesome-rust-1(IdentifierPattern)
tags: newbie
category: rust
date: 2022-09-23 16:00:32
index_img: /img/rust.png
---


# Rust *IdentifierPattern* - ref

## rust 启动！

这是awesome-rust的第一篇文章，所有文章的内容都将以问题由来+问题解决+官文链接的方案来解释（当然由于个人能力问题官文中的内容可能不能理解到位，请海涵）。

rust的官方reference：https://doc.rust-lang.org/reference/introduction.html 。我认为所有学习rust的同学都应该知道这份参考手册的存在，这份手册真的是太棒辣。

## *IdentifierPattern*

#### 问题产出

我在`codewars`平台进行练习题目 [ https://www.codewars.com/kata/52bef5e3588c56132c0003bc ] 后与他人代码对比后发现了 `ref` 正确使用的姿势之一。

这里值得提一嘴的是，对于刚步入一门新语言的“萌新”，`codewars`的题目推荐非常适合用于熟悉语言特点与语言标准库。

下面这个是Node的struct，同时Node并没有实现`Clone`或者`Copy`。

```rust
struct Node {
    value: u32,
    left: Option<Box<Node>>,
    right: Option<Box<Node>>,
}
```

#### 代码对比

**我的代码**

```rust
fn traverse(last_nodes: &mut VecDeque<&Node>, result: &mut Vec<u32>) {
    if last_nodes.len() == 0 {
        return;
    }
    let mut childrens = VecDeque::new();
    while let Some(node) = last_nodes.pop_front() {
      result.push(node.value);
      if let Some(n) = &node.left {
          childrens.push_back(n.as_ref());
      }
      if let Some(n) = &node.right {
          childrens.push_back(n.as_ref());
      }
    }
    traverse(&mut childrens, result)
}
```

**其他人的代码**

```rust
fn tree_by_levels_best(root: &Node) -> Vec<u32> {
    let mut result = vec![];
    let mut queue: VecDeque<&Node> = VecDeque::from([root]);
    while let Some(n) = queue.pop_front() {
        result.push(n.value);
        if let Some(ref l) = n.left { //关键的不同点
            queue.push_back(&*l);
        }
        if let Some(ref r) = n.right {
            queue.push_back(&*r);
        }
    }
    result
}
```

#### 解决问题

由于平时常用的`IdentifierPattern`多是 `mut`,而这里出现了 `ref`，虽然从字面上或者代码上我们可以理解 `ref` 的作用，但是我还是想要找出使用这个 pattern 的条件。 

官文链接：https://doc.rust-lang.org/reference/patterns.html#identifier-patterns 。

官文中明确指出了，标识符的pattern默认的两种做法

1.会优先考虑把标识符所表示的变量绑定到它所依赖的值的一份copy上。

2.会优先考虑把依赖数据move到变量中

关于copy，当然就是实现 Rust的 `Copy Trait`咯。

关于move，这里贴出官文的链接：https://doc.rust-lang.org/reference/expressions.html?highlight=move#moved-and-copied-types ，这里面提出了能够move的四种情况。在这里我认为不能进行move的主要原因是第一种情况

> [Variables](https://doc.rust-lang.org/reference/variables.html) which are not currently borrowed.

这里摘出关键的不同点进行拆分

```rust
if let Some(ref l) = n.left { 
    queue.push_back(&*l);
}
```

如果没有 `ref`，代码会以以下方式呈现，当rust的编译器进行checker时会发现我们的Node节点没有Copy的实现，同时在`Some(l) = n.left`这个`destructuring subpatterns`中发现l依赖的数据是`Box<Node>` 关于`Box`它相当于是一种共享型的能力，因此也并不满足`move`触发的条件。

```rust
if let Some(l) = n.left { 
    queue.push_back(&*l);
}
```

但是有了上面的`ref`我们会改变 identifier pattern 的默认绑定方式，相当于采用了下面的做法

```rust
if let Some(l) = &n.left { 
    queue.push_back(&*l);
}
```



另外，我在这个学习的过程中还发现了一个有趣的问题，就是rust的自动类型推测系统与实际需要效果的差异。

```rust
let mut queue: VecDeque<&Node> = VecDeque::from([root]); //这里是显示类型
```

```rust
let mut childrens = VecDeque::new(); //这里会infer出 VecDeque<&Box<Node>>,与实际需求略有差异
...
childrens.push_back(&*n);
...
```

欢迎各位前来指点，这是我的SO Question: https://stackoverflow.com/questions/73824648/rust-why-compiler-infers-a-different-type-beyond-our-expection

