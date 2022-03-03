# Effective-C-
effectiveC++ summary
条款2 尽量以const ,enum,inline替换#define

#defines不重视作用域

取一个const的地址是合法的，但取一个enum的地址就不合法

类中的const成员变量只能在构造函数**初始化列表**初始化，static和static const成员变量在类外初始化。



条款3 尽可能使用const

const出现在星号左边表示被指物是常量；如果出现在星号右边，表示指针自身是常量；出现在星号两边，表示被指物和指针两者都是常量。

const std::vector<int>::iterator iter = //iter的作用像个T* const

std::vector<int>::const_iterator cIter =   //cIter的作用像个const T*

两个成员函如果只是常量性不同，可以被重载。

如果函数的返回类型是个内置类型（如int），那么改动函数返回值从来就不合法。

常成员函数内可以修改static类型变量，因为static成员不属于对象。

当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复，具体实施过程中需进行两次类型转换，一是将*this加上const，第二次是从const成员函数返回值中移除const。



条款4 确定对象被使用前已被初始化

永远在使用对象之前先将它初始化。

C++规定，对象的成员变量的初始化动作发生在进入构造函数**本体**之前。

使用成员初值列可避免：首先调用成员变量的默认构造函数，再在构造函数体里赋值。

class的成员变量总是以其声明的次序被初始化。

为免除"跨编译单元内全局的静态的对象（包括全局变量和静态全局变量）初始化次序"问题，可以参考Meyer's单例模式，用**静态局部**对象替换全局的静态的对象。

任何一种**非**常静态对象，不管是局部还是全局的，在多线程环境下“等待某事发生”都会有麻烦，解决麻烦的做法是：在程序的单线程启动阶段手动调用所有的返回引用的函数，这可消除初始化有关的“竞速形式”。



条款5 了解C++默默编写并调用哪些函数

编译器可以暗自为class创建default函数、copy构造函数、拷贝赋值操作运算符以及析构函数，均为public。

当成员变量是引用类型或者const成员，需自己定义拷贝赋值操作运算符。

如果某个基类将拷贝赋值操作运算符声明为private，编译器拒绝为其派生类生成一个拷贝赋值赋值运算符。

类的析构函数会自动调用其非静态成员变量的析构函数。



条款6 若不想使用编译器自动生成的函数，就该明确拒绝

将成员函数声明为private而且不实现，当对象调用成员函数编译器会报错，当另外的成员函数或者友元函数调用这些未实现的成员函数，连接器会报错。要想将连接错误移植到编译期，可构造一个uncopyable类，功能类私有继承uncopyable类即可。



条款7 为多态基类声明virtual析构函数

任何class只要带有virtual函数都应该也有一个virtual析构函数。

为实现virtual函数，对象携带一个vptr(virtual table pointer)指针，vprt指向一个由**函数指针**组成的数组vtbl(virtual table)。**每一个带有virtual函数的class都有一个相应的vtbl**，当对象调用某一virtual函数，实际被调用的函数取决于该对象的vptr所指的那个vtbl——编译器在其中寻找合适的函数指针。

当想把一个类定义为抽象类，但没有纯虚函数时，可以将析构函数声明为纯虚析构函数，而且需要为这个纯虚析构函数提供一份定义。（当派生类调用它的析构函数时，编译器会在其中创建一个对其基类的析构函数的调用动作，如果基类的析构函数没有定义，连接器会报错）

带多态性质的基类应该声明一个virtual析构函数。



条款8 别让异常逃离析构函数

析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下他们（不传播）或结束程序。

如果客户需要对某个操作函数运行期间抛出的异常作出反应，那么class应该提供一个普通函数（而非在析构函数中）执行该操作。



条款9 绝不在构造和析构函数过程中调用vitrual函数

在派生类对象的基类构造期间，对象的类型是基类而不是派生类。对象在派生类构造函数开始执行前不会成为一个派生类对象。同理也适用于析构函数。

确定构造函数和析构函数都没有调用virtual函数，它们调用的函数也都服从同一约束。



条款10 令operator=返回一个reference to*this

链式反应，同样适用于所有赋值相关运算，例如operator+=。



条款11 在operator= 中处理“自我赋值” （妙）

当成员变量包含**指针**时，确保当对象自我赋值时operator=有良好的行为，可以选择：1.比较“来源对象”和“目标对象”的地址（解决了”自我赋值安全“，但解决不了“异常安全性”）、2.精心周到的语句顺序——先对指针做一份复件、删除原成员（在解决“异常安全性”的同时可解决“自我赋值安全性”）、3.**copy-and-seap——为参数数据制作一份副本，将*this数据和上述数据交换**（常见且足够好的operator=撰写方法）。

保证任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确。



条款12 复制对象时勿忘其每一成分

当编写一个拷贝构造函数或者拷贝赋值操作符函数应该保证：1.复制所有的局部成员变量。2.调用所有基类内的适当的拷贝构造函数或者拷贝复制操作符。

copying函数应该确保复制“对象内的所有成员变量”及“所有的base class成分”。

不要尝试以某个copying函数实现另一个copying函数。应该将共同机能放进第三个函数中，并由两个copying函数共同调用，避免代码重复。





条款13 以对象管理资源

使用智能指针可以避免动态分配在堆区的资源泄漏。

智能指针的析构函数自动对其所指对象调用delete，C++20后支持数组。

“以对象管理资源”（RAII）需要两个保证：1.获得资源后立刻放进管理对象内，2.管理对象运用析构函数确保资源释放。

C++11 引入了 3 个智能指针类型：

1. `std::unique_ptr<T>` ：独占资源所有权的指针。
2. `std::shared_ptr<T>` ：共享资源所有权的指针。
3. `std::weak_ptr<T>` ：共享资源的观察者，需要和 std::shared_ptr 一起使用，不影响资源的生命周期。

详情见：[现代 C++：一文读懂智能指针 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/150555165)



条款14 在资源管理类中小心copying行为

对于非堆上的资源，智能指针不适用，需要建立自己的资源管理类。

例如：为确保不忘记将被锁住的Mutex解锁，基于RAII守则建立一个类管理机锁。

```c++
class Lock{
public:
    explicit Lock(Mutex* pm):mutexPtr(pm)
    {
        lock(mutexPtr);
    }
    ~Lock()
    {
        unlock(mutexPtr);
    }
private:
    Mutex* mutexPtr;
}
```

