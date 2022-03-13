# Effective C++
## 1. 视C++为一个语言联邦
![](Effective%20C++/526AB6EC-A25B-418E-B7A6-5C2A86A8DAB9.png)

## 2. 尽量以const、enum、inline替换#define

## 3. 尽可能使用const
如果关键字const出现在*左边，表示被指物是常量，如果出现在*右边，表示指针本身是常量

## 4. 确定对象被使用前已先被初始化
![](Effective%20C++/37C51E54-3175-433C-AA60-8544FAF7B63D.png)

## 5. 了解C++默默编写并调用哪些函数
如果自己没声明，编译器会声明一个copy构造函数、copy assignment操作符和一个析构函数和一个default构造函数。

## 6. 若不想使用编译器自动生成的函数，就该明确拒绝
方法：
1）将相应的成员函数声明为private，并且不予以实现。
2）使用像Uncopyable这样的base class：
```
class Uncopyable {
protected:
Uncopyable() {}
~Uncopyable() {}
private:
Uncopyable(const Uncopyable&);
Uncopyable& operator=(const Uncopyable&);
};
```

## 7. 为多态基类声明virtual析构函数
* 任何class只要带有virtual函数都几乎确定应该也有一个virtual析构函数。
* 如果class不含virtual函数，通常表示它并不意图被用作一个base class，则其析构函数为virtual往往是个馊主意。
* polymorphic（带多态性质的）base classed应该声明一个virtual析构函数。如果class带有任何virtual函数，它就应该拥有一个virtual析构函数。
* classed的设计目的如果不是作为base classes使用，或不是为了具备多态性（polymorphically），就不该声明virtual析构函数。

## 8. 别让异常逃离析构函数
* C++并不禁止析构函数吐出异常，但它不鼓励你这么做。
* 因此，析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕获任何异常，然后吞下它们（不传播）或结束程序。
* 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数（而非在析构函数中）执行该操作。

## 9.绝不在构造和析构过程中调用virtual函数
因为这类调用从不下降至derived class（比起当前执行构造函数和析构函数的那层）。

## 10. 令operator=返回一个*this的引用

## 11. 在operator=中处理“自我赋值”
* 确保当对象自我赋值时 operator=有良好行为。其中技术包括比较“来源对象”和“目标对象”的地址、精心周到的语句顺序、以及copy-and-swap。
* 确定任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确。

## 12. 复制对象时勿忘其每一个成分
* Copying函数应该确保复制“对象内的所有成员变量”及“所有base class成分”。
* 不要尝试以某个copying函数实现另一个copying函数。应该将共同机能放进第三个函数中，并由两个copying函数共同调用。

## 13. 以对象管理资源
* 为防止资源泄漏，请使用RAII对象，它们在构造函数中获得资源并在析构函数中释放资源。
* 两个常被使用的RAII classes分别是 tr1::shared_ptr 和 auto_ptr。前者通常是较佳选择。
* 1）auto_ptr被销毁时会自动删除它所指之物，所以一定要注意别让多个auto_ptr同时指向同一对象。auto_ptr有一个不同寻常的性质：若通过copy构造函数或copy assignment操作符复制它们，它们会变成null，而复制所得的指针将取得资源的唯一拥有权。 以上两点，auto_ptr并非管理动态分配资源的神兵利器。
* 2）引用计数型智慧指针RCSP，其也是智能指针，持续追踪共有多少对象指向某笔资源，并在无人指向它时自动删除该资源。RCSPs提供的行为类似垃圾回收，不同的是其无法打破环状引用。shared_ptr就是个RCSP。

## 14. 在资源管理类中小心copying行为
* 复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为。
* 普遍而常见的RAII class copying 行为是：抑制copying（条款6）、施行引用计数法（shared_ptr）。不过其他行为也都可能被实现。

## 15. 在资源管理类中提供对原始资源的访问
* APIs往往要求访问原始资源，所以每一个RAII class应该提供一个“取得其所管理之资源”的办法
* 对原始资源的访问可能经由显示转换或隐式转换。一般而言显示转换比较安全，但隐式转换对客户比较方便。

## 16. 成对使用new和delete时要采取相同形式
如果你在new表达式中使用[]，必须在相应的delete表达式中也使用[]。如果你在new表达式中不使用[]，一定不要在相应的delete表达式中使用[]。

