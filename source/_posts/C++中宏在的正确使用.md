title: C++中宏在的正确使用
tags: [C++]
date: 2015-10-12 16:19:02
description: 宏在c++里面应该是最有个性的一个特点，虽然只是简单的文本替换，用的巧妙，它可以让代码行数变短，如果乱用，则代码会完全丧失可读性
---

宏在c++里面应该是最有个性的一个特点，虽然只是简单的文本替换，但
> 用的巧妙，它可以让代码行数变短，如果乱用，则代码会完全丧失可读性

正确的使用宏非常重要,宏主要作用是两点：
- 用来表达一般语法无法表达的内容，如通过一行语句，简洁的定义一个自定义类
- 用来标示一种状态，根据不同的状态来减少编译代码

# 尽量不使用宏
文本替换操作可以使用的场景很多，但是代码里面到处是宏，其他开发人员也很难阅读代码，《Effective C++》里面明确表示通过const、emum、inline语法来替代宏的使用。
如下所示，相对于VAR_INT宏，使用VAR_INT会写入符号表，编译起来报错会有“VAR_INT”信息，而不是单纯的一个“1”

```c++
#define VAR_INT 1
const int VAR_INT = 1;
```

# 定义新的类或函数
那什么时候应该使用宏呢？有些内容，比如定义一个新的类或函数，c++里面只有模版可以对一个类进行抽象，还是有一些需求无法满足，使用宏会让代码更加简洁。
如下面代码，单元测试框架代码，用P\_TEST\_宏来定义个一个**部分**新测试类，每次定义时用register_test_case方法来自动注册到TestManager中（也可以选择在构造函数中，但是这样要先new这个对象），这样就可以在main函数中执行队列中所有的测试案例
```c++
namespace ptest {

class TestClass{
    //...
};

//register test case to TestManager
TestCase* register_test_case(const char* test_case_name, const char* test_name,TestFactoryBase* factory);

class TestFactoryBase{
public:
    virtual TestCase* CreateTest()=0;
};
    
template <class TestClass>
class TestFactoryImpl : public TestFactoryBase{
public:
    virtual TestCase* CreateTest() { return new TestClass; }
};

}

#define P_TEST_CLASS_NAME_(test_case_name, test_name) \
test_case_name##_##test_name##_Test

#define P_TEST_(test_case_name, test_name, parent_class)\
class P_TEST_CLASS_NAME_(test_case_name, test_name) : public parent_class {\
public:\
    P_TEST_CLASS_NAME_(test_case_name, test_name)() {}\
private:\
    virtual void Run();\
    static ptest::TestCase* test_case_;\
};\
\
ptest::TestCase* P_TEST_CLASS_NAME_(test_case_name, test_name)::test_case_ =\
ptest::register_test_case(#test_case_name, #test_name, \
new ptest::TestFactoryImpl<P_TEST_CLASS_NAME_(test_case_name, test_name)>);\
void P_TEST_CLASS_NAME_(test_case_name, test_name)::Run()

#define PTEST(test_case_name, test_name) \
P_TEST_(test_case_name, test_name, ptest::TestCase)


```
利用宏定义一个测试案例
```c++
PTEST(MyTestCase, Macro_Test_Name){
    assert(1,3);
}
```
执行所有测试案例
```c++
int main()
{
    ptest::TestManager::GetInstance()->Run();
    return 0;
}
```
# 内置宏
内置宏就是来表达一种状态，表达在编译时刻，编译环境的一些信息
## 操作系统
表达编译时刻的操作系统：比如\_\_WIN64\_\_代表64位windows；\_\_CYGWIN\_\_代表windows上面的CYGWIN虚拟环境；ANDROID代表Android环境；\_\_linux\_\_或者\_\_linux代表linux环境
## CPU类型
表达编译时刻的CPU架构类型：比如\_\_arm\_\_表达移动平台arm芯片；\_\_i386\_\_表示的i386架构；\_\_x86\_64\_\_表示x86架构
## 编译器类型
表达编译代码的编译器类型：比如\_\_GNUC\_\_表示GNU C++编译平台；\_MSC\_VER表示windows的Microsoft Visual C/C++平台；\_\_clang\_\_表示Mac下面的LLVM编译平台
## C++实现版本
表达实现C++规范的版本，也可以用来区分C和C++，根据__cplusplus来判断是否支持某种特性，如qt源码中qcompilerdetection.h的代码片段，表示如果C++版本晚于是2011年3月份，intel的编译器值大于1200，则支持LAMBDA等特性
```c++
#  if __cplusplus >= 201103L || defined(__INTEL_CXX11_MODE__)
#    if __INTEL_COMPILER >= 1200
#      define Q_COMPILER_AUTO_TYPE
#      define Q_COMPILER_CLASS_ENUM
#      define Q_COMPILER_DECLTYPE
#      define Q_COMPILER_DEFAULT_MEMBERS
#      define Q_COMPILER_DELETE_MEMBERS
#      define Q_COMPILER_EXTERN_TEMPLATES
#      define Q_COMPILER_LAMBDA
#      define Q_COMPILER_RVALUE_REFS
#      define Q_COMPILER_STATIC_ASSERT
#      define Q_COMPILER_VARIADIC_MACROS
#    endif
#   endif
```

<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>