# 深度探索C++对象模型
- - - -
第一章
## C++对象模型
![](%E6%B7%B1%E5%BA%A6%E6%8E%A2%E7%B4%A2C++%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B/%E6%88%AA%E5%B1%8F2021-09-04%20%E4%B8%8B%E5%8D%886.35.50.png)
* Nonstatic data members：配置于每一个class object之内；
* static data members：存放在个别的class object之外；›
* static/nonstatic function members：存放在个别的class object之外；
* virtual functions：
1）每一个class产生出一堆指向virtual functions的指针，放在virtual table（vtbl）表格之中。
2）每一个class object被安插一个指针（vptr），指向相关的virtual table。每一个class所关联的type_info object（用以支持RTTI）通常放于virtual table的第一个slot。

### 继承：
C++支持单一继承和多重继承。
继承关系也可以指定为虚拟（virtual）。
在虚拟继承的情况下，base class不管在继承串链中被派生多少次，永远只会存在一个实例（subobject）。
举例：
```
class ios;
class istream : virtual public ios {...};
class ostream : virtual public ios {...};
class iostream : public istream, public ostream {...};
```
上述iostream之中就只有virtual ios base class的一个实例。
![](%E6%B7%B1%E5%BA%A6%E6%8E%A2%E7%B4%A2C++%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B/AEBB7182-8F93-421B-B19B-56257DD278EC.png)

## struct在C++中的合理用途
C struct 在C++中的一个合理用途，是当你要传递“一个复杂的class object的全部或部分”到某个C函数去时，struct声明可以将数据封装起来，并保证拥有与C兼容的空间布局。然而这项保证只在组合（composition）的情况下才存在。如果是“继承”而不是“组合”，编译器会决定是否应该有额外的data members被安插到base struct subobject之中。

## C++程序设计模型支持的三种程序设计范式（programming paradigms）
* 程序模型（procedural model）
* 抽象数据类型模型（abstract data type model，ADT）
* 面向对象模型（object-oriented model）：在此模型中有一些彼此相关的模型，通过一个抽象的base class被封装起来。

## C++多态
多态的主要用途是经由一个共同的接口来影响类型的封装，这个接口通常被定义在一个抽象的base class中。
需要付出的代价就是额外的间接性：不论在“内存的获得”或是在“类型的决断”上。

C++以下列方法支持多态：
* 经由一组隐式的转化操作。
例如把一个derived class指针转化为一个指向其public base type的指针。
* 经由virtual function机制
* 经由dynamic_case和type_id运算符

C++通过class 的pointer和reference来支持多态，这种程序设计风格就称为“面向对象”。

- - - -
第二章
## ARM
Annotated Reference Manual，ARM

## default constructor
trivial default constructor：对于class X，如果没有任何user-declared constructor， 那么会有一个default constructor被隐式（implicitly）声明出来。一个被隐式声明出来的default constructor将是一个trivial（没啥用的）constructor。
nontrivial default constructor：编译器所需要的那种，必要的话会由编译器合成出来。

### nontrivial default constructor的4种情况：
* “带有default constructor”的member class object：
如果一个class没有任何constructor，但它内含一个member object，而后者有default constructor，那么这个class的implicit default constructor就是“nontrivial”，编译器需要为该class合成出一个default constructor，不过这个合成操作只有在constructor真正需要被调用时才会发生。
```
class Foo { public: Foo(), Foo(int) ... };
class Bar { public: Foo foo; char *str; };
void foo_bar()
{
	Bar bar;
	if (str) { } ...
}
//被合成的Bar default constructor内含必要的代码，能够调用class Foo的default constructor来处理member object Bar::foo，但它并不产生任何代码来初始化Bar::str。

//下面是被合成的default constructor可能样子：
inline Bar::Bar() { foo.Foo::Foo(); }
```
如果class A内含一个或一个以上的member class objects，那么class A的每一个constructor必须调用每一个member classes的default constructor。编译器会扩张已存在的constructors，在其中安插一些代码，使得user code被执行之前，先调用必要的default constructors。
```
//比如上述Bar含程序员定义的default constructor：
Bar::Bar() { str = 0; }
//下面是编译器扩张后的fefault constructor：
Bar::Bar() { foo.Foo::Foo(); str = 0; }
```
如果有多个class member objects都要求constructor初始化操作，C++语言要求以“member objects在class中的声明顺序”来调用各个constructors。

* “带有default constructor”的base class：
如果一个没有任何constructors的class派生自一个“带有default constructor”的base class，那么这个derived class的default constructor会被视为nontrivial，并因此需要被合成出来。它将调用上一层base classes的default constructor（根据它们的声明顺序）。
如果设计者提供多个constructors，但其中都没有default constructor，那么编译器会扩张现有的每一个constructors，将“用以调用所有必要之default constructors”的程序代码加进去。

