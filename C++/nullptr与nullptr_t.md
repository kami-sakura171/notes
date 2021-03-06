#### **一. nullptr与nullptr_t**

（一）**nullptr_t是一种数据类型，而nullptr是该类型的一个实例**。通常情况下，也可以通过nullptr_t类型创建另一个新的实例。

（二）所有定义为nullptr_t类型的数据都是等价的，行为也是完全一致的。

（三）**std::nullptr_t类型，并不是指针类型**，但**可以隐式转换成任意一个指针类型**（注意不能转换为非指针类型，强转也不行）。

（四）**nullptr_t**类型的数据不适用于算术运算表达式。但**可以用于关系运算表达式**（仅能与nullptr_t类型数据或指针类型数据进行比较，当且仅当关系运算符为==、<=、>=等时）

【编程实验】nullptr与nullptr_t的关系

```c++
#include <iostream>
#include <vector>
#include <iomanip>  //for setbase;
#include <boost/type_index.hpp>

using namespace std;
using boost::typeindex::type_id_with_cvr;

//辅助类模板，用于打印T的类型
template <typename T>
void printType(string s)
{
    cout << s << " = " << type_id_with_cvr<T>().pretty_name() << endl;
}

//4. 用于测试nullptr并不是指针类型
template<typename T>
void func_ptr(T* t) {}

template<typename T>
void func_value(T t) {}

int main()
{
    //1. 查看nullptr的类型
    printType<decltype(nullptr)>("nullptr的类型: ");  //std::nullptr_t
    cout << "sizeof(nullptr) = " << sizeof(nullptr) << endl;

    //2. nullptr可以隐式转换为任何指针！
    void* vptr = nullptr;
    char* cptr = nullptr;

    //3. 使用关系运算符进行比较nullptr与nullptr_t类型的变量
    //3.1 nullptr支持关系运算
    nullptr_t my_nullptr = nullptr; //通过nullptr_t定义新的实例(vc下必须初始化，但g++默认下己初始化为nullptr)

    (nullptr == my_nullptr) ? (cout << "newptr == nullptr" << endl) : (cout << "newptr != nullptr" << endl);
    (nullptr < my_nullptr) ? (cout << "newptr < nullptr" << endl) : (cout << "newptr !< nullptr" << endl);

    //3.2 nullptr不支持算术运算
    //nullptr += 1; //nullptr是个常量
    //nullptr * 5;

    //4. nullptr_t并不是指针类型（尽管看起来、用起来都像指针类型）
    func_ptr((float*)nullptr); //ok， T = float。nullptr可以转为任意类型的指针
    //func_ptr(nullptr);       //编译失败，nullptr的类型是nullptr_t，而不是指针类型
                               //编译器并不会“智能”地推导成某种类型的指针（含void*）

    func_value(0);           // T = int;
    func_value(nullptr);     //T = nullptr_t;
    func_value((float*)nullptr);  //T = float*

    return 0;
}
/*输出结果
nullptr的类型:  = std::nullptr_t
sizeof(nullptr) = 4
newptr == nullptr
newptr !< nullptr
*/
```

#### **二、nullptr与NULL**

##### （一）nullptr与NULL的区别

　　1. **NULL****是一个宏定义，C++中通常将其定义为0**，**编译器总是优先把它当作一个整型常量**（C标准下定义为(void*）0)。

　　2. **nullptr****是一个编译期常量，其类型为nullptr_t**。**它既不是整型类型，也不是指针类型**。

　　3. 在**模板推导**中，**nullptr被推导为nullptr_t类型**，仍可隐式转为指针。**但0或NULL则会被推导为整型类型**。

　　4.**要避免在整型和指针间进行函数重载**。因为NULL会被匹配到整型形参版本的函数，而不是预期的指针版本。

##### （二）nullptr与(void*)0的区别

　　1. **nullptr****到任何指针的转换是隐式的**。（尽管nullptr不是指针类型，但仍可当指针使用）

　　2.**(void\*)0**只是一个强制转换表达式，其返回void*指针类型，**只能经过类型转换到其他指针才能用**。

【编程实验】nullptr与NULL的区别

```c++
#include <iostream>
#include<memory>
#include <mutex>
#include <iomanip>  //for setbase;
using namespace std;

//3. 避免整型和指针间的重载
void f(int)
{
    cout <<"invoke f(int)" << endl;
}

void f(void*)
{
    cout << "invoke f(void*)" << endl;
}

//4. nullptr在模板中的表现
class Widget{};
int f1(std::shared_ptr<Widget> spw) { return 0; }
double f2(std::unique_ptr<Widget> upw) { return 0; }
bool f3(Widget* pw) { return true; }
using MuxGuard = std::lock_guard<std::mutex>;

template<typename Func, typename Mux, typename Ptr>
decltype(auto) lockAndCall(Func func, Mux& mux, Ptr ptr) //C++14
{
    MuxGuard guard(mux);
    return func(ptr);
}

int main()
{
    //1. nullptr是一个右值常量
    nullptr_t my_nullptr;

    cout <<"&my_nullptr = "<< setbase(16) << &my_nullptr << endl;    //nullptr_t类型的对象取地址
    //cout << setbase(16) << &nullptr<< endl;  //nullptr是个右值常量，不能取地址。
    const nullptr_t&& def_nullptr = nullptr; //nullptr是个右值常量，可用右值引用来接。
    cout << "&def_nullptr = " << setbase(16) << &def_nullptr << endl; //可对右值引用取地址（具名变量，本身是左值）

    //2. nullptr与(void*)0的区别
    void* px = NULL;
    //int* py = (void*)0;  //编译错误，不能隐式将void*转为int*类型
    int* pz = (int*)px;    //void*不能隐式转为int*，必须强制转换！

    int* pi = nullptr;     //ok！nullptr可以隐式转为任何其他指针类型
    void* pv = nullptr;    //ok! nullptr可以隐式转为任何其他指针类型

    //3. 避免在整型和指针间重载函数
    f(0);        //invoke f(int)
    f(NULL);     //invoke f(int)
    f((char*)0); //invoke f(void*)
    f(nullptr);  //invoke f(void*)

    //4. nullptr的模板推导中，仍可当指针用。
    std::mutex m1, m2, m3;
    //auto result1 = lockAndCall(f1, m1, 0);    //0被推导为int类型，与share_ptr<Widget>类型不匹配
    //auto result2 = lockAndCall(f2, m2, NULL); //同上，NULL被优先当作整型
    auto result3 = lockAndCall(f3, m3, nullptr);//nullptr被推导为nullptr_t，但该类可以隐式转为Widget*

    return 0;
}
/*输出结果
&my_nullptr = 00DEFB28
&def_nullptr = 00DEFB10
invoke f(int)
invoke f(int)
invoke f(void*)
invoke f(void*)
*/
```

