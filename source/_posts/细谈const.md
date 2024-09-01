\---

title: 细谈const

categories: 杂谈

tags: C++

\---

# 细谈const



## 啥是const

{% notel blue Definition%

有时候我们希望定义这样一种变量，它的值不能被改变。例如，用一个变量来表示缓冲区的大小。使用变量的好处是当我们觉得缓冲区大小不再合适时，很容易对其进行调整。另一方面，也应随时警惕防止程序一不小心改变了这个值。为了满足这一要求，可以用关键字**const**对变量的类型加以限定。

​																														——*C++ primer*

{% endnotel %}