* “带有一个virtual function“的class：
下面两种情况也需要合成出default constructor：
1）class声明（或继承）一个virtual function
2）class 派生自一个继承串链，其中有一个或更多的virtual base classes。
对于那些未声明任何constructors的classes，编译器会为它们合成一个default constructor，以便正确地初始化每一个class object的vptr。

* “带有一个virtual base class“的class
对于class所定义的每一个constructor，编译器会安插那些“允许每一个virtual base class的执行期存取操作”的代码。如果class没有声明任何constructors，编译器必须为它合成一个default constructor。

对于上述4种情况之外而又没有声明任何constructor的classes，我们说它们拥有的是implicit trivial default constructors，它们实际上并不会被合成出来。

## copy constructor的构造操作
### default memberwise initialization：
当class objecs以相同class的另一个object作为初值，而class没有提供一个explicit copy constructor时，其内部是以所谓的default memberwise initialization手法完成的。也就是把每一个内建的或派生的data member的值，从某个object拷贝一份到另一个object身上，不过它并不会拷贝其中的member class object，而是以递归的方式施行memberwise initialization。

### nontrivial copy constructor
如果class没有声明一个copy constructor，就会有隐式的声明（implicitly declared）或隐式的定义（implicitly defines）出现。只有nontrivial的实例才会被合成于程序之中。
决定一个copy constructor是否为trivial的标准在于class是否展现出所谓的“bitwise copy semantics”。

### bitwise copy semantics位逐次拷贝
如果class中展现了位逐次拷贝，编译器就不会产生出copy constructor。当没有产生copy constructor时，类的产生通过bitwise copy搞定，也就是将源类中成员变量中的每一位都逐次拷贝到目标类中。

### 类不展现出“bitwise copy semantics”的情况：
（此时需要生成copy constructor）
* 当class内含一个member object而后者的class声明有一个copy constructor时（不论是被class设计者显式地声明，还是被编译器合成）
* 当class继承自一个base class而后者存在一个copy constructor时（不论被显式声明还是被合成）
* 当class声明了一个或多个virtual functions时
* 当class派生自一个继承串链，其中有一个或多个virtual base classes时。

### NRV
Named Return Value， NRV。
编译器的一种优化操作，称为NRV优化。当函数中，return指令传回相同的具名数值（named value），以result参数取代named return value。
```
X bar()
{
	X xx;
	//......处理xx
	return xx;
}
//编译器取代为：
void bar(X &_result)
{
	//default constructor被调用
	_result.X::X();

	//......直接处理_result
	return;
}
```

## member initialization list成员初始化列表
### 必须使用member initialization list成员初始化列表的情况：
* 当初始化一个reference member时
* 当初始化一个const member时
* 当调用一个base class的constructor，而它拥有一组参数时
* 当调用一个member class的constructor，而它拥有一组参数时

### 成员初始化列表的顺序
list中初始化的顺序是由class中的members声明顺序决定的，不是由initialization list中的排列顺序决定的。因此编译器会对初始化列表一一处理并有可能重新排序，以反映出members的声明顺序。它会安插一些代码到constructors体内，并置于任何explicit user code之前。

注意一点, 用成员函数的返回值来作为初始化列表的参数语法上是没有问题的, 但是需要保证这个成员函数不依赖于成员的数据对象, 因为很可能这个在调用此函数时还没有初始化其依赖的数据成员, 这就会引起难以发现的错误. 另外, 最好不要将其用于初始化基类成员。

- - - -
第三章
## 类大小
### 不含数据成员的类对象
对于不存在继承和虚函数的类, 没有数据成员时, 其大小至少是1 byte, 以保证变量有唯一的地址。
当加上虚函数后, 由于有虚函数指针, 对象大小等于一个指针的大小, 32位系统中是4 bytes, 64位系统中是8 bytes。
```
struct Empty {};
struct VirtualEmpty
{
    virtual void f() {}
};

Empty a;
Empty b;

cout<<sizeof(Empty)<<endl; // 输出为1
cout<<sizeof(VirtualEmpty)<<endl; // 输出为8
```
当其作为基类时, 在某些情况下则不必遵循上面的要求, 可以在子类中将其优化掉, 节省所占空间:
```
struct Base {};
struct Derived : Base
{
    int64_t i;
};

cout<<sizeof(Base)<<endl; // 输出为1
cout<<sizeof(Derived)<<endl // 输出为8
```
但是即使是继承, 也有不能进行优化的情况:
1)子类的第一个非静态数据成员的类型和空基类相同.
2)子类的第一个非静态数据成员的基类类型和空基类相同.
（因这两种情况下, 会有两个空基类对象(父类对象和子类数据成员对象)连续出现, 如果优化掉, 将不能区别二者）：
```
struct Base {};
struct Derived1 : Base // 情况一
{
    Base b;
    int64_t i;
}d1;
struct Derived2
{
    Base b;
};
struct Derived3 : Base
{
    Derived2 d2;
    int64_t i;
}d3;
cout<<sizeof(Derived1)<<endl; // 输出为16, 基类对象和成员b各占1 byte, 由于内存对齐补齐8 bytes
cout<<sizeof(Derived2)<<endl; // 输出为1
cout<<sizeof(Derived3)<<endl; // 输出为16, 基类对象和成员d2各占1 byte, 由于内存对齐补齐8 bytes

cout<<&d1<<' '<<&d1.b<<endl; // 前者(基类对象地址)比后者小1
cout<<&d3<<' '<<&d3.d2.b<<endl; // 前者(基类对象地址)比后者小1
```
对于空类作为虚基类的情况，同样可以进行优化：
```
struct Base {};
struct Derived1 : virtual Base {};
struct Derived2 : virtual Base {};
struct Derived3 : Derived1, Derived2 {};
struct Derived4 : Derived1, Derived2
{
    Base b;
}d4;

cout<<sizeof(Derived3)<<endl; // 输出为16.为了实现虚继承, 类Derived1和Derived2包含一个指针. 而虚基类Base被优化掉了, 因此Derived3大小为16 bytes
cout<<sizeof(Derived4)<<endl; // 输出为24.由于包含类型是Base的非静态成员, 需要占据8 bytes, 即Derived4大小为24 bytes. 

cout<<&d4<<endl; // 输出为0x55c6986ffe70
cout<<dynamic_cast<Base*>(&d4)<<endl; // 输出为0x55c6986ffe70
cout<<&(d4->b)<<endl; // 输出为0x55c6986ffe80
```

