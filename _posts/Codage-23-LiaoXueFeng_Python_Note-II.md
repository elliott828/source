title: 廖雪峰python教程学习笔记2
date: 2016-05-23 15:20:28
categories: Codage|编程
tags: [Python, 廖雪峰]
---

## 函数式编程
### 高阶函数
> 把函数作为参数传入，这样的函数称为高阶函数，函数式编程就是指这种高度抽象的编程范式。

#### map/reduce
`map()`接受俩参数，一个是函数名，一个是`Iterable`对象，最后的output是一个`Iterator`

例：
``` python
def f(x):
    return x * x
r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
list(r)
```

	## [1, 4, 9, 16, 25, 36, 49, 64, 81]

`map()`类似于R语言中的`apply()`族函数，将传入的函数依次作用到序列的每个元素，并把结果合并返回。

`reduce()`同样接受两个参数，把前一个元素的运算结果和下一个元素做累积计算

``` python
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)
```



#### filter




#### sorted 