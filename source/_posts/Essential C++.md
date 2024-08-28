---
title: Essential C++
categories:书籍阅读
tags:c++
---
# Essential C++

## 第1章 C++编程基础 Basic C++ Programming

```c++
int elem_seq[] = {1,2,3};	//编译器会自行算出此array包含了3个元素
```



## 第2章 面向过程的编程风格 Procedural Programming

### 2.1 如何编写函数 How to Write a Function

​	每一个函数必须定义四个部分：**返回类型、函数名、参数列表、函数体**。

​	函数必须先被声明，然后才可以被调用。函数声明不必提供函数体，但必须指明返回类型、函数名，以及参数列表。此即所谓的函数原型。

### 2.2 调用函数 Invoking a Function

#### Pass by Reference 语义

​	参数传递方式：传值(by value)、传址(by reference)

```C++
/*传值*/
void Function1(int parameter1, int parameter2);
//调用函数
Function1(a,b);
//parameter1 = a, parameter2 = b,仅进行赋值操作，Function1()执行完毕后，不会改变a、b的值。

/*传址*/
void Function2(int &parameter3, int &parameter4);
//调用函数
Function2(c,d);
//&parameter3 = c, &parameter4 = d, 此处parateter3与parameter4分别代表c和d两个对象，相当于为c和d取了别名parameter3和parameter4，对它们进行操作就相当于对c和d进行操作。
//例：
int ival = 1024;
int *pi = &ival;
int &rval = ival;

int jval = 4096;
rval = jval;	//此时，ival == jval == 4096
pi = &rval;	//其实就是将ival的地址赋给pi

```

​	当我们以by reference方式将对象作为函数参数传入时，对象本身并不会复制出另一份——==复制的是对象的地址==。函数中对该对象进行的任何操作，都相当于是对传入的对象进行间接操作。

​	将参数声明为reference，一是希望得以直接对所传入的对象进行修改，二是降低复制大型对象的额外负担。

```c++
//pointer参数和reference参数
int Function1(int *value1)
{
    if(!value)
    {
        cout << "this pointer is 0!";
    }
    else
    {
    	//do anything...    
    }
}

int Function2(int &value2)
{
    do anything...
}

//调用函数
Function1(&a);
Function2(b);

//pointer可能（也可能不）指向某个实际对象，所以当我们提领pointer时，一定要先确定其值不为0。至于reference，则必定会代表某个对象。
```



#### 作用域及范围		

​	对象在程序内的存活区域称为该对象的scope（作用域）。

​	对象如果在函数以外声明，即有所谓的file scope。对象如果拥有file scope，从其声明点至文件末尾都是可见的。file scope内的对象也具备所谓的static extent，意即==该对象的内存在main()开始执行之前便已分配好==，可以一直存在至程序结束。

​	内置类型的对象如果定义在file scope之内，**必定被初始化为0**。但如果被定义于local scope之内，那么除非程序员指定其初值，否则不会被初始化。



#### 动态内存管理

​	dynamic extent（动态范围）。其内存系由程序的空闲空间分配而来，也成为heap memory（堆内存）。其分配系通过new表达式来完成。而其释放则通过delete表达式完成。

```c++
//new表达式的形式如下：
    new Type;	
/*此处的Type可为任意内置类型，也可以是程序知道的class类型。new表达式亦可为：*/
	new Type(initial_value);
//例如：
    int *pi;
	pi = new int;	
/*先由heap分配出一个类型为int的对象，再将其地址赋值给pi。默认情况下，由heap分配而来的对象，皆未经初始化。*/

//new表达式的另一种形式可以赋初值,例如：
	pi = new int(1024);
/*先由heap分配出一个类型为int的对象，再将其地址赋给pi，但这个对象的值会被初始化为1024*/

//可以从heap中分配数组：
	int *pia = new int[24];
/*先由heap分配一个拥有24个整数的数组，pia会被初始化为数组第一个元素的地址*/


//delete表达式形式如下：
	delete pi;
	delete [] pia;
```

​	若不使用delete表达式，由heap分配而来的对象就永不被释放！

### 2.3 提供默认参数值 Providing Default Parameter Values

​	以“参数传递”作为函数间的沟通方式，比“直接将对象定义域file scope”更适当。函数如果过度依赖定义于file scope内的对象，就比较难以在其他环境中重用，也比较难以修改。

​	关于默认参数值的提供，有两个不很直观的规则：第一个规则是，默认值的解析（resolve）操作由最右边开始进行。如果我们为某个参数提供了默认值，那么这一参数右侧的所有参数都必须也具有默认参数值才行。

```c++
//错误：没有为vec提供默认值
void display(ostream &os = cout, const vector<int> &vec);
```

​	第二个规则是，默认值只能够指定一次，可以在函数声明处，亦可以在函数定义处，但不能够在两个地方都指定。

### 2.4 使用局部静态对象 Using Local Static Objects

​	对于函数Fibonacci()，每次调用时，便会计算出用户指定的元素个数并保存在vector中。但若对Fibonacci()进行三次调用：

```c++
Fibonacci(24);
Fibonacci(8);
Fibonacci(18);
```

​	此时，第一次调用就已经计算出第二次、第三次调用所需要计算的值。若第四次调用Fibonacci(32)，如何只进行计算#25-#32这些元素值？

```c++
const vector<int>* Fibonacci(int size)
{
    static vector<int> elems;	//elems被定义为Fibonacci()中的局部静态对象
    //函数的工作逻辑放在此处
    
    return &elems;
}
```

​	与局部非静态对象不同的是，局部静态对象所处的内存空间，即使在不同的函数调用过程中，依然持续存在。elems的内容不再像以前一样地在Fibonacci()每次被调用时就被破坏又被重新建立。这也就是现在我们可以安全地将elems的地址返回的原因。

### 2.5 声明inline函数 Declaring a Function Inline

​	一般而言，最适合声明为inline的函数具有体积小，常被调用，所从事的计算并不复杂的特点。

​	使用inline避免了频繁调用函数对栈内存重复开辟所带来的消耗。

### 2.6 提供重载函数 Providing Overloaded Functions

​	编译器无法根据函数返回类型来区分两个具有相同名称的函数。

```c++
//错误：参数列表(而非返回类型)必须不同
ostream& display_mesage(char ch);
bool display_message(char ch);
```

### 2.7 定义并使用模板函数 Defining and Using Template Functions

```c++
template <typename elemType>
void display_message(const string &msg, const vector<elemType> &vec)
{
    cout << msg;
    for(int ix = 0; ix < vec.size(); ++ix)
    {
        elemType t = vec[ix];
        cout << t << '';
    }
}

//使用
vector<int> ivec;
string msg;
//...
display_message(msg,ivec);
```

