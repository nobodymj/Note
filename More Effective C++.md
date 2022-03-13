# More Effective C++
##  1. 指针和引用的区别
应该使用引用的情况：
* 如果总是指向一个对象并且一旦指向一个对象后就不会改变指向；
* 当你重载某个操作符时，应该使用引用。

应该使用指针的情况：
* 你考虑到存在不指向任何对象的可能；
* 你需要能否在不同的时刻指向不同的对象；
* 上述使用引用外的其他情况

## 2. 尽量使用C++风格的类型转换
* const_cast<T>( expression )：用来将对象的常量性转除
* dynamic_case<T>( expression )：主要用来执行“安全向下转型”（继承）
* reinterpret_case<T>( expression )：意图执行低级转型，实际动作（及结果）可能取决于编译器，这也就表示它不可移植（比如将pointer to int转型为一个int）
* static_case<T>( expression )：用来强迫隐式转换（比如non-const对象转为const，int转为double），也可用来执行上述多种转换的反向转换（比如void*指针转为typed指针，pointer-to-base转为pointer-to-derived）

## 3. 不要对数组使用多态

## 4. 避免无用的缺省构造函数
提供无意义的缺省构造函数也会影响类的工作效率。如果成员函数必须测试所有的部分是否都被正确地初始化，那么这些函数的调用者就得为此付出更多的时间。而且还得付出更多的代码，因为这使得可执行文件或库变得更大。它们也得在测试失败的地方放置代码来处理错误。如果一个类的构造函数能够确保所有的部分被正确初始化，所有这些弊病都能够避免。缺省构造函数一般不会提供这种保证，所以在它们可能使类变得没有意义时，尽量去避免使用它们。使用这种（没有缺省构造函数的）类的确有一些限制，但是当你使用它时，它
也给你提供了一种保证：你能相信这个类被正确地建立和高效地实现。 

## 5. 谨慎定义类型转换函数
有两种函数允许编译器进行转换：
1）单参数构造函数：只用一个参数即可以调用的构造函数。可以是只定义了一个参数，也可以是虽定义了多个参数但第一个参数以后的所有参数都有缺省值。
2）隐式类型转换运算符：即operator关键字，其后跟一个类型符号的成员函数。

克服方法：
1）构造函数用explicit声明，则编译器会拒绝为了隐式类型转换而调用构造函数。
2）不声明运算符（operator）的方法。

## 6. 自增（increment）、自减（decrement）操作符前缀形式与后缀形式的区别
```
// 前缀形式：increment and fetch
UPInt& UPInt::operator++() 
{ 
 *this += 1; // 增加 
 return *this; // 取回值 
} 
// 后缀形式: fetch and increment 
const UPInt UPInt::operator++(int) 
{ 
 UPInt oldValue = *this; // 取回值 
 ++(*this); // 增加 
return oldValue; // 返回被取回的值 
} 
```
后缀操作符函数没有使用它的参数。它的参数只是用来区分前缀与后缀函数调用。
当处理用户定义的类型时，尽可能地使用前缀increment，因为它的效率较高（后缀increment需要建立一个临时对象以作为它的返回值）。

## 7. 不要重载 “&&”、“||”、“,”
```
不能重载的操作符：
.                .*                  ::               ?: 
new             delete               sizeof           typeid 
static_cast    dynamic_cast     const_cast    reinterpret_cast 

能重载的操作符：
operator new        operator delete 
operator   new[]    operator delete[] 
 +    -   *   /   %   ^     &   |     ~ 
!    =   <   >  +=   -=   *=   /=   %= 
^=  &=  |=  <<  >>   >>=  <<=  ==   != 
<=  >=  &&  ||  ++   --    ,   ->*  -> 
()  [] 
```
能重载这些操作符不是去重载的理由。如果没有一个好理由重载操作符，就不要重载。

## 8. 理解各种不同含义的new和delete
new operator（new操作符）：既分配内存又为对象调用构造函数
operator new：只分配内存。也可以自定义operator new来定制在堆对象被建立时的内存分配过程。
placement new：可以在一块已经获得指针的内存里建立一个对象。

