---
title: 细谈const
categories: 杂谈
tags: C++
---
# 细谈const

{% notel blue Definition%}

​	有时候我们希望定义这样一种变量，它的值不能被改变。例如，用一个变量来表示缓冲区的大小。使用变量的好处是当我们觉得缓冲区大小不再合适时，很容易对其进行调整。另一方面，也应随时警惕防止程序一不小心改变了这个值。为了满足这一要求，可以用关键字**const**对变量的类型加以限定。

​																														——*C++ primer*

{% endnotel %}

​	const对象一旦创建后其值就不能再改变，所以const对象必须初始化。

```c++
const int i = 42;	// correct
const int k;		// error k是一个未经初始化的常量
```



## const的使用与作用域

```c++
const int buffSize = 512;
```

编译器将在编译过程中把用到该变量的地方都替换成对应的值。也就是说，编译器会找到代码中所有用到`buffSize`的地方，然后用512替换。

​	默认情况下，const对象被设定为仅在文件内有效。当多个文件中出现了同名的const变量时，其实等同于在不同文件中分别定义了独立的变量。

​	若要在不同文件中使用同一个const对象，需要使用extern关键字：

```c++
//constTest.cc
extern const int buffSize = 12;		// 声明并定义了变量buffSize

//constMain.cc
#include <iostream>
using namespace std;
extern const int buffSize;			// 该buffSize与constTest.cc中定义的buffSize是同一个
int main() {
  cout << buffSize << endl;  		// 12
  return 0;
}
```



## const的引用

​	可以把引用绑定到const对象上，就像绑定到其他对象上一样，我们称之为**对常量的引用**。

```c++
(1) const int ci = 1024;
(2) const int &r1 = ci;		//correct
(3) r1 = 2048;				//error!
(4) int &r2 = ci;			//error!
```

因为不允许直接为`ci`赋值，当然也就不能通过引用去改变`ci`、因此，对`r2`的初始化是错误的。假设该初始化是合法的，则可以通过`r2`来改变它所引用的对象的值，这显然是不正确的。

​	了解了const的相关定义，很容易理解上述代码(1) - (3)。对于(4)也是可以理解的，但是这里隐含一个“权限放大”的概念，值得再细细探讨一番。

### 关于权限

​	权限可以分为权限放大和权限缩小。对于一个文件，如果它只有读的权限，现在给它加一个写权限，这就是**权限放大**。若它读写权限都有，但此刻限制了它的写权限，这就是**权限缩小**。
​	在指针和引用赋值中，权限可以缩小，但不可以放大：这里通常指的是变量的值可不可以改的问题，如果说是从改到不能改，这就是将权限缩小了，这是可行的；如果说是从不能改到能改，这就是权限放大了，此时是不行的。

```c++
int a = 10;
const int& ra = a;  // 加了const，不可修改ra所引用的对象(a)，权限缩小，可行
int& raa = ra;  	// rra是ra的别名，然而ra是只读的，raa此时是可读可写的，这属于权限放大，不可行！
					//error: binding reference of type ‘int&’ to ‘const int’ discards qualifiers

const int b = 20;
int& rb = b;  		// 同上，这里权限放大，不可行！
const int& rrb = b;  // 这里rrb和b都是const int，此时权限平移，即权限不变，可行。

```

​	再进一步，类型转换中也可能存在权限放大问题。

```c++
double pi = 3.14;

int pa = pi;		// correct，隐式类型转换
int pb = (int)pi;	// correct，强制类型转换

double& ra = pi;	// correct，ra是pi的别名，且类型都为double
int& rb = pi;		// error(1)
int& rd = (int)pi;	// error(2)
const int& rc = pi;	// correct
```

​	在解释上面的error(1)、error(2)前，先引入一个关于类型转换的小知识点，**临时量对象(temporary)**。

#### 	临时量对象

​	所谓临时量对象就是当编译器需要一个空间来暂存表达式的求值结果时临时创建的一个未命名的对象。

```c++
double dval = 3.1415;
const int& ri = dval;
```

此处`ri`引用了一个int型的数。对`ri`的操作应该是整数运算，但`dval`却是一个双精度浮点数而非整数。因此为了确保`ri`绑定一个整数，编译器把上述代码变成了如下形式：

```c++
const int temp = dval;	// 由双精度浮点数生成一个临时的整形常量
const int& ri = temp;	// 让ri绑定这个变量
```

​	有了临时量对象的概念后，再来看看error(1)和error(2)。

​	首先，对于error(1)，它实际的形式是：

```c++
const int temp_pi = pi;
int& rb = temp_pi;		// error(1)
```

这显然是属于权限放大，因此不可行！同理，error(2)也是一个权限放大问题，它的实际形式和error(1)一样。

除了类型转换会有临时量，函数调用也有相关问题。

```c++
void myFunc1(int& num) {
    cout << num << endl;
}
void myFunc2(const int& num) {
    cout << num << endl;
}
int main() {
    double b = 1.23;
    myFunc1(b);		//error!,这里会产生临时量 double -> const int, 函数形参类型为int, 接收一个const int 属于权限放大
    myFunc2(b);		//correct, 这里会产生临时量 double -> const int, 函数形参类型为const int，属于权限平移，可行
}
```