## 17. 以独立语句将newed对象置入智能指针
如果不这样做，一旦异常抛出，有可能导致难以察觉的资源泄漏。
```
//存在函数：
int priority();
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);
//调用：
processWidget(std::tr1::shared_ptr<Widget>(new Widge), priority());
```
执行processWidget前，创建代码顺序可以是：
1）调用priority
2）执行 new Widget
3）调用std::tr1::shared_ptr构造函数
或者：
1）执行 new Widget
2）调用std::tr1::shared_ptr构造函数
3）调用priority
或者：
1）执行 new Widget
2）调用priority
3）调用std::tr1::shared_ptr构造函数
假如调用priority时异常，则new Widget返回的指针将遗失

## 18. 让接口容易被正确使用，不易被误用
* 好的接口很容易被正确使用，不容易被误用。你应该再你的所有接口中努力达成这些性质。
* “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容。
* “防止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。
* tr1::shared_ptr支持定制型删除器。这可防止DLL问题，可被用来自动解除互斥锁等。

## 19. 设计class犹如设计type
class的设计就是type的设计。在定义一个新type之前，请确定考虑下面主题：
* 新type的对象应该如何被创建和销毁
* 对象的初始化和对象的赋值该有什么样的差别
* 新type的对象如果被pass by value，意味着什么
* 什么是新type的“合法值”
* 你的新type需要配合某个继承图系吗？
* 你的新type需要什么样的转换
* 什么样的操作符和函数对此新type而言是合理的
* 什么样的标准函数应该驳回
* 谁该取用新type的成员
* 什么是新type的“未声明接口”
* 你的新type有多么一般化
* 你真的需要一个新type吗？

## 20. 宁以pass-by-reference-to-const替换pass-by-value
* 尽量以pass-by-reference-to-const替换pass-by-value，前者通常比较高效，并可避免切割问题。
* 以上规则并不适用于内置类型，以及stl的迭代器和函数对象。对它们而言，pass-by-value往往比较适当

## 21. 必须返回对象时，别妄想返回其reference
绝不要返回pointer或reference指向一个local stack对象，或返回reference指向一个heap-allocated对象，或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象。条款4已经为“在单线程环境中合理返回reference指向一个local static对象”提供了一份设计实例。

## 22. 将成员变量声明为private
* 切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供class作者以充分的实现弹性。
* protected并不比public更具封装性。

## 23. 宁以non-member、non-friend替换member函数
这样做可以增加封装性、包裹弹性和机能扩充性。

## 24. 若所有参数皆需类型转换，请为此采用non-member函数
## 25. 考虑写出一个不抛异常的swap函数
![](Effective%20C++/%E6%88%AA%E5%B1%8F2021-08-18%20%E4%B8%8B%E5%8D%8811.08.05.png)
![](Effective%20C++/7ACFD9E9-C983-4635-889C-DB5A6E9EE816.png)
![](Effective%20C++/%E6%88%AA%E5%B1%8F2021-08-18%20%E4%B8%8B%E5%8D%8811.09.02.png)

## 26. 尽可能延后变量定义式的出现时间
这样做可增加程序的清晰度并改善程序效率

## 27. 尽量少做转型动作
C++转型语法：
（旧式转型：）
* (T)expression
* T(expression)
（新式转型：）
* const_cast<T>( expression )：用来将对象的常量性转除
* dynamic_case<T>( expression )：主要用来执行“安全向下转型”（继承）
* reinterpret_case<T>( expression )：意图执行低级转型，实际动作（及结果）可能取决于编译器，这也就表示它不可移植（比如将pointer to int转型为一个int）
* static_case<T>( expression )：用来强迫隐式转换（比如non-const对象转为const，int转为double），也可用来执行上述多种转换的反向转换（比如void*指针转为typed指针，pointer-to-base转为pointer-to-derived）

因此：
* 如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_casts。如果有个设计需要转型动作，试着封装无需转型的替代设计。
* 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需将转型放进他们自己的代码内。
* 宁可使用C++-style（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌。

## 28. 避免返回handles指向对象内部成分
避免返回handles（包括reference、指针、迭代器）指向对象内部。遵守这个条款可增加封装性，帮助const成员函数的行为像个const，并将发生“虚吊号码牌”（dangling handles）的可能性降至最低。

## 29. 为“异常安全”而努力是值得的
* 当异常被抛出时，带有异常安全性的函数会：1、不泄漏任何资源；2、不允许数据败坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型。
* “强烈保证”往往能够以copy-and-swap实现出来，但“强烈保证”并非对所有函数都可实现或具备现实意义。
* 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者。

## 30. 透彻了解inlining的里里外外
* 将大多数inlining限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。
* 不要只因为function templates出现在头文件，就将它们声明为inline。

