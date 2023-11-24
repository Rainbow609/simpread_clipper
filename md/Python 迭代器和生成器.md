> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/ybjourney/p/12078640.html)

Python 的迭代器集成在语言之中，迭代器和生成器是 Python 中很重要的用法，本文将**深入了解迭代器和生成器**。

首先，我们都知道 for 循环是一个基础迭代操作，大多数的容器对象都可以使用 for 循环，那么，我们从 **for 循环开始**：

你有没有想过，for 循环的内部实现原理呢？

其实，在 Python 中，for 循环是对迭代器进行迭代的语法糖，内部运行机理就是：首先底层对循环对象实现迭代器包装（调用容器对象的`__iter__`方法）返回一个迭代器对象，每循环一步，就调用一次迭代器对象的`__next__`方法，直到循环结束时，自动处理 StopIteration 这个异常。

对于像 list，dict 等容器对象而言，都可以使用 for 循环，但是它们并不是迭代器，它们属于可迭代对象。

**什么是可迭代对象呢？**

最简单的解释：实现了迭代方法可以被迭代的对象，可以使用 isinstance() 方法进行判断。

举个例子：

```
In [1]: from collections import Iterable, Iterator
In [2]: a = [1, 2, 3]
In [3]: isinstance(a, Iterable)
Out[3]: True
In [4]: b = a.__iter__()
In [5]: isinstance(b, Iterator)
Out[5]: True


```

可迭代对象实现了`__iter__`方法，该方法返回一个迭代器对象。

以上，可以看到，在迭代过程中，实际调用了迭代器的`__next__`方法进行迭代。

那么，**什么是迭代器？**

实现了迭代器协议的对象就是迭代器，所谓的迭代器协议可以简单归纳为：

1.  实现`__iter__()`方法，返回一个迭代器
2.  实现 next 方法，返回当前元素并指向下一个元素，如果当前位置已无元素，则抛出 StopIteration 异常 。

迭代器和可迭代对象的区别是：迭代器可以使用 next() 方法不断调用并返回下一个值，除了调用可迭代对象的`__iter__`方法来将可迭代对象转换为迭代器以外，还可以使用 iter() 方法。

举个例子来验证以上说法：

```
In [1]: iter_data = iter([1, 2, 3])
In [2]: print(next(iter_data))
1
In [3]: print(next(iter_data))
2
In [4]: print(next(iter_data))
3
In [5]: print(next(iter_data))
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-16-425d66e859b8> in <module>
----> 1 print(next(iter_data))


```

**为什么要用迭代器？**

很重要的一点是，Python 把迭代器内建在语言之中的，我们在遍历一个容器对象时并不需要去实现具体的遍历操作。

迭代器时一个惰性序列，仅仅在迭代至当前元素时才计算该元素的值，在此之前可以不存在，在此之后可以随时销毁，也就是说，在迭代过程中不是将所有元素一次性加载，这样便不需要考虑内存的问题。通过定义迭代器协议，我们可以随时实现一个迭代器。

**什么时候用迭代器？**  
具体在什么场景下可以使用迭代器：

*   数列的数据规模巨大
*   数列有规律，但是不能使用列表推导式描述。

举个最简单的例子：

```
class Fib(object):
    def __init__(self):
        self._a = 0
        self._b = 1

    def __iter__(self):
        return self

    def __next__(self):
        self._a, self._b = self._b, self._a + self._b
        return self._a


if __name__ == '__main__':
    for index, item in enumerate(Fib()):
        print(item)
        if index >= 9:
            break


```

**什么是生成器？**

生成器，顾名思义，就是按照一定的模式生成一个序列，是一种高级的迭代器，Python 中有一个专门的关键字（yield）来实现生成器。

如果一个函数，使用了 yield 语句，那么它就是一个生成器函数，当调用生成器函数函数时，它返回一个迭代器，不过这个迭代器时一个生成器对象。

举个例子：

```
from itertools import islice

def fib():
    a, b = 1, 1
    while True:
        yield a
        a, b = b, a + b

if __name__ == '__main__':
    fib_data = fib()
    print(list(islice(fib_data, 0, 10)))


```

可以看到，使用生成器后，代码简洁了很多！在上述代码中添加：

```
print(type(fib_data))
print(dir(fib_data))


```

可以看到函数返回的是一个 generator 对象，且对象实现了迭代器协议。

但是，使用生成器必须要注意的一点是：**生成器只能遍历一次**。

**什么时候用生成器呢？**

生成器可以使用更少的中间变量来写流式代码， 相比于其它容器对象占用的内存和 CPU 资源更少一些。当需要一个将返回一个序列或在循环中执行的函数时，就可以使用生成器，因为当这些元素被传递到另一个函数中进行后续处理时，一次返回一个元素可以有效的提升整体性能，最重要的是，比迭代器简洁！

除此以外，生成器还有两个很棒的用处：

1.  实现 with 语句的上下文管理器协议
2.  实现协程

**什么是生成器表达式？**

列表推导式，大家应该都用到，但是由于内存的限制，列表的容量是有限的，如果要创建一个有几百万个元素的列表，会占用很多的储存空间，当我们只需要访问几个元素时，其它元素占用的空间就白白浪费了。

这种时候你可以用生成器表达式啊，生成式表达式是一种实现生成器的便捷方式，将列表推导式的中括号替换为圆括号，生成器表达式是一种边循环边计算，使得列表的元素可以在循环过程中一个个的推算出来，不需要创建完整的列表，从而节省了大量的空间。

```
In [1]: a = (item for item in range(10))

In [2]: type(a)
Out[2]: generator

In [3]: next(a)
Out[3]: 0

In [4]: next(a)
Out[4]: 1


```

以上。

代码可参考：[my github](https://github.com/Yabea/Python/tree/master/learn_python/iterator)