#### 类与对象的权限放大

再复杂一点，在类中也存在很多权限放大问题！

{% notel green const 修饰 成员函数的规则%}

1. const对象不可以调用非const成员函数，这属于权限放大
2. 非const对象可以调用const成员函数，这属于权限缩小
3. const成员函数内不可以调用其它的非const成员函数，这属于权限放大
4. 非const成员函数内可以调用其它的const成员函数，这属于权限缩小

{% endnotel %}

```c++
class Cat {
public:
    Cat() {
        cout << "This cat build from Cat()" << endl;
    }
    Cat(Cat& c) {	// 注意这个参数类型
        cout << "This cat build from Cat(cat& c)" << endl;
    }
    ~Cat() {
        cout << "kill cat!!!" << endl;
    }
    void introduce1() {
        cout << "cat introduce1" << endl;
        introduce2();	// correct, 非const成员函数可以调用其它const成员函数。
    }
    void introduce2() const {
        cout << "cat introduce2" << endl;
        introduce1();	// error,const成员函数不可以调用其它非const成员函数！
    }
};

int main() {
    
    Cat& boy_cat = Cat();	// error，这个error似乎令你超乎想象
         					// error: cannot bind non-const lvalue reference of type ‘Cat&’ to an rvalue of type ‘Cat’
    const Cat& girl_cat = Cat();
    
    girl_cat.introduce1();	// const对象调用非const成员函数，权限放大 error
    						// error: passing ‘const Cat’ as ‘this’ argument discards qualifiers [-fpermissive]
    girl_cat.introduce2();	// const对象调用const成员函数，权限平移，可行
}
```

​	对于`Cat& boy_cat = Cat();`。`Cat()`产生的匿名对象是一个临时对象。C++标准不允许将非`const`引用绑定到临时对象上。这是因为临时对象可能会在引用的生命周期结束之前被销毁，从而导致引用悬垂（dangling reference），即引用指向了一个已经被销毁的对象。

​	再来看`const Cat& girl_cat = Cat();`。在这个例子中，`girl_cat` 是一个对临时 `Cat` 对象的 `const` 引用。由于它是 `const` 的，所以你不能通过这个引用来修改对象的状态，这避免了潜在的悬垂引用问题，因为即使临时对象被销毁了，你也无法通过这个引用来修改它。然而，你仍然可以通过这个引用来读取对象的状态，直到临时对象的生命周期结束。

{% note primary  %}匿名对象的生命周期只在那一行，那么之后`girl_cat`是否会变成野引用呢？即引用了一块已经不存在的空间？ {% endnote %}

​	其实当用const来引用一个匿名对象，匿名对象的声明周期会被延长，在本例中，这个匿名对象会随着程序结束才进行销毁。

## 常量折叠

{% notel blue Definition%}

​	常量折叠（Constant Folding）是编译器的一种优化技术，它通过在编译期间对常量表达式进行计算，将其结果替换为常量值，从而减少程序运行时的计算和开销。

{% endnotel %}

​	在编译器里进行语法分析的时候，将常量表达式计算求值，并用求得的值来替换表达式，**放入常量表**，可以算作一种编译优化，但是变量的名称是有效范围内还是可用的，并且在编译的时候从常量表中的直接替换，并不涉及到该变量的内存地址。

```c++
#define PI 3.14
int main(){
　　const int r = 10;
 
　　int p = pI; 		//这里会在预编译阶段产生宏替换，PI直接替换为3.14，其实就是int p = 3.14;
　　int len = 2*r; 	//这里会发生常量折叠，也就是对常量r的引用会替换成他对应的值，相当于int len = 2*10;
　　return 0;
}
```

​	折叠变量的效果跟宏定义很类似，但又有做不同，宏是字符常量，在预编译完宏替换完成后，**该宏名字会消失**，所有对宏的引用已经全部被替换为它所对应的值，编译器当然没有必要再维护这个符号。而常量折叠发生的情况是，对常量的引用全部替换为该常量值，但是，**常量名并不会消失**，编译器会把他**放入到符号表中**，同时，会为该变量分配空间，栈空间或者全局空间。

```c++
int main(){
　　const int i=0; 						// 定义常量i
　　int *j = (int *) &i; 					// 看到这里能对i进行取值，判断i必然有自己的内存空间
　　*j=1; 								// 对j指向的内存进行修改
    printf("%d %d %d %d \n", &i, j, i, *j);	 // 观察实验结果:0012ff7c	0012ff7c 0 1
　　return 0;
}
```

这段代码可以说明：
	1. `i`和`j`地址相同，指向同一块空间，`i`虽然是可折叠常量，但是，`i`确实有自己的空间

2. `i`和`j`指向同一块内存，但是`*j = 1`对内存进行修改后，按道理来说`i`也应该等于1，而实验结果中`i`实实在在的等于0，这是为什么呢，就是本文所说的内容，`i`是可折叠常量，在编译阶段对`i`的引用已经别替换为`i`的值了，也就是说`printf("%d %d %d %d \n", &i, j, i, *j);`被替换为`printf("%d %d %d %d \n", &i, j, 0 ,*j);`。