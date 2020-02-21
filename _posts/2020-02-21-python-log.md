# Pyhton log

在运行程序时，经常需要记录程序运行日志。一种方法是使用logging类

## 使用场景
开发者时常需要记录程序性能，记录程序运行状态，便于调试。需要通过一定方式记录一些日志。一种方式是自己通过文件的方式写出，这样经常导致效率低，多线程时会比较复杂。为了不重复造轮子，可以使用成熟模块进行。
## 简介
可以使用logging模块。
从使用角度看，该模块的核心概念是handler，和logging模块的getLogger()工厂方法。handler有多种，例如循环记录等。


## 如何使用
```python
import logging
from logging.handlers import RotatingFileHandler
formatter = logging.Formatter('%(name)-12s %(asctime)-12s [line:%(lineno)d] %(levelname)-8s %(message)s', '%Y-%m-%d %H:%M:%S',) #产生一个Formatter对象，该对象负责将相应的信息转换成固定格式。
#例如
#name:
#asctime:
#lineno
#levelname
#message
file_handler = RotatingFileHandler('/user/xxx/run.log', maxBytes=10*1024*1024, backupCount=5)#生成一个FileHandler对象, 该对象用于写数据
file_handler.setFormatter(formatter)#设置formatter， 一个项目中，日志的格式理论上是一样的，因此可能没有必要使用多种格式

logger_shell_command = logging.getLogger("ShellCommand")#生成一个Logger对象，在一个项目中，可能有多个logger，但是handdler可能共用（写到一个文件）
logger_shell_command.addHandler(file_handler)
logger_shell_command.setLevel(logging.DEBUG)#设置不同的级别，只有该级别及以上的日志才会写出



```