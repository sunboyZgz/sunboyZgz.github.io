---
title: awesome-rust-2
tags: newbooie
categories: rust
---

# awesome-rust-2(那些智能指针)

RefCell 与 Cell

总之，当非要使用内部可变性时，首选 `Cell`，只有你的类型没有实现 `Copy` 时，才去选择 `RefCell`。