### 2.8 函数指针带来更大的弹性 Pointers to Functions Add Flexibility

```c++
const vector<int>* (*seq_ptr)(int);
/*seq_ptr可以指向“具有所列返回类型及参数列表”的任何一个函数，seq_ptr是一个指针，指向返回类型为const vector<int>*的函数，且该函数的参数为int。*/

const vector<int>* (*seq_ptr)(int) = 0;	//给予函数指针初值
seq_ptr = pell_seq;		//将pell_seq()的地址赋值给seq_ptr
const vector<int> *pseq = seq_ptr(pos); //pseq会间接调用seq_ptr所指向的函数。

const vector<int>* (*seq_array[])(int) = {		//seq_array是一个数组，内放函数指针
    fibon_seq, lucas_seq, pell_seq,
    triang_seq, square_seq, pent_seq
};

enum ns_type{
    ns_fibon, ns_lucas, ns_pell,
    ns_triang, ns_square, ns_pent
};
/*关键字enum之后是一个可有可无的标识符ns_type,其后每个项称为枚举项，默认情况下，第一个枚举项的值为0，第二个值为1，以此类推。*/
seq_ptr = seq_array[ns_pell];	//枚举项的使用可以无需记住seq_array中各项位置
```

### 2.9 设定头文件 Setting Up a Header File

​	将函数声明放在头文件中，并在每个程序代码文件内包含(include)这些函数声明。

​	函数的定义只能有一份，但可以有许多份声明。但对于inline函数的定义，为了能够扩展inline函数的内容，在每个调用点上，编译器都能取得其定义。这意味着我们必须将inline函数的定义放在头文件中，而不是把它放在各个不同的程序代码文件中。

​	在file scope内定义的对象，如果可能被多个文件访问，就应该被声明于头文件中。

```c++
//这并不十分正确
const int seq_cnt = 6;		//正确，且seq_cnt是一个const object
const vector<int>* (*seq_array[seq_cnt])(int);	//错误，且seq_array不是一个const object，而是一个											//指向const object的指针

/*这并不正确，因为它会被解读为seq_array的定义而非声明。就像函数一样，一个对象只能在程序中被定义一次。对象的定义就像函数定义一样，必须放在程序代码文件中。只要在上述seq_array定义前加上关键字extern，它就会变成一个声明：*/
extern const vector<int>* (*seq_array[seq_cnt])(int);
```

​	const object就和inline函数一样，是“一次定义”规则下的例外。const object的定义只要一出文件之外便不可见。这意味着我们可以在多个程序代码文件中加以定义，不会导致任何错误。

## 第3章 泛型编程风格 Generic Programming

​	泛型算法之所以被称为泛型，是因为==它们和它们想要操作的元素类型无关==。

​	泛型算法通过function template技术，达到“与操作对象的类型相互独立”的目的。而实现“与容器无关”的诀窍，就是不要直接在容器身上操作。而是借由iterator(first和last)，标示我们要进行迭代的元素范围。

### 3.1 指针的算数运算  The Arithmetic of Pointers

​	实现find()函数，使之可以同时处理vector或者array。

```c++
template <typename elemType>
elemType* find(const elemType *first, const elemType *last, const elemType &value)
{
    if(!first || ! last)
        return 0;
    
    //当first不等于last，就把value拿来和first所指的元素比较，
    //如果两者相等，便返回first，否则将first递增1，令它指向下一个元素
    
    for(; first != last; ++first)
        if( *first == value)
            return first;
    
    return0;
}

template <typename elemType>
inline elemType* begin(const vector<elemType> &vec)
{
    return vec.empty() ? 0: &vec[0];
}

inline elemType* end(const vector<elemType> &vec)
{
    return vec.empty() ? 0: &vec[vec.size()];
}

//通过对vector的抽象，至此，find()已可以同时处理vector和array

find(begin(svec), end(svec), search_value);
```

### 3.2 了解Iterator（泛型指针） Making Sense of Iterators

```c++
vector<string> svec;

//以下是标准库中的iterator语法。
//iter指向一个vector,后者的元素类型为string。
//iter一开始指向svec的第一个元素。

vector<string>::iterator iter = svec.begin(); 
//iter可通过递增运算符进行遍历：
for(iter = svec.begin(); iter != svec.end(); ++iter)
    cout << *iter << ' ';


//对于const vector,我们使用const_iterator来进行遍历操作：
const vector<string> cs_vec;
vector<string>::const_iterator iter = cs_vec.begin();
//const_iterator允许我们读取vector的元素，但不允许任何写入操作。

//欲通过iterator取得元素值，我们可以采用一般指针的提领方式：
cout << "string value of element: " << *iter;

//欲通过iter调用底部的string元素所提供的操作，我们可以使用arrow运算符：
cout << "(" << iter -> size() << "): " << *iter << endl;
```



### 3.3 所有容器的共通操作 Operations Common to All Containers

​		下列为所有容器类（以及string类）的共通操作：

​		· equality(==)和inequality(!=)运算符，返回true或false。

​		· assignment(=)运算符，将某个容器复制给另一个容器。

​		·  empty()会在容器无任何元素时返回true，否则返回false。

​		· size()返回容器内目前持有的元素个数。

​		· clear()删除所有元素。

​		· begin()返回一个iterator，指向容器的第一个元素。

​		· end()返回一个iterator，指向容器的最后一个元素的下一位置。

​		· insert()将单一或某个容器内的元素插入容器内。

​		· erase()将容器内的单一元素或某个范围内的元素删除。

### 3.4 使用顺序性容器 Using the Sequential Containers

​		定义顺序性容器对象的五种方式：

```c++
//1. 产生空的容器：
list<string> slist;
vector<int> ivec;

//2. 产生特定大小的容器，每个元素都以其默认值作为初值：
list<int> ilist(1024);
vector<string> svec(32);

//3. 产生特定大小的容器，并为每个元素指定初值：
vector<int> ivec(10, -1);
list<string> slist(16, "unassigned");

//4. 通过一对iterator产生容器。这对iterator用来标示一整组作为初值的元素范围：
int ia[8] = {1, 1, 2, 3, 4, 8, 13, 21};
vector<int> fib(ia, ia + 8);

//5. 根据某个容器产生出新容器。复制原容器内的元素，作为新容器的初值：
list<string> slist;
//填充slist...
list<string> slist2(slist);	//将slist复制给slist2
```

push_back()和pop_back()允许我们在容器末尾进行插入和删除操作。

list和deque还提供push_front()和pop_front()，允许我们在容器最前端进行插入和删除操作。

每个容器除了拥有通用的插入函数insert()，还支持四种变形：

