# my_blog

Google Test 是一套编写C++单元测试的框架，可以运行在很多平台上。

## 如何安装
```
$ git clone https://github.com/google/googletest.git
$ cd googletest
$ mkdir build
$ cd build
$ cmake ..
$ make
$ sudo make install
```

安装之后，如何在代码中使用呢？用一个Cmake脚本说明一下：
```
cmake_minimum_required (VERSION 2.8)

project(gtest_demo)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -std=c++11 -Wall")

find_package(GTest REQUIRED)

find_library(GMOCK
NAMES gmock
PATH /usr/local/lib /usr/lib)

include_directories(${GTEST_INCLUDE_DIRS})
add_executable(MyTests test_main.cpp test.cpp)
target_link_libraries(MyTests ${GMOCK} ${GTEST_BOTH_LIBRARIES})
add_test(Test MyTests)
enable_testing()
```

OK，上面的cmake还引入了gmock，gmock中包含了gtest，它做的事情更加酷炫(例如mock、Hamcrest断言等等)。

接着编写`test_main.cpp`文件
```
#include <gmock/gmock.h>
#include <iostream>

int main(int argc, char* argv[])
{
::testing::InitGoogleMock(&argc, argv);
RUN_ALL_TESTS();
}
```

在`test.cpp`中编写测试的代码
```
#include <gmock/gmock.h>
using namespace testing;

TEST(AddTest, OnePlusOneEqTwo)
{
ASSERT_THAT(1+1, Eq(2));
}
```
## 断言
本质上，GTest通过断言帮助你构建自动化测试的代码。断言的结果有三种：成功、非终止失败和终止失败。其中，终止失败会终止当前的测试函数

断言有两种，
+ ASSERT_* 失败是会终止当前函数，ASSERT_* 后面的代码将不会运行
+ EXPECT_* 失败时不会终止，EXPECT_* 后面的代码将会继续运行

通常，一个测试单元如果只有一个断言，使用ASSERT_* ，如果有多个断言则使用 EXPECT_*

### 经典式断言
GTest支持两种断言形式，一种是经典式的断言。它们沿袭了SUnit的风格，使用这种形式并无不妥，但是你或许会考虑形式更加多样的断言形式，即 Hamcrest 断言。由于你会遇到大量的经典式代码，因此学习这两种断言是十分必要。

下表列出了经典式断言的两个主要断言。其他框架也会使用类似的名称
| 形式 | 描述 | 实例 |
| ------ | ------ | ------ |
| ASSERT_TRUE(表达式) | 表达式返回假（或者0），测试失败 | ASSERT_TRUE(4<7) |
| ASSERT_EQ(期待值，实际值) | 期待值和实际值不等时，测试失败 | ASSERT_EQ(4, 20/5) |

GTest还提供了一些额外的断言形式来提高表达能力，ASSERT_* 一共有八种断言形式，分别是：`ASSERT_TRUE()`、`ASSERT_FALSE()`、`ASSERT_EQ()`、`ASSERT_NE()`、`ASSERT_LT()`、`ASSERT_LE()`、`ASSERT_GT()`和`ASSERT_GE()`。

EXPECT_* 与之类似，也有八种。

### Hamcrest 断言
Hamcrest 断言是为了提高测试的表达能力，创建复杂断言的灵活性，以及测试错误所提供的信息

Hamcrest 断言使用**匹配器**比较实际结果。匹配器可以组成复杂但易懂的比较表达式。你也可以自己定义匹配器。

几个简单的示例胜过千言万语

```
TEST(StringTest, StringEq)
{
string actual = string("al") + "pha";
ASSERT_THAT(actual, Eq("alpha"));
}
```

断言可以从左到右读作：断定实际值等于"alpha"。对比与 `ASSERT_EQ(actual, "alpha")`，Hamcrest 断言用区区几个额外的字符就提高了阅读性。

起初，Hamcrest 断言貌似太过于炫技。但是匹配器的价值在于它能极大地提升测试的表达能力，许多匹配器能够减少所需的代码量，同时也能提升测试的抽象层次。

`  ASSERT_THAT(actual, StartsWith("alx")); `