## 9. 使用析构函数防止资源泄露
隐藏在 auto_ptr 后的思想是：用一个对象存储需要被自动释放的资源，然后依靠对象的析构函数来释放资源，这种思想不只是可以运用在指针上，还能用在其它资源的分配和释放上。
```
//一个类，获取和释放一个 window 句柄 
class WindowHandle { 
public: 
 WindowHandle(WINDOW_HANDLE handle): w(handle) {} 
 ~WindowHandle() { destroyWindow(w); } 
 operator WINDOW_HANDLE() { return w; } // see below 
private: 
 WINDOW_HANDLE w; 
 // 下面的函数被声明为私有，防止建立多个 WINDOW_HANDLE 拷贝 
 //有关一个更灵活的方法的讨论请参见条款 M28。 
 WindowHandle(const WindowHandle&); 
 WindowHandle& operator=(const WindowHandle&); 
}; 
 这看上去有些象 auto_ptr，只是赋值操作与拷贝构造被显式地禁止（参见条款 M27），有一个隐含的转换操作能把 WindowHandle 转换为 WINDOW_HANDLE。这个能力对于使用WindowHandle 对象非常重要，因为这意味着你能在任何地方象使用 raw WINDOW_HANDLE 一样来使用 WindowHandle。（参见条款 M5 ，了解为什么你应该谨慎使用隐式类型转换操作） 
 通过给出的 WindowHandle 类，我们能够重写 displayInfo 函数，如下所示： 
// 如果一个异常被抛出，这个函数能避免资源泄漏 
void displayInfo(const Information& info) 
{ 
 WindowHandle w(createWindow()); 
 在 w 对应的 window 中显式信息; 
} 
 即使一个异常在 displayInfo 内被抛出，被 createWindow 建立的 window 也能被释放。
```

## 10. 在构造函数中防止资源泄漏
如果你用对应的 auto_ptr 对象替代指针成员变量，就可以防止构造函数在 存在异常时发生资源泄漏，你也不用手工在析构函数中释放资源，并且你还能象以前使用非 const 指针一样使用 const 指针，给其赋值。 
在对象构造中，处理各种抛出异常的可能，是一个棘手的问题，但是 auto_ptr(或者类 似于 auto_ptr 的类)能化繁为简。它不仅把令人不好理解的代码隐藏起来，而且使得程序在 面对异常的情况下也能保持正常运行。 

## 11. 禁止异常信息（exceptions）传递到析构函数外 
在有两种情况下会调用析构函数：
1）在正常情况下删除一个对象，例如对象超出了作用域或被显式地 delete。
2）异常传递的堆栈辗转开解（stack-unwinding）过程中，由异常处理系统删除一个对象。

禁止异常传递到析构函数外有两个原因：
1）能够在异常转递的堆栈辗转开解（stack-unwinding）的过程中，防止 terminate 被调用。
2）它能帮助确保析构函数总能完成我们希望它做的所有事情。

## 12. 理解“抛出一个异常”与“传递一个参数”或“调用一个虚函数”间的差异
把一个对象传递给函数或一个对象调用虚拟函数与把一个对象做为异常抛出，这之间有三个主要区别：
* 第一、异常对象在传递时总被进行拷贝；当通过传值方式捕获时，异常对象被拷贝了两次。对象做为参数传递给函数时不一定需要被拷贝。
* 第二、对象做为异常被抛出与做为参数传递给函数相比，前者类型转换比后者要少（前者只有两种转换形式）。
* 最后一点，catch 子句进行异常类型匹配的顺序是它们在源代码中出现的顺序，第一个类型匹配成功的 catch 将被用来执行。当一个对象调用一个虚拟函数时，被选择的函数位于与对象类型匹配最佳的类里，即使该类不是在源代码的最前头。 

## 13. 通过引用捕获异常
异常传递方式：
* catch by value：通过传值捕获异常（比如：catch (Widget w) … ） ，会建立两个被抛出对象的拷贝，一个是所有异常都必须建立的临时对象，第二个是把临时对象拷贝进 w 中。它会产生 slicing problem，即派生类的异常对象被做为基类异常对象捕获时，那它的派生类行为就被切掉了。
* catch by reference：通过引用捕获异常（比如：catch (Widget& w) 、catch (const Widget& w)… ） ，会建立一个被抛出对象的拷贝：拷贝同样是一个临时对象。
* catch by pointer：通过指针抛出异常（比如：catch (Widget* w) … ），是一个指针的拷贝被传递。必须确保当程序控制权离开抛出指针的函数后，对象还能够继续生存。全局与静态对象或堆对象都能够做到这一点。