```c++
//1. 可将value插入position之前，它会返回一个iterator，指向被插入的元素。
iterator insert( iterator position, elemType value)
   
//2. 可在position之前插入count个元素，这些元素的值都和value相同。
void insert( iterator position, int count, elemType value)
    
//3. 可在position之前插入[first,last)所标示的各个元素：
void insert(iterator1 position, iterator2 first, iterator2 last)

int ia1[7] = {1, 1, 2, 3, 5, 55, 89};
int ia2[4] = {8, 13, 21, 34};
list<int> elems(ia1, ia1 + 7);

list<int>::iterator it = find(elems,begin(),elems,end(), 55);
elems.insert(it, ia2, ia2 + 4);
    
    
//4. 可在position之前插入元素，元素的初值为其所属类型的默认值。
iterator insert(iterator position)
```

每个容器除了拥有通用的删除函数erase()，还支持两种变形：

```c++
//1. 可删除posit所指的元素：
iterator erase( iterator posit )
    
//2. 可删除[first, last)范围内的元素：
iterator erase( iterator first, iterator last )
```

### 3.5 使用泛型算法 Using the Generic Algorithms



### 3.6 如何设计一个泛型算法 How to Design a Generic Algorithm

```c++
//初始版本
vector<int> less_than_10(const vector<int>& vec)
{

    vector<int> nvec;
    for (int ix = 0; ix < vec.size(); ++ix)
    {
        if (vec[ix] < 10)
        {
            nvec.push_back(vec[ix]);
        }
    }
    return nvec;
}

//改进1,使得此函数更通用化，让用户得以指定某个上限值。
vector<int> less_than(const vector<int>& vec, int less_than_val)
{
    vector<int> nvec;
    for (int ix = 0; ix < vec.size(); ++ix)
    {
        if (vec[ix] < less_than_val)
        {
            nvec.push_back(vec[ix]);
        }
    }
    return nvec;
}

//改进2，将“比较操作”参数化，以函数调用来取代less_than运算符
//自定义多个可传给filter()的关系比较函数：
bool less_than(int v1, int v2)
{
    return v1 < v2 ? true : false;
}

bool greater_than(int v1, int v2)
{
    return v1 > v2 ? true : false;
}


vector<int> filter(const vector<int>& vec, int filter_value, bool (*pred)(int, int))
{
    vector<int> nvec;

    for (int ix = 0; ix < vec.size(); ++ix)
    {
        pred = less_than;
        //调用pred所指函数，
        //比较vec[ix]和filter_value。
        if (pred(vec[ix], filter_value))
            nvec.push_back(vec[ix]);
    }
    return nvec;
}

//暂停改进，下述函数将说明如何在不对任一元素进行两次以上的查看的前提下，反复地在容器身上进行find():
int count_occurs(const vector<int>& vec, int val)
{
    vector<int>::const_iterator iter = vec.begin();
    int occurs_count = 0;
    while ((iter = find(iter, vec.end(), val)) != vec.end())
    {
        ++occurs_count;
        ++iter;     //指向下一个元素
    }
    return occurs_count;
}
```

#### Function Object

​		function object是某种class的实例对象，这类class对function call运算符做了重载操作，如此一来可使function object被当成一般函数来使用。

​		标准库事先定义了一组function object，分为算数运算、关系运算和逻辑运算三大类。以下列表中的type在实际使用时会被替换为内置类型或class类型：

```c++
#include<functional>
//六个算术运算
plus<type>, minus<type>, negate<type>, 
multiplies<type>, divides<type>, modules<type>
    
//六个关系运算
less<type>, less_equal<type>, greater<type>,
greater_equal<type>, equal_to<type>, not_equal_to<type>
    
//三个逻辑运算
logical_and<type>, logical_or<type>, logical_not<type>
    
//例：默认情况下，sort()会使用底部元素的类型所提供的less-than运算符，将元素升序排序，若传入great_than function object,元素就会以降序排序：
sort(vec.begin(), vec.end(), greater<int>());

```

#### Function Object Adapter

​		function object adapter会对function object进行修改才做，所谓binder adapter会将function的参数绑定至某个特定值，使二元function object转化为一元function object。

```c++

//改进3 通过使用for_if来取代for
vector<int> filter(const vector<int>& vec, int val, less<int>& lt)
{
    //bind2nd(less<int>, cal); 会把val绑定于less<int>的第二个参数身上，于是，less<int>会将每个元素拿来和val比较。
    
    vector<int> nvec;
    vector<int>::const_iterator iter = vec.begin();
    while ((iter = find_if(iter, vec.end(), bind2nd(lt, val))) != vec.end())
    {
        //每当iter != vec.end()，iter便指向某个小于val的元素。

        nvec.push_back(*iter);
        iter++;
    }
    return nvec;
}

//改进4 消除filter()与vector元素类型，以及filter()和vector容器类型的依赖关系，以使filter()更加泛型化
template <typename InputIterator, typename OutputIterator, typename ElemType, typename Comp>
OutputIterator filter(InputIterator first, InputIterator last, OutputIterator at, const ElemType &val, Comp pred)
{
    while ((first = find_if(first, last, bind2nd(pred, val))) != last)
    {
        //观察进行情形
        cout << "found value: " << *first << endl;

        //执行assign操作，然后令两个iterator前进
        *at++ = *first++;
    }
    return at;
}

int main()
{
    const int elem_size = 8;

    int ia[elem_size] = { 12, 8, 43, 0, 6, 21, 3, 7 };
    vector<int> ivec(ia, ia + elem_size);

    //下面这个容器用来储存过滤结果
    int ia2[elem_size];
    vector<int> ivec2(elem_size);

    cout << "filtering integer array for values less than 8\n";
    filter(ia, ia + elem_size, ia2, elem_size, less<int>());

    cout << "------------------------------------------------\n";

    cout << "filtering integer vector for values greater than 8\n";
    filter(ivec.begin(), ivec.end(), ivec2.begin(), elem_size, greater<int>());

}
```

### 3.7 使用Map  Using a Map

​		map被定义为一对（pair）数值，其中的key通常是个字符串，扮演索引的角色，另一个数值是value。