Google Mock的文档中列出了一些内置的[匹配器](https://github.com/google/googlemock/blob/f7d03d2734759ee12b57d2dbcb695607d89e8e05/googlemock/docs/CheatSheet.md#matchers)

使用的时候需要添加`using`声明。
`using namespace ::testing`

否则原本用来提升表达能力的断言读起来有点啰嗦卡顿。例如：
`  ASSERT_THAT(actual, ::testing::StartsWith("alx")); `

Hamcrest断言在提升失败信息的可读性方面意义更大。
```
Value of: actual
Expected: starts with "alx"
Actual: "alpha"
```

匹配器的组合能力使你用一行断言就能表达本来需要多行断言才能做到的事情。
```
TEST(StringTest, ALLOf)
{
string actual = string("al") + "pha";
ASSERT_THAT(actual, AllOf(StartsWith("al"), EndsWith("ha"), Ne("aloha")));
}
```
上面的例子的`AllOf`表明所有匹配器都成功的时候，整个断言才通过。因而`actual`必须以"al"开头,以"ha"结尾，且不等于"aloha"

对于布尔值的断言而言，大部分开发者都会避免使用Hamcrest断言而直接使用经典形式的断言
```
ASSERT_TRUE(someBoolenExpression);
ASSERT_FALSE(someBoolenExpression);
```

## fixture
大多数单元测试的文件都支持将逻辑相关的测试进行分组。在Google Mock中，你可以使用*测试用例名称*将测试分组。例如，下面的测试用例名为 `AVector`，而`EmptyWhenInit`是此测试用例的中的一个测试
`TEST(AVector, EmptyWhenInit)`

相关的测试可能需要相同的测试环境。你会发现许多测试都需要公共的初始化或者辅助函数
```
TEST(AVector, EmptyWhenInit)
{
vector<int> vec;

ASSERT_TRUE(vec.empty());
}

TEST(AVector, NotEmptyAfterPush)
{
vector<int> vec;

ASSERT_TRUE(vec.empty());
}
```
上面两个测试用例都用到了一个`vector`

为此，我们可以定义一个`fixture`——跨测试可重用的类。在Google Mock中，你可以定义一个派生自`::testing::Test`的`fixture`，通常在测试文件开始定义一个`fixture`
```
using namespace ::testing;
class AVector : public Test
{
public:
vector<int> vec;
};
```
在`fixture`中我们定义重用的数据，然后将宏`TEST`改成`TEST_F`，下面是经过整理后的测试：
```
TEST_F(AVector, EmptyWhenInit)
{
ASSERT_TRUE(vec.empty());
}

TEST_F(AVector, NotEmptyAfterPush)
{
vec.push_back(0);
ASSERT_FALSE(vec.empty());
}
```
**注意：测试用例的名字必须和fixture的名称一样**，如果没有带_F的宏，编译就会出错

### SetUp 和 TearDown
如果测试用例中所有的测试都有一条或者更多调相同的初始化语句，那么可以将它们写在`fixture`中初始化函数中，在Google Mock中，必须将这个函数命名为`SetUp`(覆盖了::testing::Test中的虚函数)

对于`fixture`中的每一个测试，Google Mock都会创建一个新的，独立的 `fixture` 实例。这种隔离有助于减少测试之间相互干扰的情况，这也暗示着每个测试都从头创建自己的上下文，并且这些上下文是相互独立的。在创建 `fixture` 后，Google Mock会执行`SetUp`函数，然后执行测试

下面的例子都用了`vec.push_back(0)`初始化上下文
```
TEST_F(AVectorContainOneElement, NotEmpty)
{
vec.push_back(0);
ASSERT_FALSE(vec.empty());
}

TEST_F(AVectorContainOneElement, EmptyAfterPop)
{
vec.push_back(0);

vec.pop_back();

ASSERT_TRUE(vec.empty());
}
```

我们可以将初始化的工作放进`SetUp`中
```
class AVectorContainOneElement : public Test
{
public:
void SetUp() override
{
vec.push_back(0);
}

vector<int> vec;
};

TEST_F(AVectorContainOneElement, NotEmpty)
{
ASSERT_FALSE(vec.empty());
}

TEST_F(AVectorContainOneElement, EmptyAfterPop)
{
vec.pop_back();

ASSERT_TRUE(vec.empty());
}
```

我们移除了重复的代码，这样有助于我们理解测试用例。

`TearDown`函数本质上是`SetUp`的逆过程。每个测试执行后，它都会执行一次，即便当前测试抛出异常也不例外。你可以将`TearDown`用于清理工作：释放内存等。



## 常用的断言写法
在使用的Google Mock的过程中，我们断言的对象常常是复杂的，例如是浮点类型的或者是一个容器，如何对这样的数据做断言呢？这里，列举了常用的几种断言

### 浮点数比较
浮点数一直都是个麻烦的事情，好在Google Mock提供了内置的[Floating-point 匹配器](https://github.com/google/googlemock/blob/f7d03d2734759ee12b57d2dbcb695607d89e8e05/googlemock/docs/CheatSheet.md#floating-point-matchers)，能让我们方便的对浮点类型进行断言

```
ASSERT_THAT(1.0f, FloatEq(1));
ASSERT_THAT(1.0, DoubleEq(1));
```

还可以设置误差范围：
```
ASSERT_THAT(1.0f, FloatNear(1.0000001, 1e-6));
ASSERT_THAT(1.0, DoubleNear(1.0000001, 1e-6));
```

有时候要判断是否为nan
```
ASSERT_THAT(1.0f/0, NanSensitiveFloatEq(1.0/0));
ASSERT_THAT(1.0/0, NanSensitiveDoubleEq(1.0/0));
```

### 容器比较
STL中大部分容器都支持 `==`，所以你可以用`Eq(expected_container)`来对比容器
```
vector<int> a = {1,2,3};
vector<int> b = {1,2,3};
ASSERT_THAT(a, Eq(b));
```

或者，可以直接判断容器中的元素

```
vector<int> a = {1,2,3};
ASSERT_THAT(a, ElementsAre(1,2,3));
```

有时候，我们不关心容器中元素的顺序
```
vector<int> a = {1,2,3};
ASSERT_THAT(a, UnorderedElementsAre(2,1,3));
```

### 指针比较
两只指针的地址比较，直接用Eq就可以了，如果要比较指针指向的值，可以用`Pointee`
```
int* a = new int(1);
ASSERT_THAT(a, Pointee(1));

delete a;
```

好吧，我知道你想问，为什么不用`ASSERT_THAT(*a, Eq(1))`。这么用当然没有问题，但是使用`Pointee`在可读性上要好得多，符合人们的阅读习惯

### 对象的比较
想要比较两个对象是否相等，需要以全局函数的形式重载`==`

```
class Foo
{
public:
Foo(int x):x_(x){}
int getX() const
{
return x_;
}
private:
int x_;
};

bool operator==(const Foo& lhs, const Foo& rhs)
{
return lhs.getX() == rhs.getX();
}

TEST(ClassFooTest, EqualsIfHavaSameX)
{
Foo a(0);
Foo b(0);

ASSERT_THAT(a, Eq(b));
}
```

同理，重载`>,<,>=,<=`之类的就比较大小关系了

### 自己写匹配器
当内置的匹配器都不能满足你的需求时，那么你需要动手自己写匹配器，直接上几个例子辅助理解匹配器怎么写

```
MATCHER(IsEven, ""){return (arg % 2) == 0;}
TEST(ANum, IsEven)
{
ASSERT_THAT(4, IsEven());
}
```

```
MATCHER_P(IsDivisibleBy, n, ""){ return (arg % n) == 0; }
TEST(ANum, IsDivisibleByThree)
{
ASSERT_THAT(6, IsDivisibleBy(3));
}
```

```
MATCHER_P2(InCloseRange, low, high, ""){
return low <= arg && arg <= high;
}
TEST(InCloseRange, 1)
{
ASSERT_THAT(3, InCloseRange(4, 5));
}
```

可以看到，`MATCHER`表示匹配器不需要参数，`MATCHER_P`需要一个参数,`MATCHER_P2`需要两个参数，以此类推，最多能到`MATCHER_P10`

`MATCHER`中最后的那个字符串在测试失败出现，如果为空，Google Mock会默认帮你生成一个

`arg`是要比较的对象，有时候它是个tuple，例如在用到`Pointwise`时。考虑一个问题，如何对比浮点型的数组，并且允许有一定的误差，你可以这样写：

```
MATCHER_P(NearWithPrecision, ferr, "")
{
return abs(get<0>(arg) - get<1>(arg)) < ferr;
}

TEST(FloatArrayTest, Eq)
{
vector<float> float_array = {1.000001f, 2.000002f, 3.000003f};
vector<float> ground_truth = {1.0f, 2.0f, 3.0f};
ASSERT_THAT(float_array, Pointwise(NearWithPrecision(1e-4), ground_truth));
}
```

## 总结
主要介绍了Google Test和Google Mock中的基本概念，包括断言的形式、fixture等。
举例说明常用的断言命令要怎么写，包括浮点型、容器等。最后动手自己实现匹配器，满足复杂的测试需求

由于时间和篇幅的原因，本文没有介绍关于Mock的部分，这部分内容我会在下一个文章中介绍给大家


## 参考
+ [GTest入门](http://www.yeolar.com/note/2014/12/21/gtest/)
+ [Define a Mock Class](https://github.com/google/googlemock/blob/master/googlemock/docs/CheatSheet.md#defining-a-mock-class)