通过引用捕获异常（catch by reference），不会为是否删除异常对象而烦恼；能够避开 slicing 异常对象；能够捕获标准异常类型；减少异常对象需要被拷贝的数目。

## 14. 审慎使用异常规格（exception specifications）
以全面的角度去看待异常规格是非常重要的。它们提供了优秀的文档来说明一个函数抛出异常的种类，并且在违反它的情况下，会有可怕的结果，程序被立即终止，在缺省时它们会这么做。同时编译器只会部分地检测它们的一致性，所以他们很容易被不经意地违反。而且他们会阻止 high-level 异常处理器来处理 unexpected 异常，即使这些异常处理器知道如何去做。 
 异常规格是一个应被审慎使用的特性。在把它们加入到你的函数之前，应考虑它们所带来的行为是否就是你所希望的行为。

## 15. 了解异常处理的系统开销
不论异常处理的开销有多大我们都得坚持只有必须付出时才付出的原则。为了使你 的异常开销最小化，只要可能就尽量采用不支持异常的方法编译程序，把使用 try 块和异常 规格限制在你确实需要它们的地方，并且只有在确为异常的情况下（exceptional）才抛出 异常。如果你在性能上仍旧有问题，总体评估一下你的软件以决定异常支持是否是一个起作 用的因素。如果是，那就考虑选择其它的编译器，能在 C++异常处理方面具有更高实现效率 的编译器。

## 16. 牢记80-20准则
80－20 准则说的是大约 20％的代码使用了 80％的程序资源；大约 20%的代码耗用了大 约 80％的运行时间；大约 20％的代码使用了 80％的内存；大约 20％的代码执行 80％的磁 盘访问；80％的维护投入于大约 20％的代码上；通过无数台机器、操作系统和应用程序上 的实验这条准则已经被再三地验证过。80－20 准则不只是一条好记的惯用语，它更是一条 有关系统性能的指导方针，它有着广泛的适用性和坚实的实验基础。 

## 17. 考虑使用lazy evaluation（懒惰计算法）
 （mutable字段：表示在任何函数里它们都能被修改，甚至在 const 成员函数里）
 lazy evaluation 在各个领域都是有用的：能避免不需要的对象 拷贝，通过使用 operator[]区分出读操作，避免不需要的数据库读取操作，避免不需要的 数字操作。但是它并不总是有用。就好象如果你的父母总是来检查你的房间，那么拖延整理房间将不会减少你的工作量。实际上，如果你的计算都是重要的，lazy evaluation 可能会减慢速度并增加内存的使用，因为除了进行所有的计算以外，你还必须维护数据结构让 lazy evaluation 尽可能地在第一时间运行。在某些情况下要求软件进行原来可以避免的计算， 这时 lazy evaluation 才是有用的。 
C++特别适合用户实现 lazy evaluation，因为它对封装的支持使得能在类里加入 lazy evaluation，而根本不用让类的使用者知道。
我们可以直接用 eager evaluation 方法来实现一 个类，但是如果你用通过 profiler 调查（参见条款 M16）显示出类实现有一个性能瓶颈， 就可以用使用 lazy evaluation 的类实现来替代它（参见 Effective C++条款 34）。对于使 用者来说所改变的仅是性能的提高（重新编译和链接后）。这是使用者喜欢的软件升级方式， 它使你完全可以为懒惰而骄傲。 

## 18. 分期摊还期望的计算
over-eager evaluation（过度热情计算法）：在要求你做某些事情以前就完成它们。
隐藏在 over-eager evaluation 后面的思想是如果你认为一个计算需要频繁进行，你就 可以设计一个数据结构高效地处理这些计算需求，这样可以降低每次计算需求时的开销。 
采用 over-eager 最简单的方法就是 caching(缓存)那些已经被计算出来而以后还有可 能需要的值。通过 over-eager 方法分摊预期计算的开销，例如caching 和 prefething。
当 你必须支持某些操作而不总需要其结果时，lazy evaluation 是在这种时候使用的用以提高 程序效率的技术。当你必须支持某些操作而其结果几乎总是被需要或被不止一次地需要时， over-eager 是在这种时候使用的用以提高程序效率的一种技术。它们所产生的巨大的性能 提高证明在这方面花些精力是值得的。 

