+++
title = "Rust中所有权,引用,生命周期小记"
date = 2018-11-02T16:13:00+08:00
tags = ["rust"]
draft = false
author = "lyraile"
+++

## 所有权 {#所有权}

存在即是为了管理 `堆数据` <br/>

-   跟踪代码正在堆上使用的数据 <br/>
-   最大限度减少堆上的重复数据 <br/>
-   清理堆上不再使用的数据 <br/>


## rules {#rules}

-   Each value in Rust has a variable that's called its owner. <br/>
    -   rust每个值都有一个被叫做owner的变量 <br/>
-   There can only be one owner at a time. <br/>
    -   值只能有一个所有者 <br/>
-   When the owner goes out of scope, the value will be dropped. <br/>
    -   当所有者离开作用域,值将会被清除 <br/>


## RAII {#raii}

Resource Acquisition Is Initialization (RAII) <br/>


## 引用与借用 {#引用与借用}

引用作为方法函数为=借用=, 脱离作用域值不会被丢弃掉 <br/>


### 数据竞争(data race) {#数据竞争data-race}

数据竞争先决条件: <br/>

-   Two or more pointers access the same data at the same time. <br/>
    -   两个或多个指针同时访问同一数据 <br/>
-   At least one of the pointers is being used to write to the data. <br/>
    -   至少有一个指针被用来写入数据 <br/>
-   There's no mechanism being used to synchronize access to the data. <br/>
    -   没有同步数据访问的机制 <br/>


### rules {#rules-1}

-   At any given time, you can have either one mutable reference or any number of immutable references. <br/>
    -   任意给定时间,只能拥有一个可变引用或者多个不可变引用 <br/>
-   References must always be valid. <br/>
    -   引用必须总是有效的 <br/>


## 生命周期 {#生命周期}

用来解决悬垂指针问题 悬垂指针(dangling pointer): <br/>
指针指向的内存已经被分配给其他自由者 <br/>

TODO ...