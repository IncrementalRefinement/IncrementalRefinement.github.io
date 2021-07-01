---
layout: post
title: Python 命名空间
tags: [Python]
---

一点关于 Python 中命名空间的理解

## Python Namespace

Python的namespaces和Scope resolution遵循LEGB (Local -> Enclosed -> Global -> Built-in) 规则, 逐层向外寻找

```
a_namespace = {'name_a':object_1, 'name_b':object_2, ...}
b_namespace = {'name_a':object_3, 'name_b':object_4, ...}
```

考虑以上情况, Namespace 可以理解为一组用于将name映射到object的dict, 同名的name在不同位置映射到object时需要遵循一定的准则.

Local: 局部变量, 很显然.

Enclosed: 定义在外部wrapper函数/类中而不在内部函数/类中的object. 

Global: 全局变量, 很显然.

Built-in: 内置变量, 如 len() 函数等 .

注意点:

1. 没有定义在嵌套的/同一个函数/类中的变量完全没有关系, 不属于local, enclosed中的任何一种.

2. 可以使用nonlocal, global等关键词将scope外部的object转入scope内, 反之亦然.