## data member的布局
nonstatic data members非静态数据成员在class object中的排列顺序将和被声明的顺序一样，任何中间介入的static data members都不会被放进对象布局之中。
而对于普通类的静态数据成员, 则具有独立于对象的静态生存期, 保存在全局数据段中。
模板类的静态数据成员如果没有被显式特化或实例化, 则在使用时会被隐式特化, 只有当特化/实例化后才是有效定义的（C++14引入的 variable template变量模板）：
```
struct Test1
{
    template<typename T> static T val; // 非模板类的模板静态成员.
};
template<typename T> T Test1::val = 0;

template<typename T>
struct Test2
{
    static T val; // 模板类的非模板静态成员.
};
template<typename T> T Test2<T>::val = 0;

template<typename T1>
struct Test3
{
    template<typename T2> static std::pair<T1, T2> val; // 模板类的模板静态成员.
};
template<typename T1>
template<typename T2>
std::pair<T1, T2> Test2<T1>::val = std::make_pair(T1(1), T2(2));
auto var = Test3<int>::val<float>; // 即pair<int, float>(1, 2)
```

## 数据成员的存取
### 静态数据成员
对静态成员, 通过对象或对象指针访问和通过类名访问没有区别, 编译器一般会将二者统一为相同形式。类成员指针不能指向静态成员, 因为对静态成员取地址得到的是一个该成员的指针：
```
class A
{
public:
    static int x;
};
&A::x; // 其类型是 int*
```
因为类静态成员都是保存在全局数据段中, 如果不同类具有相同名字的静态成员, 就需要保证不会发生名称冲突. 编译器的解决方法是对每个静态数据成员编码(这种操作称为name-mangling), 以得到一个独一无二的名称.

### 非静态数据成员
不存在虚基类时, 通过对象名或对象指针访问非静态数据成员没有区别。
存在虚基类时, 通过对象指针访问非静态数据成员需要在运行时才能确定, 因为无法确定指针所指对象的实际类型, 也就不能判断对象的内存布局, 也就不知道对象中该数据成员的偏移。
普通继承和虚继承的这个区别的原因在于, 普通继承的类对象的内存布局在编译时就可以决定, 而存在虚继承时则需要在运行时决定。
```
Point3d origin, *pt = &origin;
origin.x = 0.0;
pt->x = 0.0;
```
上述代码，通过origin和pt存取，有什么重大差异吗？
当Point3d是一个derived class，而其继承结构中有一个virtual base class，并且被存取的member（即x）是一个从该virtual base class继承而来的member时，就有重大差异。此时不能说pt必然指向哪一种class type，这个存取操作必须延迟到执行期，经由一个额外的间接导引才能解决。使用origin，其类型无疑是Point3d class，member的offset在编译时期就固定了。、

## 继承对对象布局的影响
### 单继承
单继承不会修改父类的内存布局, 例如父类由于内存对齐产生的额外空间在子类中不会被消除, 而是保持原样：
```
struct Base // 16 bytes
{
    int64_t i1;
    char c1;
};
struct Derived : Base // 24 bytes
{
    char c2;
};
```
上述代码中, 子类大小是24 bytes, 而不是16 bytes。
其原因是如果消除了这些额外空间, 将子类对象赋值给父类对象时就可能会在父类对象的额外空间位置赋值, 这改变了程序的语义, 显然是不合适的。