## 19. 理解临时对象的来源
可能建立临时对象的情况：
1）为了使函数成功调用而进行隐式类型转换：当传送给i函数的对象类型与参数类型不匹配时。仅当通过传值方式传递对象或传递常量引用参数时，才会发生这些类型转换。
如何消除：一种是重新设计你的代码，不让发生这种类型转换；二是通过修改软件而不再需要类型转换。
2）函数返回对象时。有时你能以某种方法编写返回对象的函数，以允许你的编译器优化临时对象。这些优化中，最常见和最有效的是返回值优化。 
综上所述，临时对象是有开销的，所以你应该尽可能地去除它们，然而更重要的是训练自己寻找可能建立临时对象的地方。在任何时候只要见到常量引用（reference-to-const） 参数，就存在建立临时对象而绑定在参数上的可能性。在任何时候只要见到函数返回对象， 就会有一个临时对象被建立（以后被释放）。学会寻找这些对象构造，你就能显著地增强透 过编译器表面动作而看到其背后开销的能力。 

## 20. 协助完成返回值优化
从效率的观点来看， 你不应该关心函数返回的对象，你仅仅应该关心对象的开销。你所应该关心的是把你的努力 引导到寻找减少返回对象的开销上来，而不是去消除对象本身。

## 21. 通过重载避免隐式类型转换
不过，必须谨记 80－20 规则，没有必要实现大量的重载函数，除非你
有理由确信程序使用重载函数以后其整体效率会有显著的提高。

```
class UPInt { // unlimited precision 
public: // integers 类 
 UPInt(); 
 UPInt(int value); 
 const UPInt operator+(const UPInt& lhs, const UPInt& rhs); 
 … 
}; 
```
UPInt upi1, upi2;  
UPInt upi3 = upi1 + upi2; //调用 UPInts 的 operator 函数
现在考虑下面这些语句： 
upi3 = upi1 + 10; 
upi3 = 10 + upi2; 
这些语句也能够成功运行，方法是通过建立临时对象把整形数 10 转换为 UPInts。
建立临时对象进行类型转换工作是有开销的。
用函数重载来消除类型转换：
```
const UPInt operator+(const UPInt& lhs, const UPInt& rhs); // add UPInt and UPInt  
const UPInt operator+(const UPInt& lhs, int rhs); // add UPInt and int 
const UPInt operator+(int lhs, const UPInt& rhs); // add int and UPInt 
const UPInt operator+(int lhs, int rhs); // 错误! 
```
但注意，在 C++中有一条规则是每一个重载的 operator 必须带有一个用户定义类型（user-defined type）的参数。

## 22. 考虑用运算符的赋值形式（op=）取代其单独形式（op）
operator的赋值形式（operator+=）比单独形式(operator+)效率更高。因为单独形式要返回一个新对象，从而在临时对象的构造和释放上有一些开销。operator 的赋值形式把结果写到左边的参数里，因此不需要生成临时对象来容纳 operator 的返回值。

## 23. 考虑变更程序库
不同的程序库在效率、可扩展性、移植性、类型安全和其他一些领域上蕴含着不同的设计理念，通过变换使用给予性能更多考虑的程序库，你有时可以大幅度地提高软件的效率。 

## 24. 理解虚拟函数、多继承、虚基类和 RTTI 所需的代价
* virtual table（vtbl） ：通常是一个函数指针数组（一些编译器使用链表来代替数组）。在程序中的每个类只要声明了虚函数或继承了虚函数，它就有自己的 vtbl，并且类中 vtbl 的项目是指向虚函数实现体的指针。
```
class C1 { 
public: 
 C1(); 
 virtual ~C1(); 
 virtual void f1(); 
 virtual int f2(char c) const; 
 virtual void f3(const string& s); 
 void f4() const; 
 ... 
}; 
```
C1 的 virtual table 数组看起来如下图所示：
![](More%20Effective%20C++/A0B05AD0-88E1-46C7-B86C-221E4F23AA33.png)

