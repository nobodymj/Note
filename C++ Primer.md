# C++ Primer
## 类型别名
* typedef
```
typedef double wages; 
typedef wages base, *p;
```
* using
```
using SI = Sales_item;
```
### 指针、常量和类型别名
```
typedef char *pstring;
const pstring str = 0;  //cstr时指向char的常量指针
const pstring *ps;       //ps是一个指针，它的对象是指向char的常量指针
```

## auto类型说明符
由编译器分析表达式所属的类型。
auto定义的变量必须有初始值。
使用auto可以在一条语句中声明多个变量，但所有变量的初始基础数据类型必须一样。
```
auto i = 0, *p = &i;    //正确
auto sz = 0, pi = 3.14; //错误
```

## decltype类型指示符
它的作用是选择并返回操作数的数据类型。
```
decltype(f()) sum = x; //sum的类型就是函数f()的返回类型
const int ci = 0, &cj = ci;
decltype(ci) x = 0; //x的类型是const int
decltype(cj) y = x; //y的类型是const int&,y绑定到变量x
decltype(cj) z;     //错误，z是一个引用，必须初始化
```


## string类型
cctype头文件中的函数（对应c语言的ctype.h）
![](C++%20Primer/0F25B4BB-4E0E-4904-BA19-C5BC0F8F64FB.png)

string的操作：
![](C++%20Primer/13A4E37D-1CAA-4309-8B3E-53DDF96B7D86.png)

## vector
vector初始化方法
![](C++%20Primer/2C9934AB-151C-47D1-9876-CB6F3A1E9A56.png)
![](C++%20Primer/B21CDA28-5B66-4877-B1DC-2C55C06AB428.png)

## 迭代器
![](C++%20Primer/B1F8703A-30C1-4464-8CD5-FC35403611BD.png)
![](C++%20Primer/FA3C320A-B1B1-466C-9981-9E31FD8B23B9.png)


## friend 友元 
类可以允许其他类或者函数访问它的非公有成员，方法是令其他类或者函数成为它的友元。
友元的声明，仅仅指定了访问的权限，而非一个通常意义上的函数声明。
友元类：如果一个类指定了友元类，则友元类的成员函数可以访问此类包括非公有成员在内的所有成员。

##  inline 内联 
在类中，经常有一些规模较小的函数适合于被声明成内联函数。
定义在类内部的成员函数是自动inline的。

## 构造函数
如果成员是const、引用，或者属于某种未提供默认构造函数的类类型，我们必须通过构造函数初始值列表为这些成员提供初值。
构造函数初始值列表中初始值的前后位置顺序关系不会影响世纪的初始化顺序。成员函数的初始化顺序与它们在类定义中的出现顺序一致。

## 类的静态成员
通过 在成员的声明前加上关键字static使得其与类关联在一起。
静态成员可以是public或者private的。静态数据成员的类型可以是常量、引用、指针、类类型等。
类的静态成员存在于任何对象之外，对象中不包含任何与静态数据成员有关的数据。
静态成员函数也不与任何对象绑定在一起，它们不包含this指针，也不能被声明成const的，我们也不能在static函数体内使用this指针。
定义静态成员函数，可以在类的内部也可以在类的外部定义。当在 类的外部定义静态成员时，不能重复static关键字，该关键字只出现在类内部的声明语句中。
一般来说，不能在类的内部初始化静态成员，必须在类的外部定义和初始化每个静态成员。一个静态数据成员只能定义一次。

## 聚合类
满足如下条件的类：
* 所有成员都是public的
* 没有定义任何构造函数
* 没有类内初始值
* 没有基类，也没有virtual函数


# IO库
### IO库的条件状态

![](C++%20Primer/C2721162-2C68-4E40-9588-3306F8F6675C.png)
![](C++%20Primer/FE7C523D-02F0-4F19-AB72-8EDC19156BDB.png)

### 刷新输出缓冲区
* endl 完成换行并刷新缓冲区
* flush 刷新缓冲区，但不输出任何额外的字符
* ends 向缓冲区插入一个空字符，然后刷新缓冲区

### unitbuf操纵符
cout << unitbuf;       //所有输出操作后都会立即刷新缓冲区
cout << nounitbuf;  //回到正常的缓冲方式

### tie 关联输入和输出流
* 不带参数的版本：返回指向输出流的指针
* 接受一个只想ostream的指针的版本：将自己关联到此ostream

### fstream特有的操作

