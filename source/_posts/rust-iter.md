---
title: 一文入门rust iterator
date: 2022-09-21 11:20:22
tags: newbie
category: rust
index_img: /img/rust.png
---

# 一文入门rust iterator

## 1.本文由来

秉承一贯的做法，我还是想说说为什么我会想要写下这篇文章。起因是这样的，作为一名刚想rust的新手，我找到了一个锻炼rust语法比较好的平台`codewars`，当然我在之前就已经阅读了rust最优质的文章资源：https://doc.rust-lang.org/book/ch01-00-getting-started.html ，以及使用过`rustlings`进行了一些的知识点检查。在锻炼的过程中，我发现那些简短而优雅的代码不仅仅是逻辑运用更加熟练，同时他们对于标准库的掌握与熟悉程度也是非常得高。

## 2.一个简单的入门级rust iterator

#### 1.让我们自己进行便利操作我们会怎么做？

假设我们有以下这样的结构，我们如果要遍历并对每个元素进行相应的操作时应该怎么做？

```rust
struct Bag<T> {
    elms: Vec<T>,
}
/**
struct Bag<T> (Vec<T>) 这种结构也可以，主要是为了不直接使用Vec，只使用他的装载功能
*/
```

#### 2.大部分人可能的做法

如果让我们自己来进行实现，我们认为对于一个新手来说最容易想到的就是，实现可以传入闭包的“方法”。

```rust
pub fn traverse<F>(&self, f: F)
    where
        F: Fn(T, usize) -> (),
    {
        for i in 0..self.elms.len() {
            f(self.elms.get(i).unwrap().clone(), i);
        }
    }
```

#### 3.问题分析

以上做法没有问题，并且能够实现特别多的需求但是上面的做法在不连续的遍历中就显得别不是那么得灵活。如果想要一种可以完全控制其遍历进度并且充分灵活的方案就需要迭代器的加入。

跟随着官网对于迭代器的实现，我在最开始实现了以下一个错误案例。

**注意：**这是错误案例

```rust
struct Bag<T> {
    elms: Vec<T>,
    idx: usize,
}
impl<T: Copy> Iterator for Bag<T> {
    type Item = T;
    fn next(&mut self) -> Option<Self::Item> {
        if self.idx < self.elms.len() {
            let result = Some(self.elms.get(self.idx).unwrap().clone());
            self.idx += 1;
            result
        } else {
            None
        }
    }
}
```

在这个问题中我们会发现，我们可以进行迭代，但是问题是我们的迭代只能遍历一次，因为`idx`的索引值会越界。这个时候有人会说在遍历到最后一个的时候重置索引值，这样可行但是如果我们同时需要两个Bag的迭代器呢？ok，有人会说为`Bag`提供`Clone Trait`，但是这样的问题就是不必要的内存开辟。

所以对于`Iterator`，我们应该将他看成是一个独立的个体，但同时他又必须与需要遍历的对象保持一定的联系。

#### 4.最终的简单方案

1.首先我们需要一个独立的遍历器

```rust
pub struct BagIter<T> {
    idx: usize,//这是独立的访问索引
    elms: Rc<RefCell<Vec<T>>>, //记录我们需要遍历的context
}
impl<T> BagIter<T> {
    fn new(elms: Rc<RefCell<Vec<T>>>) -> Self {
        BagIter { idx: 0, elms }
    }
}
impl<T> Iterator for BagIter<T>
where
    T: Copy,
{
    type Item = T;
    fn next(&mut self) -> Option<Self::Item> {
        let elms = self.elms.borrow();
        match self.idx < elms.len() {
            true => match elms.get(self.idx) {
                Some(&x) => {
                    self.idx += 1;
                    Some(x)
                }
                None => None,
            },
            false => None,
        }
    }
}
```



2.实现一个 `iter` 每次调用 `iter()`生成一个新的独立遍历器。

```rust
pub struct Bag<T> {
    elms: Rc<RefCell<Vec<T>>>,
}
//can only put things into Bag
impl<T> Bag<T> {
    pub fn add(&mut self, item: T) {
        self.elms.borrow_mut().push(item)
    }
    pub fn new() -> Bag<T> {
        let elms = Rc::new(RefCell::new(vec![]));
        Bag { elms: elms }
    }
    pub fn size(&self) -> usize {
        self.elms.borrow().len()
    }
    pub fn iter(&self) -> BagIter<T> {
        BagIter::new(self.elms.clone())
    }
}
```



#### 5.满足你的 for ... in expression

众所周知在一套完美的第三方库或者标准库下，rust为第三方开发者提供的体验是非常优越的，其主要源自`Trait`以及一些隐式编译的实现。

```rust
impl<T: Copy> IntoIterator for Bag<T> {
    type Item = T;
    type IntoIter = BagIter<T>;
    fn into_iter(self) -> Self::IntoIter {
        self.iter()
    }
}
```

使用`IntoIterator`可以让你的代码在编译时生成对于self.iter()的迭代器。另外根据`IntoIterator for &mut Bag<T>`, `IntoIterator for & Bag<T>`的不同也可以实现不同的借用、修改迭代器，当然咯，这个都是得重写一份逻辑的。



ok，希望这篇入门能够帮你建立如何编写迭代器的认知。