### 加上多态
为了支持动态绑定, 编译器需要在对象中添加虚表指针(vptr), 指向虚表。
 虚表中包含类的类型信息和虚函数指针, 值得注意的是, vptr并不是指向虚表的起始地址, 很多时候该地址之前会保存着对象的类型信息, 程序通过此类型信息实现RTTI。 而vptr初值的设置和其所占空间的回收, 则分别由构造函数和析构函数负责, 编译器自动在其中插入相应代码. 这是多态带来的空间负担和时间负担。
那么vptr放在什么位置呢? 这是由编译器决定的, gcc将其放在对象头部, 这导致对象不能兼容C语言中的struct, 但是在多重继承中, 通过类成员指针访问虚函数会更容易实现.。如果放在对象末尾则可以保证兼容性, 但是就需要在执行期间获得各个vptr在对象中的偏移, 在多重继承中尤其会增加额外负担。

### 多重继承
标准并没有规定不同基类在布局中的顺序, 但是大多数实现按照继承声明顺序安排. 多重继承给程序带来了这些负担:
* 将子类地址赋值给基类指针变量时, 如果是声明中的第一个基类, 二者地址相等, 可以直接赋值. 否则, 需要加上一个偏移量, 已获得对应对象的地址。
* 上面的直接加偏移并不能保证正确性, 设想子类指针值为0, 直接加上偏移后指向的是一个内容未知的地址. 正确做法应该是将0值赋给基类指针变量. 因此, 需要先判断基类指针是否为0, 再做处理. 而对于引用, 虽然其底层是指针, 但是不需要检查是否为0, 因为引用必须要绑定到一个有效地址, 不可能为0。

### 虚拟继承
主要问题是如何实现只有一个虚拟基类.。
Class如果内含一个或多个virtual base class subobject，将被分割为两部分：一个不变区域和一个共享区域。不变区域中的数据，不管后继如何衍化，总是拥有固定的offset，这部分数据可以被直接存取。共享区域所表现的就是virtual base class subobject，这部分的数据，其位置会因为每次的派生操作而有所变化，只能被间接存取。
主流方案是将虚拟基类作为共享部分, 其他类通过指针等方式指向虚拟基类, 访问时需要通过指针或其他方式获得虚拟基类的地址。

- - - -
第四章
## 成员函数的调用方式
* 普通非静态成员函数
> C++的设计准则之一就是: nonstatic member function至少必须和一般的nonmember funciton有相同的效率。  

为了保证类成员函数的效率, 编译器将对普通非静态成员函数的调用转换为对普通函数的调用。步骤如下:
1. 修改函数签名, 添加一个额外的参数(作为第一个参数), 称为this指针. 由此将函数和对象关联起来.
2. 将函数中对非静态成员的访问改为经过this指针访问.
3. 将成员函数重写为一个外部函数, 生成一个独一无二的名字(name mangling).

* 虚成员函数
编译器将对虚成员函数的调用转化为通过vptr调用函数。
 在虚继承体系下, 任何含有某一虚函数的类, 该函数在虚表中的偏移都是固定的, 因此编译器可以根据函数名在编译期确定函数指针在虚表中的下标。 所以, 虚函数带来的额外负担就是增加一个内存访问：
```
p->func(param); // 设其在虚表中的下标为index.
// 上面的语句将被转化为
(*(p->vptr)[index])(p, param) // 这里p等于this指针, 所以将其作为第一个参数.
```

* 静态成员函数
对静态成员函数的访问将被转化为对普通函数的访问, 由于静态成员不能访问非静态数据成员, 因此不需要添加this指针。
 静态函数有下面几个特点:
1. 不能直接访问类对象的非静态成员.
2. 不能被声明为const, volatile, virtual.
3. 可以通过类对象和类名来调用.

注意一点, 当通过类对象来调用静态成员函数, 并且这个对象是由一个表达式得到时, 虽然不需要执行表达式就能直接调用函数, 但是表达式仍然会被执行(evaluate), 因为此表达式可能会有副作用, 不能被忽略. 例如:
```
Object func();
func().static_func() // func()仍然会被先执行, func()中可能会有某些不可省略的操作.
```

## 虚拟成员函数的实现
### 单继承
每个类都只有一个虚表(多继承和虚继承的类对象有多个vtpr, 指向不同的虚表, 但是实际上这些虚表是一个, vptr只是指向虚表的不同偏移位置), 也就是说相同类型的对象的vptr值是相同的。
当单继承发生时, 子类不仅继承了父类的数据成员, 还继承了函数成员, 前者体现在类对象布局上, 而后者体现在虚表上。
虚表继承的步骤可能包含下面几步：
1. 将父类虚表中的虚函数指针拷贝到子类虚表的相同下标位置.
2. 如果子类重写了父类的虚函数, 就将被重写的虚函数的指针修改为对应函数的地址.
3. 如果子类加入新的虚函数, 就增加虚表容量, 在后面添加新的函数指针.
从上面可以看到, 单继承下的虚函数效率高, 实现简单, 但是多继承和虚拟继承则要复杂很多

