---
layout: post
title: Python Import Mechenism
tags: [Python]
---

一点关于 Python 中 Import 机制的理解

## Python Import Mechenism

1. python中的import有relative import(相对引用) 和 absolute import(绝对引用)两种, 两种引用的具体方式不赘述.

2. 绝对引用的路径(就是你运行python3 \<scriptname> 的那个位置)是相对于顶层作绝对引用, 每个内部package的绝对引用的路径也是相对于顶层来写的.

3. 在内部package进行相对引用并不是一种好的习惯, 相对路径没有绝对路径清晰.

4. 在内部运行package也不是一种好的习惯 具体可以参看[这篇文章](https://stackoverflow.com/questions/18496183/python-no-module-named-error-when-calling-a-submodule)

5. 基于3) 4) 两点我决定在引用时统一选取绝对引用

6. 如果要运行内部包的模块, 我们在顶层使用 "python3 -m XX.XX.XX"命令来运行脚本即可.
