> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/ybjourney/p/8859519.html)

> Python2.5 之后引入了上下文管理器（context manager），算是 Python 的黑魔法之一，它用于规定某个对象的使用范围。

[Python 中的上下文管理器和 with 语句](https://www.cnblogs.com/ybjourney/p/8859519.html)
============================================================================

本文是针对于 Python 中上下文管理器和 with 语句的思考总结。

> Python2.5 之后引入了上下文管理器（context manager），算是 Python 的黑魔法之一，它用于规定某个对象的使用范围。

**什么是上下文管理器？**

在正常情况下，管理各种系统资源（如文件）、数据库连接时，通常是先打开这些资源，执行完相应的业务逻辑，最后关闭资源。  
举两个简单的例子:

1.  使用 Python 打开一个文件写入内容，之后需要关闭这个文件。如果不正常关闭的话可能会在文件操作时出现异常，因为系统允许你打开的文件的最大数是有限的。
2.  在数据库连接时也是存在类似问题，数据库的连接算是一种比较昂贵的资源，若连接过多而没有及时关闭的话，就可能出现不能继续连接的异常错误。

但是，很多程序员经常会忘记关闭文件，或者关闭数据库的连接。这时候就引入了**上下文管理器**，它可以在你不需要该对象的时候，自动关闭它。

**上下文管理器怎么使用？**

上下文管理器的语法是：with...as...，可以理解为：上下文管理器对象存在的目的是管理 with 语句，就像迭代器的存在是为了管理 for 语句一样。

举个文件操作的实例：

```
print "不使用上下文管理器"
print "*" * 30
f = open('file.py', 'w')
print f.closed
f.write("# Hello World")
f.close()
print f.closed

print "\n使用上下文管理器"
print "*" * 30
with open("file.py", 'w') as f:
    print f.closed
    f.write('# Hello Python')
print f.closed


```

这里通过. closed 比较，我们可以看到上下文管理器可以自动关闭文件，对于上下文管理器而言，有隶属于它的程序块，当隶属于它的程序块执行结束的时候（判断缩进），上下文管理器将自动关闭文件。  
上述实例，也可以使用 try...except... 来实现，但使用 with 使得代码更加的简洁，with 语句的一个目的也是简化了 try/finally 模式。

**上下文管理具体是怎么实现的呢？**

因为文件对象是 Python 的内置对象，内置了上下文管理的特殊方法，所以它可以使用 with 语句。在 Python 中，任何对象，只要实现了上下文管理，就可以使用 with 语句，实现上下文管理需要通过`__enter__`和`__exit__`这两个方法来实现：

*   `__enter__`(self)：with 语句开始运行时，会在上下文管理器对象上调用该方法。
*   `__exit__`(self, type, value, tb)：离开上下文管理器时调用该方法，以此扮演 finally 子句的角色。如果有异常出现，返回 False，type、value 和 tb 将分别表示异常的类型、值和追踪信息，传递出上下文显示；如果没有异常，则三个变量的值均为 None。

```
with 上下文管理器：
    语法体


```

当 with 语句遇到上下文管理器时，就会在执行语法体之前，先执行`__enter__`方法，然后再执行语法体，执行完语法体之后，执行`__exit__`方法。

**手动实现一个上下文管理器**

```
class Context(object):

    def __init__(self):
        print("init")

    def __enter__(self):
        print("enter")

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("exit")

temp = Context()

with temp:
    print("running")


```

可以很清楚的看到`__enter__`方法和`__exit__`方法的调用过程。

除了手动定义以外，Python 还提供了 contextLib 库，通过 contextmanager 装饰器 + yield，将函数分隔为两部分：yield 之前的语句在`__enter__`中执行，yield 之后的语句在`__exit__`中执行，简化了上下文管理器的实现方式：

```
from contextlib import contextmanager

@contextmanager
def context_test():

    print("enter")
    yield 
    print("exit")

with context_test():
    print("running")


```

> **总结：**通过上下文管理器，我们可以更好的控制对象在不同区间的特性，并且可以使用 with 语句替代 try...except 方法，使得代码更加的简洁，主要的使用场景是访问资源，可以保证不管过程中是否发生错误或者异常都会执行相应的清理操作，释放出访问的资源。

**本文版权归作者和博客园共有，欢迎转载，但未经作者同意转载需注明出处，且在文章页面明显位置给出原文链接。**