```c++
#include<map>
#include<string>

map<string, int> words;

//输入key/value的最简单方式:
words["vermeer"] = 1;

//对字数统计程序而言，可采用以下方法：
string tword;
while(cin >> tword)
    words[tword]++;
//其中word[tword]会取出与tword相应的value，如果tword不在map内，它便会因此被放到map内，并获得默认值0。


//欲查询map内是否存在某个key，有三种方法，最直觉的做法就是把key当成索引使用：
int count = 0;
if(!(count = words["vermeer"] ))
    //vermeer并不存在于words map内，但该写法的缺点是，如果key并不存在于map内，这个key会自动被加入到map中。
    
//法二：
words.find("vermeer");
//如果key已放入其中，find()会返回一个iterator，指向key/value形成的一个pair，反之则返回end()：
int count = 0;
map<string, int>::iterator it;

it = words.find("vermeer");
if(it != words.end())
    count = it ->secend;


//法三：
int count = 0;
string search_word("vermeer");

if(words.count(search_word))	//ok,它存在
    count = words[search_word];
```

### 3.8 使用Set Using a Set

​		Set由一群key组合而成。如果我们想知道某值是否存在于某个集合内，就可以使用set。例如，在graph traversal(图形遍历)算法中，我们可以使用set储存每个遍历过的节点(node)。在移至下一节点前，我们可以先查询set，判断该节点是否已经遍历过。

```c++
#include<set>
#include<string>
#include<map>
set<string> word_exclusion;
map<string, int> words;

while(cin >> tword)
{
    if( word_exclusion.count(tword))
    {
        //如果tword涵盖于“排除字集”内，
        //就跳过此次迭代的剩余部分。
        continue;
    }
    //ok:一旦抵达此处，表示tword并不属于“排除字集”
    words[tword]++;
}
```

默认情况下，set元素皆依据其所属类型默认的less-than运算符进行排序。

```c++
int a[10] = {1, 3, 5, 8, 5, 3, 1, 5, 8, 1};

vector<int> vec( ia, ia+10);
set<int> iset(vec.begin(), vec.end());
//iset的元素将是{1, 3, 5, 8}

iset.insert(ival);	//为set加入单一元素
iset.insert(vec.begin(), vec.end());	//为set加入某个范围的元素

set::iterator it = iset.begin();
for( ; it != iset.end(); ++it)
    cout << *it << ' ';
cout << endl;
```

### 3.9 如何使用Iterator Inserter	How to Use Iterator Inserters

​		由于不知道vector保留有多少空间，因此3.6的最后使用*at++ = *first++是危险的，标准库提供了三个所谓的insertion adapter，这些adapter让我们得以避免使用容器的assignment运算符：

```c++
#include<iterator>

//1. back_inserter()会以容器的push_back()函数取代assignment运算符。
vector<int>result_vec;
unique_copy( ivec.begin(), ivec.end(), back_inserter(result_vec));

//2. inserter()会以容器的insert()函数取代assignment运算符。
vector<string> svec_res;
unique_copy( svec.begin(), svec.end(), inserter( svec_res, svec_res.end() ));

//3. front_inserter()会以容器的push_front()函数取代assignment运算符。但仅适用于list和deque
list<int> ilist_clone;
copy( ilist.begin(), ilist.end(), front_inserter( ilist_copy));
```

### 3.10 使用iostream Iterator	Using the iostream Iterators

## 第4章基于对象的编程风格	Object-Based Programming

### 4.1 如何实现一个Class	How to Implement a Class

​		Class定义由两部分组成：class的声明，以及紧接在声明之后的主体。主体内的两个关键字public和private，用来标示每个块的“member访问权限”。Public member可以在程序的任何地方被访问，Private member只能在member function或是class friend内被访问。

```c++
class Stack{
    public:
    	// 任何操作函数如果执行成功，就返回true
    	// pop和peek会将字符串内容置于elem内
    	bool push( const string&);
    	bool pop( string &elem);
    	bool peek( string &elem);
    
    	bool empty();
    	bool full();
    
    	//size()定义于class本身中，
    	//其他member则仅仅只是声明。
    	int size(){return _stack.size()};	//size()即是Stack的一个inline member
    
    private:
    	vector<string> _stack;
}
```

​		所有member function都必须在class主体内进行声明。至于是否要同时进行定义，可自由决定。如果要在class主体内定义，这个member function会自动地被视为inline函数。要在class主体之外定义member function，必须使用特殊的语法，目的在于分辨该函数究竟属于哪一个class。如果希望该函数为inline，应该在最前面指定关键字inline：

```c++
inline bool
Stack::empty()	
   //empty是Stack class的一个member。class名称之后的"::"即所谓的class scope resolution(类作用域解析)运算符。
{
	return _stack.empty();
}

bool
Stack::pop( string &elem)
{
    if(empty())
        return fasle;
    
    elem = _stack.back();
    _stack.pop_back();
    return true;
}
```

​		对inline函数而言，定义于class主体内或主体外，并无分别。但也应该被放在头文件中。class定义及其inline member function通常会被放在与class同名的头文件中。例如Stack class的定义和其empty()函数定义都应该放在Stack.h头文件中。此即用户想要使用Stack时应该包含的文件。

### 4.2 什么是构造函数和析构函数 	What Are Class Constructors and the Class Destructor?

​		Constructor(构造函数)的函数名必须与class名称相同。Constructor不应指定返回类型，亦不用任何返回值。它可以被重载(overloaded)：

```c++
class Triangular {
    public:
    // 一组重载的consructor
    Triangular();	//default constructors
    Triangular( int len);
    Triangular( int len, int beg_pos );
    
    //...
};

Traingular t;	//会对t应用无任何参数的constructor(default constructor)
Traingular t2(10, 3);	//会调用带有两个参数的constructor。
Traingular t3 = 8;	//不是赋值操作。此举会调用带有单一参数的constructor。

Traingular t5();	//错误写法，此行将t5定义为一个函数，其参数列表是空的，返回Triangular object。

```

​		对于constructor：

```c++
//最简单的constructor就是default constructor。它不需要任何参数，这就意味着两种情况：
//第一：
Triangular::Triangular()
{
    //default constructor
    _length = 1;
    _beg_pos = 1;
    _next = 0;
}

//第二：这种情况更常见，它为每个参数提供了默认值：
class Triangular{
public:
    //也是default constructor
    Triangular( int len = 1; int bp = 1);	//对此，若不传入任何参数，将对len、bp赋初值1，1
    									 //若Triangular(5,5)即对len bp赋值5，5
    // ...
};
Triangular::Triangular( int len, int bp)
{
    _length = len > 0 ? len : 1;
    _beg_pos = bp > 0 ? bp : 1;
    _next = _beg_pos - 1;
}
```

#### Member Initialization List (成员初始化列表)

```c++
//Constructor的第二种初始化语法，是所谓的member initialization list:
Triangular::Triangular(const Triangular &rhs)
    : _length(rhs._length), _beg_pos(rhs._beg_pos), _next(ths.beg_pos-1)
    {}

//若Triangular包含一个string member：
class Triangular{
    public:
    //...
    private:
    string _name;
    int _next, _length, _beg_pos;
};

Triangular::Triangular(int len, int bp)
    :_name("Triangular")
    {
        _length = len > 0 ? len : 1;
        _beg_pos = bp > 0 ? bp : 1;
        _next = _beg_pos - 1;
    }
```