当一个RAII的对象被复制时，可以选择：

​	1.禁止复制。见条款6。

​	2.对底层资源祭出“引用计数法”。

保有资源，直到最后一个使用者被销毁，此种情况下复制RAII对象，应将资源的“被引用次数”递增。通常只要内含一个std::shared_ptr成员变量即可，但需要改变std::shared_ptr的“删除器”，因为对一个Mutex，我们需要的释放动作是解除锁定而非删除。Mutex的析构函数会自动调用shared_ptr的”删除器“(unlock)，所以不用在RAII类中定义析构函数了。

```c++
class Lock{
public:
    explicit Lock(Mutex* pm):mutexPtr(pm,unlock)
    {
        lock(mutexPtr.get());
    }
private:
    std::shared_ptr<Mutex> mutexPtr;
}
```

​	3.使用“深拷贝”复制底部资源。

​	4.转移底部资源拥有权。资源的拥有权从被复制物转移到目标物。



条款15 在资源管理类中提供对原始资源的访问

APIs往往要求访问原始资源，所以每一个RAII类应该提供一个“取得其所管理之资源”的办法，需要一个函数将RAII类的对象转换为其所内含之原始资源，有两种方法：显示转换和隐式转换。显示转换较为安全：智能指针如shared_ptr都提供一个get成员函数，也可使用->和*操作符将对象类型隐式转换为底部原始指针。隐式转换对客户较为方便，但可能增加错误发生机会，可通过隐式转换函数实现。



条款16 成对使用new和delete时要采用相同形式

new与delete对应，new []与delete []对应。

为避免delete的形式错误，应尽量不要对数组形式使用typedef形式。



条款17 以独立语句将new**ed**对象置入智能指针

shared_ptr的构造函数是个explicit函数，无法进行隐式转换。

以下语句可能会造成资源泄漏：

```c++
processWidget(shared_ptr<Weight>(new Widget),priority());
//priority是一个函数，processWidget也是一个函数
```

C++编译器在调用processWidget之前，编译器需要做三件事：

​	1.调用priority函数

​	2.执行"new Widget"

​	3.调用shared_ptr构造函数

2一定在3之前执行，但1的执行在C++里不确定，可能为123、213、231，当顺序为213时，如果调用priority函数异常，那么new Widget返回的指针将遗失，资源可能会泄露。所以**需要以独立的语句将new**ed**对象存储于智能指针内，否则可能引发资源泄漏**。

故代码可改为：

```c++
shared_ptr<Widget> pw(new Widget);
processWidget(pw,priority);
```



条款18 让接口容易被使用，不易被误用

“促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容。

“阻止误用”的方法包括建立新类型（如将Date类中的Month封装为一个类）、限制类型上的操作（常见的限制是加上const）、束缚对象值以及消除客户的资源管理责任。

智能指针支持定制型删除器——自动使用它的“每个指针专属的删除器”，这可防范跨DLL问题，可被用来自动解锁互斥锁。

好的接口可以防止无效的代码。



条款19 设计class犹如设计type

应该带着和“语言设计者当初设计语言内置类型时”一样的严谨来研讨class的设计。



条款20 宁以pass-by- reference-to-const替换pass-by-value

以值传递一个派生类对象会导致调用一次派生类的拷贝构造函数，一次基类的构造函数，以及派生类和基类的成员对象的构造函数，销毁时还会调用析构函数，效率低。以**常**引用的方式传递值效率较高。

以引用方式传递参数可以避免对象切割问题。当一个派生类对象以值方式传递并被视为一个基类对象时，基类的拷贝构造函数会切割掉除基类对象之外的派生类对象的特化性质。

对内置类型以及STL的迭代器和函数对象，对他们而言，以值方式传递往往比较适当。



条款21 必须返回对象时，别妄想返回reference(引用)

用上static对象的设计时需考虑多线程的安全性。

绝不要**返回指针或者引用**指向一个局部栈对象，或**返回引用**指向一个分配的堆对象，或**返回指针或引用**指向一个局部静态变量而有可能同时需要多个这样的对象（因为需要多个这样的对象所有的对象值都相同）。例如对operator* 函数，返回一个新对象即可：

```c++
inline const Rational operator*(const Rational& lhs,const Rational& rhs)
{
    return Rational(lhs.n*rhs.n,lhs.d*rhs.d);
}
```



条款22 将成员变量生明为private

切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供class作者以充分的实现弹性。

成员变量的封装性与“成员变量的内容改变时所破坏的代码数量”成反比。protected成员变量就像public成员变量一样缺乏封装性，因为在这两种情况下，如果成员变量被改变，都会有不可预知的大量代码受到破坏。



条款23 宁以一个non-member non-friend的函数替换member函数

non-member non-friend函数相较于一个member函数导致类有更大的封装性，因为non-member non-friend函数导致更少的函数可以访问类内数据。

在C++里为了提供“使用类的客户无法以其他方式获得”的机能，比较自然的做法是让non-member non-friend函数与类在同一namespace(命名空间)里。将所有便利函数放在多个头文件但隶属同一个命名空间，同时意味着客户可以轻松扩展这一组便利函数，它们需要做的就是添加更多non-member non-friend函数到此命名空间内。这是class定义式不能做到的，因为class定义式对客户而言不能扩展。



条款24 若所有参数皆需类型转换，请为此采用non-member函数

编译器会为只有一个参数或者除了第一个参数外其余参数都有默认值的类构造函数提供隐式类型转换。

若不想发生隐式类型转换，在只有一个参数或者除了第一个参数外其余参数都有默认值的类构造函数前加上explicit关键字。

成员函数的反面是非成员函数，不是友元函数。

友元函数能避免就避免。

如果需要为某个函数的所有参数（包括被this指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是个非友元函数。



条款25 考虑写出一个不抛异常的swap函数

swap函数是异常安全性编程的脊柱（条款29），以及用来处理自我赋值可能性（条款11）。

只要类型T支持copying（通过拷贝构造函数和拷贝赋值操作符完成），缺省（默认）的swap实现代码就会帮你置换类型为T的对象。

如果一个类型T中数据有“以一个指针指向对象，内含真正数据”那种类型，即所谓的"pimpl手法"(point to implementation)，缺省的swap函数效率很低。可以按以下步骤进行swap：