## 31. 将文件间的编译依存关系降至最低
* 支持“编译依存性最小化”的一般构想时：相依于声明式，不要相依于定义式。
* 如果使用object reference或object pointers可以完成任务，就不要使用objects。你可以只靠一个类型声明式就定义出指向该类型的references和pointers，但如果定义某类型的objects，就需要用到该类型的定义式。
* 如果能够，尽量以class声明式替换class定义式。注意，当你声明一个函数而它用到某个class时，你并不需要该class的定义，纵使函数以by value方式传递该类型的参数/返回值。
* 为声明式和定义式提供不同的头文件。
* 基于此构想的两个手段是Handle classes和Interface classes。
* 程序库头文件应该以“完全且仅有声明式”的形式存在。这种做法不论是否涉及templates都适用。

## 32. 确定你的public继承塑模出is-a关系
“public继承”意味着is-a。适用于base classes身上的每一件事情一定也适用于derived classes身上，因为每一个derived class对象也都是一个base class对象。

## 33. 避免遮掩继承而来的名称
![](Effective%20C++/77EBF807-8A98-4869-8E33-75B0F941CAB2.png)
* derived classes内的名称会遮掩base classes内的名称。在public继承下从来没有人希望如此。
* 为了让被遮掩的名称再见天日，可使用using声明式或转交函数（forwarding functions）。

## 34. 区分接口继承和实现继承
* 接口继承和实现继承不同。在public继承之下，derived classes总是继承base class的接口。
* 声明一个pure virtual函数的目的是为了让derived classes只继承函数接口。
* 声明简朴的（非纯）impure virtual函数的目的，是让derived classes继承该函数的接口和缺省实现。
* 声明non-virtual函数的目的是为了令derived classes继承函数的接口及一份强制性实现。

## 35. 考虑virtual函数以外的其他选择
* 藉由Non-Virtual Interface（NVI）手法实现Template Method模式。
NVI：令客户通过public non-virtual成员函数包裹较低访问性（private或protected）的virtual函数。
* 将virtual函数替换为“函数指针成员变量”，这是Strategy设计模式的一种分解表现形式。
* 以tr1::function成员变量替换virtual函数，因而允许使用任何可调用物（callable entity）搭配一个兼容于需求的签名式。这也是Strateg设计模式的某种形式。
* 将继承体系内的virtual函数替换为另一个继承体系内的virtual函数。这是Strategy设计模式的传统实现手法。

记住：
* virtual函数的替代方案包括NVI手法及Strategy设计模式的多种形式。NVI手法自身是一个特殊形式的Template Method设计模式。
* 将机能从成员函数移到class外部函数，带来的一个缺点是：非成员函数无法访问class的non-public成员。
* tr1::function对象的行为就像一般函数指针。这样的对象可接纳“与给定之目标签名式（target signature）兼容”的所有可调用物（callable entities）。

## 36. 绝不重新定义继承而来的non-virtual函数
![](Effective%20C++/1FBEE61A-9490-478A-8659-82A0E1276163.png)

## 37. 绝不重新定义继承而来的缺省参数值
virtual函数系动态绑定，而缺省参数值却是静态绑定（为了运行时效率）。
你唯一应该覆写的东西，却是动态绑定。

## 38. 通过复合塑膜模出has-a或“根据某物实现出”
* 复合（composition）的意义和public继承完全不同。
* 在应用域（application domain），复合意味着has-a。在实现域（implementation domain），复合意味着is-implemented-in-terms-of（根据某物实现出）。

## 39. 明智而审慎地使用private继承
private继承的行为：
1）如果classes之间的继承关系是private，编译器不会自动将一个derived class对象转换为一个base class对象。
2）由private base class继承而来的所有成员，在derived class中都会变成private属性，纵使它们在base class中原本是protected或public属性。

* private继承意味着is-implemented-in-terms-of（根据某物实现出）。这与复合（composition）意义一样，因此尽可能使用复合，必要时才使用private继承。何时才是必要？主要是当protected成员和/或virtual函数牵扯进来的时候，还有一种是当空间方面的利害关系足以踢翻private继承的支柱时。

* 和复合（composition）不同，private继承可以造成empty base最优化。这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要。
EBO：empty base optimization空白基类最优化。EBO一般只在单一继承（而非多重继承）下才可行。

## 40. 明智而审慎地使用多重继承
* 多重继承比单一继承复杂。它可能导致新的歧义性，以及对virtual继承的需要。
* virtual继承会增加大小、速度、初始化（及赋值）复杂度等等成本。如果virtual base classes 不带任何数据，将时最具实用价值的情况。
* 多重继承的确有正当用途。其中一个情节涉及“public继承某个Interface class”和“private继承某个协助实现的class”的两相组合。

