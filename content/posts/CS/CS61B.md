---
title: "CS61B 2025Fall" #标题
date: 2025-11-03T17:49:22+01:00 #创建时间
lastmod: 2025-11-03T17:49:22+01:00 #更新时间
categories: [""]
tags: [""]
description: "" #描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
# draft: false # 是否为草稿
# comments: true #是否展示评论
# showToc: true # 显示目录
# TocOpen: true # 自动展开目录
# hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
# disableShare: true # 底部不显示分享栏
# showbreadcrumbs: true #顶部显示当前路径
# cover:
#     image: "" #图片路径：posts/tech/文章1/picture.png
#     caption: "" #图片底部描述
#     alt: ""
#     relative: false
---

# [代码书写规范](https://fa25.datastructur.es/resources/style-guide/)

## 空格
- 只能用空格，不要用tab
- package 和 类之间空行
- 在 冒号、逗号、类型声明、比较符号、加减乘除、等号、问好和冒号组成的条件判断 之间空格
- 在 `assert catch do else finally for if return try while` 两边加空格
- 在<>两边不加空格，例如`List<Integer>`
- 在!, --, ++, unary -, or unary + 之后不加空格
- 在分号之前、句号之后、前括号后以及后括号前不能有空格

- 如果想断行，从运算符号之前另起一行

## 缩进
- 基础是四个空格


## 花括号
- if
- while
- for
- do


## 大小写
- 静态变量用大写字母
- 参数、局部变量、方法首字母小写，之后每个单词首字母用大写
- 类型、类型参数的命名首字母用大写
- packages 小写开头
- 实例变量和非final className变量用小写字母或者 `_` 开头

## Java 风格规范
- String[] names
- 只在必要的时候 public

书写一般顺序：
- public, protected, or private.
- abstract or static.
- final, transient, or volatile.
- synchronized.
- native.
- strictfp.

## 容易出错的构造需要避免
- 不要用 == 比较字符串
- 每个 switch 都记得写一个默认行为

## 限制
单个文件代码不超过 2000 行，每行不超过 120 字符；
每个方法不超过 80 行，不超过 8 个参数；
每个文件只包含一个 outer className （nested classNamees 不算）


# Java Introduction

1. 所有的代码都要放在className中: `public className className { }`
2. 每行结束都需要分号`;`
3. 我们需要运行的代码放在 `public static void main(String[] args)`中
4. 声明变量需要确定变量类型 `int x = 0`，声明之后变量类型不可更改
5. 代码在运行之前先进行验证，
6. 在java中定义Function：必须在className中定义函数（methods）——`public static`，所有的参数都要定义数据类型，返回值也需要定义数据类型，只能返回一个值

## 静态类型的优缺点
优点：
- 捕捉类型错误，更容易调试
- 客户端几乎不会出现类型错误
- 增加可读性
- 不需要增加类型检查，代码运行更有效率

缺点：
- 代码更冗长
- 代码通用性更差

## Java编译（Compilation）
Hello.java -> javac compiler编译器 -> Hello.className -> java interpreter解释器 -> 运行

.className文件的作用
- 类型检查，确保代码安全
- 对于机器更好运行
- 保护知识产权（一定程度上）

# Data Struction

## The Disjoint Sets Data Structure


### Asymptotics

- for loop (嵌套循环 sum of first natural numbers): O(N^2) 
- for loop (sum of first powers of 2): O(N) 
- recursion: O(2^N)
- binary search: O(logN)
- merge sort: O(NlogN)

### Tree

要求数据 comparable

#### BST (binary search trees)

Runtime: O(N)

#### 2-3 tree (B trees)

Runtime: O(logN)

很难实现

#### red black trees (LLRBs)

Runtime: O(logN)
最长路径：2N + 1

Five rules:
1. 每个节点都是红色或者黑色 Chaque nœud soit rouge, soit noir.
2. 根结点必须是黑色 Le racine est noire.
3. 所有叶节点必须是黑色 Toutes les feuilles sont noires
4. 如果有一个节点是红色，那么他的两个子节点都必须是黑色， 红色节点不能相邻 Si un nœud est rouge, alors ses deux enfants doivent être noirs.
5. 从任一节点到所有叶子节点的简单路径上所包含的黑色节点数量相同 Pour tout nœud, le nombre de nœuds noirs sur tout chemin simple menant àun descendant feuille est identique.

### Hash

重要规则：
- override equals 方法，确保添加相同数据有相同的 hashcode
- Never mutate an Object being used as a key (修改后的数据会丢失)

``` Immutable data types： `public final int month` -- an instance cannot change in any observable way ```