### 多继承
多继承的复杂性在于下面几个问题：
* 通过第2,3,…个父类的指针访问子类的虚函数.
* 通过子类指针访问第2,3,…个父类的虚函数.
* 重写的虚函数的返回类型可能和父类的被重写函数的返回类型不一样, 这是标准允许的.
> 相关概念：  
> 1）明确虚函数重写的概念：  
> 父类声明了一个虚函数, 如果其(直接或间接)子类定义了函数, 与父类虚函数具有相同的：（名字、参数类型列表(不包含返回值)、 const/volatile类型、引用类型(三种: 无引用符号, &, &&)），则子类函数为虚函数(无论是否声明为virtual), 并且重写了父类的虚函数；  
> 2） 多继承时, 我们通过子类指针可以访问所有父类的函数, 但是不能通过一个父类的指针访问其他父类的函数。  

通过父类指针直接调用子类定义的函数时有两种情况:
1）通过第一个基类指针访问时, 直接将指针值作为this指针值传给函数.
2）通过第2,3,…个基类指针访问时, 需要调整指针值, 加上/减去一个偏移, 再作为this指针传给函数。由于编译时无法确定指针所指对象的实际类型，需要在运行时调整this指针的值。
3）除此之外, 再考虑一种特殊情况(间接调用子类虚函数)：对一个父类指针调用delete。如果析构函数被声明为virtual, 那么程序将根据指针所指对象的实际类型决定调用哪个析构函数. 这就需要在运行时需要调整指针的值, 以保证能够访问正确的vptr, 从而获得对应的析构函数。

问题的复杂性在于需要在运行时根据指针所指对象的实际类型来调整指针的值, 使之指向子类对象。
比较有效率的解决方法：利用所谓的thunk。
thunk是一小段assembly代码，用来（1）以适当的offset值调整this指针，（2）跳到virtual function去。
thunk技术允许virtual table slot继续内含一个简单的指针，因此多重继承不需要任何空间上的额外负担。slots中的地址可以直接指向virtual function，也可以指向一个相关的thunk（如果需要调整this指针的话）。
Microsoft以所谓的“address points”来取代thunk策略。即将用来改写别人的那个函数（也就是overriding function）期待获得的是“引入该virtual function之class”（而非derived class）的地址。这就是该函数的“address point”。 

### 虚继承
其复杂性同样在于指针值的运行时修改, 书中建议不要在虚基类中声明非静态的函数。

## 指向成员函数的指针
取一个nonstatic member function的地址，如果该函数是nonvirtual，得到的结果是它在内存中真正的地址。它需要被绑定于某个class object的地址上。
使用一个“member function指针”，如果并不用于virtual function、多重继承、virtual base class等情况的话，并不会比使用一个“nonmember function指针”的成本更高。

### 指向virtual成员函数的指针
成员函数指针可以指向一个普通函数,（此时它可以是函数地址） ，也可以指向一个虚函数,（它可以是该函数在虚表中的偏移）。
这两种值可以保存在相同类型的变量中, 如何区分呢? 
早期C++限制最多有128个虚函数(应该是限制虚表长度为128吧), 所以偏移值最大为127. 而程序空间起始地址必定大于127, 因此可以通过将指针值和127做”&”(按位与)运算来判断是偏移还是函数地址：
```
(((int)pmf) & ~127) ? (*pmf)(ptr) : (*ptr->vptr[(int)pmf])(ptr);
```

### 多继承和虚继承
为了让指向成员函数的指针也能够支持多重继承和虚拟继承，Stroustrup设计了下面一个结构体：
```
struct __mptr {
	int delta;
	int index;
	union {
		ptrtofunc faddr;
		int       v_offset;
	};
};
```
index和faddr分别（不同时）持有virtual table索引和nonvirtual member function地址（index不指向virtual table时会设为-1）。
delta字段表示this指针的offset值，而v_offset字段放的是一个virtual（或多重继承中的第二或后继的） base class的vptr位置。
```
//下面调用：
( ptr->*pmf )();
//会变成：
( pmf.index < 0 ) ? ( *pmf.faddr )( ptr ) : ( * ptr->vptr[ pmf.index] (ptr) );
```
因每一个调用操作都得付出上述成本，检查其是否为virtual 或nonvirtual。
Microsoft把这项检查拿掉，导入一个它所谓的vcall thunk。在此策略下，faddr被指定的腰部就是真正的member function地址，要不就是vcall thunk的地址。vcall thunk会选出并调用相关virtual table中的适当slot。

### inline函数
在下面的情况下, 一个函数是inline函数：
* 声明中包含inline关键字的函数
* 当一个函数(成员函数或非成员友元函数)的定义在类内部时
* 被声明为constexpr的函数(since C++11)

inline函数只是一种建议, 建议编译器将对inline函数的调用转换, 但是编译器并不一定会接受该建议, 而且非inline函数也有可能被转换, 这依赖于具体实现。
使用inline函数时要注意下面几点：
* inline函数可能会增加生成的文件的大小.
* inline函数尽可能简单. 减少不必要的局部变量, 否则可能会在结果中产生大量的局部变量

