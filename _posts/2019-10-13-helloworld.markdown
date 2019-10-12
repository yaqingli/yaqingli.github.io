---
layout: post
title:  "hello world"
date:   2019-10-13 02:37:46 +0800
categories: hi
---

# Java





## 条件执行

### if, if/else, if/else if/else

Key: 条件，条件语句，代码块，大括号

根据条件执行部分代码块。

三元运算符： 条件判断?表达式1：表达式2

```java
//求x与y的最大值
int max = x>y?x:y
```

###Switch 语句

Key: swith, break,跳转，CPU跳转指令

一般情况下，一定不要忘了写break

问题：下面的代码输出是什么？， 该怎么改？

```java
char c = 'A';
	    switch(c) {
			case 'A':
				System.out.println('A');
			case 'B':
				System.out.println('B');
		}
```

答案：

输出的是A， B，因为A之后没有加break，因此，A代码块之后继续后续的代码块。因此，理论上都要加上break

``` java
char c = 'A';
	    switch(c) {
			case 'A':
				System.out.println('A');
        break;
			case 'B':
				System.out.println('B');
        break;
		}
```



#### switch实现原理

key：指令跳转，条件跳转，无条件跳转，跳转表

程序是指令序列，CPU有指令指示器，指向下一条需要执行的指令，CPU根据指示器的指示加载指令并且执行。

![image-20190904172516254](/Users/liyaqing/Library/Application Support/typora-user-images/image-20190904172516254.png)

转换成下图的，请问3行能去掉么？

![image-20190904172442609](/Users/liyaqing/Library/Application Support/typora-user-images/image-20190904172442609.png)



![image-20190904172804160](/Users/liyaqing/Library/Application Support/typora-user-images/image-20190904172804160.png)

问题：跳转表为什么会更高效？

答案：因为表的数据结构可以使用更高效的算法查找代码块地址。

问题：跳转表一般用什么数据类型实现？

答案：一般是数组。如果是连续的，则switch传入的本身就是index， 因此直接索引就好了（考虑初始偏移量）。如果switch里的值并不连续，那么排序，通过快排，找到该数据，然后返回其对应的代码块地址。

问题：String类型用于switch的值，怎么办？

答案：通过hash将string类型转换成int32类型的，因为这样会有冲突的，在冲突之后，需要进一步比较string本身。



## 循环

### 循环的4种形式

key: while, do/while, for, foreach

在while循环中，需要有中断退出的条件，但经常不知道什么时候会中断退出。

问题：while一般适用于哪些场景？

答案：一般适用于难以直接判断退出时机的场景。



问题：从屏幕输入中读取字符，直到读到abc为止退出

答案：

题目：从循环打印数字

```java
int[] a = {1, 2, 3};
for(int i =0; i<a.length; i++) {
  System.out.println(i);
}
```



题目：循环打印数组

方法1， 直接使用传统的for(;;)语法

```java
private static void test_for_1() {
  int[] a = {1, 2, 3};
  for(int i =0; i<a.length; i++) {
    System.out.println(a[i]);
  }
}
```



方法2，直接使用foreach语法，foreach语法依赖于for实现，没有专门的for关键字

```java
private static void test_foreach() {
		int[] a = {1, 2, 3};
		for (int i : a) {
			System.out.println(i);
		}
	}
```

比较python的for, python其实没有for(;;)语法，只有foreach语法，这里有冒号

```python
a = [1,2,3]
for i in a:
  print(a)
```

问题：什么样的数据类型支持foreach语法？

答案：数组？iter类型？



### 循环控制

#### break



#### continue



小结：
解决复杂问题的基本策略都是分治。



## 函数：

什么是函数？

问题：函数如何定义？

名称， 参数名称，修饰符（访问限制符），返回类型![image-20190904182538488](/Users/liyaqing/Library/Application Support/typora-user-images/image-20190904182538488.png)



传参：

1. 数组

注意区分内容和指针本身。尤其是传入容器类，内容可以被改变。

2. 可变长度的参数

问题：什么是函数重载(overload)

答案：同一类中，同名，参数不全相同的函



问题：为什么需要重载，举例说明。

答案：同义，但是不同参数个数或者类型

```java
//例子
public static double max(double a, double b)
public static int max(int a, int b)
public static float max(float a, float b)
public static long max(long a, long b)
```



调用的匹配过程

寻找最佳匹配签名

Java编译器实际上

问题：什么是递归函数？

问题：递归调用的方式和性能问题，描述递归调用的内部机制。

描述函数调用的过程。





## 二进制

数字的二进制表示方法

什么是补码表示法，什么是原码表示法？