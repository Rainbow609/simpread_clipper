> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/ybjourney/p/12241697.html)

装饰器在 Python 中是一个强大的高级用法，并且在流行 Python 框架中变得越来越常见。经常会用到装饰器来增强函数的行为（动态的给一个对象添加一些额外的职责），包括记录日志，权限校验，性能测试，数据封装等。有了装饰器，我们可以抽离出大量和函数功能本身无关的雷同代码并继续重用。

Python 装饰器有两种：

1.  函数装饰器：管理函数调用和函数对象
2.  类装饰器：管理类实例和类自身

**为什么使用装饰器？**

经常会遇到给函数或类增加新功能的场景，当然我们可以使用函数调用或者其它技术来实现，但是使用装饰器意图明确，最小化扩展代码的冗余，使用 @语法糖，相对优雅。

**装饰器的原理是什么？**

我们先来看一个最简单的装饰器：

```
import time
from functools import wraps

def time_it(func):
    """
    输出函数的运行时间
    :param func:
    :return:
    """
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func()
        end_time = time.time()
        process_time = end_time - start_time
        print(func.__name__, process_time)
        return result
    return wrapper

@time_it
def func_a():
    time.sleep(2)


```

对上述代码进行解释：

1.  time_it 返回 wrapper 函数对象
2.  使用 time_it 装饰 func_a 函数
3.  调用被装饰的 func_a 函数会运行 wrapper 函数，func_a 其实是 wrapper 的引用

原理：我们知道 Python 中一切皆对象，可以将函数作为其它函数的返回值。可以看到，装饰器的本质是一个函数，返回一个函数对象，通过 "@" 语法糖在包装函数中引入装饰器。

装饰器的一个关键特性是，在被装饰的函数定义之后立即执行。

**@wraps**

上述装饰器中用到的了 @wraps(func)，在创建装饰器时，一定要记得为包装函数添加 functools 库中的 @wraps 装饰器，以保证函数的元数据（包括函数名，函数注解等）不被丢失。

当我们需要访问为被装饰器修饰的原包装函数时，可以使用 @wraps 的`__wrapped__`属性来访问。

**内置装饰器**

Python 有三个内置装饰器：@staticmathod、@classmethod 和 @property

*   @staticmethod：类的静态方法，跟成员方法的区别是没有 self 参数，并且可以在类不进行实例化的情况下调用。
*   @classmethod：跟成员方法的区别是接收的第一个参数不是 self，而是 cls（当前类的具体类型）
*   @property：表示可以直接通过类实例直接访问的信息。

**装饰器嵌套**

为了支持多步骤的扩展，装饰器语法允许我们向一个装饰的函数或方法添加多个装饰器，若多个装饰器同时装饰一个函数，那么装饰器的调用顺序和 @语法糖的声明顺序相反，也就是：

```
@decorator1
@decorator2
def func():
    pass


```

等效于：

`func = decorator1(decorator2(func()))`

**装饰器参数**

函数装饰器和类装饰器都能接收参数，这些参数传递给了真正返回装饰器的可调用对象，而装饰器反过来又返回一个可调用对象。

装饰器参数在装饰发生之前就解析了，并且它们通常用来保持状态信息供随后的调用使用。

上述实例中，func_a() 是没有参数的，那如果添加参数的话，装饰器该如何编写以接收参数呢？可以在装饰器中使用 * args 和 **kwargs 代替参数：

```
def time_it(func):
    """
    输出函数的运行时间
    :param func:
    :return:
    """
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        process_time = end_time - start_time
        print(func.__name__, process_time)
        return result
    return wrapper


```

**带参数的装饰器**

我们有时候需要提供给被装饰的函数特定的功能，需要在装饰器中带参数。比如在业务处理中我们需要限定函数的执行超时时间，由于每个函数所对应的超时时间不一样，所以需要在装饰器中带参数以实现。

装饰器的语法允许我们在调用时，提供其它参数，实现上述场景：

```
import time
import signal
import functools


def func_timeout(timeout):
    """
    超时时间装饰器
    :param timeout:
    :return:
    """
    def decorator(func):
        def handler(signum, frame):
            raise RuntimeError("run %s timeout !" % func.__name__)

        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            signal.signal(signal.SIGALRM, handler)
            signal.alarm(timeout)
            func(*args, **kwargs)
            signal.alarm(0)
        return wrapper
    return decorator


@func_timeout(timeout=10)
def func():
    time.sleep(11)
    print("#" * 100)


if __name__ == '__main__':
    func()


```

**类装饰器**

上述实例都是函数装饰器，相比函数装饰器，类装饰器更加灵活，主要依靠类的`__call__`方法，当使用 @形式将装饰器附加到函数上时，就会调用此方法。

举个例子：

```
class Foo(object):
    def __init__(self, func):
        self._func = func

    def __call__(self, *args, **kwargs):
        print("class decorator start")
        self._func(*args, **kwargs)
        print("class decorator end")


@Foo
def func():
    print("test123")

if __name__ == '__main__':
    func()


```

以上，代码见 [my github](https://github.com/Yabea/Python/tree/master/learn_python/decorator)。