​	1.提供一个public swap成员函数，让它高效地置换你的类型的两个对象值，这个成员版swap函数绝不该抛出异常，因为成员版swap的一个最好的应用是帮助classes（或者class templates）提供强烈的异常安全性保证。

​	2.在你的class或template所在的命名空间内提供一个non-member swap，并令它调用上述swap成员函数。

​	3.如果正在编写一个class（而非class template），为你的class特化std::swap。并令它调用你的swap成员函数。

​	4.如果调用swap，请确定包含一个using声明式(using std::swap)，以便让std::swap在你的函数内曝光可见，然后不加任何namespace修饰符，赤裸裸地调用swap。

通常不允许改变std命名空间里的任何东西，但可以为标准templates(如swap)制造特化版本，使它专属于我们自己的classes（例如Widget）。**特化版std::swap**:

```c++
namespace std{
    template<>
    void swap<Widget>(Widget& a, Widget& b)
    {
        a.swap(b);//调用已经写好的Widget的swap成员函数。
    }
}
```

C++只允许偏特化class templates，不允许对function template，以下虽可能通过编译，但不合法：

```c++
namespace std{
	template<typename T>
    void swap<Widget<T> >(Widget<T>& a, Widget<T>& b)//偏特化function templates不合法
    {
        a.swap(b);
   }
}
```

swap函数的调用顺序：一旦编译器看到对swap的调用，C++的名称查找法则确保找到全局作用域或类所在的命名空间内的任何类的专属swap。如果类在某命名空间内，编译器会使用“实参取决之查找规则”找到此命名空间内的swap。如果没有类专属的swap，由于using std::swap的提前曝光，便可使用std内的特化swap。具体实现代码可见"重难点.md/针对pimpl手法的swap函数实现方法"。

```c++
#include<iostream>
using namespace std;
//针对pimpl手法的swap函数实现方法

//创建widget命名空间
namespace widget
{
    
    class WidgetImpl
    {
    public:
        WidgetImpl(int aa, int bb, int cc)
            :a(aa), b(bb), c(cc){};
        WidgetImpl(const WidgetImpl& rhs)
            :a(rhs.a), b(rhs.b), c(rhs.c){};
        void printWidgetIpml()
        {
            cout << a << b << c << endl;

        }
    private:
        int a, b, c;
    };
#include<iostream>
using namespace std;
//针对pimpl手法的swap函数实现方法

//创建widget命名空间
namespace widget
{
    
    class WidgetImpl
    {
    public:
        WidgetImpl(int aa, int bb, int cc)
            :a(aa), b(bb), c(cc){};
        WidgetImpl(const WidgetImpl& rhs)
            :a(rhs.a), b(rhs.b), c(rhs.c){};
        void printWidgetIpml()
        {
            cout << a << b << c << endl;

        }
    private:
        int a, b, c;
    };

    class Widget
    {
    public:
        //有参构造
        Widget(int a, int b, int c)
            :pImpl(new WidgetImpl(a, b, c)) {};
        ~Widget()
        {
            if (pImpl != NULL)
            {
                delete pImpl;
                pImpl = NULL;
            }
        }
        //拷贝构造
        Widget(const Widget& rhs)
            :pImpl(new WidgetImpl(*rhs.pImpl)) {};
        //拷贝复制操作符
        Widget& operator=(const Widget& rhs)
        {
            //拷贝赋值操作符方法1
            /*WidgetImpl* temp = pImpl;
            pImpl = new WidgetImpl(*rhs.pImpl);
            delete temp;
            return *this;*/
            //拷贝赋值操作符方法2(常用)
            Widget temp(rhs);
            swap(temp);
            return *this;
        }
        //成员版swap
        void swap(Widget& rhs)
        {
            using std::swap;//using 声明式让std::swap提前曝光
            swap(pImpl, rhs.pImpl);
        };
        void printWidget()
        {
            pImpl->printWidgetIpml();

        }
    private:
        WidgetImpl* pImpl;
    };
    //非成员版swap函数,Widget专属swap
    void swap(Widget& a, Widget& b)
    {
        a.swap(b);
    }
}

//在std命名空间特化swap
namespace std 
{
    //特化swap
    template<>
    void swap(widget::Widget& a, widget::Widget& b)
    {
        a.swap(b);
    }
}
int main()
{
    widget::Widget w1(1, 2, 3);
    widget::Widget w2(4, 5, 6);
    //提前曝光swap()
    using std::swap;
    swap(w1, w2);
    w1.printWidget();//456
    w2.printWidget();//123

    //使用swap函数的赋值操作符验证
    widget::Widget w3(7, 8, 9);
    w3 = w2;
    w3.printWidget();//123
    system("pause");
    return 0;
}
    class Widget
    {
    public:
        //有参构造
        Widget(int a, int b, int c)
            :pImpl(new WidgetImpl(a, b, c)) {};
        ~Widget()
        {
            if (pImpl != NULL)
            {
                delete pImpl;
                pImpl = NULL;
            }
        }
        //拷贝构造
        Widget(const Widget& rhs)
            :pImpl(new WidgetImpl(*rhs.pImpl)) {};
        //拷贝复制操作符
        Widget& operator=(const Widget& rhs)
        {
            WidgetImpl* temp = pImpl;
            pImpl = new WidgetImpl(*rhs.pImpl);
            delete temp;
            return *this;
        }
        //成员版swap
        void swap(Widget& rhs)
        {
            using std::swap;//using 声明式让std::swap提前曝光
            swap(pImpl, rhs.pImpl);
        };
        void printWidget()
        {
            pImpl->printWidgetIpml();

        }
    private:
        WidgetImpl* pImpl;
    };
    //非成员版swap函数,Widget专属swap
    void swap(Widget& a, Widget& b)
    {
        a.swap(b);
    }
}

//在std命名空间特化swap
namespace std 
{
    //特化swap
    template<>
    void swap(widget::Widget& a, widget::Widget& b)
    {
        a.swap(b);
    }
}
int main()
{
    widget::Widget w1(1, 2, 3);
    widget::Widget w2(4, 5, 6);
    //提前曝光swap()
    using std::swap;
    swap(w1, w2);
    w1.printWidget();
    w2.printWidget();
    system("pause");
    return 0;
}
```