​		和constructor对立的是destructor。destructor是用户自定义的一个class member。一旦一个class提供有destructor，当其object结束生命时，便会自动调用destructor处理善后。Destructor主要用来释放在constructor中或对象生命周期中分配的资源。

​		Destructor的名称需要在class名称前面再加上'~'前缀。它绝对不会有返回值，也没有任何参数，由于其参数列表是空的，所以也绝不可能被重载。

```c++
class Matrix
{
	public:
    	Matrix(int row, int col)
            :_row(row), _col(col)
            {
                _pmat = new double[row * col];
            }
    	~Matrix()
        {
            delete [] _pmat;
        }
    private:
    	int _row, _col;
    	double* _pmat;
}
```

Destructor并非绝对必要，以Triangular为例，三个data member皆以by value方式存放，这些member在class object被定义之后便已存在，并在class object结束其生命时被释放。

#### Memberwise Initialization (成员逐一初始化)

```c++
Triangular tri1(8);
Triangular tri2 = tri1;
//class data member会被依次复制。本例中的_length、_beg_pos、_next都会依从tri1复制到tri2

//以下对于上一节定义的Matrix，使用default memberwise initialization会将mat2的_pmat设为mat的_pmat的值，
//这会使得两个对象的_pmat都指到heap内的同一个数组。当Matrix destructor应用于mat2上，该数组空间便被释放。不幸
//的是，此时mat的_pmat仍指向那个数组。
{
    Matrix mat(4,4);
    //此处，constructor发生作用
    {
        Matrix mat2 = mat;
        //此处，进行default memberwise initialization
        // ...在这里使用mat2
        //此处，mat2的destructor发生作用
    }
    //...在这里使用mat
    //此处，mat的destructor发生作用
}

//我们可以为Matrix提供另一个copy constructor从而解决上述问题。
Matrix::Matrix(const Matrix &rhs)
    :_row(rhs._row), _col(rhs._col)
    {
        //对rhs.pmat所指的数组产生一份完全副本
        int elem_cnt = row * col;
        _pmat = new double[elem_cnt];
        
        for(int ix = 0; ix < elem_cnt; ++ix)
            _pmat[ix] = rhs.pmat[ix];
    }
```

### 4.3 何谓mutable(可变)和const(不变)	What Are mutable and const?

```c++
class Triangular{
    public:
    //以下是const member function
    int length()	const {return _length;}
    int beg_pos()	const {return _beg_pos;}
    int elem(int pos)	const;
    //以下是non-const member function
    bool next( int &val);
    void next_reset()	{_next = _beg_pos - 1;}
    
    //...
    private:
    int _length;	//元素个数
    int _beg_pos;	//起始位置
    int _next;		//下一个迭代目标
    
    static vector<int> _elems;
};

//对于以下函数：
int sum(const Triangular &trian)
{
    //...
}
//trian是个const reference参数，因此编译器必须保证trian在sum()之中不会被修改。


//const修饰符紧接于函数参数列表之后。凡是在class主题以外定义者，如果它是一个const member function，那就必须同时在声明与定义中指定const。例如：
int Triangular::elem(int pos) const
{
    return _elems[pos - 1];
}
```

#### Mutable Data Member (可变的数据成员)

```C++
class Triangular{
    public:
    	bool next(int &val) const;
    	void next_reset() const { _next = _brg_pos - 1;}
    	//...
    
    private:
    	mutable int _next;
    	int _beg_pos;
    	int _length;
};
//现在，next()和next_reset()既可以修改_next的值，又可以被声明为const member functio
```

### 4.4 什么是this指针 	What Is the this Pointer?

​		

```c++
class Box
{
   public:
      // 构造函数定义
      Box(double l=2.0, double b=2.0, double h=2.0)
      {
         cout <<"Constructor called." << endl;
         length = l;
         breadth = b;
         height = h;
      }
      double Volume()
      {
         return length * breadth * height;
      }
      int compare(Box box)
      {
         return this->Volume() > box.Volume();
      }
   private:
      double length;     // Length of a box
      double breadth;    // Breadth of a box
      double height;     // Height of a box
};
//每一个对象都能通过 this 指针来访问自己的地址。this 指针是所有成员函数的隐含参数。
Box Box1(3.3, 1.2, 1.5);    // Declare box1
Box Box2(8.5, 6.0, 2.0);    // Declare box2 
if(Box1.compare(Box2))
{
	cout << "Box2 is smaller than Box1" <<endl;
}
else
{
	cout << "Box2 is equal to or larger than Box1" <<endl;
}
```

### 4.5 静态类成员	Static Class Members

​		static data member用来表示唯一的、可共享的member。它可以在同一类的所有对象中被访问：

```c++
class Triangular {
public:
	//...
private:
    static vector<int> _elems;	//class只需唯一的容器来储存数列元素。
};
```

​		对class而言，static data member只有唯一的一份实体，因此我们必须在程序代码文件中提供其清楚的定义。这种定义看起来很想全局对象(global object)的定义。唯一的差别是，其名称必须附上class scope运算符：

```c++
//以下放在程序代码文件中，例如Triangular.cpp
vector<int> Triangular::_elems;
//也可以为它指定初值：
int Triangular:: _initial_size = 8;
```

​		若要在class member function内访问static data member，其方式有如访问一般(non-static)数据成员：

```c++
Triangular::Triangular( int len, int beg_pos)
    :_length( len > 0 ? len:1),
	 _beg_pos( beg_pos > 0 ? beg_pos : 1)
{
	_next = _beg_pos - 1;
    int elem_cnt = _brg_pos + _length - 1;
         
    if(_elems.size() < elem_cnt)
        gen_elements( elem_cnt);
}
```

​		对于const static int data member，可以在声明时为其明确指定初值：

```c++
class intBuffer {
public:
    //...
private:
    static const int _buf_size = 1024;
    int _buffer[_buf_size];
}
```

#### Static Member Function(静态成员函数)

​		一般情况下，member function必须通过其类的某个对象来调用。这个对象会被绑定至该member function的this指针。通过储存于每个对象中的this指针，member function才能够访问储存于每个对象中的non-static data member。

```c++
bool Triangular::is_elem(int value){
    //...
}
//若is_elem()并未访问任何non-static data member。它的工作方式和任何对象都没有任何关联，因而应该可以很方便地以一般non-member function的方式来调用：
if( is_elem(8))	//×
if( Triangular::is_elem(8))	//√
```

