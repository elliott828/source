NOTES

## 基本数据类型相关
### tuple的不可修改性
list可修改而tuple不可修改。tuple不可修改针对的是内存的指向，如：
``` python
L = ("a", "b", ["c", "d"])
```
第一个元素和第二个元素均不能修改，因为tuple的前两个元素直接指向了内存中的"a"和"b"。第三个元素是["c", "d"]这个list，不可修改仅针对他最基础的那层属性，即list，不可修改为其它属性的对象，而list内的元素是可以变更的。

### 判断一个key在dict中是否存在
方法1：`in`: 如`'a' in dict`, 如不存在，返回`False`
方法2: `.get()`: 如`dict.get('a')`，如不存在返回`None`。可指定返回值：`dict.get('a', -1)`，如不存在，则返回-1

## 函数相关
### 空函数
`pass`语句什么都不做，那有什么用？实际上`pass`可以用来作为占位符，比如现在还没想好怎么写函数的代码，就可以先放一个`pass`，让代码能运行起来。

### 函数返回多个值
如果函数内部定义了同时返回多个值，比如函数`test(lambda)`返回`x`, `y`, `z`，运行后分别在函数内部赋值为1, 2, 3。我们将结果保存：
```python
a, b, c = test(lambda)
print(a, b, c)
```
可以得到返回值为
```python
1, 2, 3
```
如果将函数仅赋值为一个变量：
```python
p = test(lambda)
print(p)
```
那么我们可以得到如下结果：
```python
(1, 2, 3)
```
返回值是一个tuple。
> 在语法上，返回一个tuple可以省略括号，而多个变量可以同时接收一个tuple，按位置赋给对应的值。所以，Python的函数返回多值其实就是返回一个tuple，但写起来更方便。

### 可变(长度)参数
当函数可处理的参数个数不确定时，如：
``` python
def calc(numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```
调用这个函数时，如果要计算多个值相加的结果，需要将这些值塞进一个list或者一个tuple，而不能直接写成`calc(1, 2, 3)`。想要实现后一种调用方式，则需要在参数前添加符号：`*`。如下:
``` python
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```
在函数内部，`*`的作用是让参数`numbers`接收到的是一个tuple，这样，调用这个函数时，可以传入任意数量(理论上来说，0到正无穷)的参数。比如有了一个list:
``` python
nums = [1, 2, 3]
```
我们并不用这样调用：
```python
calc(nums[0], nums[1], nums[2])
```
而是可以直接在`nums`前添加`*`号，再传入函数：
```python
calc(*nums)
```

### 关键字参数
``**``加参数名即为关键字参数，允许传入0个或任意个含参数名的参数，并在函数内部自动组成dict。
上面提到的可变参数——在函数内部自动组成tuple，在函数调用时，可以直接使用list和tuple。关键字参数的使用方法也一样，函数调用时，也可以以dict的形式设定参数。

函数示例：
``` python
def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)
```

可以这样调用：
``` python
person('Adam', 45, city='Beijing', job='Engineer')
```

和可变参数类似，也可以先组装出一个dict，然后，把该dict转换为关键字参数传进去：

``` python
extra = {'city': 'Beijing', 'job': 'Engineer'}
person('Jack', 24, city=extra['city'], job=extra['job'])
```
当然，上面复杂的调用可以用简化的写法：

``` python
extra = {'city': 'Beijing', 'job': 'Engineer'}
person('Jack', 24, **extra)
```
> `**extra`表示把extra这个dict的所有key-value用关键字参数传入到函数的``**kw``参数，kw将获得一个dict，注意`kw`获得的dict是`extra`的一份拷贝，对kw的改动不会影响到函数外的`extra`

### 命名关键字参数
可变参数和关键字参数的好处是参数的传入不受限制，但如果想控制，比如关键字参数传入了哪些，就必须在函数内部进行检查。
如果想限制关键字参数的名字而省掉关键字的检查步骤，则应考虑使用**命名关键字参数**。比如，只接受`city`和`job`做为关键字参数：

``` python
def person(name, age, *, city, job):
    print(name, age, city, job)
```

命名关键字参数需要一个特殊分隔符`*`，`*`后面的参数被视为命名关键字参数。如果函数定义中已经有了一个可变参数，则后面不再需要一个特殊分隔符`*`。
> 使用命名关键字参数时，要特别注意，如果没有可变参数，就必须加一个*作为特殊分隔符。如果缺少*，Python解释器将无法识别位置参数和命名关键字参数

``` python
def person(name, age, *, city, job):
    print(name, age, city, job)

# or

def person(name, age, *args, city, job):
    print(name, age, args, city, job)
```

要注意的是命名关键字参数必须传入参数名，如果没有传入参数名，python视所有参数为位置参数，进而报错。而上面定义的`person()`只有两个位置参数。

> 在Python中定义函数，可以用必选参数、默认参数、可变参数、关键字参数和命名关键字参数，这5种参数都可以组合使用。但是请注意，`**`参数定义的顺序必须是：必选参数、默认参数、可变参数、命名关键字参数和关键字参数`**`。

### 递归函数的尾递归优化
先贴一句让我撞墙的话：
> 在计算机中，函数调用是通过栈（stack）这种数据结构实现的，每当进入一个函数调用，栈就会加一层栈帧，每当函数返回，栈就会减一层栈帧。由于栈的大小不是无限的，所以，递归调用的次数过多，会导致栈溢出。

举的例子是阶乘函数：
``` python
def fact(n):
    if n==1:
        return 1
    return n * fact(n - 1)

print(fact(1))
```
    ### 1

``` python
print(fact(5))
```
    ### 120

``` python
print(fact(10))
```
    ### 3628800

然而当我们尝试`fact(1000)`的时候，函数会报错。于是有了这一小节开头的那一句话。解决方案是使用**尾递归优化**。

> 尾递归是指，在函数返回的时候，调用自身本身，并且，`return`语句不能包含表达式。这样，编译器或者解释器就可以把尾递归做优化，使递归本身无论调用多少次，都只占用一个栈帧，不会出现栈溢出的情况。

``` python
def fact(n):
    return fact_iter(n, 1)

def fact_iter(num, product):
    if num == 1:
        return product
    return fact_iter(num - 1, num * product)
```

# http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431756044276a15558a759ec43de8e30eb0ed169fb11000