## 41. 了解隐式接口和编译期多态
* classes和templates都支持接口（interfaces）和多态（polymorphism）。
* 对classes而言接口是显式的（explicit），以函数签名为中心。多态则是通过virtual函数发生于运行期。
* 对template参数而言，接口是隐式的（implicit），奠基于有效表达式。多态则是通过template具现化和函数重载解析（function overloading resolution）发生于编译期。

## 42. 了解typename的双重意义
* 声明template参数时，前缀关键字class和typename可互换。
```
//下面两种声明式没有不同：
template<class T> class Widget;
template<typename T> class Widget;
```
* 请使用关键字typename标识嵌套从属类型名称：但不得在base class lists（基类列）或member initialization list（成员初值列）内以它作为base class修饰符。
从属名称（dependent names）：template内出现的名称如果相依于某个template参数，称之为从属名称。
嵌套从属名称（nested dependent name）：如果从属名称在class内呈嵌套状。
非从属名称（non-dependent names）：不倚赖任何template参数的名称。

## 43. 学习处理模板化基类内的名称
可在derived class templates内通过“this->”指涉base class templates内的成员名称，或藉由一个明白写出的“base  class资格修饰符”完成。

## 44. 将与参数无关的代码抽离templates
* Templates生成多个classes和多个函数，所以任何template代码都不该与某个造成膨胀的template参数产生相依关系。
* 因非类型模板参数（non-type termplate parameters)而造成的代码膨胀，往往可消除，做法是以函数参数或class成员变量替换template参数。
* 因类型参数（type parameters）而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述（binary representations）的具现类型（instantiation types）共享实现码。

## 45. 运用成员函数模板接受所有兼容类型
* 请使用member function templates（成员函数模板）生成“可接受所有兼容类型”的函数。
* 如果你声明member templates用于“泛化copy构造”或“泛化assignment操作”，你还是需要声明正常的copy构造函数和copy assignment操作符。

## 46. 需要类型转换时请为模板定义非成员函数
当我们编写一个class template，而它所提供之“与此template相关的”函数支持“所有参数之隐式类型转换”时，请将那些函数定义为“class template内部的friend函数”。
```
template<typename T> class Rational;

template<typename T>
const Rational<T> doMultiply(const Rational<T>& lhs, const Rational<T>& rhs);

template<typename T>
class Rational {
public:
...
friend const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs) 
{
	return doMultiply(lhs, rhs);
}
...
};

template<typename T>
const Rational<T> doMultiply(const Rational<T>& lhs, const Rational<T>& rhs)
{
	return Rational<T>(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
}
```

## 47. 请使用traits class表现类型信息
1）如何设计并实现一个traits class：
* 确认若干你希望将来可取得的类型相关信息。
* 为该信息选择一个名称
* 提供一个template和一组特化版本，内含你希望支持的类型相关信息

2）如何使用一个traits class：
* 建立一组重载函数或函数模板，彼此间的差异只在于各自的traits参数。令每个函数实现码与其接受之traits信息相应和。
* 建立一个控制函数或函数模板，它调用上述那些重载函数并传递traits class所提供的信息

记住：
* traits class使得“类型相关信息”在编译期可用。它们以tempaltes和“templates特化”完成实现。
* 整合重载技术后，traits classes有可能在编译期对类型执行if…else测试。

## 48. 认识tempalte元编程
TMP：Template metaprogramming模板元编程，是编写tempalte-based C++程序并执行于编译期的过程。
* 它可将工作由运行期移往编译期，因而得以实现早期错误侦测和更高的执行效率。
* TMP可被用来生成“基于政策选择组合”（based on combinations of policy choices）的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码。

## 49. 了解new-handler的行为
当operator new 抛出异常以反映一个未获满足的内存需求之前，它会先调用一个客户指定的错误处理函数，一个所谓的new-handler。

一个设计良好的new-handler函数必须要做的事：
* 让更多内存可被使用。这便造成operator new内的下一次内存分配动作可能成功。
* 安装另一个new-handler。如果当前new-handler知道另外哪个new-handler有能力获得更多可用内存，就可以安装另外那个new-handler以替换自己（set_new_handler）。
* 卸除new-handler，即将null指针传给set_new_handler。一旦没有安装任何new-handler，operator new会在内存分配不成功时抛出异常。
* 抛出bad_alloc（或派生自bad_alloc）的异常。这样的异常不会被operator new捕捉，因此会被传播到内存索求处。
* 不返回，通常调用abort或exit。

记住：
* set_new_handler允许客户指定一个函数，在内存分配无法获得满足时被调用。
* Nothrow new是一个颇为局限的工具，因为它只适用于内存分配；后继的构造函数调用还是可能抛出异常。