​		于是static member function便可以在这种“与任何对象都无瓜葛”的情形之下被调用。member function只有在“不访问任何non-static member”的条件下才能够被声明为static：

```c++
class Triangular {
public:
    static bool is_elem(int);
    static void gen_elements(int length);
    static void gen_elems_to_value(int value);
    static void display(int length, int beg_pos, ostream &os = cout);
    //...
private:
    static const int _max_elems = 1024;		//Visual C++不允许在此直接指定初值
    static vector<int> _elems;
}

//当我们在class主体外部进行member function的定义时，无须重复加上关键字static(这个规则也适用于static data member)：
void Triangular::gen_elems_to_values(int value)
{
    //...
}
```

### 4.6 打造一个Iterator Class	Building an Iterator Class

```c++
//引入
Triangular trian(1, 8);
Triangular::iterator
    		it = trian.begin(),
			end_it = trian.end();
while( it != end_it )
{
    cout << *it << ' ';
    ++it;
}
//为了让上述代码得以工作，我们必须为iterator class定义!=、*、++等符号。运算符函数看起来很像普通函数，唯一差别是它不用指定 名称，只需在运算符前面加上关键字operator即可。例如：
class Triangular_iterator
{
public:
    //为了不要在每次访问元素时都执行-1操作。
    //此处将_index的值设为index - 1
    Triangular_iterator(int index) : _index(index - 1){}
    bool operator==(const Triangular_iterator&) const;
    bool operator!=(const Triangular_iterator&) const;
    int operator*() const;
    Triangular_iterator& operator++();	//前置(prefix)版
    Triangular_iterator operator++(int);	//后置(prefix)版
private:
    void check_integrity() const;
    int _index;
};

inline bool Triangular_iterator::
    operator==(const Triangular_iterator &rhs) const
			{return _index == rhs._index; }

//所谓运算符，可以直接作用于其class object：
if( trian1 == trian2)...
if( *ptri1 == *ptri2)...

//任何运算符如果和另一个运算符性质相反，我们通常会以后者实现出前者，如：
inline bool Triangular_iterator::
    operator != (const Triangular_iterator &rhs)const
			{return !(*this == rhs); }
```

运算符重载的规则：

· 不可引入新的运算符。

· 运算符的操作数个数不可改变。每个二元运算符都需要两个操作数，每个一元运算符都需要恰好一个操作数。

· 运算符的优先级不可改变。

· 运算符函数的参数列表中，必须至少有一个参数为class类型。

```c++
//运算符的定义方式，就像member function一样：
operator *() const
{
    check_integrity();
    return Triangular::_elems[_index];
}
//但也可以像non-member function一样：
inline int
    operator*(const Triangular_iterator &rhs)
	{
    	rhs.check_integrity();
    	return Triangular::_elems[_index];
	}

//Non-member运算符的参数列表中，一定会比相应的member运算符多出一个参数，也就是this指针。对member运算符而言，这个this指针隐式代表左操作数。
```

```c++
inline Triangular_iterator& Triangular_iterator::
    operator++()
{
    //前置版本
    ++_index;
    check_integrity();
    return *this;
}

inline Triangular_iterator Triangular_iterator::
    operator++( int )
{
    //后置版本
    Triangular_iterator tmp = *this;
    ++_index;
    check_integrity();
    return tmp;
}
//后置版本中，其唯一的那个int参数，编译器会自动为后置版产生一个int参数（其值必为0）。
```

​		下面是对Triangular做的修正：

```c++
#include "Triangular_iterator.h"

class Triangular{
public:
    typedef Triangular_iterator iterator;
    
    Triangular_iterator begin() const
    {
        return Triangular_iterator( _beg_pos);
    }
    Triangular_iterator end() const
    {
        return Triangular_iterator( _beg_pos + _length);
    }
    //...
private:
	int _beg_pos;
    int _length;
    //...
};
```

### 4.7 合作关系必须建立在友谊的基础上	Collaboration Sometimes Requires Friendship

​		

```c++
//以下的non-member operator*()会直接访问Triangular的private _elems以及Triangular _iterator的private check_integrity()
inline int operator*(const Triangular_iterator &rhs)
{
    rhs.check_integrity();		//直接访问private member
    return Triangular::_elems[rhs.index()];		//直接访问private member
}
```

​		任何class都可以将其他function或class指定为朋友(friend)。而所谓friend，具备了与class member function相同的访问权限，可以访问class的private member。

​		为了让operator\*()通过编译，不论Triangular或Triangular_iterator都必须将operator\*()声明为“朋友”：

```c++
class Triangular{
	friend int operator*(const Triangular_iterator &rhs);
    //...
};

class Triangular_iterator{
    friend int operator*(const Triangular_iterator &rhs);
}
```

​		也可以令class A和class B建立friend关系，借此让class A的所有member function都成为class B的friend。例如：

```c++
class Triangular {
    friend class Triangular_iterator;
    //...
};
```

​		如果以这种形式来声明class间的友谊，就不需要在友谊声明之前显现class的定义。

### 4.8 实现一个copy assignment operator  Implementing a Copy Assignment Operator

​		对于在4.2节中定义的Matrix，我们需要一个copy constructor和一个copy assignment operator。

```c++
Matrix& Matrix::
operator=(const Matrix &rhs)
{
    if(this != &rhs)
    {
        _row = rhs._row;
        _col = rhs._col;
        int elem_cnt = _row * _col;
        delete [] _pmat;
        _pat = new double[elem_cnt];
        for( int ix = 0; ix < elem_cnt; ++ix)
            _pmat[ix] = rhs._pmat[ix];
	}
    return *this;
}
```

### 4.9 实现一个function object	Implementing a Function Object

​		function object乃是一种“提供有function call运算符”的class。

```c++
class LessThan
{
public:
	LessThan( int val) : _val(val){}
    int comp_val() const {return _val;}	//基值的读取
    void comp_val(int nval){_val = nval;}	//基值的写入
    
    bool operator()(int _value) const;
private:
    int _val;
}

inline bool LessThan::
    operator()(int value) const {return value < _val;}

//将function call运算符应用于对象身上，便可以调用function call运算符；
int count_less_than(const vector<int> &vec, int comp)
{
    LessThan lt(comp);
    
    int count = 0;
    for(int ix = 0; ix < vec.size(); ++ix)
        if( lt( vec[ix]))
            ++count;
    
    return count;
}

//通常我们把function object当作参数传给泛型算法：
void print_less_than( const vector<int> &vec, int comp. ostream &os = cout)
{
    LessThan lt(comp);
    vector<int>::const_iterator iter = vec.begin();
    vector<int>::const_iterator it_end = vec.end();
    
    os << "elements less than " <<lt.comp_val() << endl;
    while ((iter = find_if(iter,it_end,lt)) != it_end)
    {
        os << *iter << ' ';
        ++iter;
    }
}
```

