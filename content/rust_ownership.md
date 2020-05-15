+++
title = "Rust中所有权,引用,生命周期小记"
date = 2018-11-29
category = "Rust"

[taxonomies]
tags = ["rust", "ownership", "lifetime", "RAII"]
+++

## rust 所有权,引用,生命周期

### 所有权

存在即是为了管理`堆数据`

- 跟踪代码正在堆上使用的数据
- 最大限度减少堆上的重复数据
- 清理堆上不再使用的数据

### rules
- Each value in Rust has a variable that’s called its owner.
    - rust每个值都有一个被叫做owner的变量
- There can only be one owner at a time.
    - 值只能有一个所有者
- When the owner goes out of scope, the value will be dropped.
    - 当所有者离开作用域,值将会被清除

### RAII

Resource Acquisition Is Initialization (RAII)

## 引用与借用

引用作为方法函数为`借用`, 脱离作用域值不会被丢弃掉

### 数据竞争(data race)

数据竞争先决条件:
- Two or more pointers access the same data at the same time.
    - 两个或多个指针同时访问同一数据
- At least one of the pointers is being used to write to the data.
    - 至少有一个指针被用来写入数据
- There’s no mechanism being used to synchronize access to the data.
    - 没有同步数据访问的机制

### rules
- At any given time, you can have either one mutable reference or any number of immutable references.
    - 任意给定时间,只能拥有一个可变引用或者多个不可变引用
- References must always be valid.
    - 引用必须总是有效的


## 生命周期
用来解决悬垂指针问题
悬垂指针(dangling pointer): 指针指向的内存已经被分配给其他自由者


TODO ...
