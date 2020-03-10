Python-Insight

object可以像函数一样被call，只要实现  ____call____方法

```python
class CallableObject(object):
    def __init__(self):
        pass

    #让对象可以像函数方法一样被调用
    #这也说明函数和对象在这一点上是相同的，即，对象是可调用的函数
    def __call__(self):
        print(CallableObject.__name__,' is called')
        
if __name__ == '__main__':
    CallableObject()()
    myfunc = CallableObject()
    myfunc()
```