```
class C2: public C1 { 
public: 
 C2(); // 非虚函数 
 virtual ~C2(); // 重定义函数 
 virtual void f1(); // 重定义函数 
 virtual void f5(char *str); // 新的虚函数 
...
}; 
```
C2的virtual table：
![](More%20Effective%20C++/4E64323C-2F1D-409E-8C08-BF3263BE91A1.png)
如果你有大量的类或者在每个类中有大量的虚函数，你会发现 vtbl 会占用大量的地址空间。 

* virtual table pointer（vptr）：指出每个对象对应的 vtbl。
![](More%20Effective%20C++/43EDE100-4DC3-4C0A-ACD3-4D257282A3E4.png)

在实际运行中，虚函数所需的代价与内联函数有关。实际上虚函数不能是内联的。

* 多继承：
![](More%20Effective%20C++/F9EC9E8B-6AD4-4D1C-8DB7-841541655867.png)
对象D的内存布局：
![](More%20Effective%20C++/708C5C23-0393-4A38-804A-13E62AC16A2F.png)

* RTTI：Runtime Type Identification运行时类型识别。 能让我们在运行时找到对象和类的有关信息。这些信息被存储在类型为 type_info 的对象里，你能通过使用 typeid 操作符访问一个类的 type_info 对象。
￼
* 总结：
![](More%20Effective%20C++/0C2A1691-2B85-4E63-AC2A-67EF9022A952.png)


## 25. 将构造函数和非成员函数虚拟化
虚拟构造函数是指能够根据输入给它的数据的不同而建立不同类型的对象。
虚拟拷贝构造函数：一种特殊种类的虚拟构造函数。虚拟拷贝构造函数能返回一个指针，指向调用该函数的对象的新拷贝。因为这种行为特性，虚拟拷贝构造函数的名字一般都是 copySelf，cloneSelf 或者是clone。
具有虚拟行为的非成员函数很简单。你编写一个虚拟函数来完成工作，然后再写一个非虚拟函数，它什么也不做只是调用这个虚拟函数。为了避免这个句法花招引起函数调用开销，你当然可以内联这个非虚拟函数。
```
class NLComponent { public: 
  virtual ostream& print(ostream& s) const = 0; 
  ...  
};  
class TextBlock: public NLComponent { 
public: 
  virtual ostream& print(ostream& s) const; 
  ...  
};  
class Graphic: public NLComponent { public: 
  virtual ostream& print(ostream& s) const; 
  ...  
};  
inline 
ostream& operator<<(ostream& s, const NLComponent& c) { 
  return c.print(s); 
} 
```

## 26. 限制某个类所能产生的对象数量
```
template<class BeingCounted> class Counted { 
public: 
  class TooManyObjects{};                    // 用来抛出异常    	static int objectCount() { return numObjects; }  
protected:   
	Counted(); 
  Counted(const Counted& rhs);    
	~Counted() { --numObjects; }  
private: 
	static int numObjects; 
  static const size_t maxObjects;  
  void init();            // 避免构造函数的代码重复
};         
template<class BeingCounted> 
Counted<BeingCounted>::Counted() 
{ init(); }  

template<class BeingCounted> 
Counted<BeingCounted>::Counted(const Counted<BeingCounted>&) { init(); }  

template<class BeingCounted> 
void Counted<BeingCounted>::init() { 
  if (numObjects >= maxObjects) throw TooManyObjects();   ++numObjects; 
} 

class Printer: private Counted<Printer> { 
public: 
  // 伪构造函数 
  static Printer * makePrinter(); 
  static Printer * makePrinter(const Printer& rhs);    	
	~Printer();  
  void submitJob(const PrintJob& job); 
  void reset(); 
  void performSelfTest(); 
  ...  
  using Counted<Printer>::objectCount;     // 参见下面解释   	
	using Counted<Printer>::TooManyObjects;  // 参见下面解释  private: 
  Printer(); 
  Printer(const Printer& rhs); 
};
```
Printer 使用了Counter 模板来跟踪存在多少 Printer 对象。
定义 Counted 内的静态成员（在 Counted的实现文件里定义）： 
```
template<class BeingCounted> 
int Counted<BeingCounted>::numObjects; //定义numObjects,自动把它初始化为 0 
const size_t Counted<Printer>::maxObjects = 10; 
```


