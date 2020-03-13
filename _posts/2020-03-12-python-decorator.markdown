# Python-Decorator

Decorator可以实现逻辑分离。对于某些场景，函数主要用于实现业务逻辑，但是如果业务逻辑中包含大量的其他逻辑，则，测试困难，逻辑不清晰。

下面run1， run2, 和run3实现业务逻辑，但是在调用时，需要进行另一部分任务-即CallableObject1。

本来在run1中的部分任务delegate给了CallableObject1

注意：

1.被执行的逻辑本身需要执行，除非decorator的一部分目的就是阻止执行业务逻辑

2.被执行的函数返回什么，被修饰后也要返回什么，不能随意修改返回值。除非业务需求。

## 抽象1

```python
class CallableObject1(object):
    def __init__(self, func):
        self.func = func

    #让对象可以像函数方法一样被调用
    #这也说明函数和对象在这一点上是相同的，即，对象是可调用的函数
    def __call__(self):
        print(self.func.__name__,' is called')
        self.func()


@CallableObject1
def run1():
    print('run1')

@CallableObject1
def run2():
    print('run2')

@CallableObject1
def run3():
    print('run3')


if __name__ == '__main__':
    run1()
    run2()
    run3()
#run1  is called
#run1
#run2  is called
#run2
#run3  is called
#run3

```



## Case1， 计时

### 虽然计时，但忘了返回值

下面实现了time，但丢弃了自己的值

```python
import time
class TimeIt(object):
    def __init__(self, func):
        self.func = func

    #用于计时
    def __call__(self):
        start = time.time()
        self.func() #由于没有返回原本被代理的函数的值，因此调用原本函数的返回值为None
        end = time.time()
        print(end-start)


@TimeIt
def run1():
    time.sleep(1)
    
    print('run1')

@TimeIt
def run2():
    time.sleep(2)
    print('run2')

@TimeIt
def run3():
    time.sleep(3)
    print('run3')


if __name__ == '__main__':
    print(run1())
    print(run2())
    print(run3())


#run1
#1.0007882118225098
#None
#run2
#2.0005271434783936
#None
#run3
#3.000214099884033
#None


```

### 修正

```python
import time
class TimeIt(object):
    def __init__(self, func):
        self.func = func

    #用于计时
    def __call__(self):
        start = time.time()
        result = self.func()
        end = time.time()
        print(end-start)
        return result


@TimeIt
def run1():
    time.sleep(1)
    print('run1')
    return 1

@TimeIt
def run2():
    time.sleep(2)
    print('run2')
    return 2

@TimeIt
def run3():
    time.sleep(3)
    print('run3')
    return 3


if __name__ == '__main__':
    print(run1())
    print(run2())
    print(run3())
   
#run1
#1.003661870956421
#1
#run2
#2.001512050628662
#2
#run3
#3.004601240158081
#3

```



## Case2，被wrap的方法/函数有参数

当被代理的方法本身有参数时，那，方法名，参数都是该传入代理函数的参数。

**注意**, 作为代理对象的TimeIt的初始化参数第一个是被代理函数对象名，第二个是本身的参数， 并不是被代理对象的参数。

```python
import time
class TimeIt(object):
    def __init__(self, func):
        self.func = func

    #用于计时
    def __call__(self, *args):#注意，被代理函数的传入从这传入
        start = time.time()
        result = self.func(*args)
        end = time.time()
        print(end-start)
        return result


@TimeIt
def run1(seconds):
    time.sleep(seconds)
    print('run1')
    return seconds



if __name__ == '__main__':
    print(run1(1))

```

如果要增加named param, 需要增加**args, 不然会报：

TypeError: __call__() got an unexpected keyword argument 

```python
import time
class TimeIt(object):
    def __init__(self, func):
        self.func = func

    #用于计时
    def __call__(self, *args, **kwargs):#**kwargs要加进去
        start = time.time()
        result = self.func(*args, **kwargs)#**kwargs要加进去
        end = time.time()
        print(end-start)
        return result


@TimeIt
def run1(seconds):
    time.sleep(seconds)
    print('run1')
    return seconds



if __name__ == '__main__':
    print(run1(seconds=1))


```



## Case3, delegate函数有参数

 #### 方法1， 定义方法

```python
import time
def TimeIt(argument):
    def real_TimeIt(function):
        def wrapper(*args, **kwargs):
            print('1')
            print(argument)
            result = function(*args, **kwargs)
            print('2')
            return result
        return wrapper
    return real_TimeIt


@TimeIt('name')
def run1(seconds):
    time.sleep(seconds)
    return seconds



if __name__ == '__main__':
    print(run1(1))
```



示例

```python
import time
def logit(log_name, message):
    def real_logit(function):
        def wrapper(*args, **kwargs):
            d = {'a':'abc', 'b':'bcd'}
            print(d[log_name], message)
            result = function(*args, **kwargs)
            return result
        return wrapper
    return real_logit


@logit(log_name='b', message='download')
def run1(seconds):
    time.sleep(seconds)
    return seconds



if __name__ == '__main__':
    print(run1(1))
```



#### 方法2，定义对象

```python
import time

class TimeIt(object):
    def __init__(self, name):
        self.name = name

    def __call__(self, func):
        class RealTimeIt(object):
            def __init__(self, func):
                self.func = func

            #用于计时
            def __call__(self, *args, **kwargs):
                start = time.time()
                result = self.func(*args, **kwargs)
                end = time.time()
                print(end-start)
                return result
        return RealTimeIt(func)

@TimeIt(name = 'stage1')
def run1(seconds):
    time.sleep(seconds)
    print('run1')
    return seconds



if __name__ == '__main__':
    print(run1(seconds=1))


```







## 获取代理的信息

在很多场景下，内部函数需要获取当前代理的信息，例如，要求用户登陆，但是登陆后，函数内部要能获取用户信息等。

场景：任务

假设你需要执行一系列的方法，方法本身需要知道当前任务的一些信息。







## Refs:

https://www.artima.com/weblogs/viewpost.jsp?thread=240808#function-decorators