![](C++%20Primer/3D444118-E9F2-4DCF-B738-F99CBD217B1E.png)
文件模式：
![](C++%20Primer/2067BF12-D2AA-4AE8-ADB7-8AAA5074E26C.png)
文件模式限制：
![](C++%20Primer/2007141A-E526-446F-9EB9-4654812616CB.png)


### stringstream特有的操作

![](C++%20Primer/6FEA6822-F7F6-4751-A710-5EB12441B73A.png)


## 顺序容器

![](C++%20Primer/01992C3E-C4AF-4670-AB08-D12CFBD2BF8A.png)
选择容器的基本原则：
![](C++%20Primer/217F940F-A117-41EC-9950-48D3EBB079A0.png)

### 容器操作
![](C++%20Primer/8FC4776A-8FBF-48CA-8050-5E326F042923.png)
![](C++%20Primer/1787F94E-A166-493C-8C85-B7015B764EF3.png)

容器定义和初始化：
![](C++%20Primer/BAC9E97A-C473-430F-9834-75369D3A14A3.png)
容器赋值运算：
![](C++%20Primer/B929A078-5018-4884-A9EB-F8E347BEC6F8.png)
向顺序容器中添加元素：
![](C++%20Primer/848B5E96-A53A-42F7-9D32-C8304A50AD6B.png)
在顺序容器中访问元素：
![](C++%20Primer/765BBD28-CCDA-402E-AEC9-5DC97ECA9A5A.png)
删除元素：
![](C++%20Primer/E88F13CC-C839-40AB-8C30-559C91FF9448.png)
forward_list的插入和删除操作：
![](C++%20Primer/D3972C41-5AAC-4455-B950-04AB90C9837C.png)
改变容器大小：
![](C++%20Primer/2003646C-0C86-4EC0-80D6-B487C4BB3D28.png)
容器大小管理操作：
![](C++%20Primer/9A6755B3-83F5-4D5D-A587-C07D8627EE5A.png)

### string额外操作
构造string：
![](C++%20Primer/03A50318-F7A3-431E-9020-73E887CCC390.png)
子字符串操作：
![](C++%20Primer/45904B93-582D-480A-B7AD-B3F545B23B6C.png)
修改string操作：
![](C++%20Primer/B220D5E0-2EC1-4B8A-B9A4-AA99B3550315.png)
![](C++%20Primer/0E42B815-FDBF-4E60-AB07-F7EF2502946C.png)
string搜索操作：
![](C++%20Primer/1865EB5E-AE34-4FD7-8981-36A98DD0BB30.png)
![](C++%20Primer/79EAE0DF-A78F-4888-9E00-00EB7A78F351.png)
compare函数：
![](C++%20Primer/208E77E2-7222-4A2F-8E6B-0A00E7A01560.png)
数值转换：
![](C++%20Primer/581864A6-0DDC-43E9-A59B-1E14FC255468.png)

### 容器适配器
顺序容器适配器：stack、queue、priority_queue
默认情况下，stack和queue是基于deque实现的，priority_queue是在vector之上实现的。
所有容器适配器都支持的操作和类型：
![](C++%20Primer/FD9AAAA0-97BB-4FD2-B2D9-11C9A921DF7A.png)
栈支持的操作：
![](C++%20Primer/10279857-100C-463C-83D2-93676FDA77F7.png)
queue和priority_queue支持的操作：
![](C++%20Primer/7F98CDED-12A3-4A34-B056-786B482697AA.png)
![](C++%20Primer/D7EE8298-5A49-46B4-B6E6-9CFA8A9BF8A5.png)


## 可调用的概念
对于一个对象或一个表达式，如果可以对其使用调用运算符，则称它为可调用的。即如果e是一个可调用的表达式，则我们可以编写代码e(args)，其中args是一个逗号分隔的一个或多个参数的列表。
目前可调用对象有：函数、函数指针、重载了函数调用运算符的类、lambda表达式。

## lambda表达式
lambda表达式形式：
![](C++%20Primer/8B411B91-EBC1-4F88-9A51-014FB4FCCAD8.png)
lambda捕获列表：
![](C++%20Primer/6933C621-7447-496B-B891-A42D82298D9E.png)

默认情况下，对于一个值被拷贝的变量，lambda不会改变其值。如果希望能改变一个被捕获的变量的值，就必须在参数列表首加上关键字mutable。
举例：
auto f = [v1] () mutable { return ++v1; };