- - - -
第五章
## 纯虚函数
当纯虚函数需要被调用时, 可以提供其定义, 但是定义必须放在类外。可以通过类名加域限定符调用纯虚函数。
唯一的例外是pure virtual destructor：class设计者一定得定义它。因为每一个derived class destructor会被编译器加以扩张，以静态调用的方式调用其“每一个virtual base class”以及“上一层base class”的destructor。只要缺乏任何一个base class destructors的定义就会导致链接失败。

抽象类有下面的性质:
* 抽象类用来表示一般的概念, 不能被实例化, 即不能创建抽象类的对象, 但是可以作为基类被实例化.
* 不能作为参数类型, 函数返回值类型, 显式转化的目的类型, 但是可以声明抽象类的指针和引用.
* 抽象类可以继承自非抽象类, 可以用纯虚函数重写非纯的虚函数.
* 抽象类可以定义构造函数和析构函数, 对象构造和析构时会被调用(只会被调用一次).
* 在抽象类的构造函数和析构函数中可以调用其他成员函数, 但是在其中调用纯虚函数的行为是未定义的.
* 如果析构函数被声明为纯虚函数, 必须提供其定义.(构造函数不能是虚函数, 所以不必讨论)

### 对象构造
> 从C++20开始, *POD*这一概念就被废止, 取而代之的是更为精确的定义, 如*TrivialType*.  
对于POD(plain old data)类型, 定义一个对象时编译器不会调用其构造函数, 复制时也不会调用复制构造函数, 而是像C语言那样的按位复制。

在初始初始化列表中初始化成员比在构造函数函数体内对成员赋值效率更高. 如果函数体内部是简单地对每个成员指定一个常量, 那么编译器可能会进行优化, 将常量抽取出来对成员初始化, 结果就好像成员初始化列表一样。

构造函数按顺序执行下面这些操作:
1. 如果当前对象是继承体系的最底层, 就初始化虚基类
2. 初始化直接基类
3. 初始化vptr
4. 进行初始化列表对数据成员的初始化操作
5. 如果有成员没有出现在初始化列表, 但是有默认构造函数, 调用之.
6. 执行构造函数函数体.

### 在构造函数中调用成员函数
可以在构造函数初始化列表中调用成员函数, 但是如果调用函数时存在直接基类没有被初始化, 行为就是未定义的.
如果调用的是虚函数, 并且调用时基类已初始化, 那么调用时的实际类型就是函数调用点所在构造函数的类类型. 这是很容易理解的, 对象类型绝不会沿着继承体系向下, 因为最底层的对象还没有完成构造. 如果是纯虚函数, 如上文所说, 是UB. 在析构函数中调用虚函数同理.
如果从编译器的角度来解释上面的两条规则, 就需要考虑vptr的初始化. 我们知道, 编译器在构造函数中插入代码来初始化对象的vptr. 但是具体这段代码放在什么位置呢? 如上文所述, 是放在调用基类构造函数之后, 成员初始化语句之前. 所以, 在构造函数的数据成员初始化语句和函数体内调用成员函数时, 对象vptr刚刚被设置为构造函数所属类对应的vptr. 那么, 在调用虚函数时结果就如上文所说。

### 对象复制
当将一个对象赋值给另一个对象时, 有下面三种选择:
* 采用默认行为, 即不提供复制赋值运算符或使用默认复制赋值运算符.
* 显式定义复制赋值运算符,
* 拒绝赋值行为.
对于第三点, C++11之前需要将*operator =*声明为private, 并且不提供其定义. 而C++11之后, 可以用下面的语句实现:
```
ClassName& ClassName::operator =(const ClassName&) = delete;
```
另外C++11提供的一个语法是可以将其显式声明为default, 虽然用户显式声明之, 但是定义是由编译器隐式生成的:
```
ClassName& ClassName::operator =(const ClassName&) = default;
```

当不需要拒绝赋值时, 就需要考虑是不是显式提供一个*operator =*. 一个原则是:**只有在默认复制赋值运算符的行为不安全或不正确时, 才需要显式定义复制赋值运算符**.

### copy assignment operator不表现bitwise copy语意的情况：
* 当class内含一个member object，而其class有一个copy assignment operator时
* 当一个class的base class有一个copy assignment operator时
* 当一个class声明了任何virtual functions时（我们一定不要拷贝右端class object的vptr地址，因为它可能是一个derived class object）
* 当class继承自一个virtual base class时（不论此base class有没有copy operator）

### Trivial copy assignment operator
当复制赋值运算符满足下面的条件是, 它就是tirivial的：
* 不是用户提供的(隐式定义的或声明为default).
* 类没有虚函数.
* 类没有虚基类.
* 直接基类的复制赋值运算符都是trivial的.
* 非静态成员的复制赋值运算符是tirvial的.
满足这个条件的对象的赋值行为是bitwise的, 就如同调用std::memmove一样. 所有与C语言兼容的数据类型都满足此条件. 不满足上面的的条件时, 就采用member-wise复制赋值行为。

