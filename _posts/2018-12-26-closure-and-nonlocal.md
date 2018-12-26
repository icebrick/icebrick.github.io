---
title: python闭包和nonlocal的小应用
key: 20181226
tags: python 闭包
---

在阅读tornado源码时发现一段有意思的关于闭包和局部变量的用法的小技巧，理解之后可以加深自己对python这个语言特性的理解
<!--more-->

最近在看tornado源码的时候，发现了一段挺有意思的代码：
```python
def run_sync(self, func, timeout=None):
    
    future_cell = [None]
    
    def run():
        try:
            result = func()
        except Exception:
            future_cell[0] = TracebackFuture()
            future_cell[0].set_exc_info(sys.exc_info())
        else:
            if is_future(result):
                future_cell[0] = result
            else:
                future_cell[0] = TracebackFuture()
                future_cell[0].set_result(result)
        self.add_future(future_cell[0], lambda future: self.stop())
    self.add_callback(run)
    if timeout is not None:
        timeout_handle = self.add_timeout(self.time() + timeout, self.stop)
    self.start()
    if timeout is not None:
        self.remove_timeout(timeout_handle)
    if not future_cell[0].done():
        raise TimeoutError('Operation timed out after %s seconds' % timeout)
    return future_cell[0].result()
```

这里不需要管这段代码的逻辑和功能，从他的写法里我们有没有看到比较奇怪的地方，这个函数里定义了一个列表变量`future_cell`，但是整个函数都只是用到了它的第一个元素`future_cell`，那为什么要把它定义为一个列表而不直接定义成`future_cell = None`呢？

这里其实涉及到`闭包`以及`局部变量`作用域的概念，这是`python`语言的一个特性。下面详细介绍。

首先解释下闭包，闭包是指延伸了作用域的函数，其中包含在函数体中引用，但是不在函数体中定义的非全局变量。在函数`run_sync`中定义了一个内部函数`run`。这个内部函数`run`会在`run_sync`函数调用完成后某个时刻才会执行，这里会就形成一个闭包，即`run`函数可以使用在自己函数体外面定义的`future_cell`这个变量。直觉上`future_cell`变量作为函数`run_sync`函数的局部变量，在函数`run_sync`执行结束退出后，变量`future_cell`应该不复存在了。但是因为闭包的机制，在函数`run_sync`退出后，在另外某个地方运行`run`函数时，`run`函数还是能取得`future_cell`这个变量的值，该变量是一个`自由变量`。

上面这个机制可能看过python闭包概念的同学都了解，但是这解释不了为什么需要把`future_cell` 这个变量定义为一个列表而不是普通变量。

这里涉及另外一个概念，即python变量在赋值时定义，当我们在`run`函数里为`future_cell`赋值时，其实相当于重新定义了一个只在`run`函数中可见的局部变量，这个变量和`run`函数外面定义的同名变量已经完全不是同一个了。对这种方式产生的`run`内部的这个`future_cell`对`run`函数外部是不可见的，这肯定不能满足这段代码的逻辑需求。

我们要怎么解决这个问题呢，其实很简单，就像代码里做的那样，将`future_cell`定义成一个列表（也可以定义为一个`dict`），在`run`函数中对这个列表的第一个元素赋值，这样就再是对`future_cell`这个变量重新复制，而只是更新其中的元素值，`future_cell`也不会变成`run`函数的局部变量，这样就能获得我们想要的那种效果。

当然，如果我们在内部函数中只是使用`future_cell`变量而不给它赋值（比如下面这个常见的装饰器的写法），则不需要用到这里介绍的技巧。

```python
def timer(func):
    """简单的记录函数运行时间的装饰器"""
    def inner(*args, **kwargs):
        start = time.time()
        res = func(*args, **kwargs)
        print time.time() - start
        return res
    return inner
```



最后说明下，只有在`python2`中需要用这种曲线救国的方法，`python3`中新增加了一个关键词`nonlocal`，使用这个关键词，我们可以强制将某个变量声明为`自由变量`，以达到同样的效果。如果使用nonlocal，则这段代码可以改成这样：

```python
def run_sync(self, func, timeout=None):
    
    future_cell = None
    
    def run():
        nonlocal future_cell
        try:
            result = func()
        except Exception:
            future_cell = TracebackFuture()
            future_cell.set_exc_info(sys.exc_info())
        # 剩下的代码类似
```