## 50. 了解new和delete的合理替换时机
了解何时可在“全局性的”或“class专属的”基础上合理替换缺省的new和delete：
* 用来检测运用上的错误
* 为了收集动态分配内存之使用统计数据
* 为了增加分配和归还的速度
* 为了降低缺省内存管理器带来的空间额外开销
* 为了弥补缺省分配器中的非最佳齐位
* 为了将相关对象成簇集中
* 为了获得非传统的行为

## 51. 编写new和delete时需固守常规
* operator new应该内存一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就该调用new-handler。它也应该有能力处理0bytes申请。Class专属版本则还应该处理“比正确大小更大的（错误）申请”。
* operator delete应该在收到null指针时不做任何事。Class专属版本则还应该处理“比正确大小更大的（错误）申请”。

## 52. 写了placement new也要写placement delete
placement new：如果operator new接受的参数除了一定会有的那个size_t之外还有其他，这便是个所谓的placement new。
placement delete：如果operator delete接受额外参数，便称为placement delete。
缺省情况下C++在global作用于内提供的operator new：
```
void* operator new(std::size_t) throw(std::bad_alloc);//normal new
void* operator new(std::size_t, void*) throw();//placement new
void* operator new(std::size_t, const std::nothrow_t&) throw();//nothrow new
```

记住：
* 当你写一个placement operator new，请确定也写出对应的placement operator delete。如果没有这样做，你的程序可能会发生隐薇而时断时续的内存泄漏。
* 当你声明placement new和placement delete，请确定不要无意识（非故意）地遮掩了它们的正常版本。

## 53. 不要轻忽编译期的警告
* 严肃对待编译期发出的警告信息。努力在你的编译期的最高（最严苛）警告级别下争取“无任何警告”的荣誉。
* 不要过度倚赖编译期的报警能力，因为不同的编译期对待事情的态度并不相同。一旦移植到另一个编译期上，你原本倚赖的警告信息有可能消失。

## 54. 让自己熟悉包括TR1在内的标准程序库
C++98列入的C++标准程序库主要成分：
* STL，覆盖容器、迭代器、算法、各自容器适配器和函数对象适配器
* Iostream，覆盖用户自定缓冲功能、国际化I/O，以及预先定义号的对象cin、cout、cerr和clog。
* 国际化支持，包括多区域能力（wchar_t、wstring等）
* 数值处理：包括复数模板和纯数值数组。
* 异常阶层体系，包括base class exception及其derived class logic_error和runtime_error，以及更深继承的各个classes。
* C89标准程序库

TR1:Technical Report 1
TR1组件实例：
* 智能指针：tr1::shared_ptr和 tr1::weak_ptr
* tr1::function：此物得以表示任何callable entity（可调用物，也就是任何函数或函数对象），只要其签名符合目标。
void registerCallback(std::tr1::function<std::string (int)>func);
* tr1::bind：它能够做STL绑定器bindlst和bind2nd所做的每一件事，而又更多。它可以和const及non-const成员函数协同运作，可以和by-reference参数协同运作，而且它不需特殊协助就可以处理函数指针。
（下面这部分提供彼此互不相干的独立机能）
* Hash tables：用来实现sets，multisets，maps和multi-maps，名称为：tr1::unordered _set, r1::unordered _multiset, r1::unordered _map, r1::unordered _multimap。
* 正则表达式，包括以正则表达式为基础的字符串查找和替换，或是从某个匹配字符串到另一个匹配字符串的逐一迭代等等。
* Tuples（变量组），这是标准程序库中的pair template的新一代制品。pair智能持有两个对象，tr1::tuple可持有任意个数的对象。
* tr1::array：本质是个“STL化”数组，即一个支持成员函数如begin和end的数组。tr1::array的大小固定，并不使用动态内存。
* tr1::mem_fn：这是个语句构造上与成员函数指针一致的东西。
* tr1::reference_wrapper：一个“让reference的行为更像对象”的设施。它可以造成容器“犹如持有reference”。
* 随机数生成工具
* 数学特殊函数，包括Laguerre多项式、Bassel函数完全椭圆机分，以及更多数学函数。
* C99兼容扩充：一大堆函数和模板。
（下面这部分由更精巧的template编程技术包括TMP构成）
* Type traits：一组traits classes，用以提供类型（types）的编译期信息。给予一个类型T，TR1的type traits可以指出T是否是个内置类型，是否提供virtual析构函数，是否是个empty class，可隐式转换为其他类型U吗等等，也可以显现该给定类型之适当齐位（alignment）。
* tr1::result_of：这是个template，用来推导函数调用的返回类型。

## 55. 让自己熟悉Boost