另一个问题是存在虚基类时复制赋值运算符可能会多次对基类子对象调用operator =。C++并没有提供类似复制构造函数的语法来保证虚基类只会被复制一次. 所以, 书中建议将虚基类的复制赋值运算符声明为delete, 甚至不要再虚基类中声明数据成员。

### 对象析构
并不是定义了构造函数就需要定义析构函数, 这种”对称”是无意义的。
只有当需要一个析构函数时, 我们才应该显式定义之. 那么什么时候需要呢? 
首先要搞清楚析构函数的作用, 它是对象的生命周期的终结, 而函数体内执行的主要是是对对象持有的资源的释放, 例如在构造函数中动态申请的空间. 析构函数的操作与构造函数类似, 但是顺序相反.

### Trivial destructor
类T的析构函数如果满足下面的条件, 就是trivial的:
* 析构函数不是用户定义的.(隐式声明或声明为default)
* 析构函数非虚.(这就要求父类的虚函数也非虚)
* 直接父类的析构函数是trivial的.
* 非静态数据成员(数组的数据成员)的析构函数是trivial的.
trivial析构函数不进行任何操作, 析构时只需要释放对象的空间即可. 所有与C语言兼容的数据类型都是*trivial destructible*的.

### 一个由程序员定义的destructor被扩展方式：
1. destructor的函数本体首先被执行
2. 如果class拥有member class objects，而后者拥有destructors，那么它们会以其声明顺序的相反顺序被调用
3. 如果object内含一个vptr，现在被重新设定，指向适当之base class的virtual table
4. 如果有任何直接的（上一层）nonvirtual base classes拥有destructor，它们会以其声明顺序的相反顺序被调用
5. 如果有任何virtual base classes拥有destructor，而目前讨论的这个class是最尾端（most-derived）的class，那么它们会以其原来的构造顺序的相反顺序被调用

- - - -
第六章
## 对象的构造与析构
### 全局对象
C++程序中所有的global objects都被放置在程序的data segment中，如果显式地指定给它一个值，此object将以之为初值，否则内存内容为0。
书中建议不要用那些需要被静态初始化的global objects（因那些objects不能被放置于try区域之内防止throw出发terminate()函数，及控制需要跨越模块做静态初始化之objects的相依顺序的复杂度等原因）。

### 局部静态变量
局部静态变量的constructor必须只能施行一次（需要时才被构建），destructor必须只能施行一次。