当需要为一个lambda定义返回类型时，必须使用尾置返回类型。
举例：
transform(vi.begin(), vi.end(), vi.begin(), [](int i) -> int { if (i < 0) return -i; else return i; });

## 插入迭代器
![](C++%20Primer/74EA0362-D613-4183-B7DF-1CDB6A825531.png)

## iostream迭代器
![](C++%20Primer/082CB3CF-F10C-4270-B460-5E83D52D2888.png)
![](C++%20Primer/441CC038-0F38-4FE2-9FFC-C5048243695E.png)

## 链表相关算法
list和forward_list应该优先使用成员函数版本的算法，而不是通用算法。
![](C++%20Primer/C63EAE91-FC23-4CD6-A3B0-FC7A65E46D51.png)
![](C++%20Primer/CEC891EA-DCCD-4A8D-8AF8-CD3B287B816A.png)
![](C++%20Primer/51425E52-327F-47D5-A417-1E1F8D88F6DF.png)

## 关联容器
类型：

![](C++%20Primer/B774340C-218E-40B8-A4F5-2EEC470E5609.png)

![](C++%20Primer/F12B5F20-93F8-4A1E-A14E-9F04ADCCDF03.png)

![](C++%20Primer/D83F6373-6FF2-4AE0-A24F-DF371BAA5005.png)

![](C++%20Primer/466E0ECA-C0AF-4C68-AF3D-6B1A090415C9.png)

![](C++%20Primer/AA6C5482-EA5C-48FE-AA0D-320E27583D9B.png)

![](C++%20Primer/D3D35A4D-4F9A-4DF0-9289-AC454C3E1120.png)
![](C++%20Primer/83FF93D6-0BA2-4326-AC6F-0B5CF860367B.png)

### 无序容器
实现：哈希表

![](C++%20Primer/493D96BF-ECF0-4F46-A861-F3DA978D00C1.png)

## 智能指针
shared_ptr：允许多个指针指向同一个对象；
unique_ptr：独占所指向的对象；
weak_ptr：弱引用，指向shared_ptr所管理的对象。

![](C++%20Primer/7857B403-8FAA-4BC8-B9B2-A3D61DA3802D.png)

![](C++%20Primer/7FC4686F-4533-48E0-8E5D-F7D2641E6D47.png)

![](C++%20Primer/6E74F33E-AD39-462F-A88C-41D972606F91.png)
![](C++%20Primer/1D060A3F-9EFB-4674-88FA-4DBDC49C28F5.png)

![](C++%20Primer/C4B6FAB9-FE2F-4562-AFFE-184A054C0E6A.png)

![](C++%20Primer/F0820263-517E-417C-BEAC-2E500F1ED738.png)

![](C++%20Primer/9102F99A-483D-4D0A-906A-870FF3B014DE.png)
shared_ptr不直接支持管理动态数组。如果希望使用shared_ptr管理一个动态数组，必须提供自己定义的删除器。
举例：
shared_ptr<int> sp(new int[10], [](int *p) { delete [] p; });
sp.reset();

### allocator

![](C++%20Primer/402EB515-9B29-49B1-8167-4140D74B3436.png)

![](C++%20Primer/2CC67C7E-CD7A-4A68-9D36-B90C30DAA015.png)

## 拷贝控制
### 拷贝初始化发生的情况：
![](C++%20Primer/37A921CF-1B45-4302-A731-B8C3AC7F7048.png)
![](C++%20Primer/3C487B32-1C00-4235-A2BC-D0F22F0B68F2.png)
### 使用=default
将拷贝控制成员定义为=default来显示地要求编译器生成合成的版本。
### 阻止拷贝
可以通过将拷贝构造函数和拷贝赋值运算符定义为删除的函数来阻止拷贝。
删除的函数：我们虽然声明了它们，但不能以任何方式使用它们。
在函数的参数列表后面加上=delete来指出我们希望将它定义为删除的。
注意：不能删除析构函数。
对某些类来说，编译器将这些合成的成员定义 为删除的函数：

![](C++%20Primer/2CFAF987-E204-4C2E-A768-4728220A6995.png)

### 右值引用与移动构造/移动赋值运算符


### 运算符重载
可以/不可以被重载的运算符：
![](C++%20Primer/C550AFCD-917D-4951-A07C-F2ABF52CFC84.png)
运算符选择作为成员/非成员函数的考虑：
![](C++%20Primer/2789EC4D-15E2-4539-849E-95130D8D3661.png)