### 4.10 重载iostream运算符 	Providing Class Instances of the iostream Operator

​		

```c++
ostream& operator<<( ostream &os, const Triangular &rhs)
{
    os << "(" << rhs.beg_pos() << ","
       << rhs.length();
    
    rhs.display( rhs.length(), rhs.beg_pos(), os);
    return os;
}

istream& operator>>( istream &is, Triangular &rhs )
{
    char ch1, ch2;
    int bp, len;
    
    is >> ch1 >> bp
       >> ch2 >> len;
    //设定rhs的三个data member···
    rhs.beg_pos(bp);
    rhs.length(len);
    rhs.next_reset();
    
    return is;
}
```

### 4.11 指针，指向	Class Member Function  Pointers to Class Member Functions

```c++
class num_sequence{
public:
    typedef void (num_sequence::*PtrType)(int);
    //将PtrType声明为一个指针，指向num_sequence的member function，后者的返回类型必须是void，并且只接受单一参数，参数类型为int。
    
    //_pmf可指向下列任何一个函数
    void fibonacci(int);
    void pell(int);
    void lucas(int);
    void sequare(int);
    void pentagonal(int);
    //...
private:
    vector<int>* _elem;	//指向目前所用的vector
    PtrType _pmf;	//指向目前所用的算法(用以计算数列元素)
    static const int num_seq = 7;
    static PtrType func_tbl[num_seq];
    static vector<vector<int> > seq;
};
//定义一个指针，并令它指向member function fibonacci():
PtrType pm = &num_sequence::fibonacci;

const int num_sequence::num_seq;
vector<vector<int> > num_sequence::seq(num_seq);
//等价于：
//typedef num_sequence::PtrType PtrType;
//PtrType num_sequence::func_tbl[num_seq] = ...
    
num_sequence::PtrType
    num_sequence::func_tbl[num_seq] = {
    0,
    &num_sequence::fibonacci,
    &num_sequence::pell,
    &num_sequence::lucas,
    &num_sequence::triangular,
    &num_sequence::square,
    &num_sequence::pentagonal
}
```

​		Pointer to member function和pointer to function的一个不同点是，前者必须通过同一类的对象加以调用，而该对象便是此member function内的this指针所指之物。假设有以下定义：

```c++
num_sequence ns;
num_sequence *pns = &ns;
PtrType pm = &num_sequence::fibonacci;
//为了通过ns调用_pmf，有如下写法：
ns.fibonacci(pos);
(ns.*pm)(pos);
pns->fibonacci(pos);
(pns->*pm)(pos);
// .*符号是个pointer to member selection运算符，系针对class object工作。
//针对pointer to class object工作的pointer to member selection运算符，其符号是->*
```

## 第5章 面向对象编程风格	 Object-Oriented Programming

### 5.1 面向对象编程概念 	Object-Oriented Programming Concepts

​		面向对象编程概念的两项最主要特质是：继承(inheritance)和多态(polymorphism)。

​		继承使我们得以将一群相关的类组织起来，并让我们得以分享其间的共通数据和操作行为。

​		多态让我们在这些类之上进行编程时，可以如同操控单一个体，而非相互独立的类，并赋予我们更多弹性来加入或移除任何特定类。

​		继承机制定义了父子关系。父类定义了所有子类共通的公有接口(public interface)和私有实现(private implementation)。每个子类都可以增加或覆盖(override)继承而来的东西，以实现其自身独特的行为。

​		在C++中，父类被称为基类(base class)，子类被称为派生类(derived class)。父类和子类之间的关系则称为继承体系(inheritance hierarchy)。

​		动态绑定(dynamic binding)是面向对象编程风格的第三个独特概念。“找出实际被调用的究竟是哪一个派生类的函数”这一解析操作会延迟至运行时才进行。此即我们所谓的动态绑定。

### 5.2 漫游：面向对象编程思维 	A Tour of Object-Oriented Programming

​		默认情况下，member function的解析(resolution)皆在编译时静态地进行。若要令其在运行时动态进行，我们就得在它的声明前加上关键字virtual。

```c++
class LibMat {
public:
    LibMat(){ cout << "LibMat::LibMat() default constructor!\n"; }
    virtual ~LibMat() { cout << "LibMat::~LibMat() destructor!\n"; }
    virtual void print() const
    	{ cout << "LibMat::print() --I am a LibMat object!\n"; }
};
```

​		定义一个non-member function print():

```c++
void print(const LibMat &mat)
{
	cout << "in global print(): about to print mat.print()\n";
    
    //下一行会根据mat实际指向的对象，
    //解析该执行哪一个print() member function
    mat.print();
}
```

​		当程序定义出一个派生对象，基类和派生类的constructor都会被执行。当派生对象被销毁，基类和派生类的destructor也都会被执行。

​		为了清楚标示这个新类乃是继承自一个已存在的类，其名称之后必须接一个冒号(:), 然后紧跟着关键字public和基类的名称：

```c++
class Book : public LibMat{
public:
    Book(const string &title, const string &author)
        :_title(title), _author(author){
        cout << "Book::Book(" << _title << ", " << _author <<" ) constructor\n";
        }
    virtual ~Book(){
        cout << "Book::~Book() destructor!\n";
    }
    virtual void print() const {
        cout << "Book::print() --I am a Book object!\n"
             << "My title is : " << _title << '\n'
             << "My author is " << _author << endl;
    }
    const string& title() const{ return _title; }
    const string& author() const{ return _author; }

//被声明为protected的所有成员都可以被派生类直接访问，除此(派生类)之外，都不得直接访问protected成员。
protected:
    string _title;
    string _author;
};
```

### 5.3 不带继承的多态 	Polymorphism without Inheritance

​		4.11节的num_sequence class模拟了多态行为。该类的每一个对象皆能在程序执行过程中的任一时间点，通过member function set_sequence()，摇身一变为六个数列之一：