条款26 尽可能延后变量定义式的出现时间

不只应该延后变量的定义，直到非得使用该变量的前一刻为止，甚至应该尝试延后这份定义直到能够给它初值实参为止。

当1.知道赋值成本比“构造+析构”成本低，或者2.正在处理的代码中效率高度敏感的部分，应该将变量定义在循环体内，除此之外，尽量将变量定义在循环体内。

```c++
//变量定义于循环外
Widget w;
for(int i = 0; i < n; i++)
{
    w = 取决于i的某个值;
}
//成本：1个构造函数+1个析构函数+n个赋值函数
```

```c++
//变量定义于循环内
for(int i = 0; i < n; i++)
{
    Widget w(取决于i的某个值);
}
//成本：n个构造函数+n个析构函数
```



条款27 尽量少做转型动作

旧式转换：

```c++
(T)expression //将expression转换为T

T(expression)//将expression转换为T


```

新式转换：

```c++
const_cast<T>(expression)
dynamic_cast<T>(expression)
reinterpret_cast<T>(expression)
static_cast<T>(expression)
```

const_cast通常被用来将对象的常量性转除。只有它能将const转换为non-const。

dynamic_cast主要执行安全向下转型，即用来决定某对象是否归属继承体系中的某个类型。它是唯一无法由旧式语法执行的动作，可能耗费重大运行成本。

reinterpret_cast执行低级转型，实际动作取决于编译器，这就表示它不可移植。

static_cast用来强迫隐式转换。

当一个基类指针指向派生类对象时，**有时候**上述两个地址不相同，即单一对象可能拥有一个以上的地址，这种情况下会有个偏移量在运行期被施行于派生类指针上，用来获取正确的基类指针。对象的布局方式和它们的地址计算方式随编译器的不同而不同。

如果确实需要转型，试着将它隐藏于某个函数背后，客户可以调用该函数，而不需将转型放进他们自己的代码内。

尽量使用C++新式转型，因为很容易辨识且有着分门别类的职责。



条款28 避免返回handles指向对象内部成分

引用、指针和迭代器统统都是handles。

第一，成员变量的封装性最多只等于”返回其handles“的函数的访问级别。第二，如果const成员函数传出一个handles，后者所指数据与对象自身有关联，而它又被存储与对象之外，那么这个函数的调用者可以修改那笔数据，与我们的设想不同。以上两个问题可以通过为他们的返回类型加上const解决。但第三、它可能导致空悬的handles——即这种handles所指东西的所属对象不存在。

应避免返回handles指向对象内部。



条款29 为“异常安全”而努力是值得的

当异常被抛出时，带有异常的函数会：1.不泄露任何资源。2.不允许数据破坏。

异常安全函数需提供以下三个保证之一：

​	1.基本承诺——如果异常被抛出，程序内的任何事物仍然保持在有效状态下，但程序的现实状态用户不可预料。

​	2.强烈保证——如果异常被抛出，程序状态不改变，即不成功程序回复到调用函数之前的状态。

​	3.不抛掷保证——承诺不抛出异常，因为总是能够完成它们原先承诺的功能。内置类型（ints，指针）身上所有操作都提供不抛掷保证。

任何使用动态内存的东西（例如所有STL容器）如果无法找到足够内存以满足需求，通常会抛出一个bad_alloc异常（条款49）。

大多数函数抉择往往落在基本保证和强烈保证之间。

copy and swap会导致“强烈保证”，copy and swap——为你打算修改的对象做出一份副本，然后在副本上做出修改，修改成功后再将副本和原对象在一个不抛出异常的操作中置换（swap）。

当一个想要实现“强烈保证”函数调用了其他多个函数或者实现“强烈保证”效率太低时，“强烈保证”就不太实际了，需要保证“基本承诺”。

如果一个系统内有一个函数不具备异常安全性，整个系统就不具备异常安全性。

函数提供的“异常安全保证”通常**最高**只等于其所调用之各个函数“异常安全保证”中的最弱者，例如某函数调用两个“强烈保证”函数，一个函数完成，一个函数异常那这个函数“异常安全保证”就不是“强烈保证”。



条款30 透彻了解inlining的里里外外

inline只是对编译器的一个申请，不是强制命令，编译器可加以忽略。

将函数**定义于class定义式**内可隐喻提出inline函数。

inline函数被定义于头文件内，因为大多数建置环境在编译过程中进行inling，如果编译器无法将函数inline化，会给出一个警告信息。

templates通常也被置于头文件内，因为它一旦使用，编译器为了将它具现化，需要知道它长什么样子。

在inline函数内调用virtual函数会使inlining落空，因为virtual意味着"等待，直到运行期才确定调用哪个函数"，而inline意味着”执行前，先将调用动作替换为被调用的函数本体“。

编译器通常不对”通过函数指针而进行的调用“实施inlining。

构造函数和析构函数往往是inlining的糟糕候选人，因为编译器在编译期间在其中做出一些必须的行为。

将大多数inlining限制在小型、被频繁调用的函数身上。



条款31 将文件间的编译依存关系降至最低