### 重载前置运算符和后置运算符
前置运算符：
StrBlobPtr operator++();
StrBlobPtr operator—();
后置运算符：递增/递减对象的值但是返回原值
StrBlobPtr operator++(int);
StrBlobPtr operator—(int);

## 标准库定义的函数对象
![](C++%20Primer/BB77F322-C4A0-4FE8-BE0C-BA6FE8C5A141.png)

### C++可调用的对象
函数、函数指针、lambda表达式、bind创建的对象、重载了函数调用运算符的类。
### 标准库function类型
![](C++%20Primer/3D311A3C-2ACA-4303-B277-5C7E06D5E076.png)

## 模板
![](C++%20Primer/0E4AEBB8-BDE6-43E2-A003-8DD1EAD2D352.png)


## tuple
tuple支持的操作：
![](C++%20Primer/FF11321C-5D38-4AF0-8F33-43F71570E858.png)


## bitset
初始化bitset：
![](C++%20Primer/5A3B1C83-23A2-4749-908C-C0029D0C34AB.png)
bitset操作：
![](C++%20Primer/2CCEFE3C-8FA2-495C-AC80-08ACF9F84E4A.png)

## 正则表达式
正则表达式库组件：
![](C++%20Primer/2D49D903-2B43-44C5-A7A1-9A9700BE3EEB.png)
regex_search和regex_match的参数：
![](C++%20Primer/F21AFE36-C43D-414F-BDC7-39AFD3EA0B59.png)
regex（和wregex）选项：
![](C++%20Primer/9AAB9A66-C427-42BE-9A06-8559CCEA0881.png)
正则表达式错误类型：
![](C++%20Primer/E03A8FAD-7AA3-4B9D-8931-3AD95425B50B.png)
正则表达式库类：
![](C++%20Primer/587E1B57-2D46-4B6E-A259-CF1AB7483689.png)

sregex_iterator操作：
![](C++%20Primer/0C10CA06-1B05-4353-BDBD-3DCB04E430DF.png)
![](C++%20Primer/09FA2AD1-9502-4767-84F5-C74865183801.png)
smatch操作：
![](C++%20Primer/1A3C2D2E-C439-43F7-8CA1-DD21C73B5138.png)

子匹配操作：
![](C++%20Primer/9645CC78-32BE-4C28-970E-41568A181540.png)

正则表达式替换操作：
![](C++%20Primer/8A572C1D-3E56-4D27-B8AA-29CC161EB9E4.png)

就像标准库定义标志来指导如何处理正则表达式一样，标准库还定义了用来在替换过程中控制匹配或格式的标志：
匹配标志：
![](C++%20Primer/82A5D6E4-BBEF-4B8D-9A7D-DD9D360F4F38.png)


## 随机数
![](C++%20Primer/36A72A6D-3A8D-4EB7-B8A9-584A7589418E.png)
### 引擎
![](C++%20Primer/F4263F13-DDB9-4279-B554-FDC8351C0B64.png)
对于大多数场合，随机数引擎的输出是不能直接使用的。
得到一个指定范围内的数：
```
//生成0到9之间（包含）均匀分布的随机数
uniform_int_distribution<unsigned> u(0,9);
default_random_engine e;
cout << u(e) << endl;
```
设置随机数发生器种子：
1）在创建引擎对象时提供种子：
default_random_engine e1(235632);
2）调用引擎的seed成员：
default_random_engine e2;
e2.seed(38757594);
常用：default_random_engine e1(time(0));
### 分布
![](C++%20Primer/3128DA94-40FD-449C-AC29-F3D3526AAA33.png)

## 定义在iostream中的操纵符
![](C++%20Primer/441C7A87-3C50-4FC5-B2EF-7452EBA1E57C.png)
![](C++%20Primer/44B8567C-443E-4143-9DBD-BC432C4CDD51.png)
### 定义在imanip中的操纵符
![](C++%20Primer/7E58AB0C-9767-4229-AEEE-BC76AB8D6880.png)
### 单字节低层IO操作
![](C++%20Primer/7FBC7E0E-50C2-4EF0-A39A-BC8C2033DC10.png)
### 多字节低层IO操作
![](C++%20Primer/B32818BE-3105-4571-9274-5B5F35B03AFE.png)
![](C++%20Primer/2F6C5330-3BC2-4653-BF8F-EB140DAD70C1.png)
### seek和tell函数

![](C++%20Primer/BCC16137-37D2-449B-8E34-5EC4797A6F03.png)