## new运算符的实现
```
extern void* operator new(size_t size)
{
	if (size == 0)
		size = 1;

	void *last_alloc;
	while( !(last_alloc = malloc(size) )) {
		if (_new_handler)
			(*_new_handler)();
		else
			return 0;
	}
	return last_alloc;
}
```
上述实现的两个有趣之处：
1. 传回的指针，指向一个默认为1-byte的内存区块
2. 它允许使用者提供一个属于自己的_new_handler(0函数

### 对象数组
构建内含virtual base class的数组：
```
void* vec_new(void *array, size_t elem_size, int elem_count, void (*constructor)(void*), void (*destructor)(void*,  char));
void* vec_delete(void *array, size_t elem_size, int elem_count, void (*destructor)(void*,  char));
```

### 针对数组的new
避免以一个base class指针指向一个derived class objects所组成的数组（如果derived class object比其base大的话）。
程序员必须迭代走过整个数组，把delete运算符实施于每一个元素身上（virtual）

### placement new operator
```
void* operator new(size_t, void* p)
{
	return p;
}
Point2w *ptw = new( arena ) Point2w;
```
placement new operator所扩充的另一半是将Point2w constructor自动实施于arena所指的地址上：
```
Point2w *ptw = (Point2w*) arena;
if (ptw != 0)
	ptw->Point2w::Point2w();
```
决定了object被放置在哪里，编译系统保证object的constructor会施行于其上。
一般而言，placement new operator不支持多态。被交给new的指针，应该适当地指向一块预先配置好的内存。

### 临时对象
临时性对象的被摧毁，应该是对完整表达式求值过程中的最后一个步骤。该完整表达式造成临时对象的产生。
凡持有表达式执行结果的临时性对象，应该存留到object的初始化操作完成为止。
如果一个临时性对象被绑定于一个reference，对象将残留，直到被初始化之reference的生命结束，或知道临时对象的生命范畴（scope）结束—视哪一种情况先到达而定。

- - - -
第七章
## template
目前的编译器，面对一个template声明，在它被一组实际参数实例化之前，只能施行以有限的错误检查。
nonmember和member template functions在实例化行为发生之前也一样没有做到完全的类型检验。

##  template中的名称决议法：
* scope of the template definition：定义出template的程序端
* scope of the template instantiation：实例化template的程序端

template之中，对于一个nonmember name的决议结果，是根据这个name的使用是否与“用以实例化该template的参数类型”有关而决定的。如果其使用互不相关，那么就以“scope of the template definition”来决定name；如果其使用互有关联，那么就以“scope of the template instantiation”来决定name。

* scope of the template definition：用以专注于一般的template class
* scope of the template instantiation：用以专注于特定的实例
编译器的决议（resolution）算法必须决定哪一个才是适当的scope，然后在其中搜寻适当的name。

## template中member function的实例化行为
对于template的支持，最困难的莫过于template function的实例化。
目前编译器提供了两个策略：一个是编译时期策略，程序代码必须在program text file中备妥可用；另一个是链接时期策略，有一些meta-compilation工具可以导引编译器的实例化行为。
### 编译器如何找出函数的定义：
* 一种方法是包含template program text file，就好像它是一个header文件一样；
* 另一种方法是要求一个文件命名规则
### 编译器如何能够只实例化程序中用到的member functions：
* 一种方法是，根本忽略这项要求，把一个已经实例化的class的所有member functions都产生出来；
* 另一种策略就是模拟链接操作，检测看看哪一个函数真正需要，然后只为它（们）产生实例。
### 编译器如何阻止member definitions在多个.o文件中都被实例化：
* 一种方法就是产生多个实例，然后从链接器中提供支持，只留下其中一个，其余都忽略
* 另一种办法就是由使用者来导引“模拟链接阶段”的实例化策略，决定哪些实例才是所需求的。

目前，不论是编译时期还是链接时期的实例化策略，均存在以下弱点：当template实例被产生出来时，有时候会大量增加编译时间。

![](%E6%B7%B1%E5%BA%A6%E6%8E%A2%E7%B4%A2C++%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B/B45776B9-E1F8-4C10-8269-523DF947D0CE.png)
![](%E6%B7%B1%E5%BA%A6%E6%8E%A2%E7%B4%A2C++%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B/C4E1600A-70AB-467F-BCCB-9CED8B3415B1.png)

## 异常处理
欲支持exception handling，编译器的主要工作就是找出catch子句，以处理被抛出的exception。
需要条件：
* 追踪程序堆栈中的每一个函数的目前作用区域（包括追踪函数中local class objects当时的情况）。
* 编译器必须提供某种查询exception objects的方法，以知道其实际类型。
* 还需要某种机制以管理被抛出的object，包括它的产生、存储、可能的析构、清理以及一般存取。

### exception handling的主要组成语汇组件：
* 一个throw子句。它在程序某处发出一个exception。被抛出的exception可以是内建类型，也可以是使用者自定类型
* 一个或多个catch子句。每一个catch子句都是一个exception handler。
* 一个try区域。它被围绕以一系列的statements，这些statements可能会引发catch子句起作用

### 决定throw发生的区段
一个函数可以被想象成几个区域：
1. try区段以外的区域，而且没有active local objects
2. try区段以外的区域，但有一个（或以上）的active local objects需要析构
3. try区段以内的区域
编译器必须标示出以上各区域，并使它们对执行期的exception handling系统有所作用。一个很棒的策略就是构造出program counter-range表格。
为了在一个内含try区段的函数中标示出某个区域，可以把program counter的起始值和结束值（或是起始值和范围）存储在一个表格中。
当throw操作发生时，目前的program counter值被拿来与对应的“范围表格”进行比对，以决定目前作用中的区域是否在一个try区段中。如果是，就需要找出相关的catch子句。如果这个exception无法被处理（或者它被再次抛出），目前的这个函数会从程序堆栈中被推出，而program counter会被设定为调用端地址，然后这样的循环再重新开始。

## 执行期类型识别RTTI
### Type-Safe Dynamic Cast保证安全的动态转换
dynamic_cast运算符可以在执行期决定真正的类型。如果downcast是安全的，这个运算符会传回被适当转换过的指针，如果不安全则传回0。

### dynamic_cast运用于指针和引用
1）程序执行中对一个class指针类型施以dynamic_cast运算符，会获得tru或false：
* 如果传回真正的地址，则表示这一object的动态类型被确认了，一些与类型有关的操作现在可以施行于其上
* 如果传回0，则表示没有指向任何object，意味着应该以另一种逻辑施行于这个动态类型未确认的object身上。

2）当dynamic_cast运算符施行于一个refenence时：
* 如果reference真正参考到适当的derived class（包括下一层或下下…层）,downcase会被执行而程序可以继续进行
* 如果reference并不真正是某一种derived class，那么，由于不能传回0，因此抛出一个bad_cast exception

typeid运算符
使用typeid运算符，就有可能以一个reference达到相同的执行期替代路线。
typeid运算符传回一个const reference，类型为type_info：
![](%E6%B7%B1%E5%BA%A6%E6%8E%A2%E7%B4%A2C++%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B/EFF5E4A3-3FC8-4ECC-873F-C8D30F44F20B.png)
在程序中使用typeid方法：typeid(expression) 或 typeid(type)，会传回一个const type_info&