## 27.要求或禁止在堆中产生对象
*  要求在堆中建立对象
为了执行这种限制，你必须找到一种方法禁止以调用“new”以外的其它手段建立对象。非堆对象（non-heap object）在定义它的地方被自动构造，在生存时间结束时自动被释放，所以只要禁止使用隐式的构造函数和析构函数，就可以实现这种限制。 
1）最直接的方法是把构造函数和析构函数声明为 private，但这样做副作用太大。最好让析构函数成为private，让构造函数成为 public，然后引进一个专用的伪析构函数，用来访问真正的析构函数。客户端调用伪析构函数释放他们建立的对象。
```
class UPNumber { 
public: 
 UPNumber(); 
 UPNumber(int initValue); 
 UPNumber(double initValue); 
 UPNumber(const UPNumber& rhs); 
 // 伪析构函数 (一个 const 成员函数， 因为即使是 const 对象也能被释放。) 
 void destroy() const { delete this; } 
 ... 
private: 
 ~UPNumber(); 
}; 
 
UPNumber n; // 错误! (在这里合法， 但是当它的析构函数被隐式地调用时，就不合法了) 
UPNumber *p = new UPNumber; //正确 
... 
delete p; // 错误! 试图调用private 析构函数 
p->destroy(); // 正确 
```
2)把全部的构造函数都声明为 private。这种方法的缺点是一个类经常有许多构造函数，类的作者必须记住把它们都声明为 private。

* 判断一个对象是否在堆中：
```
class HeapTracked { // 混合类; 跟踪从operator new 返回的ptr 
public: 
 class MissingAddress{}; // 异常类
 virtual ~HeapTracked() = 0; 
 static void *operator new(size_t size); 
 static void operator delete(void *ptr);
 bool isOnHeap() const; 
private:
 typedef const void* RawAddress; 
 static list<RawAddress> addresses;//跟踪从operator new返回的所有指针
}; 

// mandatory definition of static class member 
list<RawAddress> HeapTracked::addresses; 
// HeapTracked 的析构函数是纯虚函数，使得该类变为抽象类。 然而析构函数必须被定义，所以我们做了一个空定义。
HeapTracked::~HeapTracked() {} 
void * HeapTracked::operator new(size_t size) 
{ 
 void *memPtr = ::operator new(size); // 获得内存 
 addresses.push_front(memPtr); // 把地址放到 list 的前端 
 return memPtr; 
} 
void HeapTracked::operator delete(void *ptr) 
{ 
 //得到一个 "iterator"，用来识别 list 元素包含的 ptr
 list<RawAddress>::iterator it = find(addresses.begin(), addresses.end(), ptr); 
 if (it != addresses.end()) { // 如果发现一个元素 
 addresses.erase(it); //则删除该元素 
 ::operator delete(ptr); // 释放内存 
 } else { // 否则 
 throw MissingAddress(); // ptr 就不是用 operator new 
 } // 分配的，所以抛出一个异常 
} 
bool HeapTracked::isOnHeap() const 
{ 
 // 得到一个指针，指向*this 占据的内存空间的起始处，  
 const void *rawAddress = dynamic_cast<const void*>(this); 
 // 在 operator new 返回的地址 list 中查到指针 
 list<RawAddress>::iterator it =  find(addresses.begin(), addresses.end(), rawAddress); 
 return it != addresses.end(); // 返回 it 是否被找到 
} 

//使用：继承HeapTracked，可判断Asset对象指针指向的是否是堆对象：
class Asset: public HeapTracked { 
private: 
 UPNumber value; 
 ... 
}; 
//我们能够这样查询 Assert*指针： 
void inventoryAsset(const Asset *ap) 
{ 
 if (ap->isOnHeap()) { 
 ap is a heap-based asset — inventory it as such;
 } 
 else { 
 ap is a non-heap-based asset — record it tha
 } 
} 
```
像HeapTracked 这样的混合类有一个缺点，它不能用于内建类型，因为象 int 和 char这样的类型不能继承自其他类型。不过使用像HeapTracked 的原因一般都是要判断是否可以调用“delete this”，你不可能在内建类型上调用它，因为内建类型没有 this 指针。 

