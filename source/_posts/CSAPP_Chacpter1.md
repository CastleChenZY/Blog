---

title: 第一章 计算机系统漫游
categories: [书籍阅读, CSAPP, 第一章 计算机系统漫游]
tags: 计算机基础

---

# CSAPP

Our story will begin with "Hello, World."

```c
// hello.c
#include <stdio.h>
int main() 
{
    printf("hello, world\n");
    return 0;
}
```

hello 程序的生命周期是从一个**高级** C 语言程序开始的。然而，为了在系统上运行 `hello.c` 程序，每条 C 语句都必须被<u>其他程序</u>转化为一系列的**低级**机器语言指令。然后这些指令按照一种称为可执行目标程序的格式打好包，并以 **二进制**磁盘文件的形式存放起来。目标程序也称为<u>可执行目标文件</u>。

