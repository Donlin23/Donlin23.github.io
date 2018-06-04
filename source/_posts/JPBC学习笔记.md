---
title: JPBC学习笔记
date: 2018-06-04 10:54:45
tags: 密码学
categories: 信息安全
---

最近这段时间，读了2004年的一篇公钥密码体制下可搜索加密（PEKS）的论文，论文提到一种使用双线性对的方案实现关键字的可搜索加密。这里用到了双线性映射的计算方法，接下来就是对 JPBC 库进行了学习以及实验。在看论文的过程中总是对双线性对的部分内容模棱两可，但是使用了 JPBC 库进行一些实验的时候就轻而易举地理解了很多未解的问题了。不熟悉这部分内容我建议先阅读一下《深入理解密码学》一书，有关公钥密码的群论部分知识，浅显易懂，老少咸宜，居家旅行必备良书，比我上本科时候的什么《应用密码学》之类的好太多了。下面给出我学习的那一篇公钥密码体制下的可搜索加密方案论文以及 JPBC 库的主页。

[论文传送门->](https://crypto.stanford.edu/~dabo/papers/encsearch.pdf) <br>
[JPBC主页](http://gas.dia.unisa.it/projects/jpbc/index.html#.WYHRI4iGPcs) <br>

JPBC 是 Pairing-Based Cryptography Library (PBC)的一个 Java 语言实现（ PBC 是 C++ 写的），使用一个装饰器的设计模式对双线性对计算做了一个代理，提供了双线性对下的各类数学计算，并集成了几种经典的能够构造双线性对的具体函数，这部分在 JPBC 主页提到的几篇论文有提到。主页还提供了不同的 .properties 文件代表不同的 e(g1,g2) 映射函数，这样非常方便初学者在进行双线性对的计算时，对其计算过程有一个直观的理解。

### 初始化
JPBC 提供了一个工厂模式的类方法去读取作者写好的几个双线性对函数的参数以初始化一个 Pairing 对象，同时，也提供了专门生成双线性对函数参数的 PairingParametersGenerator 类以及表示函数参数的 PairingParameters 类。

```
import it.unisa.dia.gas.plaf.jpbc.pairing.PairingFactory;

//params.properties是双线性对函数的参数文件
Pairing pairing = PairingFactory.getPairing("params.properties");
```

```
import it.unisa.dia.gas.jpbc.PairingParametersGenerator;
import it.unisa.dia.gas.jpbc.PairingParameters;

// 初始化参数的生成器，
ParametersGenerator parametersGenerator = ...

// 调用这个生成器生成一组双线性对的参数
PairingParameters params = parametersGenerator.generate();
```

### 获取一个群域
群域是群论计算中的一个大范围，简单来说就是一组正整数和这组正整数的算术运算。在 JPBC 中，群域使用 Field 类来表示，通过生成好的 Pairing 对象中获取相对应的群域。每一个 Pairing 对象实例化后都会相应的生成相应的几个特定的群域。通过下面的方法获取。

```
// Return Zr ，返回一个素数域
Field Zr = pairing.getZr();

// Return G1 ，返回一个操作为加法的G1域
Field G1 = pairing.getG1();

// Return G2 ，返回一个操作为加法的G2域
Field G2 = pairing.getG2();

// Return GT ，返回一个操作为乘法的GT域
Field GT = pairing.getGT();
```

### 群域中元素的计算
在 JPBC 库中，使用了 Element 类表示一个群域中的元素。获取元素可以使用以下的方法：
```
// 初始化一个未赋值的元素  
Element element = field.newElement();

// 初始化一个赋值为0的元素
Element element = field.newZeroElement();

// 初始化一个赋值为1的元素
Element element = field.newOneElement();

// 初始化一个赋值为随机数的元组（该随机数一定是在群域 field 中）
Element element = field.newRandomElement();

// 初始化一个赋值为5的元素
Element element = field.newElement(5);

// 初始化一个赋值为大整数的元素
Element element = field.newElement(new BigInteger("5390843849083028490328"));

// set方法可以给一个元素赋值
element.set(5);
```
这些使用群域对象 Field 初始化的元素，需要注意的是，一个元素必定是来自于某个群域，而且两个元素必须是来自同一个 Pairing 生成的群域才能进行算术运算。例如元素 a 是通过群域 field1 生成的`a = field1.newRandomElement()`，而元素 b 是通过群域 field2 生成的`b = field2.newRandomElement()`，这两个元素在进行群操作时会出现错误，因为 a 和 b 不是同一个 Pairing 生成的群域的元素。以下是几个简单的在群上的算术运算。
```
// 执行e + a mod field操作
e.add(a);

// 执行e - a mod field操作
e.sub(a);

// 执行e * a mod field操作
e.mul(a);

// 执行e / a mod field操作
e.div(a);
```

### Other
我在进行 JPBC 的学习时进行了一个使用的小实验，实现双线性对的加密和解密，参考的实验代码：[Github - JPBC ](https://github.com/Donlin23/JPBC_Experience)

参考资料：

[JPBC主页](http://gas.dia.unisa.it/projects/jpbc/index.html#.WYHRI4iGPcs) <br>
[CSDN-JPBC配置与测试](http://blog.csdn.net/liuweiran900217/article/details/23414629) <br>
[CSDN-Java实现双线性对方案](http://blog.csdn.net/liuweiran900217/article/details/45080653) <br>