* 禁止堆对象
通常对象的建立这样三种情况：对象被直接实例化；对象做为派生类的基类被实例化； 对象被嵌入到其它对象内。
1）禁止用户直接实例化对象：利用 new 操作符总是调用 operator new 函数这点，自己声明这个函数，而且可以把它声明为 private。
```
class UPNumber { 
private: 
  static void *operator new(size_t size);   
  static void operator delete(void *ptr); 
  ... 
}; 
 
UPNumber n1; // okay 
static UPNumber n2;  // also okay 
UPNumber *p = new UPNumber; // error! attempt to call private operator new  
```
把 operator new 声明为private，也要把operator delete 一起声明。
如果你也想禁止 UPNumber 堆对象数组，可以把 operator new[]和 operator delete[]也声明为 private。
2）把 operator new 声明为private 经常会阻碍 UPNumber 对象做为一个位于堆中的派生类对象的基类被实例化。因为 operator new 和 operator delete 是自动继承的， 如果 operator new 和 operator delete 没有在派生类中被声明为 public（进行改写， overwrite），它们就会继承基类中 private 的版本。
```
class UPNumber { ... };             // 同上 class NonNegativeUPNumber:public UPNumber { //假设这个类没有声明 operator new 
  ... 
}; 
NonNegativeUPNumber n1;             // 正确 
static NonNegativeUPNumber n2;      // 也正确 
NonNegativeUPNumber *p = new NonNegativeUPNumber;  // 错误! 试图调用private operator new 
```
如果派生类声明它自己的 operator new，当在堆中分配派生对象时，就会调用这个函数， 于是得另找一种不同的方法来防止 UPNumber 基类的分配问题。
3）UPNumber 的 operator new 是 private这一点，不会对包含 UPNumber 成员对象的对象的分配产生任何影响： 
```
class Asset { 
public: 
  Asset(int initValue); 
  ... 
private: 
  UPNumber value; 
}; 
Asset *pa = new Asset(100); // 正确, 调用Asset::operator new或::operator new, 不是UPNumber::operator new 
```

## 28. smart指针

## 29. 引用计数
引用计数是这样一个技巧，它允许多个有相同值的对象共享这个值的实现。
这个技巧有两个常用动机：
1）第一个是简化跟踪堆中的对象的过程。一旦一个对象通过调用new被分配出来，最要紧的就是记录谁拥有这个对象；
2）第二个动机是由于一个简单的常识。如果很多对象有相同的值，将这个值存储多次是很无聊的。更好的办法是让所有的对象共享这个值的实现。这么做不但节省内存，而且可以使得程序运行更快，因为不需要构造和析构这个值的拷贝。 

## 30. 代理类
Proxy 类可以完成一些其它方法很难甚至不可能实现的行为。多维数组是一个例子，左 /右值的区分是第二个，限制隐式类型转换（见 Item M5）是第三个。 
同时，proxy类也有缺点。作为函数返回值，proxy 对象是临时对象（见 Item 19），它们必须被构造和析构。这不是免费的，虽然此付出能从具备了区分读写的能力上得到更多的 补偿。Proxy对象的存在增加了软件的复杂度，因为额外增加的类使得事情更难设计、实现、 理解和维护。 
最后，从一个处理实际对象的类改换到处理 proxy 对象的类经常改变了类的语义，因 为 proxy 对象通常表现出的行为与实际对象有些微妙的区别。有时，这使得在设计系统时无 法选择使用 proxy 对象，但很多情况下很少有操作需要将 proxy 对象暴露给用户。例如，很 少有用户取上面的二维数组例子中的 Array1D对象的地址，也不怎么有可能将下标索引的对 象（见 Item M5）作参数传给一个期望其它类型的函数。在很多情况下，proxy 对象可以完 美替代实际对象。当它们可以工作时，通常也是没有其它方法可采用的情况。

## 31. 让函数根据一个以上的对象来决定怎么虚拟

## 32. 在未来时态下开发程序

## 33. 将非尾端类设计为抽象类

## 34. 如何在同一程序中混合使用C++和C

## 35. 让自己习惯使用标准C++语言