[怎样削减C++代码间依赖 - SegmentFault 思否](https://segmentfault.com/a/1190000002790854)讲的挺清楚

如果头文件有任何改变，或者这些头文件所依赖的其他头文件有任何改变，那么每个包含这个文件的文件就得重新编译，任何使用这个文件的文件也得重新编译。

为降低依赖关系，将类分割为两个类，一个只提供接口，另一个负责实现接口。接口类只内含一个指针成员指向实现类的设计称为pimpl idiom。

如果使用对象引用或者对象指针可以完成任务，就不要使用对象本身。

如果能够，尽量以class声明式替换class定义式。

为声明式和定义式提供不同的头文件。

使用pimpl idiom的class往往被称为handle class。还有一种方法是令class成为一种特殊的抽象基类，成为interface class。Interface class通常调用一个特殊函数，此函数扮演“真正被具现化”的那个derived classes的构造函数，它们返回指针（或更为可取的智能指针）指向动态分配所得的对象。这样的函数往往在Interface class内被声明为static：

```c++
//Interface class
class Person
{
public:
    virtual ~Person();
    virtual string name()const = 0;
    virtual string birthDate()const = 0;
    virtual string address()const = 0;
    static std::shared_ptr<Person> //返回一个shared_ptr，指向一个新的Person，并以给定之参数初始化。
        create(const string& name,
              const Date& birthday,
              const Address& addr);

}

//Interface class的具象的派生类
class RealPerson:public Person{
public:
    RealPerson(const string& name,const Date& birthday,const Address& addr)
        :thName(name),theBirthDate(birthday),theAddress(addr)
        {}
    virtual ~RealPerson(){};
    string name()const;
    string birthDate()const;
    string address()const;
private:
    string name;
    Date theBirthDate;
    Address theAddress;
}
```

有了RealPerson之后，就可以写出Person::create了

```c++
static std::shared_ptr<Person> create(const string& name,const Date& birthday,const Address& addr)
{
    return 
        shared_ptr<Person>(new RealPerson(name,birthday,addr));
}
```



条款32 确定你的public继承塑膜出is-a（是一种）关系

public继承意味着is-a。适用于基类身上的每一件事情也一定适用于派生类身上，因为每一个派生类对象也都是一个基类对象。正方形类不应继承长方形类，因为长方形的宽度改变之后高度可以不改变，但正方形不满足这件事。



条款33 避免遮掩继承而来的名称

派生类内的名称会遮掩基类内的名称，在public继承下从来没有人希望如此，可以在派生类中使用using声明式（令继承而来的给定名称的所有同名函数在派生类中都可见）或者转交函数（调用基类中某一特定同名函数）使基类同名函数可用。

转交函数：

```c++
class Derived: private Base{
public:
    virtual void mf1()//转交函数，暗自成为inline
    {
        Base::mf1();
    }
}
```



条款34 区分接口继承和实现继承

纯虚函数有两个最突出的特性：它们必须被任何”继承了它们“的具象类重新声明，而且它们在抽象类中通常没有定义。

声明一个纯虚函数的目的是为了让派生类只继承函数接口。

声明简朴的非纯虚函数的目的，是让派生类继承该函数的接口和缺省实现。

声明非虚函数的目的是为了令派生类继承函数的接口及一份强制性实现。不应在派生类中重新定义。



条款35 考虑virtual函数以外的其他选择

virtual函数的替代方案：

1.使用非虚interface（NVI）手法，那是Template Method设计模式的一种特殊形式，它用public非虚成员函数包裹较低访问性（private或protected）的虚函数。把public非虚成员函数称为virtual函数的外覆器，外覆器确保得以在一个虚函数调用之前设定好适当场景，并在调用结束之后清理场景。

2.将virtual函数替换为“函数指针成员变量”，在构造函数中初始化这个函数指针，这是Strategy设计模式的一种分解表现形式。优点是每个对象可拥有自己的函数实现方式，缺点是函数如果需要访问类内非public成员，需降低类的封装性。

3.以function成员变量替换virtual函数，因而允许使用任何可调用物搭配一个兼容于需求的签名式。这也是Strategy设计模式的某种形式。

4.将继承体系内的virtual函数替换为另一个继承体系内的virtual函数。这是Strategy设计模式的传统实现手法。



条款36 绝不重新定义继承而来的non-virtual函数

类内的非虚函数都是静态绑定，基类的一个指针或引用调用的非虚函数永远都是基类定义的版本，即使基类指针指向一个派生类对象。

虚函数是动态绑定，基类指针或引用指向一个派生类对象时调用的虚函数是派生类定义的虚函数。

任何情况下都不该重新定义一个继承而来的非虚函数。



条款37 绝不重新定义继承而来的缺省参数值

当继承一个带有缺省参数值的virtual函数，virtual函数是动态绑定，而缺省参数值是静态绑定。

静态类型：在程序中被声明时所采用的类型。

动态类型：目前所指对象的类型。

绝对不要重新定义一个继承而来的缺省参数值。基类指针指向一个派生类对象，调用一个派生类内的虚函数，即使派生类内虚函数重写缺省参数值，也会使用基类为它所指定的缺省参数值。



条款38 通过复合塑膜出has-a或”根据某物实现出“

复合是类型之间的一种关系，当某种类型的对象内含它种类型的对象，便是这种关系。

在应用域，复合意味has-a（有一个）。在实现域，复合意味着is-implemented-in-terms-of（根据某物实现出）。



条款39 明智而审慎地使用private继承

如果classes之间的继承关系是private，编译器不会自动将一个派生类对象转换为一个基类对象。

由private基类继承而来的所有成员，在派生类中都会变成private属性。

**private继承意味着is-implemented-in-terms-of（根据某物实现出），与复合类似**。只有实现部分被继承，接口部分应略去，。

private继承比复合级别低，当你面对”并不存在is-a关系“的两个classes，其中一个需要访问另一个protected成员，或需要重新定义其一个或多个virtual函数，使用private继承是合理的。

和复合不同，private继承还可以造成empty base最优化。



条款40 明智而审慎地使用多重继承

为解决”钻石型多重继承“（菱形继承）问题，保证派生类内只有继承一个成员变量，令所有直接继承自它的classes采用”virtual继承“。

virtual继承会增加大小、速度、初始化（及赋值）复杂度等成本，如果virtual 继承基类不带任何数据，将是最具使用价值的情况。

多重继承的确有正当用途。书中一个情节涉及”public 继承某个Interface class“和”private 继承某个协助实现的class“的两相组合实例。



条款41 了解隐式接口和编译期多态

classes和templates都支持接口和多态。

对classes而言接口是显示的，以函数签名（函数名称、参数类型、返回类型）为中心。多态性是通过virtual函数发生于运行期。

对template参数而言，接口是隐式的，奠基于有效表达式。“以不同的template参数具现化”会导致调用不同的函数。多态通过template具现化和函数重载解析发生于编译期。



条款42 了解typename的双重意义

template内出现的名称如果相依于某个template参数，称之为**从属名称**。

如果从属名称在class内呈嵌套状，我们称它为**嵌套从属名称**。C::const_iterator就是这样一个名称。

```c++
template<typename C>
void print2nd(const C& container)
{
    if(container.size() >= 2)
    {
        C::const_iterator iter(container.begin());//非有效代码，不能通过编译，后续提出改进
        ++iter;
        int value = *iter;
        cout << value;
    }
}
```

一个并不依赖任何template参数的名称，称为谓非从属名称，例如上个template中的int类型的value。

**如果template中遭遇一个嵌套从属名称，它便假设这名称不是个类型**，除非你告诉它是，所以缺省情况下嵌套从属名称不是类型，若要加以矫正，需在之前放置typename关键字，告诉C++他是个类型。

```c++
template<typename C>
void print2nd(const C& container)
{
    if(container.size() >= 2)
    {
        typename C::const_iterator iter(container.begin());//有效代码
        ++iter;
        int value = *iter;
        cout << value;
    }
}
```

任何时候当你想要在template中指涉一个嵌套从属类型名称，就必须在紧邻它的前一个位置放上关键字typename，但typename不可以出现在base classes list 内的嵌套从属类型名称之前，也不可在member ititialization list（成员初值列）中作为base class修饰符。

```c++
template<typename T>
class Derived: public Base<T>::Nested//base class list不允许 typename
{
public:
    explicit Derived(int x):Base<T>::Nested(x)//member initialization list不允许 typename
    {
        typename Base<T>::Nested temp;//嵌套从属类型名称
    }
}
```

假如正在撰写一个function template，它应该接受一个迭代器，当打算为迭代器指涉的对象做一份局部副本，可以：

```c++
template<typename IterT>
void workWithIterator(IterT iter)
{
    typename std::iterator_traits<IterT>::value_type temp(*iter);
}
```

上述语句声明一个局部变量，使用IterT对象所指物的相同类型，并将temp初始化为iter所指物。如果IterT是vector<int>::iterator，temp的类型就是int。std::iterator_traits<IterT>::value_type是个嵌套从属类型名称。



条款43 学习处理模板化基类内的名称

```c++
template<typename Company>
class LoggingMsgSender:public MsgSender<Company>
{
public:
   void sendClearMsg(const MsgSender& info)
   {
       sendClear(info);//调用基类函数，这段代码无法通过编译
   }
}
```

当编译器遭遇派生类模板LoggingMsgSender定义式时，并不知道它继承什么样的基类，因为Company是个template，无法知道它是否有个sendClear函数，即使泛化版本有sendClear接口，但基类MsgSender可能有个特化版的MsgSender，特化版的MsgSender可能没有定义sendClear接口，所以C++拒绝在templatized base classes（模板化基类）内寻找继承而来的名称。

为令C++进入templatized base classes观察，对编译器承诺"基类模板的任何特化版本都将支持其泛化版本所提供的接口"有三种办法：

​	1.在基类函数调用之前加上"this->''。 

```c++
this->sendClear(info);//成立，假设sendClear被继承
```

​	2.使用using声明式。

```c++
using MsgSender<Company>::sendClear;//告诉编译器，请它假设sendClear位于base class内
void sendClearMsg(const MsgSender& info)
{
	sendClear(info);//成立，假设sendClear被继承
}
```

​	3.明白指出被调用的函数位于基类内。最糟糕的做法，因为如果被调用的时virtual函数，上述的明确资格修饰会关闭“virtual绑定行为”。

```c++
MsgSender<Company>::sendClear(info);//成立，假设sendClear被继承
```



条款44 将与参数无关的代码抽离templates

使用template可能会导致代码膨胀：其二进制代码带着重复的代码、数据，或两者。使用共性与变性分析解决代码膨胀。

假设想为固定尺寸的正方矩阵编写一个template，使用下述代码当操作同一类型，尺寸不同矩阵时，会具现化两份invert()，但其代码完全相同：

```c++
template<typename T,std::size_t n>
class SquareMatrix
{
public:
    void invert();//求逆矩阵
};
```

可以将**非类型参数**n以函数参数方式替代template：

```c++
template<typename T>
class SquareMatrixBase
{
protected:
    void invert(std::size_t matrixSize);//以给定的尺寸求逆矩阵
};
template<typename T,std::size_t n>
class SquareMatrix: private SquareMatrixBase<T>
{
private:
    using SquareMatrixBase<T>::invert;//避免遮掩base版的invert；见条款33
public:
    void invert(){ this->invert(n);}//制造一个inline调用，调用base class版的invert。基于条款43使用this->。
};
```

Templates生成多个classes和多个函数，所以任何template代码都不该与某个造成膨胀的template参数产生相依关系。

因非类型模板参数而造成的代码膨胀，往往可以消除，做法是**以函数参数或class成员变量替换template参数**。

因类型参数而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述的具体类型实现共享码。例如许多平台上的int和long。



条款45 运用成员函数模板接受所有兼容类型

STL容器的迭代器几乎总是智能指针。

真实指针支持隐式转换，派生类指针可以隐式转换为基类指针，指向non-const对象的指针可以转换为指向const对象。

```c++
class Top{...};
class Middle: public Top{...};
class Bottom: public Middle{...};
Top* pt1 = new Middle;//将Middle*转换为Top*
Top* pt2 = new Bottom;//将Bottom*转换为Top*
const Top* ptc2 = pt1;//将Top*转换为const Top*
```

但用户想在自定义的智能指针中模拟上述变换，稍稍有点麻烦。可以使用**Templates和泛型编程**为智能指针写一个构造函数模板，对任何类型T和任何类型U，这里可以根据SmartPtr<U>生成一个SmartPtr<T>，此**泛化拷贝构造函数**没有声明为explicit是因为效仿真实指针可以隐式转换：

```c++
template<typename T>
class SmartPtr
{
public:
    template<typename U>//成员模板为了生成拷贝构造函数
    SmartPtr(const SmartPtr<U>& other);
};
```

但如果我们需要根据一个SmartPtr<Bottom>创建一个SmartPtr<Top>，但不希望根据一个SmartPtr<Top>创建一个SmartPtr<Bottom>，可以使用成员初始列来初始化SmartPtr<T>之内类型为T *的成员变量，并以类型U *的指针作为初值，这个行为只有当“某个隐式转换可将一个U *指针转换为一个T *指针”时才能通过编译：

```c++
template<typename T>
class SmartPtr
{
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other)
    :heldPtr(other.get())//以other的heldPtr初始化this的heldPet
    {}
    T* get() const {return heldPtr;}
private:
    T* heldPtr;//这个SmartPtr持有的内置指针
};
```

在class内声明泛化拷贝构造函数并不会阻止编译器生成它们自己的拷贝构造函数。如果想控制拷贝构造的方方面面，必须同时声明泛化拷贝构造和正常的拷贝构造，对赋值操作符也同样适用。



条款46 需要类型转换时请为模板定义非成员函数

本条款基于条款24讨论，将Rational和operator*模板化之后：

```c++
template<typename T>
class Rational
{
public:
    Rational(const T& numerator = 0, const T& denominator = 1);
    const T numerator() const;
    const T denominati() const;
};

template<typename T>
const Rational<T> operator*(const Rational<T>& lhs,const Rational<T>& rhs)
{...}
```

下列代码无法通过编译，因为编译器不知道调用哪个函数，operator*的第二参数为Rational<T>，但传递给operator *的第二实参类型（2）是int，template实参推导从不将隐式类型转换函数纳入考虑：

```c++
Rational<int> oneHalf(1,2);
Rational<int> result = oneHalf*2;//无法通过编译
```

但**Class template并不依赖template实参推导（实参推导只施行于function template上）**，所以编译器总是能在class Rational<T>具现化时得知T。将Rational<T> class声明适当的operator* 为其**友元函数**，在类外定义后，**编译通过，但连接不通过**，因为具现化后找不到具现化后的函数定义。所以需将友元函数的声明式和定义式合并于类内，自动成为inline函数：

```c++
template<typename T>
class Rational
{
public:
    friend const Rational operator*(const Rational<T>& lhs,const Rational<T>& rhs)//友元函数，自动inline
    {
        return Rational(lhs.numerator()*rhs.numerator(),lhs.denominator()*rhs.denominator());
    }
    Rational(const T& numerator = 0, const T& denominator = 1);
    const T numerator() const;
    const T denominati() const;
};
```

同样如果要抵抗inline，可以在类外声明一个template函数，友元函数调用这个函数即可。

当我们编写一个class template，而它提供的“于此template相关的”函数支持“所有参数之隐式类型转换”时，请为那些函数定义为“class template内部的友元函数”。



条款47 请使用traits classes表现类型信息

STL中有一个工具性template是advance用来将某个迭代器移动某个距离。

**Input Iterator**：只能单步向前迭代元素，不允许修改由该类迭代器引用的元素，且只能读一次。
**Output Iterator**：该类迭代器和Input Iterator极其相似，也只能单步向前迭代元素，不同的是该类迭代器对元素只有写的权力，只能写一次。
**Forward Iterator**：该类迭代器可以在一个正确的区间中进行读写操作，它拥有Input Iterator和Output Iterator的所有特性，以及单步向前迭代元素的能力。
**Bidirectional Iterator**：该类迭代器是在Forward Iterator的基础上提供了单步向后迭代元素的能力。
**Random Access Iterator**：该类迭代器能完成上面所有迭代器的工作，它自己独有的特性就是可以像指针那样进行算术计算，而不是仅仅只有单步向前或向后迭代。

如果想利用迭代器类别的不同用不同方式实现advance,我们需要利用traits 在编译期取得某些类型信息：

```c++
template<typename IterT, typename DistT>
void advance(IterT& iter,DistT& d)
{
    if( iter is a random access iterator)//针对随机迭代器
    {
        iter += d;
    }
    else
    {
        if(d >= 0)while(d--) {++iter};
        else {while(d++) --iter;}
    }
}
```

Traits并不是C++关键字或一个预先定义好的构件，它们是一种技术，也是一个C++程序员共同遵守的协议。这个技术的要求之一是：它们对内置类型和用户自定义类型表现必须一样好。标准技术是把它放进一个template及其或多个特化版本中，这样的templates在标准程序库中有若干个，其中针对迭代器的被命名为iterator_traits，习惯上traits总是被实现为structs，iterator_traits的运作方式是针对每一个类型IterT，在struct iterator_traits<IterT>内一定声明某个typedef名为iterator_category，用来确认IterT的迭代器分类。

首先对每一个“用户自定义的迭代器类型”必须**嵌套**一个typedef，名为iterator_category，例如list迭代器支持双向迭代：

```c++
template<...>//略写参数
class list
{
public:
    class iterator{
    public:
        typedef bidirectional_iterator_tag iterator_category;
    ...
    }
}
```

**对于iteration_traits，只需要响应iterator class的嵌套式typedef**：

```c++
template<typename IterT>
struct iterator_traits
{
    typedef typename IterT::iterator_category iterator_category;//条款42解释为什么加typename
}
```

**因为指针不能嵌套typedef，为支持指针迭代器，iterator_traits特别针对指针类型提供一个偏特化版本**:

```c++
template<typename IterT>
struct iterator_traits<IterT*>
{
    typedef random_access_iterator_tag iterator_category;//条款42解释为什么加typename
}
```

**所以设计并实现traits class的步骤如下：**

​	1.确认若干你希望将来可取得的类型相关信息。对迭代器而言，我们希望可取得其分类（category）。

​	2.为该信息选择一个名称（例如iterator_category）

​	3.提供一个template和一组特化版本（例如iterator_traits），内含希望支持的类型相关信息。

**if语句编译期才能确定，所以为了在编译期间**就可完成对类型核定的功能，**使用函数重载**doAdvance实现advance：

```c++
template<typename IterT,typename DistT>//用于实现random access迭代器
void doAdvance(IterT& iter,DistT& d,std::random_access_iterator_tag)
{
    itet += d;
}

template<typename IterT,typename DistT>//用于实现birdirectional迭代器
void doAdvance(IterT& iter,DistT d,std::birdirectional_iterator_tag)
{
    if(d >= 0){while(d--) ++iter;}
    else{while(d++)--iter;}
}

template<typename IterT,typename DistT>
//用于实现input迭代器,因为forward迭代器继承自input迭代器，所以此版本也能处理forward迭代器
void doAdvance(IterT& iter,DistT d,std::input_iterator_tag)
{
    if(d >= 0){while(d--) ++iter;}
    else{throw std::out_of_range("Negative distance");}
}

//于是advance可以实现为：
template<typename IterT,typename DistT>
void advance(IterT& iter,DistT d)
{
    doAdvance(iter,d,typename std::iterator_traits<IterT>::iterator_category);
}
```

**使用traits class的步骤：**

​	1.建立一组重载函数或函数模板（doAdvance），彼此间差异只在于各自的traits参数。令每个参数实现码与其接受traits的信息相应和。

​	2.建立一个控制函数或函数模板（advance）,调用上述函数并传递traits class所提供的信息。

Traits除了供应iterator_category还供应另四份迭代器相关信息，value_type最有用。此外char_traits用于保存字符类型相关信息，numeric_limits用来保存数值类型相关信息。

**Traits class是的"类型相关信息"在编译期间可用。**以templates和”templates特化“完成实现。

**整合重载技术，traits classes有可能在编译期对类型执行if...else测试。**



条款48 认识template元编程

TMP模板元编程是编写template-baseC++程序并执行于编译期的过程。

编译器必须确保所有源码有效，即使不会执行起来的代码，当iter不是random_access迭代器时“iter+=d”无效。

TMP并没有真正的循环结构，循环借由递归完成。

TMP的作用：

​	1.确保度量单位正确。

​	2.优化矩阵运算。

​	3.可以从生成客户定制的设计模式。



条款49 了解new-handler的行为

当operator new抛出异常以反映一个未获满足的内存需求之前，它会先调用一个客户指定的错误处理函数，即new-handler。为了指定这个“用以处理内存不足”的函数，客户必须调用set_new_handler，那是声明于<new>的一个标准程序库函数。

```c++
namespace std
{
    typedef void (*new_handler)();
    new_handler set_new_handler(new_handler p) throw();
}
```

set_new_handler则是“获得一个new_handler并返回一个new_handler”的函数。set_new_handler声明式尾端的“throw()”是一份异常明细，表示该函数不抛出任何异常。

set_new_handler的参数是个指针，指向operator new无法分配足够内存时该被调用的函数。返回值也是个指针，指向set_new_handler被调用前正在执行（但马上就要被替换）的那个new_handler函数。

当operator new无法满足内存申请时，它会不断调用new_handler函数，直到找到足够内存，一个设计良好的new_handler函数必须做以下事情：

​	让更多内存可被使用

​	安装另一个new_handler

​	卸除new_handler

​	抛出bad_alloc的异常。这样的异常不会被operator new捕捉，因此会被传播到内存索求处

​	不返回，通常调用abort或exit

C++并不支持class专属的new_handler，但可以令每一个class提供自己的set_new_handler和operator new即可。

```c++
class Widget
{
public:
    static std::new_handler set_new_handler(std::new_handler p)throw();
    static void* operator new(std::size_t size) throw(std::bad_alloc);
private:
    static std::new_handler currentHandler;
}
std::new_handler Widget::currentHandler = 0;//在class实现文件内初始化为null
std::new_handler Widget::set_new_handler(std::new_handler p) throw()
{
    std::new_handler oldHandler = currentHandler;
    currentHandler = p;
    return oldHandler;
}
```

Widget的operator new需要做：

1.调用标准set_new_handler，告知Widget的错误处理函数。这会将Widget的new_handler安装为global new_handler。

2.调用global operator new，执行实际的内存分配。如果分配失败，由第一步知global operator new调用Widget的new_handler。如果无法分配足够内存，抛出一个bad_alloc异常，此情况下Widget的operator new必须恢复原本的global new_handler，然后再传播该异常。为确保new_handler总是被重新安装回去，Widget将global new_handler视为资源并遵守条款13，运用资源管理对象防止资源泄露。

3.如果global operator new能完成分配内存，Widget的operator new会返回一个指针，指向分配所得。Widget析构函数会管理global new_handler，他会自动将Widget’s operator new被调用前的global new_handler恢复过来。

```c++
class NewHandlerHolder{
public:
    explicit NewHandlerHolder(std::new_handler nh)//取得目前new_handler
        :handler(nh){}
    ~NewHandlerHolder()//释放它
    {
        std::set_new_handler(handler);
    }
private:
    std::new_handler handler;//记录下来
    NewHandlerHolder(const NewHandlerHolder&);//阻止copying
    NewHandlerHolder& operator=(const NewHandlerHolder&);
}
void* Widget::operator new(std::size_t size) throw(std::bal_alloc)
{
    NewHandlerHolder h(std::set_new_handler(currentHandler));//安装Widget的new_handler
    return ::operator new(size);//分配内存或抛出异常，恢复global new_handler
}
```

实现这个方案的代码不因class的不同而不同，所以可以建立一个“mixin”风格的base class，此种base class允许派生类继承单一特定能力——本例中是“设定class专属new_handler”能力。

```c++
template<typename T>
class NewHandlerSupport{//“mixin”风格的base class
{
public:
    static std::new_handler set_new_handler(std::new_handler p)throw();
    static void* operator new(std::size_t size) throw(std::bad_alloc);
private:
    static std::new_handler currentHandler;
}
    
template<typename T>
std::new_handler NewHandlerSupport::currentHandler = 0;//在class实现文件内初始化为null

template<typename T>
std::new_handler NewHandlerSupport::set_new_handler(std::new_handler p) throw()
{
    std::new_handler oldHandler = currentHandler;
    currentHandler = p;
    return oldHandler;
}
    
template<typename T>
void* NewHandlerSupport::operator new(std::size_t size) throw(std::bal_alloc)
{
    NewHandlerHolder h(std::set_new_handler(currentHandler));//安装new_handler
    return ::operator new(size);//分配内存或抛出异常，恢复global new_handler
}  
```

为Widget添加set_new_handler只要令Widget继承自**NewHandlerSupport<Widget>**就好。

Nothrow new 是一个颇为局限的工具，因此它只适用于内存分配。



条款50：了解new和delete的合理时机

Inter x86体系结构上doubles可被对齐于任何byte边界，但如果它是8-byte齐位，齐访问速度会快许多。32位操作系统采用4byte对齐，64位操作系统采用8byte对齐。

C++要求operator news返回的指针都有适当的对齐（取决于数据类型）。

Boost程序库的Pool对于常见的“分配大量小型对象”很有帮助。

合理替换缺省的new和delete：

​	1.为了检测运用错误

​	2.为了收集动态分配内存之使用统计信息。

​	3.为了增加分配和归还的速度。

​	4.Class专属分配器是“区块尺寸固定”的分配器实例，例如Boost提供的Pool程序库便是。

​	5.为了降低缺省内存管理器带来的空间额外开销。

​	6.为了弥补缺省分配器中的非最佳齐位。

​	7.为了将相关对象成簇集中。

​	8.为了获得非传统行为。