```c++
for( int ix = 1; ix < num_sequence::num_of_sequences(); ++ix)
{
    ns.set_sequence( num_sequence::nstype(ix));
    int elem_val = ns.elem(pos);
    //...
}
//为类定义一个data member _isa,用以记录它目前代表的数列类型：
class num_sequence{

private:
    vector<int> *_elem;	// 指向一个vector,后者用来储存数列元素
    PtrType _pmf;	//	指向元素产生器(函数)
    ns_type	_isa;	// 目前的数列元素
    //...
    
public:
   //_isa被赋予一个常量名称，此名称用以表示某个数列类型。所有数列类型名称被放在一个名为ns_type的枚举类型中：
    enum ns_type {
        ns_unset, ns_fibonacci, ns_pell, ns_lucas,
        ns_triangular, ns_square, ns_pentagonal
    };
    
    
    //ns_type()函数会检验其整数参数是否代表一个有效数列。
    //如果参数有效，nstype()就返回对应的enumerator(枚举项)，否则返回ns_unset(无效值):
    static ns_type nstype( int num)
    {
        return num <= 0 || num >= num_seq ? ns_unset : static_cast<ns_type>(num);
        //static_cast是个特殊转换记号，可将整数num转换为对应的ns_type枚举项。
    }
    
    
    //set_sequence()将当前数列的_pfm、_isa、_elem等data member设定妥当：
    void set_sequence(ns_type nst)
    {
        switch(nst)
        {
            default:
                cerr << "invalid type: setting to 0 \n";
                //刻意让它继续执行下去，不做break操作！
            case ns_unset:
                _pmf = 0;
                _elem = 0;
                _isa = ns_unset;
                break;
                
            case ns_fibonacci: 
            case ns_pell:
            case ns_lucas:
            case ns_triangular:
            case ns_square:
            case ns_pentagonal:
                //以下func_tbl是个指针表，每个指针指向一个member function。
                //以下seq是个vector，其内又是一些vector，用来储存数列元素。
                _pmf = func_tbl[nst];
                _elem = &seq[nst];
                _isa = nst;
                break;
        }
    }
};

//为了让外界查询此对象目前究竟代表何种数列，下面提供一个what_am_i()操作函数，返回一个表示数列类型的字符串：
inline void disply( ostream &os, const num_sequence &ns, int pos)
{
    os << "The element at position "
       << pos << "for the "
       << ns.what_am_i() << " sequence is "
       << ns.elem(pos) << endl;
}
const char* num_sequence:: what_am_i() const{
    stataic char *names[num_seq] = {
        "notSet", "fibonacci", "pell",
        "lucas", "triangular", "square", "pentagonal"
    };
    return names[_isa];
}
```

### 5.4 定义一个抽象基类 	Defining an Abstract Base Class

​		定义抽象类的第一个步骤就是找出所有子类共通的操作行为：

```c++
class num_sequence {
public:
    int elem( int pos);		//返回pos位置上的元素
    void gen_elems( int pos);	//产生直到pos位置的所有元素
    const char* what_am_i() const;	//返回确切的数列类型
    ostream& print(ostream &os = cout) const;	//将所有元素写入os
    bool check_integrity( int pos);		//检查pos是否为有效位置
    static int max_elems();		//返回所支持的最大位置值
    // ...
};
```

​		设计抽象基类的下一步，便是设法找出哪些操作行为与类型相关——也就是说，有哪些操作行为必须根据不同的派生类而有不同的实现方式。这些操作行为应该成为整个类继承体系中的虚函数(virtual function)。在上述代码中，每个数列类都必须提供它们自己的gen_elems()实现，但check_integrity()就不会因为类型的不同而有任何差异。注意：static member function无法被声明为虚函数。

​		设计抽象基类的第三步，便是试着找出每个操作行为的访问层级(access level)：

​		如果某个操作行为应该让一般程序皆能访问，我们应该把它声明为public。

​		如果某个操作行为在基类之外不需要被用到，我们就将它声明为private。即使是该基类的派生类，也无法访问基类中的private member。

​		如果某个操作行为可让派生类访问，却不允许一般程序使用，我们就把它声明为protected。

```c++
class num_sequence {

public:
    virtual ~num_sequence(){};
    
    virtual int elem(int pos) const = 0;	//纯虚函数
    virtual const char* what_am_i() const = 0;
    static int max_elems(){return _max_elems;}
    virtual ostream& print(ostream &os = cout ) const = 0;
    
protected:
    virtual void gen_elems( int pos )const = 0;
    bool check_integrity( int pos ) const;
    const static int _max_elems = 1024;
};
```

​		每个虚函数，要么得有其定义，要么可设为“纯”虚函数，如果对于该类而言，这个虚函数并无实质意义的话，例如gen_elems()之于num_sequence class。将虚函数赋值为0，意思便是令它为一个纯虚函数。

​		任何类如果声明有纯虚函数，那么由于其接口的不完整性，程序无法为它产生任何对象。这种类只能作为派生类的子对象使用，而且前提是这些派生类必须为所有虚函数提供确切的定义。

​		根据一般规则，凡基类定义有一个（或多个）虚函数，应该将其destructor声明为virtual，考虑以下程序片段：

```c++
num_sequence *ps = new Fibonacci(12);
//...使用数列
delete ps;		//通过ps调用的destructor一定是Fibonacci的destructor，不是num_sequence的destructor。
//ps是基类num_sequence的指针，但它实际上指向派生类Fibonacci的对象。当delete表达式被应用于该指针，destructor会先应用于指针所指的对象身上，于是将此对象占用的内存空间归还给程序的空闲空间。
```

​		在本例中，并不建议将其destructor声明为pure virtual——虽然它其实不具有任何实质意义的实现内容。对这类destructor而言，最好是提供空白定义：

```c++
inline num_sequence::~num_sequence(){}

```

### 5.5 定义一个派生类	 Defining a Derived Class



### 5.6 运用继承体系 	Using an Inheritance Hierarchy



### 5.7 基类应该多么抽象 	How Abstract Should a Base Class Be?



### 5.8 初始化、析构、复制 	Initialization, Destruction, and Copy



### 5.9 在派生类中定义一个虚函数 	Defining a Derived Class Virtual Function



### 5.10 运行时的类型鉴定机制 	Run-Time Type Identification



## 第6章 以template进行编程 Programming with Template

### 6.1 被参数化的类型 	Parameterized Types

### 6.2 Class Template的定义	The Template Class Definition

### 6.3 Template类型参数的处理 	Handing Template Type Parameters

### 6.4 实现一个 Class Template	 Implementing the Template Class

### 6.5 一个以Function Template完成的Output运算符	A Function Template Output Operator

### 6.6 常量表达式与默认参数值	Constant Expressions and Default Parameters

### 6.7 以Template参数作为一种设计策略	Template Parameters as Strategy

### 6.8 Member Template Function





## 第7章 异常处理 Exception Handling

### 7.1 抛出异常	Throwing an Exception

### 7.2 捕获异常	Catching an Exception

### 7.3 提炼异常	Trying for an Exception

### 7.4 局部资源管理	Local Resource Management

### 7.5 标准异常	The Standard Exceptions
