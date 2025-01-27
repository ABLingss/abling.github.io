---
layout: post
title: C++学习笔记
subtitle: 你说的对，但是要常看捏
date: 2025-01-27
categories: 技术
tags:
  - 技术
  - 设计
---
*这是2024夏季InfiniTensor开源社区 清华ai训练营的[c++课程部分](https://opencamp.cn/InfiniTensor/camp/2024summer)*
## L1 c++输出
### 流输出std::cout和printf
- 区别在于 std::cout是类型安全的，printf要标注类型，cout不用；
- cout输出bool的时候会输出int的1 0 
  -  如果你不想这样，就要先输出一个流修饰符，而且他改的是全局状态
    ```c++
    std::cout << std::boolalpha << true << ' ' << flase << std::endl;
    /*输出结果为true false*/
    std::cout << true << ' ' << flase << std::endl;
    /*这个没有流修饰符，但是输出也是true flase*/
    /*流修饰符改变的是全局状态*/
    ```
- 流修饰符 <https://zh.cppreference.com/w/cpp/io/manip>
  - ***流修饰符改变的是全局状态***
  - 这也太唐了 真不如printf
### 解决流输出的问题 用fmt库 他是类型安全的  支持流不支持的输出
  - 已经进入c++20标准
  - 你可以
    ```c++
  #include <fmt>
  // 字符串
  auto name = fmt::format("Hello, {}\n", "tlanyan");
  std::cout << name; // 输出 Hello, tlanyan
  // 或者直接使用fmt::print
  fmt::print("Hello, {}\n", "tlanyan");

  // 整数
  fmt::print("The answer is {}\n", 42); // 输出 The answer is 42

  // 浮点数
  fmt::print("PI is {:.2f}\n", 3.14159); // 输出 PI is 3.14

  // 布尔
  fmt::print("Are you human being? {}\n", true); // 输出Are you human being? true

  // vector
  // 需要 #include <fmt/ranges.h>
  fmt::print("vec: {}\n", std::vector{1, 4, 7}); // 输出 vec: [1, 4, 7]    fmt::print("The answer is {}\n", 42); 
    ```
    - 在c++20后可以
    ```c++
    #include <format>

    std::cout << std::format("hello,{}", "ABLing") << std::endl;
              << std::format("{} + {} = {}",23,19,23+19);
    ```
### 其他的流操作
- `std::cin`
- `std::error`

## L2 运算符
- 运算符 <https://zh.cppreference.com/w/cpp/language/expressions#.E8.BF.90.E7.AE.97.E7.AC.A6>
- 在流输出里面，你里面的内容可能对函数本身造成影响，因为<<本质就是座椅运算符 所以你要多多加括号
- 但是有时候加括号也不行，比如用宏的时候
        ```
        #define left(x, w) (x) << (w)
        std::cout << left(7 , 10) << std::endl:
        // 这会打印710 因为你用宏定义的左移在流输出里面被理解为<<输出

## L3 声明
### noexcept 关键字
   是 C++11 引入的，用于标记一个函数声明为“不抛出异常”（即该函数不会抛出任何异常）。它在编译时帮助进行优化，同时也让程序员明确声明某个函数不会抛出异常，提供了更好的异常安全性和性能优化。
### 知识点
https://zh.cppreference.com/w/cpp/language/declarations

https://learn.microsoft.com/zh-cn/cpp/c-language/interpreting-more-complex-declarators?view=msvc-170

## SP 什么是左值什么是右值 移动语义
- READ: 左值右值（概念）<https://learn.microsoft.com/zh-cn/cpp/c-language/l-value-and-r-value-expressions?view=msvc-170>
- READ: 左值右值（细节）<https://zh.cppreference.com/w/cpp/language/value_category>
- READ: 关于移动语义 <https://learn.microsoft.com/zh-cn/cpp/cpp/rvalue-reference-declarator-amp-amp?view=msvc-170#move-semantics>

在 C++ 中，**左值（lvalue）**和**右值（rvalue）**是表达式中不同类型的值，它们的区别在于它们是否可以出现在赋值操作符的左侧。

### 左值（Lvalue）

- **定义**：左值是一个可以出现在赋值操作符左侧的表达式，它代表的是一个可以定位的对象的内存地址。
- **特点**：
  - 左值可以是对象、数组、引用等。
  - 左值的值可以在程序中修改（但不一定修改）。
  - 左值通常指向内存中的某个位置，因此可以通过该位置来读取或修改数据。
  
**常见场景**：
- 变量、数组元素、解引用后的指针等。

#### 示例：
```cpp
#include <iostream>

int main() {
    int x = 10;  // 'x' 是左值
    int y = 20;
    x = y;        // 'x' 在此为左值，'y' 是右值
    std::cout << x << std::endl;  // 输出 20
}
```

### 右值（Rvalue）

- **定义**：右值是一个不能出现在赋值操作符左侧的表达式，它通常表示一个临时值或者一个将被销毁的对象。
- **特点**：
  - 右值通常没有名称，或者说它的生命周期比较短。
  - 右值通常表示的是一个即将消失的临时对象或常量，不能直接被修改。

**常见场景**：
- 字面量（如数字、字符常量）、临时对象、函数返回值等。

#### 示例：
```cpp
#include <iostream>

int main() {
    int x = 10;
    x = x + 5;  // 'x + 5' 是右值，结果会被赋值给左值 'x'
    std::cout << x << std::endl;  // 输出 15
}
```

在上面的例子中，`x + 5` 是一个右值，它表示一个临时计算结果，不能直接出现在赋值操作符的左侧。

### 右值引用（Rvalue Reference）

C++11 引入了右值引用（`T&&`）来允许对右值进行操作，特别是在**移动语义**和**完美转发**中非常重要。

右值引用允许我们**“移动”**资源而不是复制，从而提高性能，特别是在涉及动态内存分配的场景中。

#### 示例：
```cpp
#include <iostream>
#include <vector>

void func(std::vector<int>&& v) {
    // 通过右值引用来接收临时对象
    std::cout << "Vector size: " << v.size() << std::endl;
}

int main() {
    std::vector<int> vec = {1, 2, 3};
    func(std::move(vec));  // 使用 std::move 将左值转换为右值
}
```

在这个例子中，`std::move(vec)` 将 `vec` 转换为右值引用，从而调用 `func` 函数，这样可以避免复制整个 `vec` 容器，而是直接将其内容移动到新的位置。

### 左值引用和右值引用的区别

- **左值引用（Lvalue Reference）**：只能绑定到左值。
  ```cpp
  int a = 10;
  int& ref = a;  // 左值引用绑定到左值
  ```

- **右值引用（Rvalue Reference）**：只能绑定到右值，允许通过移动语义优化性能。
  ```cpp
  int&& rref = 10;  // 右值引用绑定到右值
  ```

### 应用场景

1. **移动语义（Move Semantics）**：
   - 在涉及容器（如 `std::vector`）等数据结构时，使用右值引用可以避免不必要的深度复制，从而提高性能。
   
2. **完美转发（Perfect Forwarding）**：
   - 在模板编程中，右值引用和 `std::forward` 配合使用，实现完美转发，确保参数能够按照其原始类型（左值或右值）传递到函数中。

#### 示例：移动语义
```cpp
#include <iostream>
#include <vector>

class MyClass {
public:
    MyClass() { std::cout << "Constructor\n"; }
    MyClass(const MyClass& other) { std::cout << "Copy Constructor\n"; }
    MyClass(MyClass&& other) noexcept { std::cout << "Move Constructor\n"; }
};

int main() {
    MyClass obj1;               // 输出: Constructor
    MyClass obj2 = std::move(obj1);  // 输出: Move Constructor
}
```

在这个例子中，`std::move(obj1)` 将 `obj1` 转换为右值引用，从而触发移动构造函数而不是复制构造函数。

通过对右值的支持，C++ 能够减少不必要的资源复制，尤其是在处理大型数据结构时，能够显著提高程序的效率。
#### [完美转发](#完美转发用到模板)

## L4传参
- READ: <https://stackoverflow.com/questions/156767/whats-the-difference-between-an-argument-and-a-parameter>

  - **ARGUMENTS == 实参**
  - **Parameter == 形参**
### 参数都有哪些传递方式？如何选择传递方式？
#### 值传递
```c++
#include <iostream>

void printValue(int x) {
    std::cout << "x = " << x << std::endl;
    x = 100;  // 修改形参的值，不影响实参
}

int main() {
    int a = 10;
    printValue(a);
    std::cout << "a = " << a << std::endl;  // a的值仍为10
    return 0;
}
```
在这个例子中，printValue函数按值接收参数x。在函数内部对x的修改不会影响到main函数中的变量a。
#### 左值引用传递
```c++
#include <iostream>

void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

int main() {
    int x = 10, y = 20;
    std::cout << "Before swap: x = " << x << ", y = " << y << std::endl;
    swap(x, y);
    std::cout << "After swap: x = " << x << ", y = " << y << std::endl;
    return 0;
}
```
这里swap函数通过按引用传递参数a和b，在函数内部交换了两个变量的值，这种修改会直接反映到main函数中的变量x和y上。
#### 按指针传递
```c++
#include <iostream>

void increment(int* p) {
    (*p)++;  // 通过指针修改实参的值
}

int main() {
    int num = 5;
    std::cout << "Before increment: num = " << num << std::endl;
    increment(&num);
    std::cout << "After increment: num = " << num << std::endl;
    return 0;
}
```
increment函数通过指针p接收参数，在函数内部通过解引用(*p)来修改实参num的值。
#### 左值常引用传递（可以引用临时对象（不用在调用前声明））
```c++
#include <iostream>
#include <string>

void printString(const std::string& str) {
    std::cout << "The string is: " << str << std::endl;
    // str = "New string";  // 错误，不能修改常量引用
}

int main() {
    std::string myString = "Hello, World!";
    printString(myString);
    return 0;
}
```
在这个例子中，printString函数按常量引用接收参数str。这种方式可以避免对std::string对象的复制，同时保证函数内部不能修改str的值，确保了数据的安全性。

#### 右值引用传递
```c++
#include <iostream>
#include <string>

void printString(std::string&& str) {
    std::cout << str << std::endl; //两个引用号
}

int main() {
    std::string s = "Hello";
    printString(std::move(s));  // 明确将左值s转换为右值
    printString("World");       // 字面量"World"本身就是右值
    return 0;
}
在这个例子中，printString函数通过右值引用接收参数str，可以接收右值字符串。
```

#### 完美转发（用到模板）
```c++
#include <iostream>
#include <utility>

template<typename T>
void wrapper(T&& arg) {
    // 使用std::forward进行完美转发
    wrappedFunction(std::forward<T>(arg));
}

void wrappedFunction(int& x) {
    std::cout << "wrappedFunction(int&): " << x << std::endl;
}

void wrappedFunction(int&& x) {
    std::cout << "wrappedFunction(int&&): " << x << std::endl;
}

int main() {
    int a = 5;
    wrapper(a);        // 调用wrappedFunction(int&)
    wrapper(10);       // 调用wrappedFunction(int&&)
    return 0;
}
在这个例子中，wrapper函数模板使用右值引用和std::forward来完美转发参数arg给wrappedFunction函数。根据传入的实参类型，wrappedFunction的重载版本会被正确调用。
```

## L5 存储类说明符STATIC AUTO REGISTER THREAD_LOCAL EXTERN MUTABLE
<https://zh.cppreference.com/w/cpp/language/storage_duration>
```c++
// THINK: 这个函数的两个 `static` 各自的作用是什么？
// 第一个 `static` 修饰 `func` 函数，使其只在当前文件中可见；第二个 `static` 修饰 `static_` 变量，使其在多次函数调用之间保持其值并且只初始化一次。
static int func(int param) {
    static int static_ = param;
    // std::cout << "static_ = " << static_ << std::endl;
    return static_++;
}
```

## L6 编译期计算 CONSTEXPR声明
在 C++ 和 Rust 中，计算可以发生在编译期或运行期，分别涉及编译器优化和 `constexpr` 声明。我们可以通过这两种方式来减少运行时的计算开销，提高程序的性能。

### 1. **编译器优化 (来自编译器的优化)**

编译器优化指的是编译器在编译时对代码进行的一些智能优化，以减少运行时的计算负担。这些优化通常是在程序编译阶段自动进行的，不需要开发者显式指定。编译器会分析代码并尝试将某些计算转化为在编译时完成的计算，从而避免在程序运行时进行这些计算。

#### 示例（C++）
```cpp
#include <iostream>

int square(int x) {
    return x * x;
}

int main() {
    int result = square(5);  // 编译器可以将这行代码优化为 result = 25;
    std::cout << result << std::endl;
    return 0;
}
```

**解释**：
- 如果在编译过程中，编译器能够确定 `square(5)` 是一个常量表达式（编译时可以计算的值），它会将 `square(5)` 直接替换为 `25`。
- 这样，在运行时，`result` 就是 `25`，而没有进行额外的乘法计算。
- 这类优化通常依赖于编译器的优化级别和具体实现，不需要显式的 `constexpr` 声明。

### 2. **`constexpr` 进行声明 (编译期常量表达式)**

`constexpr` 是 C++11 引入的一个关键字，用于显式声明常量表达式，指明一个函数或变量在编译时可以被计算。这意味着 `constexpr` 函数的结果可以在编译时计算出来，而不是在程序运行时计算。

- **`constexpr` 函数**：在编译时执行，可以像常量一样使用。
- **`constexpr` 变量**：在编译时确定其值，且该值不能改变。

#### 示例（C++）
```cpp
#include <iostream>

constexpr int square(int x) {
    return x * x;
}

int main() {
    constexpr int result = square(5);  // 计算在编译期完成
    std::cout << result << std::endl;   // 输出 25
    return 0;
}
```

**解释**：
- `square` 函数被声明为 `constexpr`，这意味着它可以在编译期计算并返回结果。
- `result` 也被声明为 `constexpr`，意味着它的值（`square(5)`）会在编译时就被确定为 25。
- 这意味着在运行时，不会再进行乘法计算，`result` 已经是一个常量。


- **编译器优化**：通过编译器的自动优化，某些计算可以在编译时完成。例如，在 C++ 中，编译器会尝试将常量表达式提前计算，从而减少运行时的计算。
- **`constexpr` 和 `const fn`**：这是一种显式声明常量表达式的方法，允许开发者在编写代码时指明某些计算可以在编译时进行，这样计算结果就会在编译阶段完成，而不是在程序运行时进行。

这两种方式都能够提高程序性能，减少运行时的计算，尤其是在涉及到大量常量计算时。

## sp 斐波那契数列求和递归
- 以下的代码 运行的时候算的顺序不定
- 这说明多个参数 +两边的情况 << <<的流等 计算顺序不定 是**未定义型** 编译器会因各种原因改变顺序
- **因此 一行式子不要出现超过一个i++ 否则会让顺序出错**
```c++
/*这个直接跑不了的 要放到05的函数体里*/
constexpr unsigned long long fibonacci(int i,int level = 0) {
    for(int j = 0;j < level;j ++){
        std::cout << "; ";
    }
    std::cout << i << std::endl;
    switch (i) {
        case 0:
            return 0;
        case 1:
            return 1;
        default:
            return fibonacci(i - 1) + fibonacci(i - 2);
    }
}
int main(int arg, int **argv){
    constexpr auto ANS_N = 5;
    auto ANS = fibonacci(ANS_N);
    std::cout << "fibonacci(" << ANS_N << ") ="  << ANS << std::endl;
    return 0;
}
```

## L7 数组
https://zh.cppreference.com/w/cpp/language/array

## L8 纯函数
- 一个函数被称为纯函数（pure function），如果它满足以下两个条件：

  - 无副作用（No Side Effects）：函数的执行不应改变外部状态（如修改全局变量、输入输出等），并且它不依赖于外部状态。
  - 对于相同输入总是返回相同输出（Deterministic Output）：给定相同的输入，函数每次执行的结果必须是一样的。
- 下面是一个使用了缓存的斐波那契，很快
```cpp
// THINk: 这个函数是一个纯函数（pure function）吗？
// READ: 纯函数 <https://zh.wikipedia.org/wiki/%E7%BA%AF%E5%87%BD%E6%95%B0>
static unsigned long long fibonacci(int i) {
    static unsigned long long cache[96]{0, 1}, cached = 2;
    for (; cached <= i; ++cached) {
        cache[cached] = cache[cached - 1] + cache[cached - 2];
    }
    return cache[i];
}
```
- 这是纯函数，只要相同输入会有相同输出，就是纯函数

## L9 枚举 联合 类型转换
- cpp的类型转换这样写
    ```cpp
    static_cast<int>(b) //把b转换为int型，效果等同于`(int)b`
    ```
- 如果要逐比特的转换，就利用指针
```cpp 
    float a = 2.5;
    int* p = reinterpret_cast<int*>(&a); 

    std::cout << *(int*)p << std::endl; //c写法
    std::cout << *reinterpret_cast<int*>(p) << std::endl; //cpp写法
```
```cpp
#include "../exercise.h"

// READ: 枚举类型 <https://zh.cppreference.com/w/cpp/language/enum>

// `enum` 是 C 的兼容类型，本质上其对应类型的常量。
// 在 `enum` 中定义标识符等价于定义 constexpr 常量，
// 这些标识符不需要前缀，可以直接引用。
// 因此 `enum` 定义会污染命名空间。
enum ColorEnum : unsigned char {
    COLOR_RED = 31,
    COLOR_GREEN,
    COLOR_YELLOW,
    COLOR_BLUE,
};

// 有作用域枚举型enum class是 C++ 引入的类型安全枚举。
// 其内部标识符需要带前缀引用，如 `Color::Red`。
// 作用域枚举型可以避免命名空间污染，并提供类型安全保证。
enum class Color : int {
    Red = COLOR_RED,
    Green,
    Yellow,
    Blue,
};

ColorEnum convert_by_pun(Color c) {
    // READ: <https://zh.cppreference.com/w/cpp/language/union>
    // `union` 表示在同一内存位置存储的不同类型的值。
    // 其常见用法是实现类型双关转换，即将一种类型的值转换为另一种无关类型的值。
    // 但这种写法实际上仅在 C 语言良定义，在 C++ 中是未定义行为。
    // 这是比较少见的 C++ 不与 C 保持兼容的特性。
    // READ: 类型双关 <https://tttapa.github.io/Pages/Programming/Cpp/Practices/type-punning.html>
    union TypePun {
        ColorEnum e;
        Color c;
    };

    TypePun pun;
    // TODO: 补全类型双关转换

    return pun.e;
}

int main(int argc, char **argv) {
    ASSERT(convert_by_pun(Color::Red) == COLOR_RED, "Type punning conversion");
    ASSERT(convert_by_pun(Color::Green) == COLOR_GREEN, "Type punning conversion");
    ASSERT(convert_by_pun(Color::Yellow) == COLOR_YELLOW, "Type punning conversion");
    ASSERT(convert_by_pun(Color::Blue) == COLOR_BLUE, "Type punning conversion");
    return 0;
}

```

## L10 初始化
https://zh.cppreference.com/w/cpp/language/initialization
多看!!!!!

根据 C++ 的官方参考文档，C++ 支持几种不同的初始化方法。以下是列举的初始化方式：

1. **直接初始化**（Direct Initialization）  
   通过括号或圆括号初始化对象。
   ```cpp
   int x(10);  // 直接初始化
   ```

2. **拷贝初始化**（Copy Initialization）  
   通过赋值的方式初始化对象。
   ```cpp
   int x = 10;  // 拷贝初始化
   ```

3. **列表初始化**（List Initialization）  

    1. **普通变量的初始化**：

       ```cpp
       int x{10};  // x 被初始化为 10
       ```

       这里，`x` 被初始化为 `10`，如果没有适合的构造函数或者类型转换，则会产生错误。

    2. **数组的初始化**：

       ```cpp
       int arr[3]{1, 2, 3};  // 数组 arr 被初始化为 {1, 2, 3}
       ```

    3. **类的初始化**（使用构造函数）：

       如果你有一个类，使用列表初始化时，可以通过调用构造函数来初始化成员变量：

       ```cpp
       class MyClass {
       public:
           int a;
           double b;

           MyClass(int x, double y) : a(x), b(y) {}
       };

       MyClass obj{10, 3.14};  // obj 被初始化为 a=10, b=3.14
       ```


4. **统一初始化**（Uniform Initialization）  
   这是 C++11 引入的语法，花括号初始化提供一种统一的方式初始化所有对象。
   ```cpp
   int x{10};  // 统一初始化
   double d{3.14};
   ```

5. **零初始化**（Zero Initialization）  
   使用花括号初始化时，如果没有给出值，则会将对象初始化为零。
   ```cpp
   int x{};  // 零初始化，x 的值为 0
   ```

6. **默认初始化**（Default Initialization）  
   如果没有为变量提供初始化值，那么根据变量的类型，C++ 会决定默认的初始化方式：
   - 基本数据类型（如 `int`、`double`）会有未定义值。
   - 类类型会调用默认构造函数。
   ```cpp
   int x;  // 默认初始化，x 的值不确定
   ```

7. **值初始化**（Value Initialization）  
   使用 `T()` 初始化对象时，会调用默认构造函数（对于类类型）并将成员初始化为零（对于内置类型，值初始化会给对象赋予零值）。
   ```cpp
   int x{};  // 值初始化
   MyClass obj{};  // 调用默认构造函数，成员初始化为零
   ```

8. **引用初始化**（Reference Initialization）  
   初始化引用类型时，引用必须绑定到某个对象。
   ```cpp
   int x = 10;
   int& ref = x;  // 引用初始化
   ```

9.  **动态初始化**（Dynamic Initialization）  
   使用 `new` 操作符动态分配内存并初始化。
   **返回值是指针**
   ```cpp
   int* ptr = new int(10);  // 动态初始化
   ```

10. **类类型初始化**（Class Initialization）  
    类对象初始化时，调用相应的构造函数。
    ```cpp
    class MyClass {
    public:
        MyClass(int val) : x(val) {}
        int x;
    };

    MyClass obj(10);  // 类对象初始化
    ```


## sp 有关&和*用法的终极辨析
### 1. **& — 引用符号**(仅cpp,表示一个非空的指针)

#### **在声明时作为引用**
- `&` 在声明变量时表示引用类型。引用是一个别名，它与原始变量共享同一内存地址，因此修改引用的值会直接影响原始变量。
  
  ```cpp
  int a = 10;
  int& b = a; // b 是 a 的引用
  b = 20;      // 修改 b 的值，会影响 a
  std::cout << a << std::endl; // 输出 20
  ```

#### **在函数参数中传递引用**
- 传递引用可以避免复制大量数据（尤其是大对象），并且可以让函数修改传入的变量。

  ```cpp
  void addTen(int& x) {
      x += 10;
  }

  int main() {
      int n = 5;
      addTen(n); // 通过引用传递，n 会被修改
      std::cout << n << std::endl; // 输出 15
  }
  ```

#### **注意：**
- 引用一旦绑定到某个对象后，无法再指向其他对象。它与指针不同，指针可以指向不同的内存地址。

### 2. **& — 地址符号**

- 在取地址时，`&` 用于获取变量的内存地址，即返回该变量的指针。

  ```cpp
  int a = 10;
  int* p = &a; // 获取 a 的地址并赋给 p
  std::cout << p << std::endl; // 输出 a 的地址
  ```

### 3. **\* — 指针符号**

#### **在声明时作为指针**
- `*` 在声明变量时表示指针类型。指针是一个存储内存地址的变量，它指向另一个变量。

  ```cpp
  int a = 10;
  int* p = &a; // p 是一个指向 int 类型的指针
  std::cout << *p << std::endl; // 输出 p 所指向的变量的值，即 a 的值 10
  ```

#### **通过指针解引用**
- `*` 用于解引用，获取指针指向的变量的值。如果你有一个指针，你可以通过解引用操作访问它所指向的对象。

  ```cpp
  int a = 10;
  int* p = &a;
  *p = 20;  // 修改 p 指向的值，即修改 a 的值
  std::cout << a << std::endl; // 输出 20
  ```

### 4. **& 和 * 在赋值中的区别**

#### **赋值给引用：**
- 引用会直接修改原始变量的值，因为引用是原始变量的别名。

  ```cpp
  int a = 10;
  int& b = a; // b 是 a 的引用
  b = 20;      // b 被赋值为 20，a 也变为 20
  std::cout << a << std::endl; // 输出 20
  ```

#### **赋值给指针：**
- 指针的赋值需要间接操作，指针本身保存的是地址，而解引用指针时可以修改指针指向的值。

  ```cpp
  int a = 10;
  int* p = &a;
  *p = 20;   // 修改 p 指向的值，即修改 a 的值
  std::cout << a << std::endl; // 输出 20
  ```

### 5. **& 和 * 在传参中的区别**

#### **传值（默认行为）**
- 通过传值传递时，函数会接收变量的副本，修改副本不会影响原始变量。

  ```cpp
  void addTen(int x) {
      x += 10;  // 修改副本
  }

  int main() {
      int n = 5;
      addTen(n);
      std::cout << n << std::endl; // 输出 5，n 不受影响
  }
  ```

#### **传引用**
- 通过引用传递时，函数操作的是原始变量本身，修改会影响传入的变量。

  ```cpp
  void addTen(int& x) {
      x += 10;  // 修改原始变量
  }

  int main() {
      int n = 5;
      addTen(n);
      std::cout << n << std::endl; // 输出 15，n 被修改
  }
  ```

#### **传指针**
- 通过指针传递时，函数接收的是指针，可以通过解引用修改指针指向的变量。

  ```cpp
  void addTen(int* x) {
      *x += 10;  // 修改指针指向的值
  }

  int main() {
      int n = 5;
      addTen(&n);
      std::cout << n << std::endl; // 输出 15，n 被修改
  }
  ```

### 总结

- **`&`**：
  - 在声明时：表示引用类型。
  - 在取地址时：用于获取变量的地址。
  
- **`*`**：
  - 在声明时：表示指针类型。
  - 在解引用时：访问指针指向的值。
  
- **传参时的区别**：
  - 传值：函数接收变量的副本。
  - 传引用：函数接收原始变量的引用，可以修改原始变量。
  - 传指针：函数接收变量的地址，通过解引用可以修改原始变量。

---

## sp （下面的几章着眼于面向对象，这是前置知识）面向对象与实例化


### 1. 面向对象编程 (Object-Oriented Programming, OOP)

面向对象编程（OOP）是一种编程范式，它将程序视为一组相互作用的对象，而这些对象是通过类来定义的。OOP 通过将数据和操作数据的代码封装在一起，使得程序更加模块化、灵活和易于维护。面向对象编程的四大基本特性包括：
1. **封装**（Encapsulation）
2. **继承**（Inheritance）
3. **多态**（Polymorphism）
4. **抽象**（Abstraction）

#### 1.1 封装 (Encapsulation)

封装是将数据和操作数据的行为封装成对象的过程。通过封装，类的内部实现细节对外部隐藏，外部只能通过公开的接口来访问类的成员。这样有助于保护数据，并提高代码的可维护性。

#### 1.2 继承 (Inheritance)

继承允许一个类（派生类）继承另一个类（基类）的属性和方法。通过继承，可以实现代码的重用，并为类添加新的功能。

#### 1.3 多态 (Polymorphism)

多态允许通过父类指针或引用来调用子类的重写函数，从而表现出不同的行为。通过虚函数和继承，C++ 实现了运行时多态。

#### 1.4 抽象 (Abstraction)

抽象指的是从现实世界中提取出重要的特征和行为，并忽略不重要的细节。在编程中，抽象通常通过类和接口来实现。

---

### 2. 实例化 (Instantiation)

**实例化** 是指根据类创建对象的过程。类是对象的蓝图或模板，而对象则是类的一个具体实例。实例化过程是将类定义转换为内存中的实际数据结构，从而允许访问类的属性和方法。

#### 2.1 类和对象的关系

- **类** 是对象的模板，定义了对象的属性和行为。
- **对象** 是类的一个具体实例，代表了类在内存中的一个特定实现。

#### 2.2 创建对象的过程

在 C++ 中，实例化对象通常使用 **构造函数** 来完成。构造函数用于初始化对象的成员变量，并为对象分配内存空间。构造函数的名称与类名相同，并且没有返回类型。

#### 例子：
```cpp
class Car {
public:
    string make;
    string model;
    int year;

    // 构造函数
    Car(string m, string mod, int y) {
        make = m;
        model = mod;
        year = y;
    }

    void displayInfo() {
        cout << "Car: " << year << " " << make << " " << model << endl;
    }
};

int main() {
    Car myCar("Toyota", "Camry", 2020);  // 实例化对象 myCar
    myCar.displayInfo();  // 调用对象的方法
    return 0;
}
```
在这个例子中，`Car` 类通过构造函数接受参数 `make`、`model` 和 `year` 来实例化一个对象 `myCar`。`myCar` 是 `Car` 类的一个实例，可以调用类中的成员函数 `displayInfo()`。

#### 2.3 动态实例化

在 C++ 中，除了静态实例化（在栈上创建对象）外，还可以通过动态内存分配在堆上实例化对象。使用 `new` 关键字来动态创建对象，并返回指向该对象的指针。

#### 例子：
```cpp
int main() {
    Car* carPtr = new Car("Honda", "Civic", 2021);  // 动态实例化
    carPtr->displayInfo();  // 通过指针调用对象方法
    delete carPtr;  // 释放内存
    return 0;
}
```
在这个例子中，`new` 用于动态分配内存，创建了 `Car` 类的一个对象，并返回该对象的指针。使用 `delete` 释放动态分配的内存。

#### 2.4 默认构造函数

默认构造函数是没有参数的构造函数，如果你没有为类定义构造函数，编译器会自动提供一个默认的构造函数。默认构造函数会初始化成员变量为默认值。

#### 例子：
```cpp
class Person {
public:
    string name;
    int age;

    // 默认构造函数
    Person() {
        name = "Unknown";
        age = 0;
    }

    void displayInfo() {
        cout << "Name: " << name << ", Age: " << age << endl;
    }
};

int main() {
    Person p;  // 使用默认构造函数
    p.displayInfo();  // 输出：Name: Unknown, Age: 0
    return 0;
}
```

### 3. 构造函数与实例化

#### 3.1 构造函数的作用

构造函数在对象实例化时被调用，主要负责初始化对象的状态。构造函数可以有多个版本，这样可以支持不同的初始化方式：

- **默认构造函数**：无参数，提供默认值。
- **带参数的构造函数**：接受参数并根据参数初始化对象。
- **拷贝构造函数**：用于创建一个新对象作为现有对象的副本。

#### 3.2 拷贝构造函数

拷贝构造函数用于创建一个对象，它是另一个同类型对象的副本。拷贝构造函数的格式通常是：
```cpp
ClassName(const ClassName& other);
```

#### 例子：
```cpp
class Point {
public:
    int x, y;

    Point(int x_val, int y_val) : x(x_val), y(y_val) {}

    // 拷贝构造函数
    Point(const Point& p) {
        x = p.x;
        y = p.y;
    }

    void display() {
        cout << "Point(" << x << ", " << y << ")" << endl;
    }
};

int main() {
    Point p1(10, 20);
    Point p2 = p1;  // 使用拷贝构造函数
    p2.display();  // 输出：Point(10, 20)
    return 0;
}
```
在这个例子中，`Point` 类定义了一个拷贝构造函数，用于将 `p1` 的值复制到 `p2`。

### 4. 静态实例化

除了实例化普通对象外，C++ 还支持**静态实例化**。静态成员变量属于类本身，而不是类的实例，因此它们在所有对象之间共享。

#### 4.1 静态成员变量

静态成员变量在类的所有对象之间共享，而不是每个对象都有一份副本。静态成员变量必须在类外定义。

#### 例子：
```cpp
class Counter {
public:
    static int count;  // 静态成员变量

    Counter() {
        count++;
    }

    static void displayCount() {
        cout << "Count: " << count << endl;
    }
};

int Counter::count = 0;  // 定义静态成员变量

int main() {
    Counter c1;
    Counter c2;
    Counter::displayCount();  // 输出：Count: 2
    return 0;
}
```
在这个例子中，`count` 是静态成员变量，它在所有 `Counter` 类的对象之间共享。

---

### 5. 总结

- **面向对象编程**：一种基于对象和类的编程方法，四大基本特性是封装、继承、多态和抽象。
- **实例化**：根据类创建对象的过程，使用构造函数进行对象初始化。对象可以在栈上静态创建，也可以在堆上动态创建。
- **构造函数**：用于初始化对象的成员变量。可以有默认构造函数、带参数的构造函数和拷贝构造函数。
- **静态成员**：静态成员变量在所有对象之间共享，可以通过类名直接访问。


## L11 方法
- cpp可以把函数的定义写到结构体里面,这样这个函数就成为了这个结构体的方法
### this关键字
- 在 C++ 中，**this 是一个特殊的类型为self(结构体是什类型它就是什么类型)的指针**，指向当前对象的地址。它只   能在类的成员函数内部使用，表示调用该成员函数的对象本身this    关键字允许你在类的方法内部访问和操作当前对象的员变量和方法， 尤其是在对象的成员与局部变量或函数参数发命名冲突时，它非常有   用。
this 关键字的基本概念
this 是 指针 类型，指向当前对象的地址。
它只能在 成员函数 中使用，指向调用该成员函数的对象。
this 允许你直接访问当前对象的 成员变量 和 成员函数。
this 不能在 静态成员函数 中使用，因为静态成员函数不与某特定   对象关联。
示例：
```cpp
#include <iostream>
using namespace std;
class MyClass {
public:
    int value;
    // 成员函数，使用 this 指针
    void setValue(int value) {
        this->value = value;  // 使用 this 指针来区成员   变量和参数
    }
    void showValue() {
        cout << "The value is " << this->value <<  endl;  // 使用 this 访问成员变量
    }
};
int main() {
    MyClass obj;
    obj.setValue(10);  // 调用 setValue 方法，this 向     obj
    obj.showValue();   // 输出：The value is 10
    return 0;
}
```
在上面的例子中：
this->value 是指访问当前对象的成员变量 value。
this 指向调用成员函数的对象。在 setValue 中，它指向obj，在     showValue 中，它同样指向 obj。

### 用const修饰成员函数
### 1. **`const` 修饰成员函数：`int get(int i) const`**

`const` 修饰符放在成员函数的末尾，表示 **该成员函数不会修改对象的状态**，也就是说，这个函数不会修改类的成员变量。

- `const` 修饰符放在成员函数的后面表示 **"成员函数不会修改成员变量"**，并且保证它是一个 **常量成员函数**（const member function）。
- 这种修饰符非常重要，特别是在你希望函数能够对 `const` 对象进行调用时。例如，允许你在 `const` 类型的对象上调用该方法，而不会发生编译错误。

#### 为什么要用 `const` 修饰成员函数？

- **限制行为**：`const` 成员函数不会修改类的状态，所以它只能访问类的 `const` 成员变量。
- **支持 `const` 对象**：如果一个对象是 `const` 类型（即 `const Fibonacci FIB`），那么只能调用那些被 `const` 修饰的成员函数。这样可以确保你不会意外地修改一个本应保持不变的对象。

##### 示例：

```cpp
struct Fibonacci {
    int numbers[11];

    // const 成员函数：承诺不会修改类的成员变量
    int get(int i) const {
        return numbers[i];
    }
};
```

这里，`get` 是一个 `const` 成员函数，它承诺 **不会修改 `numbers` 数组中的元素**。这使得你可以通过 `const` 对象调用 `get` 方法，且不会改变对象的状态。

如果没有 `const` 修饰符，你将不能在 `const` 对象上调用 `get` 方法：

```cpp
const Fibonacci FIB{{0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55}};
FIB.get(10);  // 只能调用 const 成员函数
```

否则编译器会报错。

### 2. **`const` 修饰 `FIB` 对象：`Fibonacci constexpr FIB{{0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55}};`**

在这行代码中，`const` 关键字的作用是使得 `FIB` 对象本身成为 **常量对象**。一旦对象被定义为 `const`，就意味着 **该对象的状态（成员变量）不能被修改**。

- `const Fibonacci FIB`：表示 `FIB` 是一个 `Fibonacci` 类型的常量对象，这意味着 `FIB` 对象的所有成员变量在程序中都不可修改。
- `constexpr` 修饰符：表示 `FIB` 对象是一个 **常量表达式**，在编译时已经确定了值。它要求 `FIB` 的构造函数中的所有数据必须是 **常量表达式**，也就是编译时已知的常量值。

这种定义方式确保了 `FIB` 对象在程序中不可被修改，同时它也需要在编译时就能够确定。

#### 关键点：

- `const` 确保对象不可变。
- `constexpr` 确保对象的值在编译时是已知的。
---



## L12 类 访问修饰符
- C++ 中，`class` 和 `struct` 之间的**唯一区别**是
`class` 默认访问控制符是 `private`，
`struct` 默认访问控制符是 `public`。
READ: 访问说明符 <https://zh.cppreference.com/w/cpp/language/access>
这个 class 中的字段被 private 修饰，只能在 class 内部访问。
因此必须提供构造器来初始化字段。
READ: 构造器 <https://zh.cppreference.com/w/cpp/language/constructor>
### public和private访问修饰符

- 在 C++ 中，public 和 private 是访问修饰符（access specifiers），它们决定了类的成员（变量和函数）对外界的可访问性。C++ 提供了三种主要的访问修饰符：
> public：表示该成员是公开的，任何地方都可以访问。
> private：表示该成员是私有的，只有该类的成员函数可以访问，外部代码不能直接访问。
> protected：表示该成员对类的成员函数和派生类（子类）的成员函数是可访问的，但外部代码不能直接访问。
- 写法举例:
```cpp
class MyClass {
public: //访问修饰符后面的变量都是这个类型,直到遇到下一个访问修饰符
    int publicVar;  // public 成员变量

    void setPublicVar(int value) {
        publicVar = value;  // 可以访问 public 成员变量
    }
};
```

## L13 构造函数
构造器（又称构造函数）是类中的一种特殊函数，它在 对象创建时自动调用，用于 初始化对象的状态。  
**就是类中一个用来初始化的函数**
- 构造器的基本特点
    - 与类名相同：构造器的名字必须与类名相同。
    - 没有返回类型：构造器没有返回值类型，甚至不能是 void 类型。
    - 自动调用：构造器在创建对象时由编译器自动调用。
    - 可以重载：构造器是可以重载的，意味着可以定义多个构造器来初始化对象的成员变量。
- 写法(这是把三种写到一起了)
```cpp
class MyClass {
public:
    int a;
    double b;
   // 默认构造器
    MyClass() {
        a = 0;  // 初始化成员变量
        b = 0.0;
        cout << "Default constructor called!" << endl;
    }
    // 带参构造器
    MyClass(int x, double y) {
        a = x;  // 使用参数初始化成员变量
        b = y;
        cout << "Parameterized constructor called!" << endl;
    }
    // 拷贝构造器
    MyClass(const MyClass &other) {
        a = other.a;  // 复制成员变量
        b = other.b;
        cout << "Copy constructor called!" << endl;
    }
}

    MyClass obj;  // 自动调用默认构造器
    MyClass obj(10, 3.14);  // 使用带参构造器
    MyClass obj2 = obj1;     // 调用拷贝构造器
```
- 要特别注意[列表初始化](#l10-初始化)


## L14 析构函数
- 凡是new 必须有特定delete
- 析构器就是用来delete的
在 C++ 中，**析构函数**（Destructor）是一个特殊的成员函数，用于在对象生命周期结束时进行清理工作，通常用于释放对象动态分配的资源。析构函数的主要作用是确保当对象被销毁时，所分配的资源（如内存、文件句柄、数据库连接等）能够被正确释放，防止资源泄漏。

### 1. **析构函数的定义**
析构函数的名字与类名相同，但在前面加上波浪号 `~`。析构函数不接受任何参数，并且没有返回值。

```cpp
class MyClass {
public:
    MyClass() {
        // 构造函数：可以进行资源分配
    }

    ~MyClass() {
        // 析构函数：清理资源
    }
};
```

### 2. **析构函数的作用**
析构函数的主要作用是：

- 释放对象的动态内存：如果类的成员是通过 `new` 动态分配的，析构函数需要调用 `delete` 来释放这些内存。
- 清理其他资源：例如关闭文件、断开网络连接、释放图形资源等。

### 3. **析构函数和 `delete` 的关系**
当使用 `new` 创建一个对象时，系统会在堆上分配内存；相应地，当对象不再使用时，必须调用 `delete` 来释放这块内存。如果使用 `new` 分配了内存，但没有及时调用 `delete`，就会发生内存泄漏。

**析构函数的作用**之一就是在对象销毁时自动调用 `delete`，以释放通过 `new` 分配的内存。具体来说，析构函数会被隐式地调用，当对象超出作用域或手动调用 `delete` 时，析构函数会清理对象的资源。

### 4. **示例代码**
```cpp
#include <iostream>

class MyClass {
private:
    int* data;  // 动态分配的内存

public:
    // 构造函数
    MyClass(int value) {
        data = new int(value);  // 动态分配内存
        std::cout << "Constructor: data = " << *data << std::endl;
    }

    // 析构函数
    ~MyClass() {
        delete data;  // 释放内存
        std::cout << "Destructor: data deleted." << std::endl;
    }
};

int main() {
    MyClass obj(10);  // 创建对象
    // 当 obj 离开作用域时，析构函数会自动调用
    return 0;
}
```

**输出：**
```
Constructor: data = 10
Destructor: data deleted.
```

在上面的例子中：
- 构造函数使用 `new` 动态分配了一个整数。
- 析构函数在对象销毁时自动调用 `delete`，释放了 `new` 分配的内存。

### 5. **析构函数的调用时机**
析构函数在以下几种情况下被调用：
- **对象生命周期结束**：当对象超出其作用域时，析构函数会被自动调用。
- **通过 `delete` 删除对象**：当使用 `new` 创建了对象并通过 `delete` 删除时，析构函数会被调用。

### 6. **注意事项**
- **一个类只能有一个析构函数**，因为析构函数没有参数，不能重载。
- **基类和派生类的析构函数**：如果派生类对象通过基类指针删除，应该保证基类的析构函数是虚拟的，以确保派生类的析构函数被正确调用。
  
```cpp
class Base {
public:
    virtual ~Base() {
        std::cout << "Base Destructor" << std::endl;
    }
};

class Derived : public Base {
public:
    ~Derived() override {
        std::cout << "Derived Destructor" << std::endl;
    }
};

int main() {
    Base* obj = new Derived();
    delete obj;  // 触发 Derived 和 Base 的析构函数
}
```

**输出：**
```
Derived Destructor
Base Destructor
```

### 7. **总结**
析构函数是确保对象销毁时能正确释放资源的关键。任何通过 `new` 动态分配的资源都应该在析构函数中释放。合理使用析构函数可以避免内存泄漏等问题。


## sp new的语法
```cpp
new T
new T[c]
new T()
new T{}
new T({})
new T{{}}
new T[]{t0, t1}
new T[c]{t0, t1}
new T(a0, a1)
new T{a0, a1}
```
- 实际的例子
```cpp
#include <iostream>

class MyClass {
public:
    int x, y;

    // 默认构造函数
    MyClass() : x(0), y(0) {}

    // 带参数的构造函数
    MyClass(int a, int b) : x(a), y(b) {}

    // 打印成员
    void print() {
        std::cout << "x: " << x << ", y: " << y << std::endl;
    }
};

int main() {
    // new T
    MyClass* obj1 = new MyClass;  // 分配一个 MyClass 对象，并调用默认构造函数
    obj1->print();  // 输出 x: 0, y: 0
    delete obj1;

    // new T[c]
    MyClass* arr1 = new MyClass[2];  // 分配一个大小为 2 的 MyClass 数组，并调用默认构造函数
    arr1[0].print();  // 输出 x: 0, y: 0
    arr1[1].print();  // 输出 x: 0, y: 0
    delete[] arr1;

    // new T()
    MyClass* obj2 = new MyClass();  // 分配一个 MyClass 对象，并调用默认构造函数
    obj2->print();  // 输出 x: 0, y: 0
    delete obj2;

    // new T{}
    MyClass* obj3 = new MyClass{};  // 分配一个 MyClass 对象，并调用默认构造函数，进行值初始化
    obj3->print();  // 输出 x: 0, y: 0
    delete obj3;

    // new T({})
    MyClass* obj4 = new MyClass({});  // 分配一个 MyClass 对象，并进行列表初始化
    obj4->print();  // 输出 x: 0, y: 0
    delete obj4;

    // new T{{}}
    MyClass* obj5 = new MyClass{{}};  // 分配一个 MyClass 对象，并通过嵌套列表初始化
    obj5->print();  // 输出 x: 0, y: 0
    delete obj5;

    // new T[]{t0, t1}
    MyClass* arr2 = new MyClass[]{ MyClass(1, 2), MyClass(3, 4) };  // 创建一个 MyClass 数组，并用给定的值初始化
    arr2[0].print();  // 输出 x: 1, y: 2
    arr2[1].print();  // 输出 x: 3, y: 4
    delete[] arr2;

    // new T[c]{t0, t1}
    MyClass* arr3 = new MyClass[3]{ MyClass(1, 2), MyClass(3, 4) };  // 创建一个大小为 3 的 MyClass 数组，初始化前两个元素
    arr3[0].print();  // 输出 x: 1, y: 2
    arr3[1].print();  // 输出 x: 3, y: 4
    arr3[2].print();  // 输出 x: 0, y: 0 (默认构造)
    delete[] arr3;

    // new T(a0, a1)
    MyClass* obj6 = new MyClass(5, 6);  // 创建一个 MyClass 对象，并通过带参数的构造函数初始化
    obj6->print();  // 输出 x: 5, y: 6
    delete obj6;

    // new T{a0, a1}
    MyClass* obj7 = new MyClass{7, 8};  // 创建一个 MyClass 对象，并使用列表初始化
    obj7->print();  // 输出 x: 7, y: 8
    delete obj7;

    return 0;
}
```

## L15 类的相关操作
### 复制构造器
### 移动构造器
**先复习移动语义**
// READ: 如果实现移动构造 <https://learn.microsoft.com/zh-cn/cpp/cpp/move-constructors-and-move-assignment-operators-cpp?view=msvc-170>

// READ: 移动构造函数 <https://zh.cppreference.com/w/cpp/language/move_constructor>
// READ: 移动赋值 <https://zh.cppreference.com/w/cpp/language/move_assignment>
// READ: 运算符重载 <https://zh.cppreference.com/w/cpp/language/operators>
### 类的继承 基类
#### 1. **基本继承**

在 C++ 中，类的继承使得子类能够继承父类的成员（属性和方法），并且可以重写父类的方法来扩展其功能。继承的基本语法为：

```cpp
class 子类 : 访问权限 基类 {
    // 子类成员
};
```

- **访问权限**：包括 `public`、`protected` 和 `private`，决定了继承后子类可以访问父类哪些成员。

##### 示例：基本继承

```cpp
#include <iostream>

// 基类（父类）
class Animal {
public:
    void speak() {
        std::cout << "Animal is making a sound" << std::endl;
    }
};

// 子类（派生类）
class Dog : public Animal {
public:
    void speak() {  // 重写父类的 speak 方法
        std::cout << "Dog is barking" << std::endl;
    }
};

int main() {
    Dog dog;
    dog.speak();  // 输出 "Dog is barking"
    
    Animal animal;
    animal.speak();  // 输出 "Animal is making a sound"
    
    return 0;
}
```

- **子类**继承自**父类**`Animal`，并重写了父类的`speak`方法。
- **父类**的`speak`方法输出"Animal is making a sound"。
- **子类**的`speak`方法输出"Dog is barking"。

#### 2. **构造函数与继承**

在 C++ 中，父类的构造函数不会自动被子类继承。如果子类需要使用父类的构造函数，则必须显式调用父类构造函数。

##### 示例：构造函数与继承

```cpp
#include <iostream>
#include <string>

// 基类（父类）
class Animal {
public:
    Animal(std::string name) {
        this->name = name;
    }
    
    void speak() {
        std::cout << "Animal " << name << " is making a sound" << std::endl;
    }
    
private:
    std::string name;
};

// 子类（派生类）
class Dog : public Animal {
public:
    Dog(std::string name) : Animal(name) {}  // 调用父类的构造函数
    
    void speak() {  // 重写父类的 speak 方法
        std::cout << "Dog is barking" << std::endl;
    }
};

int main() {
    Dog dog("Buddy");
    dog.speak();  // 输出 "Dog is barking"
    
    Animal animal("Generic Animal");
    animal.speak();  // 输出 "Animal Generic Animal is making a sound"
    
    return 0;
}
```

- **父类**`Animal`有一个构造函数接收一个名字，并将其存储在成员变量`name`中。
- **子类**`Dog`在构造时调用了父类的构造函数来初始化`name`。

#### 3. **多重继承**

C++ 支持多重继承，即一个子类可以继承多个父类的成员。需要注意的是，子类继承多个父类时，如果父类有相同的成员变量或方法，可能会出现冲突，需要解决。

##### 示例：多重继承

```cpp
#include <iostream>

// 基类1
class Animal {
public:
    void speak() {
        std::cout << "Animal is making a sound" << std::endl;
    }
};

// 基类2
class Mammal {
public:
    void breathe() {
        std::cout << "Mammal is breathing" << std::endl;
    }
};

// 子类
class Dog : public Animal, public Mammal {
public:
    void speak() {
        std::cout << "Dog is barking" << std::endl;
    }
};

int main() {
    Dog dog;
    dog.speak();    // 输出 "Dog is barking"
    dog.breathe();  // 输出 "Mammal is breathing"
    return 0;
}
```

- **子类**`Dog`继承了两个父类：`Animal`和`Mammal`，并重写了父类的`speak`方法。
- 子类可以访问父类的所有公共方法，包括`Animal`的`speak`和`Mammal`的`breathe`。

#### 4. **虚拟继承（Virtual Inheritance）**

虚拟继承解决了多重继承中的“钻石继承”问题。当多个类通过继承共享同一个父类时，子类可能会继承多份父类数据。虚拟继承确保子类只继承父类的数据一次。

##### 示例：虚拟继承

```cpp
#include <iostream>

// 基类 A
class A {
public:
    int x;
};

// 基类 B 和 C 都虚拟继承 A
class B : virtual public A {};  
class C : virtual public A {};  

// 子类 D
class D : public B, public C {
public:
    void print() {
        std::cout << x << std::endl;  // 只会输出 A 中的 x
    }
};

int main() {
    D d;
    d.x = 10;
    d.print();  // 输出 10
    return 0;
}
```

- **虚拟继承**确保`B`和`C`共享同一个`A`的实例，因此`D`类只会继承一份`A`中的成员。
- `D`类可以访问`A`类中的成员变量`x`，而不会出现重复继承的问题。

### 总结

- **继承**是 C++ 面向对象编程中的一个核心特性，允许子类复用父类的成员，并扩展或重写父类的功能。
- C++ 支持**单继承**和**多重继承**，并通过**虚拟继承**解决多重继承中的钻石问题。
- 子类可以通过构造函数初始化父类成员，并且可以重写父类的方法以修改行为。
- 在多重继承中，子类可能继承多个父类的数据或方法，这可能导致冲突，需要小心处理。

通过继承，代码重用性和扩展性得到了显著提升，但在设计时需要考虑访问权限、构造函数调用以及可能出现的继承冲突。


## L16 重载
### C++ 中的函数重载

**重载（Overloading）**是 C++ 中允许在同一个作用域内定义多个同名但参数不同的函数或运算符的特性。重载允许我们使用相同的函数名来执行不同的操作，这样可以提高代码的可读性和可维护性。

#### 1. **函数重载**

函数重载指的是在同一个类或命名空间中，可以定义多个同名但参数列表（参数个数或类型）不同的函数。编译器会根据函数调用时传递的参数类型和数量来决定调用哪个版本的函数。

##### 函数重载的规则：
- **函数名相同**。
- **参数个数不同**，或者
- **参数类型不同**，但不能仅通过返回值类型的不同来重载。

##### 示例：函数重载

```cpp
#include <iostream>
using namespace std;

class Print {
public:
    // 打印整数
    void display(int i) {
        cout << "Integer: " << i << endl;
    }
    
    // 打印浮点数
    void display(double d) {
        cout << "Double: " << d << endl;
    }
    
    // 打印字符串
    void display(string s) {
        cout << "String: " << s << endl;
    }
};

int main() {
    Print p;
    p.display(5);          // 调用 display(int)
    p.display(3.14);       // 调用 display(double)
    p.display("Hello!");   // 调用 display(string)
    
    return 0;
}
```

输出：

```
Integer: 5
Double: 3.14
String: Hello!
```

在这个例子中，`display` 函数被重载了三次，分别接受 `int`、`double` 和 `string` 类型的参数。根据传递的参数类型，编译器会选择调用合适的重载版本。

#### 2. **构造函数重载**

构造函数也可以被重载，允许我们在创建对象时根据不同的参数类型或数量来初始化对象的状态。

##### 示例：构造函数重载

```cpp
#include <iostream>
using namespace std;

class Rectangle {
private:
    int width, height;
    
public:
    // 默认构造函数
    Rectangle() {
        width = 0;
        height = 0;
    }
    
    // 带参数的构造函数
    Rectangle(int w, int h) {
        width = w;
        height = h;
    }
    
    void display() {
        cout << "Width: " << width << ", Height: " << height << endl;
    }
};

int main() {
    Rectangle r1;        // 调用默认构造函数
    Rectangle r2(10, 20); // 调用带参数的构造函数
    
    r1.display();  // 输出 Width: 0, Height: 0
    r2.display();  // 输出 Width: 10, Height: 20
    
    return 0;
}
```

在这个例子中，`Rectangle` 类有两个构造函数，一个是默认构造函数，另一个是带有两个参数的构造函数。我们可以根据需要选择不同的构造函数来初始化对象。

#### 3. **运算符重载**

C++ 允许我们为内置类型（如整数、浮点数）以外的数据类型重载运算符。运算符重载使得我们能够使用自定义类型与内置类型一样进行运算。

##### 示例：运算符重载

```cpp
#include <iostream>
using namespace std;

class Complex {
private:
    float real;
    float imag;
    
public:
    Complex(float r, float i) : real(r), imag(i) {}

    // 重载加法运算符
    Complex operator + (const Complex& other) {
        return Complex(real + other.real, imag + other.imag);
    }
    
    void display() {
        cout << "Real: " << real << ", Imaginary: " << imag << endl;
    }
};

int main() {
    Complex c1(3.5, 2.5);
    Complex c2(1.5, 4.5);
    
    Complex c3 = c1 + c2;  // 调用重载的加法运算符
    
    c3.display();  // 输出 Real: 5, Imaginary: 7
    
    return 0;
}
```

在这个例子中，我们重载了 `+` 运算符，使得我们可以直接使用 `+` 运算符对两个 `Complex` 类型的对象进行相加。

#### 4. **重载注意事项**

- **返回类型不能作为区分重载的唯一依据**：也就是说，两个函数如果只有返回类型不同，编译器无法判断调用哪个函数，因此不允许仅通过返回类型来进行重载。
- **不能通过参数的默认值来重载函数**：函数默认参数值的不同不会导致函数的重载。

##### 错误的重载示例：

```cpp
#include <iostream>
using namespace std;

class Test {
public:
    int add(int x, int y) {
        return x + y;
    }

    // 错误的重载：返回类型不同
    float add(int x, int y) {
        return (float)(x + y);
    }
};
```

这个代码将导致编译错误，因为 `add` 函数的返回类型不同，但它们的参数完全相同，编译器无法根据返回类型来区分这两个函数。

#### 5. **总结**

- **函数重载**允许在同一个作用域中定义多个同名函数，它们的参数个数或类型不同。
- 函数重载提高了代码的可读性和可维护性，使得相同的操作可以使用相同的函数名来处理不同类型的数据。
- **构造函数重载**可以让我们根据不同的需求初始化对象。
- **运算符重载**允许自定义类型使用与内置类型相同的操作符。
  
重载的本质是通过函数名相同、参数不同来让编译器根据调用时传递的参数来选择正确的函数。通过合理使用重载，能够使代码更简洁、灵活和易于理解。

### C++ 中的 **重写（Overriding）** 和 **重载（Overloading）** 的区别

**重写**（Overriding）和**重载**（Overloading）是面向对象编程中的两个重要概念，虽然它们看起来相似，但有本质的不同。下面我们来详细讲解这两个概念及其区别。

#### 1. **重载（Overloading）**

**重载**是指在同一个类中，可以定义多个同名但参数不同的函数或方法。重载的目的是根据不同的参数类型或参数个数，执行不同的操作。

##### 主要特点：
- 发生在**同一个类**中。
- 通过**参数不同**来区分重载函数（可以是参数个数、参数类型、参数顺序等）。
- **返回类型**不能作为重载的依据，必须通过参数来区分。
- 可以在**同一个类**中定义多个重载的函数。

##### 示例：函数重载

```cpp
#include <iostream>
using namespace std;

class Printer {
public:
    // 打印整数
    void print(int i) {
        cout << "Printing integer: " << i << endl;
    }

    // 打印浮点数
    void print(double d) {
        cout << "Printing double: " << d << endl;
    }

    // 打印字符串
    void print(string s) {
        cout << "Printing string: " << s << endl;
    }
};

int main() {
    Printer p;
    p.print(5);        // 调用 print(int)
    p.print(3.14);     // 调用 print(double)
    p.print("Hello");  // 调用 print(string)
    
    return 0;
}
```

输出：

```
Printing integer: 5
Printing double: 3.14
Printing string: Hello
```

在这个例子中，`print` 函数被重载了三次，它们接受不同类型的参数：`int`、`double` 和 `string`。

#### 2. **重写（Overriding）**

**重写**是指在子类中重新定义父类中已经定义过的虚函数，目的是修改或扩展父类的功能。重写要求父类的方法是**虚函数**（`virtual`），这样子类就可以通过自己的实现覆盖父类的方法。

##### 主要特点：
- 发生在**父类和子类**之间。
- 子类重新定义父类的**虚函数**，并改变或扩展其行为。
- **返回类型**必须与父类的返回类型一致（或者是父类返回类型的派生类）。
- **参数列表**必须与父类中的虚函数完全相同。

##### 示例：函数重写

```cpp
#include <iostream>
using namespace std;

class Animal {
public:
    // 虚函数
    virtual void sound() {
        cout << "Animal is making a sound" << endl;
    }
};

class Dog : public Animal {
public:
    // 重写父类的 sound 方法
    void sound() override {
        cout << "Dog is barking" << endl;
    }
};

int main() {
    Animal* animal = new Dog();
    animal->sound();  // 输出 "Dog is barking"
    
    delete animal;
    return 0;
}
```

输出：

```
Dog is barking
```

在这个例子中，`Dog` 类重写了父类 `Animal` 的 `sound` 方法。因为 `sound` 是一个虚函数（`virtual`），所以当通过 `Animal*` 指针调用 `sound` 方法时，实际执行的是 `Dog` 类中的 `sound` 方法（即多态）。

#### 3. **重载 vs 重写：对比**

| 特性           | **重载（Overloading）**                              | **重写（Overriding）**                                 |
|----------------|-----------------------------------------------------|-------------------------------------------------------|
| **定义**       | 同一个类中多个函数使用相同的名字，但参数不同（参数个数或类型不同）。 | 子类重新定义父类的虚函数，改变其实现。                |
| **作用域**     | 发生在同一个类内。                                  | 发生在父类和子类之间。                                 |
| **函数签名**   | 通过**函数名**和**参数列表**来区分。                 | 通过函数的**签名**（函数名和参数列表）相同。          |
| **返回类型**   | 可以有不同的返回类型，但不能仅通过返回类型来区分。    | 必须和父类的返回类型一致（或者是父类返回类型的派生类）。|
| **多态性**     | 不涉及多态性，编译时根据参数选择调用哪个重载函数。    | 通过多态，运行时决定调用父类还是子类的函数。          |
| **函数体**     | 可以在同一类中定义多个具有相同名字但不同参数的函数。   | 子类必须提供对父类虚函数的具体实现。                  |

#### 4. **总结**

- **重载**（Overloading）是指在同一个类中定义多个同名但参数不同的函数，用于处理不同类型或数量的参数。
- **重写**（Overriding）是指子类重新定义父类的虚函数，修改其行为并覆盖父类的实现。
- **重载**是编译时多态，通过函数参数的不同来选择调用哪个函数。
- **重写**是运行时多态，依赖于虚函数和继承关系，允许子类改变父类方法的行为。

理解这两个概念有助于你在 C++ 编程中更好地利用面向对象的特性，提高代码的可读性、可维护性和灵活性。


## L17 虚函数与多态
先看L16的[函数重写](#2-重写overriding)

### 1. 函数重写 (Overriding)
在面向对象编程中，函数重写指的是在子类中重新定义父类中已经实现的函数。重写要求子类函数的函数签名（函数名、参数列表和返回类型）必须与父类函数保持一致。通过重写，子类可以改变父类的默认行为。

而且override关键字可以不写，主要是增加可读性，防止和重载混淆

#### 例子：
```cpp
class Base {
public:
    void show() { 
        cout << "Base show" << endl; 
    }
};

class Derived : public Base {
public:
    void show() override {  // 重写父类的 show 函数
        cout << "Derived show" << endl;
    }
};
```

在这里，`Derived` 重写了 `Base` 类中的 `show()` 函数。调用 `show()` 会根据对象的实际类型决定执行哪个版本。

### 2. 虚函数 (Virtual Functions)
虚函数是基类中声明为 `virtual` 的成员函数，允许在派生类中重写该函数的行为。虚函数通过虚表（vtable）机制实现运行时的多态性。

#### 2.1 虚函数的作用
虚函数的关键作用是：在运行时通过基类指针或引用调用子类的重写版本，而不是基类的版本。通过虚函数，我们可以实现 **动态多态**。

#### 2.2 如何声明虚函数？
要声明虚函数，只需要在基类中的函数声明前加上 `virtual` 关键字。

```cpp
class Base {
public:
    virtual void show() {  // 基类中的虚函数
        cout << "Base show" << endl;
    }
};
```

#### 2.3 如何重写虚函数？
在派生类中使用相同的函数签名来重写基类的虚函数。可以使用 `override` 关键字明确表示这是一个重写的虚函数。

```cpp
class Derived : public Base {
public:
    void show() override {  // 派生类重写虚函数
        cout << "Derived show" << endl;
    }
};
```

### 3. 虚表 (VTable) 和 虚指针 (VPtr)
虚函数的实现依赖于虚表（VTable）和虚指针（VPtr）。每个类如果包含虚函数，编译器会为该类生成一个虚表，虚表是一个函数指针数组，每个元素指向该类的虚函数。

#### 3.1 虚表（VTable）
虚表是由编译器自动生成的，它包含了类的所有虚函数的指针。每个含有虚函数的类都有一个虚表。

#### 3.2 虚指针（VPtr）
每个对象都包含一个虚指针（VPtr），它指向该类的虚表。通过虚指针，程序能够在运行时根据对象的实际类型查找到正确的虚函数。

#### 3.3 虚函数的调用
当我们通过基类指针或引用调用虚函数时，程序会通过虚指针查找虚表，然后调用相应的虚函数。

#### 例子：
```cpp
class Base {
public:
    virtual void show() {  
        cout << "Base show" << endl;
    }
};

class Derived : public Base {
public:
    void show() override {  
        cout << "Derived show" << endl;
    }
};

int main() {
    Base* base = new Derived();  // 基类指针指向派生类对象
    base->show();  // 调用的是 Derived 类的 show()，而不是 Base 类的 show()
}
```
在这个例子中，`base->show()` 会调用 `Derived` 类中的 `show()` 方法，而不是 `Base` 类中的方法。

### 4. 虚析构函数 (Virtual Destructor)
虚析构函数是指在类的析构函数前加上 `virtual` 关键字。当类中包含虚函数时，析构函数也应该是虚函数。这样，在删除通过基类指针指向的派生类对象时，能够正确地调用派生类的析构函数，避免内存泄漏。

#### 例子：
```cpp
class Base {
public:
    virtual ~Base() {  // 基类虚析构函数
        cout << "Base destructor" << endl;
    }
};

class Derived : public Base {
public:
    ~Derived() override {  // 派生类虚析构函数
        cout << "Derived destructor" << endl;
    }
};

int main() {
    Base* base = new Derived();  // 基类指针指向派生类对象
    delete base;  // 输出: Derived destructor  Base destructor
}
```
通过将析构函数声明为虚函数，我们确保通过基类指针删除派生类对象时，派生类的析构函数会先被调用。

### 5. 静态字段 (Static Fields)
静态字段是类级别的成员，所有对象共享该字段，而不是每个对象拥有一份自己的副本。静态成员通常用于存储一些全局或类级别的数据。

#### 例子：
```cpp
class MyClass {
public:
    static int count;  // 静态成员变量
};
int MyClass::count = 0;  // 静态成员变量的定义
```
静态成员变量不属于任何对象，而是属于整个类，所有对象共享同一份静态数据。


## sp 理解多态

### 1. 什么是多态？
多态（Polymorphism）指的是同一个操作（方法或函数）能够作用于不同类型的对象，并根据对象的实际类型表现出不同的行为。多态是面向对象编程的核心特性之一，它使得程序更具可扩展性和灵活性。

多态有两种主要类型：
- **编译时多态（静态多态）**：通过函数重载、运算符重载等方式实现。
- **运行时多态（动态多态）**：通过虚函数和继承关系实现。

### 2. 运行时多态
运行时多态是通过虚函数来实现的，基类的指针或引用指向派生类的对象，调用虚函数时，程序会动态选择调用哪一个函数实现。

#### 例子：
```cpp
class Shape {
public:
    virtual void draw() {  // 基类虚函数
        cout << "Drawing a shape" << endl;
    }
};

class Circle : public Shape {
public:
    void draw() override {  // 派生类重写虚函数
        cout << "Drawing a circle" << endl;
    }
};

class Square : public Shape {
public:
    void draw() override {  // 派生类重写虚函数
        cout << "Drawing a square" << endl;
    }
};

void render(Shape* shape) {
    shape->draw();  // 多态调用，根据传入的对象类型决定调用哪个函数
}

int main() {
    Circle circle;
    Square square;
    render(&circle);  // 输出: Drawing a circle
    render(&square);  // 输出: Drawing a square
}
```

在这个例子中，`render` 函数通过基类 `Shape` 的指针来调用派生类 `Circle` 或 `Square` 的 `draw()` 方法。多态的机制使得 `draw()` 方法根据对象的实际类型来决定执行哪个版本。

### 3. 多态的优势
- **灵活性**：通过多态，代码能够在运行时根据对象类型选择行为，增加了代码的灵活性。
- **可扩展性**：新的派生类可以在不修改已有代码的情况下添加，只需要继承基类并重写虚函数。
- **代码复用**：多态减少了重复代码，通过基类指针操作不同的派生类对象。

### 4. 多态的实现机制
多态依赖于以下几个机制：
- **虚函数**：基类中声明为虚函数的成员函数，允许在派生类中进行重写。
- **虚表（VTable）**：虚表是类的虚函数指针数组，指向该类的虚函数。
- **虚指针（VPtr）**：每个对象包含一个指向虚表的指针，用来执行动态绑定。

### 5. 多态的优势
- **提高代码的灵活性**：允许不同类型的对象共享相同的接口，增加了系统的扩展性。
- **代码复用**：通过父类指针或引用，代码可以统一处理不同派生类的对象，避免重复编写类似的代码。

## sp final 关键字、纯虚函数、抽象类和虚继承

### 1. `final` 关键字

在 C++ 中，`final` 关键字用于限制类或成员函数的继承或重写行为，增强了程序的安全性和可维护性。

#### 1.1 `final` 用于类

当 `final` 用于类时，表示该类不能被继承，即禁止其他类继承这个类。

#### 例子：
```cpp
class Base final {  // 使用 final 禁止继承
public:
    void show() {
        cout << "Base show" << endl;
    }
};

// 错误：无法继承 Base 类
class Derived : public Base {
};
```
在这个例子中，`Base` 类被标记为 `final`，所以无法再派生出 `Derived` 类。编译器会报错。

#### 1.2 `final` 用于成员函数

当 `final` 用于成员函数时，表示该函数不能在派生类中被重写。

#### 例子：
```cpp
class Base {
public:
    virtual void show() final {  // 使用 final 禁止重写
        cout << "Base show" << endl;
    }
};

class Derived : public Base {
public:
    // 错误：不能重写 show() 函数
    void show() override {
        cout << "Derived show" << endl;
    }
};
```
在这个例子中，`Base` 类中的 `show()` 函数被标记为 `final`，因此在 `Derived` 类中无法重写这个函数。

### 2. 纯虚函数 (Pure Virtual Functions)

纯虚函数是指在基类中声明但不实现的虚函数，它以 `= 0` 作为函数声明的结束标志。含有纯虚函数的类称为 **抽象类**。纯虚函数没有函数体，在派生类中必须进行实现，否则派生类也会成为抽象类，不能直接实例化。

#### 2.1 纯虚函数的声明
纯虚函数的声明形式是：在虚函数后加上 `= 0`，表示该函数没有实现，需要在派生类中重写。

#### 例子：
```cpp
class Shape {
public:
    virtual void draw() = 0;  // 纯虚函数，没有实现
};

class Circle : public Shape {
public:
    void draw() override {  // 派生类实现纯虚函数
        cout << "Drawing a circle" << endl;
    }
};
```

在这个例子中，`Shape` 类包含了一个纯虚函数 `draw()`，它没有函数体。`Circle` 类继承 `Shape` 类并实现了 `draw()` 函数。由于 `Shape` 是抽象类，它不能直接实例化；但是可以创建 `Circle` 类的对象，因为它实现了所有纯虚函数。

### 3. 抽象类 (Abstract Class)

抽象类是指含有至少一个纯虚函数的类。抽象类不能直接实例化，它的目的是提供一个统一的接口供派生类实现。抽象类可以包含已经实现的成员函数、纯虚函数以及数据成员。

#### 3.1 抽象类的特点
- 含有至少一个纯虚函数。
- 不能实例化对象。
- 作为基类，供派生类继承和实现。

#### 例子：
```cpp
class AbstractShape {
public:
    virtual void draw() = 0;  // 纯虚函数
    virtual void resize() = 0;  // 纯虚函数
};

class Rectangle : public AbstractShape {
public:
    void draw() override {
        cout << "Drawing a rectangle" << endl;
    }

    void resize() override {
        cout << "Resizing a rectangle" << endl;
    }
};
```

在这个例子中，`AbstractShape` 是一个抽象类，包含两个纯虚函数 `draw()` 和 `resize()`。`Rectangle` 类继承了 `AbstractShape`，并实现了所有的纯虚函数，因此 `Rectangle` 类可以实例化。

### 4. 虚继承 (Virtual Inheritance)

虚继承是 C++ 中的一种特殊的继承方式，解决了多重继承中的 **菱形继承问题**。菱形继承问题指的是在多重继承中，多个派生类继承自同一个基类时，可能会导致基类成员的重复继承。虚继承通过共享基类的唯一实例来避免这种重复继承。

#### 4.1 菱形继承问题
假设我们有四个类，`A` 是基类，`B` 和 `C` 继承自 `A`，然后 `D` 类同时继承自 `B` 和 `C`。如果 `B` 和 `C` 都继承了 `A`，那么 `D` 会从 `B` 和 `C` 各继承一份 `A` 类的成员，这会导致 `A` 类的成员在 `D` 类中被重复继承。

#### 4.2 虚继承的解决方案
通过虚继承，`D` 类只会从 `B` 和 `C` 继承一个 `A` 类的实例，从而避免了重复继承的问题。

#### 例子：
```cpp
class A {
public:
    void show() {
        cout << "Class A" << endl;
    }
};

class B : virtual public A {
public:
    void showB() {
        cout << "Class B" << endl;
    }
};

class C : virtual public A {
public:
    void showC() {
        cout << "Class C" << endl;
    }
};

class D : public B, public C {
public:
    void showD() {
        cout << "Class D" << endl;
    }
};

int main() {
    D obj;
    obj.show();  // 通过虚继承，避免了重复继承 A 的成员
}
```

在这个例子中，`B` 和 `C` 都通过虚继承继承了 `A` 类，这样 `D` 类就只会有一个 `A` 类的实例，而不是两个。调用 `obj.show()` 时，正确地调用了 `A` 类的方法，而不会因为重复继承导致错误。

### 5. `final` 关键字与虚继承

`final` 关键字与虚继承结合使用时，可以限制虚继承的派生类不再被继承，或者禁止虚函数在派生类中被重写。

#### 例子：
```cpp
class A {
public:
    virtual void show() final {  // 不允许派生类重写 show()
        cout << "Class A" << endl;
    }
};

class B : virtual public A {
    // 错误：不能重写 show()
    void show() override {
        cout << "Class B" << endl;
    }
};
```
在这个例子中，`A` 类的 `show()` 函数被标记为 `final`，因此 `B` 类不能重写它。



好的，宝宝！下面是关于 **面向对象** 和 **实例化** 的详细笔记：

---


## sp 静态和动态字段的初始化
```cpp
// 在类外初始化静态字段
class MyClass {
public:
    static int staticVar;  // 声明静态字段

    MyClass() {
        // 构造函数
    }

    static void staticMethod() {
        // 静态方法
    }
};
int MyClass::staticVar = 10;  // 在类外初始化静态字段
int main(){
    ...
}
```
```cpp
class MyClass {
public:
    int* dynamicVar;  // 声明动态字段

    MyClass() {
        dynamicVar = new int(20);  // 动态字段在构造器里动态分配并初始化
    }

    ~MyClass() {
        delete dynamicVar;  
    }
};
```

## sp 四种类型转换


在 C++ 中，类型转换有多种方式，其中有四种 **显式类型转换** 方式，通常称为“四种类型转换”。它们分别是：

### 1. `static_cast`
`static_cast` 是最常见的类型转换方式，用于执行 **编译时类型检查** 的转换，通常用于在 **相关类型之间**（如父类和子类之间）转换。

#### 适用场景：
- 基类与派生类之间的转换（不涉及多态）。
- 基本数据类型之间的转换（如 `int` 和 `float`）。

#### 示例：

```cpp
class Base {};
class Derived : public Base {};

Base *base = new Derived;  // 派生类指针转换为基类指针
Derived *derived = static_cast<Derived*>(base);  // 基类指针转换为派生类指针
```

注意：如果类型不兼容，`static_cast` 编译时会报错。

### 2. `dynamic_cast`
`dynamic_cast` 用于处理 **多态类型**（类中含有虚函数）。它主要用于指针或引用类型之间的转换，并且进行 **运行时类型检查**。当转换失败时，它会返回 `nullptr`（对于指针）或抛出 `std::bad_cast` 异常（对于引用）。

#### 适用场景：
- 基类和派生类之间的指针或引用转换，且涉及 **多态**。
- 安全地进行向下转换（例如基类指针转派生类指针）。

#### 示例：

```cpp
class Base {
public:
    virtual ~Base() {}  // 虚析构函数，确保多态
};
class Derived : public Base {};

Base *base = new Derived;
Derived *derived = dynamic_cast<Derived*>(base);  // 安全地转换
if (derived) {
    // 转换成功
} else {
    // 转换失败
}
```

如果 `base` 不是指向 `Derived` 类型的对象，`dynamic_cast` 会返回 `nullptr`。

### 3. `const_cast`
`const_cast` 用于 **修改常量属性**，可以 **移除** 或 **添加** `const` 或 `volatile` 限定符。它并不会改变对象的实际类型，只是改变类型的 `const` 限定符。

#### 适用场景：
- 移除对象的 `const` 限定符。
- 添加 `const` 限定符。

#### 示例：

```cpp
void foo(const int* p) {
    int* q = const_cast<int*>(p);  // 移除 const 限定符
    *q = 10;  // 修改对象的值
}

const int x = 5;
foo(&x);  // 通过 const_cast 修改 x 的值（虽然 x 被声明为 const，但会被强制转换为非 const）
```

使用 `const_cast` 时需要小心，如果尝试修改原本是 `const` 的对象，且该对象本身是 `const`，那么这会导致 **未定义行为**。

### 4. `reinterpret_cast`
`reinterpret_cast` 是最强大的类型转换方式，它可以进行 **几乎所有类型的转换**，包括指针类型之间的转换，以及将指针转换为整数等。由于它是低级的类型转换，因此需要小心使用。

#### 适用场景：
- 将一个指针类型转换为另一个不相关的指针类型。
- 将指针类型转换为整数类型或反之。
- 用于将对象转换为字节流等。

#### 示例：

```cpp
int a = 10;
void* ptr = reinterpret_cast<void*>(&a);  // int 指针转为 void 指针

int* p = reinterpret_cast<int*>(ptr);  // void 指针转回 int 指针
```

`reinterpret_cast` 能够绕过类型系统，进行几乎所有的内存级别的转换，因此它通常用在底层编程中。

### 总结：


1. **`static_cast`**：用于已知类型之间的安全转换，编译时检查。
2. **`dynamic_cast`**：用于多态类型之间的转换，运行时检查，通常用于指针和引用的转换。
3. **`const_cast`**：用于修改 `const` 或 `volatile` 限定符。
4. **`reinterpret_cast`**：低级类型转换，用于指针类型和整数类型之间的转换。

不同的类型转换方法有不同的适用场景，选择时需要根据需求来决定。如果不确定，通常优先使用 `static_cast`，而对于多态类型转换则使用 `dynamic_cast`。


## L18 函数模板
// read：https://zh.cppreference.com/w/cpp/language/function_template
#### **什么是函数模板？**

函数模板是 C++ 中实现 **泛型编程** 的一种方式，它允许你编写一个函数模板（即一个函数的蓝图），可以在不同的类型上进行操作。通过函数模板，我们可以编写通用的代码，不需要为每种数据类型重复编写多个版本的函数。

函数模板在编译时会根据传入的类型自动生成具体的函数版本，减少了代码的冗余。

#### **函数模板的基本语法**

函数模板的基本语法格式如下：

```cpp
template <typename T>
return_type function_name(parameter_list) {
    // 函数体
}
```

- `template <typename T>`：这是模板声明，告诉编译器该函数是一个模板函数，`T` 是模板参数，可以是任何合法的类型。你也可以使用 `class` 来代替 `typename`，两者是等价的。
- `return_type`：返回值类型，可以是任意合法的类型。
- `function_name`：函数的名字。
- `parameter_list`：函数的参数列表，可以是任意类型。

#### **示例：一个简单的函数模板**

```cpp
#include <iostream>

// 函数模板：交换两个变量的值
template <typename T>
void swap_values(T &a, T &b) {
    T temp = a;
    a = b;
    b = temp;
}

int main() {
    int x = 10, y = 20;
    swap_values(x, y);  // 使用模板函数交换整数
    std::cout << "x = " << x << ", y = " << y << std::endl;  // 输出：x = 20, y = 10

    double m = 3.14, n = 2.71;
    swap_values(m, n);  // 使用模板函数交换浮点数
    std::cout << "m = " << m << ", n = " << n << std::endl;  // 输出：m = 2.71, n = 3.14

    return 0;
}
```

在这个例子中，`swap_values` 是一个函数模板，它可以交换任意类型 `T` 的两个变量的值。我们可以看到，通过模板，我们可以在 `main` 函数中同时交换 `int` 类型和 `double` 类型的值，避免了为每种类型编写不同的交换函数。

#### **自动推导模板参数**

当调用模板函数时，C++ 编译器可以自动推导模板参数的类型。我们无需显式指定类型，编译器会根据传入的参数类型推导出正确的类型。

```cpp
#include <iostream>

template <typename T>
void print(T value) {
    std::cout << value << std::endl;
}

int main() {
    print(10);        // 自动推导为 int
    print(3.14);      // 自动推导为 double
    print("Hello");   // 自动推导为 const char*
    
    return 0;
}
```

在这个例子中，`print` 函数模板根据传入的参数自动推导出正确的类型，因此我们无需显式指定模板类型。

#### **模板特化（Template Specialization）**

有时我们需要为某些特定的类型提供不同的实现。这时可以使用 **模板特化**，即为特定类型定义一个特殊版本的模板。

##### **完全特化（Full Specialization）**

如果你想为特定类型定义一个特定的函数实现，可以使用完全特化：

```cpp
#include <iostream>

// 通用模板
template <typename T>
void print(T value) {
    std::cout << "Generic template: " << value << std::endl;
}

// 完全特化：为 int 类型提供一个特殊版本
template <>
void print<int>(int value) {
    std::cout << "Specialized template for int: " << value << std::endl;
}

int main() {
    print(10);    // 调用特化版
    print(3.14);  // 调用通用版
    return 0;
}
```

在这个例子中，当 `print(10)` 被调用时，编译器会选择 `print<int>` 函数模板，而对于 `print(3.14)`，则会选择通用的 `print` 函数模板。

##### **偏特化（Partial Specialization）**

偏特化是指对模板的部分参数进行特化，即对于某些特定类型的组合提供不同的实现。

```cpp
#include <iostream>

// 通用模板
template <typename T, typename U>
void print(T a, U b) {
    std::cout << "Generic template: " << a << ", " << b << std::endl;
}

// 偏特化：当两个类型相同
template <typename T>
void print(T a, T b) {
    std::cout << "Specialized template for same types: " << a << ", " << b << std::endl;
}

int main() {
    print(10, 20);      // 使用偏特化版本
    print(3.14, 2.71);  // 使用偏特化版本
    print(10, 3.14);    // 使用通用版本
    return 0;
}
```

在此例中，当两个参数类型相同（如 `int` 和 `int`）时，编译器会选择偏特化的 `print` 版本，否则选择通用版本。


## sp 浮点数比较大小
浮点数（如 `float` 和 `double`）由于精度限制，在进行计算时会产生微小的误差。因此，直接使用 `==` 来判断浮点数是否相等可能会导致错误的结果。通常，我们使用一个小的容差（epsilon）来判断两个浮点数是否足够接近。

为了通过 `ASSERT(plus(0.1, 0.2) == 0.3, "How to make this pass?");`，你可以将浮点数比较的部分修改为判断它们的差值是否足够小。

### 解决方法：
可以使用 `std::abs` 来计算两个浮点数的差，并与一个非常小的容差值（如 `1e-9`）进行比较。

修改后的代码：

```cpp
#include <cmath>  // 引入 abs 函数

template <class T>
T plus(T a, T b) {
    return a + b;
}

template <class T>
bool are_equal(T a, T b) {
    const T epsilon = 1e-9;  // 定义容差值
    return std::abs(a - b) < epsilon;  // 判断两个数的差值是否小于容差
}

int main(int argc, char **argv) {
    ASSERT(plus(1, 2) == 3, "Plus two int");
    ASSERT(plus(1u, 2u) == 3u, "Plus two unsigned int");

    ASSERT(plus(1.25f, 2.5f) == 3.75f, "Plus two float");
    ASSERT(plus(1.25, 2.5) == 3.75, "Plus two double");

    // 修改判断条件
    ASSERT(are_equal(plus(0.1, 0.2), 0.3), "How to make this pass?");

    return 0;
}
```

### 解释：
- **`are_equal`** 函数：用来判断两个浮点数是否相等。通过计算它们的差值并与容差 `epsilon` 进行比较。如果差值小于容差，认为这两个数“相等”。
- **`epsilon`**：一个很小的常量，用于定义允许的误差范围。对于不同的应用场景，容差值可能需要根据实际需求调整。

通过这种方式，可以避免由于浮点数精度问题导致的错误比较结果。

## L19 类模板
### 类模板的基本语法

类模板的定义类似于普通的类定义，不同之处在于类名之前会加上模板参数。模板参数可以是类型，也可以是非类型参数。其基本语法如下：

```cpp
template <typename T>
class MyClass {
public:
    T value;

    MyClass(T val) : value(val) {}

    void print() {
        std::cout << "Value: " << value << std::endl;
    }
};
```

- `template <typename T>`：声明了一个模板参数 `T`，它是一个类型参数。类模板将为 `T` 类型生成类的定义。
- `class MyClass`：类的定义，它使用 `T` 作为成员类型。

### 类模板的实例化

定义了类模板之后，我们可以通过具体的类型实例化类模板。这相当于“生成”了一个类，根据指定的类型参数构建类。例如：

```cpp
int main() {
    MyClass<int> obj1(42);    // 创建一个类型为 int 的对象
    obj1.print();              // 输出：Value: 42

    MyClass<double> obj2(3.14);  // 创建一个类型为 double 的对象
    obj2.print();                // 输出：Value: 3.14

    return 0;
}
```

- `MyClass<int>`：实例化 `MyClass` 模板为 `int` 类型的类。
- `MyClass<double>`：实例化 `MyClass` 模板为 `double` 类型的类。

这就像为每个类型单独生成一个类，因此同一个模板可以为不同的类型生成不同的类。

### 非类型模板参数

除了类型模板参数，C++ 还支持非类型模板参数。非类型模板参数不是类型，而是某种值（例如整数、指针等）。它可以在模板定义时指定，用来控制类的行为。语法如下：

```cpp
template <typename T, int N>
class Array {
public:
    T data[N];

    Array() {
        for (int i = 0; i < N; ++i) {
            data[i] = T();  // 默认初始化
        }
    }

    void print() {
        for (int i = 0; i < N; ++i) {
            std::cout << data[i] << " ";
        }
        std::cout << std::endl;
    }
};
```

在这个例子中，`N` 是一个非类型模板参数，它表示数组的大小。

实例化时，我们可以指定 `N` 的值：

```cpp
int main() {
    Array<int, 5> arr1;  // 创建一个大小为 5 的 int 类型数组
    arr1.print();         // 输出：0 0 0 0 0

    Array<double, 3> arr2;  // 创建一个大小为 3 的 double 类型数组
    arr2.print();            // 输出：0 0 0

    return 0;
}
```

### 模板特化（Template Specialization）

模板特化允许我们为某些特定类型定义不同的类实现。C++ 支持两种类型的模板特化：
1. **完全特化**：为某个特定类型提供完全不同的实现。
2. **偏特化**：为类型的一部分（例如某些类型组合）提供特定实现。

#### 完全特化

```cpp
template <>
class MyClass<bool> {
public:
    bool value;

    MyClass(bool val) : value(val) {}

    void print() {
        std::cout << "Boolean value: " << value << std::endl;
    }
};
```

这里我们为 `MyClass<bool>` 提供了一个特定实现。

#### 偏特化

偏特化允许为某些类型组合提供特定的实现，例如：

```cpp
template <typename T>
class MyClass<T*> {  // 为指针类型的 T 提供特化
public:
    T* ptr;

    MyClass(T* p) : ptr(p) {}

    void print() {
        std::cout << "Pointer value: " << *ptr << std::endl;
    }
};
```

这样，`MyClass<int*>` 和 `MyClass<double*>` 会使用特化的版本，而其他类型仍然使用通用版本。

### 模板的成员函数

类模板的成员函数可以像普通成员函数一样定义，可以在类内部定义，也可以在外部定义。成员函数的定义和普通类成员函数一样，但需要在函数定义时指定模板参数。

例如：

```cpp
template <typename T>
class MyClass {
public:
    T value;

    MyClass(T val) : value(val) {}

    void print() const {
        std::cout << "Value: " << value << std::endl;
    }

    T getValue() const {
        return value;
    }
};
```

如果成员函数定义在类外部，必须在定义时指定模板参数：

```cpp
template <typename T>
T MyClass<T>::getValue() const {
    return value;
}
```

### 模板的成员变量

类模板中的成员变量可以是类型化的，和成员函数一样，可以根据模板参数进行定义。

```cpp
template <typename T>
class MyClass {
public:
    T value;
    static int count;  // 静态成员变量

    MyClass(T val) : value(val) {
        ++count;
    }

    void print() const {
        std::cout << "Value: " << value << std::endl;
    }
};

template <typename T>
int MyClass<T>::count = 0;  // 静态成员变量的定义
```

### 总结

类模板是 C++ 中强大的特性之一，它允许我们编写能够处理多种类型的类，而不需要为每种类型编写重复的代码。类模板可以通过类型参数和非类型参数来实现通用类的定义，还支持特化机制来为特定类型提供不同的实现。类模板是泛型编程的核心部分，可以让代码更加灵活和高效。

## sp 单向广播
### 单向广播（Broadcasting）概述：

1. 单向广播是指在不同形状的数组之间进行运算时，较小的数组会自动扩展为较大数组的形状。
2. 扩展是根据广播规则自动完成的，不需要手动调整数组大小。
3. 规则：如果两个数组的维度数不同，较小维度的数组会在最左侧加上额外的维度。
4. 规则：如果两个数组在某一维度大小相同，或其中一个维度为1，则可以进行广播。
5. 规则：如果两者在某维度大小都不相同且不为1，无法进行广播。

### C++ 示例：向量与矩阵的广播加法

```cpp
#include <iostream>
#include <vector>

void broadcast_add(const std::vector<std::vector<int>>& A, const std::vector<int>& B, std::vector<std::vector<int>>& result) {
    for (size_t i = 0; i < A.size(); ++i)
        for (size_t j = 0; j < A[0].size(); ++j)
            result[i][j] = A[i][j] + B[j];
}

int main() {
    std::vector<std::vector<int>> A = {{1, 2, 3}, {4, 5, 6}};
    std::vector<int> B = {10, 20, 30};
    std::vector<std::vector<int>> result(2, std::vector<int>(3));

    broadcast_add(A, B, result);

    for (const auto& row : result)
        for (int val : row) std::cout << val << " ";
    std::cout << std::endl;
}
```

### Python 示例：向量与矩阵的广播加法

```python
import numpy as np

A = np.array([[1, 2, 3], [4, 5, 6]])
B = np.array([10, 20, 30])

result = A + B  # 自动广播 B 到每一列
print(result)
```

### 结果：
- **C++ 输出**：`[11, 22, 33] [14, 25, 36]`
- **Python 输出**：`[[11 22 33] [14 25 36]]`

广播让不同形状的数组可以进行数学运算，避免了手动调整形状的复杂性。


## L20 模板形参与实参

### 1. 模板形参（Template Parameters）

模板形参是模板定义中声明的参数，用于表示未知的类型或值。模板形参可以是：
- **类型参数**：表示某种类型（如 `typename T`）。
- **非类型参数**：表示常量值（如 `int N`）。

#### 示例：模板形参的定义
```cpp
template <typename T, int N> // T 是类型参数，N 是非类型参数
class Array {
private:
    T data[N]; // 使用模板形参定义数组
public:
    void set(int index, T value) {
        data[index] = value;
    }
    T get(int index) const {
        return data[index];
    }
};
```

在上面的代码中：
- `T` 是一个类型参数，表示数组元素的类型。
- `N` 是一个非类型参数，表示数组的大小。

---

### 2. 模板实参（Template Arguments）

模板实参是模板实例化时提供的具体值或类型，用于替换模板形参。模板实参可以是：
- **类型实参**：具体的类型（如 `int`、`double`）。
- **非类型实参**：具体的常量值（如 `10`、`100`）。

#### 示例：模板实参的使用
```cpp
int main() {
    Array<int, 5> intArray; // 实例化模板，T = int, N = 5
    intArray.set(0, 10);
    std::cout << intArray.get(0) << std::endl; // 输出: 10

    Array<double, 3> doubleArray; // 实例化模板，T = double, N = 3
    doubleArray.set(1, 3.14);
    std::cout << doubleArray.get(1) << std::endl; // 输出: 3.14

    return 0;
}
```

在上面的代码中：
- `Array<int, 5>` 中的 `int` 和 `5` 是模板实参，分别替换了模板形参 `T` 和 `N`。
- `Array<double, 3>` 中的 `double` 和 `3` 是模板实参，分别替换了模板形参 `T` 和 `N`。

---

### 3. 模板形参的默认值

C++ 允许为模板形参提供默认值。如果实例化时未提供实参，则使用默认值。

#### 示例：模板形参的默认值
```cpp
template <typename T = int, int N = 10> // 为 T 和 N 提供默认值
class Buffer {
private:
    T data[N];
public:
    void fill(T value) {
        for (int i = 0; i < N; ++i) {
            data[i] = value;
        }
    }
    void print() const {
        for (int i = 0; i < N; ++i) {
            std::cout << data[i] << " ";
        }
        std::cout << std::endl;
    }
};

int main() {
    Buffer<> defaultBuffer; // 使用默认模板实参，T = int, N = 10
    defaultBuffer.fill(42);
    defaultBuffer.print(); // 输出: 42 42 42 42 42 42 42 42 42 42

    Buffer<double, 5> customBuffer; // 使用自定义模板实参，T = double, N = 5
    customBuffer.fill(3.14);
    customBuffer.print(); // 输出: 3.14 3.14 3.14 3.14 3.14

    return 0;
}
```

在上面的代码中：
- `Buffer<>` 使用了模板形参的默认值 `T = int` 和 `N = 10`。
- `Buffer<double, 5>` 提供了自定义的模板实参。

---

### 4. 模板形参的类型推导

在 C++17 中，模板实参可以通过函数参数的类型自动推导，无需显式指定。

#### 示例：模板形参的类型推导
```cpp
template <typename T>
void print(T value) {
    std::cout << value << std::endl;
}

int main() {
    print(42);       // 推导 T = int
    print(3.14);     // 推导 T = double
    print("Hello");  // 推导 T = const char*

    return 0;
}
```

在上面的代码中：
- 编译器根据函数参数的类型自动推导出模板实参。

---

### 5. 模板形参的约束（C++20）

C++20 引入了 **概念（Concepts）**，用于对模板形参进行约束，确保模板实参满足特定条件。

#### 示例：模板形参的约束
```cpp
#include <concepts>

template <typename T>
requires std::integral<T> // 约束 T 必须是整数类型
T add(T a, T b) {
    return a + b;
}

int main() {
    std::cout << add(10, 20) << std::endl; // 合法，T = int
    // std::cout << add(3.14, 2.71) << std::endl; // 错误，T = double 不满足约束

    return 0;
}
```

在上面的代码中：
- `std::integral<T>` 约束了模板形参 `T` 必须是整数类型。

## L21: `std::array`, `std::vector`, 和 `std::string`

在 C++ 标准库中，`std::array`、`std::vector` 和 `std::string` 是三个非常常用的容器类。它们分别用于管理固定大小的数组、动态数组和字符串。以下是它们的详细介绍和对比。

### 1. `std::array`

#### 概述
- `std::array` 是一个封装了固定大小数组的容器。
- 它是 C++11 引入的，位于 `<array>` 头文件中。
- 大小在编译时确定，不可动态调整。

#### 特点
- **固定大小**：大小在编译时确定，无法在运行时改变。
- **性能高效**：与原生数组性能相当，没有额外的动态内存分配开销。
- **支持迭代器**：可以使用标准库的算法和范围 for 循环。
- **安全性**：提供 `at()` 方法进行边界检查。

#### 示例代码
```cpp
#include <iostream>
#include <array>

int main() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5}; // 定义一个大小为 5 的数组

    // 访问元素
    std::cout << "Element at index 2: " << arr[2] << std::endl; // 输出: 3
    std::cout << "Element at index 2 (using at): " << arr.at(2) << std::endl; // 输出: 3

    // 遍历数组
    for (int i : arr) {
        std::cout << i << " "; // 输出: 1 2 3 4 5
    }
    std::cout << std::endl;

    // 获取数组大小
    std::cout << "Size of array: " << arr.size() << std::endl; // 输出: 5

    return 0;
}
```

###  `std::array` 常用方法

`std::array` 提供了许多常用的方法来操作和访问数组中的元素。以下是一些常用的方法及其说明：

#### 1.1 `at(size_type pos)`
- **功能**：访问指定位置的元素，并进行边界检查。
- **参数**：`pos` 表示要访问的元素的位置。
- **返回值**：返回指定位置的元素的引用。
- **异常**：如果 `pos` 超出数组范围，抛出 `std::out_of_range` 异常。

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
int element = arr.at(2); // 访问索引为 2 的元素，值为 3
```

#### 1.2 `operator[]`
- **功能**：访问指定位置的元素，不进行边界检查。
- **参数**：`pos` 表示要访问的元素的位置。
- **返回值**：返回指定位置的元素的引用。
- **注意**：与 `at()` 不同，`operator[]` 不会进行边界检查，访问越界会导致未定义行为。

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
int element = arr[2]; // 访问索引为 2 的元素，值为 3
```

#### 1.3 `front()`
- **功能**：访问数组的第一个元素。
- **返回值**：返回第一个元素的引用。

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
int first_element = arr.front(); // 访问第一个元素，值为 1
```

#### 1.4 `back()`
- **功能**：访问数组的最后一个元素。
- **返回值**：返回最后一个元素的引用。

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
int last_element = arr.back(); // 访问最后一个元素，值为 5
```

#### 1.5 `data()`
- **功能**：返回指向数组第一个元素的指针。
- **返回值**：返回指向数组首元素的指针。

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
int* ptr = arr.data(); // 获取指向数组首元素的指针
```

#### 1.6 `size()`
- **功能**：返回数组的大小（元素个数）。
- **返回值**：返回数组的大小。

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
std::size_t size = arr.size(); // 获取数组大小，值为 5
```

#### 1.7 `empty()`
- **功能**：检查数组是否为空。
- **返回值**：如果数组大小为 0，返回 `true`，否则返回 `false`。

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
bool is_empty = arr.empty(); // 检查数组是否为空，值为 false
```

#### 1.8 `fill(const T& value)`
- **功能**：将数组中的所有元素设置为指定的值。
- **参数**：`value` 表示要设置的值。

```cpp
std::array<int, 5> arr;
arr.fill(10); // 将数组中的所有元素设置为 10
// arr 现在为 {10, 10, 10, 10, 10}
```

#### 1.9 `begin()` 和 `end()`
- **功能**：返回指向数组第一个元素和最后一个元素之后位置的迭代器。
- **返回值**：返回指向数组首元素和尾后元素的迭代器。

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
for (auto it = arr.begin(); it != arr.end(); ++it) {
    std::cout << *it << " "; // 输出: 1 2 3 4 5
}
```

#### 1.10 `cbegin()` 和 `cend()`
- **功能**：返回指向数组第一个元素和最后一个元素之后位置的常量迭代器。
- **返回值**：返回指向数组首元素和尾后元素的常量迭代器。

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
for (auto it = arr.cbegin(); it != arr.cend(); ++it) {
    std::cout << *it << " "; // 输出: 1 2 3 4 5
}
```

#### 1.11 `rbegin()` 和 `rend()`
- **功能**：返回指向数组最后一个元素和第一个元素之前位置的逆向迭代器。
- **返回值**：返回指向数组尾元素和首前元素的逆向迭代器。

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
for (auto it = arr.rbegin(); it != arr.rend(); ++it) {
    std::cout << *it << " "; // 输出: 5 4 3 2 1
}
```

#### 1.12 `crbegin()` 和 `crend()`
- **功能**：返回指向数组最后一个元素和第一个元素之前位置的常量逆向迭代器。
- **返回值**：返回指向数组尾元素和首前元素的常量逆向迭代器。

```cpp
std::array<int, 5> arr = {1, 2, 3, 4, 5};
for (auto it = arr.crbegin(); it != arr.crend(); ++it) {
    std::cout << *it << " "; // 输出: 5 4 3 2 1
}
```

#### 1.13 `swap(std::array& other)`
- **功能**：交换两个数组的内容。
- **参数**：`other` 表示要交换的另一个数组。

```cpp
std::array<int, 5> arr1 = {1, 2, 3, 4, 5};
std::array<int, 5> arr2 = {6, 7, 8, 9, 10};
arr1.swap(arr2); // 交换 arr1 和 arr2 的内容
```


### 2. `std::vector`

#### 概述
- `std::vector` 是一个动态数组容器。
- 位于 `<vector>` 头文件中。
- 大小可以动态调整，支持在尾部高效地添加和删除元素。

#### 特点
- **动态大小**：可以在运行时动态调整大小。
- **连续存储**：元素在内存中连续存储，支持随机访问。
- **自动内存管理**：自动处理内存分配和释放。
- **支持迭代器**：可以使用标准库的算法和范围 for 循环。

#### 示例代码
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3}; // 初始化一个 vector

    // 添加元素
    vec.push_back(4); // vec: {1, 2, 3, 4}
    vec.push_back(5); // vec: {1, 2, 3, 4, 5}

    // 访问元素
    std::cout << "Element at index 2: " << vec[2] << std::endl; // 输出: 3
    std::cout << "Element at index 2 (using at): " << vec.at(2) << std::endl; // 输出: 3

    // 遍历 vector
    for (int i : vec) {
        std::cout << i << " "; // 输出: 1 2 3 4 5
    }
    std::cout << std::endl;

    // 获取 vector 大小
    std::cout << "Size of vector: " << vec.size() << std::endl; // 输出: 5

    // 删除最后一个元素
    vec.pop_back(); // vec: {1, 2, 3, 4}
    std::cout << "Size after pop_back: " << vec.size() << std::endl; // 输出: 4

    return 0;
}
```

---

### `std::vector` 常用方法

#### 1. **元素访问**
| 方法                          | 功能描述                                                                 |
|-------------------------------|--------------------------------------------------------------------------|
| `operator[]`                  | 访问指定位置的元素，**不进行边界检查**。                                   |
| `at(size_t pos)`              | 访问指定位置的元素，**进行边界检查**，如果越界则抛出 `std::out_of_range`。 |
| `front()`                     | 返回第一个元素的引用。                                                   |
| `back()`                      | 返回最后一个元素的引用。                                                 |
| `data()`                      | 返回指向底层数组的指针。                                                 |

**示例：**
```cpp
std::vector<int> vec = {1, 2, 3};
std::cout << vec[0] << std::endl;    // 输出: 1
std::cout << vec.at(1) << std::endl; // 输出: 2
std::cout << vec.front() << std::endl; // 输出: 1
std::cout << vec.back() << std::endl;  // 输出: 3
int* ptr = vec.data();               // 获取底层数组指针
```

---

#### 2. **容量相关**
| 方法                          | 功能描述                                                                 |
|-------------------------------|--------------------------------------------------------------------------|
| `size()`                      | 返回 `vector` 中元素的数量。                                             |
| `capacity()`                  | 返回 `vector` 当前分配的内存容量（可容纳的元素数量）。                   |
| `empty()`                     | 检查 `vector` 是否为空。                                                 |
| `reserve(size_t n)`           | 预分配至少能容纳 `n` 个元素的内存空间。                                  |
| `shrink_to_fit()`             | 释放未使用的内存，使 `capacity()` 等于 `size()`。                        |

**示例：**
```cpp
std::vector<int> vec = {1, 2, 3};
std::cout << vec.size() << std::endl;     // 输出: 3
std::cout << vec.capacity() << std::endl; // 输出: 3
vec.reserve(10);
std::cout << vec.capacity() << std::endl; // 输出: 10
vec.shrink_to_fit();
std::cout << vec.capacity() << std::endl; // 输出: 3
```

---

#### 3. **修改容器**
| 方法                          | 功能描述                                                                 |
|-------------------------------|--------------------------------------------------------------------------|
| `push_back(const T& value)`   | 在尾部添加一个元素。                                                     |
| `pop_back()`                  | 删除尾部的一个元素。                                                     |
| `insert(iterator pos, const T& value)` | 在指定位置插入一个元素。                                     |
| `erase(iterator pos)`         | 删除指定位置的元素。                                                     |
| `clear()`                     | 清空所有元素。                                                           |
| `resize(size_t n)`            | 调整 `vector` 的大小为 `n`，多出的元素用默认值填充。                     |
| `swap(std::vector& other)`    | 交换两个 `vector` 的内容。                                               |

**示例：**
```cpp
std::vector<int> vec = {1, 2, 3};
vec.push_back(4); // vec: {1, 2, 3, 4}
vec.pop_back();   // vec: {1, 2, 3}
vec.insert(vec.begin() + 1, 5); // vec: {1, 5, 2, 3}
vec.erase(vec.begin() + 2); // vec: {1, 5, 3}
vec.clear(); // vec: {}
```

---

#### 4. **迭代器**
| 方法                          | 功能描述                                                                 |
|-------------------------------|--------------------------------------------------------------------------|
| `begin()`                     | 返回指向第一个元素的迭代器。                                             |
| `end()`                       | 返回指向末尾（最后一个元素之后）的迭代器。                               |
| `cbegin()`                    | 返回指向第一个元素的常量迭代器。                                         |
| `cend()`                      | 返回指向末尾的常量迭代器。                                               |
| `rbegin()`                    | 返回指向最后一个元素的反向迭代器。                                       |
| `rend()`                      | 返回指向开头之前的反向迭代器。                                           |
| `crbegin()`                   | 返回指向最后一个元素的常量反向迭代器。                                   |
| `crend()`                     | 返回指向开头之前的常量反向迭代器。                                       |

**示例：**
```cpp
std::vector<int> vec = {1, 2, 3};

// 使用迭代器遍历
for (auto it = vec.begin(); it != vec.end(); ++it) {
    std::cout << *it << " "; // 输出: 1 2 3
}

// 使用反向迭代器遍历
for (auto it = vec.rbegin(); it != vec.rend(); ++it) {
    std::cout << *it << " "; // 输出: 3 2 1
}
```

---

#### 5. **比较操作**
| 方法                          | 功能描述                                                                 |
|-------------------------------|--------------------------------------------------------------------------|
| `operator==`                  | 比较两个 `vector` 是否相等。                                             |
| `operator!=`                  | 比较两个 `vector` 是否不相等。                                           |
| `operator<`                   | 比较两个 `vector` 的字典序。                                             |
| `operator>`                   | 比较两个 `vector` 的字典序。                                             |
| `operator<=`                  | 比较两个 `vector` 的字典序。                                             |
| `operator>=`                  | 比较两个 `vector` 的字典序。                                             |

**示例：**
```cpp
std::vector<int> vec1 = {1, 2, 3};
std::vector<int> vec2 = {1, 2, 3};

if (vec1 == vec2) {
    std::cout << "Vectors are equal." << std::endl; // 输出: Vectors are equal.
}
```

---


### 3. `std::string`

#### 概述
- `std::string` 是一个用于表示和操作字符串的类。
- 位于 `<string>` 头文件中。
- 支持动态调整大小，提供了丰富的字符串操作功能。

#### 特点
- **动态大小**：字符串长度可以动态调整。
- **丰富的操作**：支持拼接、查找、替换、子串等操作。
- **支持迭代器**：可以使用标准库的算法和范围 for 循环。
- **自动内存管理**：自动处理内存分配和释放。

#### 示例代码
```cpp
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello"; // 初始化一个字符串

    // 拼接字符串
    str += ", World!"; // str: "Hello, World!"

    // 访问字符
    std::cout << "First character: " << str[0] << std::endl; // 输出: H
    std::cout << "First character (using at): " << str.at(0) << std::endl; // 输出: H

    // 获取字符串长度
    std::cout << "Length of string: " << str.length() << std::endl; // 输出: 13

    // 查找子串
    size_t pos = str.find("World");
    if (pos != std::string::npos) {
        std::cout << "'World' found at position: " << pos << std::endl; // 输出: 7
    }

    // 获取子串
    std::string sub = str.substr(7, 5); // 从位置 7 开始，长度为 5
    std::cout << "Substring: " << sub << std::endl; // 输出: World

    // 遍历字符串
    for (char c : str) {
        std::cout << c; // 输出: Hello, World!
    }
    std::cout << std::endl;

    return 0;
}
```

User: ### 3. `std::string`

#### 概述
- `std::string` 是一个用于表示和操作字符串的类。
- 位于 `<string>` 头文件中。
- 支持动态调整大小，提供了丰富的字符串操作功能。

#### 特点
- **动态大小**：字符串长度可以动态调整。
- **丰富的操作**：支持拼接、查找、替换、子串等操作。
- **支持迭代器**：可以使用标准库的算法和范围 for 循环。
- **自动内存管理**：自动处理内存分配和释放。

#### 示例代码
```cpp
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello"; // 初始化一个字符串

    // 拼接字符串
    str += ", World!"; // str: "Hello, World!"

    // 访问字符
    std::cout << "First character: " << str[0] << std::endl; // 输出: H
    std::cout << "First character (using at): " << str.at(0) << std::endl; // 输出: H

    // 获取字符串长度
    std::cout << "Length of string: " << str.length() << std::endl; // 输出: 13

    // 查找子串
    size_t pos = str.find("World");
    if (pos != std::string::npos) {
        std::cout << "'World' found at position: " << pos << std::endl; // 输出: 7
    }

    // 获取子串
    std::string sub = str.substr(7, 5); // 从位置 7 开始，长度为 5
    std::cout << "Substring: " << sub << std::endl; // 输出: World

    // 遍历字符串
    for (char c : str) {
        std::cout << c; // 输出: Hello, World!
    }
    std::cout << std::endl;

    return 0;
}
```

在这个后面补充string的常用方法
Kimi: ### `std::string` 常用方法

### `std::string`常用方法

#### 1. 构造函数
- **默认构造**：创建一个空字符串。
  ```cpp
  std::string s;
  ```
- **从 C 风格字符串构造**：从 C 风格字符串构造 `std::string`。
  ```cpp
  std::string s = "Hello";
  ```
- **从另一个字符串构造**：从另一个字符串的子串构造。
  ```cpp
  std::string s1 = "Hello, World!";
  std::string s2 = s1.substr(7, 5); // s2: "World"
  ```
- **重复字符构造**：创建一个包含重复字符的字符串。
  ```cpp
  std::string s(10, 'a'); // s: "aaaaaaaaaa"
  ```

#### 2. 字符串操作
- **拼接**：使用 `+` 或 `+=` 操作符拼接字符串。
  ```cpp
  std::string s1 = "Hello";
  std::string s2 = "World";
  std::string s3 = s1 + ", " + s2 + "!"; // s3: "Hello, World!"
  ```
- **赋值**：使用 `=` 操作符赋值。
  ```cpp
  std::string s1 = "Hello";
  std::string s2;
  s2 = s1; // s2: "Hello"
  ```
- **清空**：使用 `clear()` 方法清空字符串。
  ```cpp
  std::string s = "Hello";
  s.clear(); // s: ""
  ```

#### 3. 字符访问
- **通过下标访问**：使用 `[]` 操作符访问字符。
  ```cpp
  std::string s = "Hello";
  char c = s[0]; // c: 'H'
  s[0] = 'h'; // s: "hello"
  ```
- **通过 `at` 方法访问**：使用 `at` 方法访问字符，会进行边界检查。
  ```cpp
  std::string s = "Hello";
  char c = s.at(0); // c: 'H'
  s.at(0) = 'h'; // s: "hello"
  ```

#### 4. 字符串属性
- **长度**：使用 `length()` 或 `size()` 方法获取字符串长度。
  ```cpp
  std::string s = "Hello";
  size_t len = s.length(); // len: 5
  size_t size = s.size(); // size: 5
  ```
- **是否为空**：使用 `empty()` 方法检查字符串是否为空。
  ```cpp
  std::string s = "";
  bool isEmpty = s.empty(); // isEmpty: true
  ```

#### 5. 查找和替换
- **查找**：使用 `find()` 方法查找子串。
  ```cpp
  std::string s = "Hello, World!";
  size_t pos = s.find("World"); // pos: 7
  if (pos != std::string::npos) {
      std::cout << "'World' found at position: " << pos << std::endl;
  }
  ```
- **替换**：使用 `replace()` 方法替换子串。
  ```cpp
  std::string s = "Hello, World!";
  s.replace(7, 5, "C++"); // s: "Hello, C++!"
  ```

#### 6. 子串
- **获取子串**：使用 `substr()` 方法获取子串。
  ```cpp
  std::string s = "Hello, World!";
  std::string sub = s.substr(7, 5); // sub: "World"
  ```

#### 7. 比较
- **比较字符串**：使用 `==`、`!=`、`<`、`>`、`<=`、`>=` 操作符比较字符串。
  ```cpp
  std::string s1 = "Hello";
  std::string s2 = "World";
  bool isEqual = (s1 == s2); // isEqual: false
  bool isLess = (s1 < s2); // isLess: true
  ```

#### 8. 迭代器
- **使用迭代器**：使用标准库的算法和范围 for 循环。
  ```cpp
  std::string s = "Hello, World!";
  for (char c : s) {
      std::cout << c; // 输出: Hello, World!
  }
  ```

#### 9. 输入输出
- **输入**：使用 `std::cin` 读取字符串。
  ```cpp
  std::string s;
  std::cin >> s; // 读取一个单词
  std::getline(std::cin, s); // 读取一行
  ```
- **输出**：使用 `std::cout` 输出字符串。
  ```cpp
  std::string s = "Hello, World!";
  std::cout << s << std::endl; // 输出: Hello, World!
  ```

#### 10. 其他方法
- **转换为 C 风格字符串**：使用 `c_str()` 方法获取 C 风格字符串。
  ```cpp
  std::string s = "Hello";
  const char* cstr = s.c_str(); // cstr: "Hello"
  ```
- **转换为大写或小写**：使用 `std::transform` 和 `std::toupper` 或 `std::tolower`。
  ```cpp
  std::string s = "Hello";
  std::transform(s.begin(), s.end(), s.begin(), ::toupper); // s: "HELLO"
  std::transform(s.begin(), s.end(), s.begin(), ::tolower); // s: "hello"
  ```

这些方法涵盖了 `std::string` 的大部分常用功能，希望对您有帮助！如果有其他问题或需要进一步的解释，请随时告诉我。

## L22 模板特化
### 1. **模板全特化（Full Specialization）**
全特化是指为模板的所有参数提供具体的类型或值，从而为特定类型提供完全不同的实现。

#### 语法：
```cpp
template <>
class/struct 模板名<具体类型> {
    // 特化实现
};
```

#### 示例：
```cpp
#include <iostream>

// 通用模板
template <typename T>
class MyClass {
public:
    void print() {
        std::cout << "Generic template" << std::endl;
    }
};

// 全特化：针对 int 类型
template <>
class MyClass<int> {
public:
    void print() {
        std::cout << "Specialized template for int" << std::endl;
    }
};

int main() {
    MyClass<double> obj1;
    obj1.print(); // 输出: Generic template

    MyClass<int> obj2;
    obj2.print(); // 输出: Specialized template for int

    return 0;
}
```

#### 特点：
- 全特化是针对所有模板参数的完全定制。
- 全特化的实现可以与通用模板完全不同。

---

### 2. **模板偏特化（Partial Specialization）**
偏特化是指为模板的部分参数提供具体类型或值，从而为某些特定类型组合提供定制化的实现。

#### 语法：
```cpp
template <typename T1, typename T2>
class 模板名<T1, T2*> { // 偏特化：T2 是指针类型
    // 特化实现
};
```

#### 示例：
```cpp
#include <iostream>

// 通用模板
template <typename T1, typename T2>
class MyClass {
public:
    void print() {
        std::cout << "Generic template" << std::endl;
    }
};

// 偏特化：T2 是指针类型
template <typename T1, typename T2>
class MyClass<T1, T2*> {
public:
    void print() {
        std::cout << "Partial specialization for T2*" << std::endl;
    }
};

int main() {
    MyClass<int, double> obj1;
    obj1.print(); // 输出: Generic template

    MyClass<int, double*> obj2;
    obj2.print(); // 输出: Partial specialization for T2*

    return 0;
}
```

#### 特点：
- 偏特化是针对部分模板参数的定制。
- 偏特化可以用于更复杂的类型组合（如指针、引用、特定类型等）。

---

### 3. **函数模板特化**
函数模板也可以特化，但通常更推荐使用函数重载来实现类似的功能。

#### 示例：
```cpp
#include <iostream>

// 通用函数模板
template <typename T>
void print(T value) {
    std::cout << "Generic template: " << value << std::endl;
}

// 特化：针对 int 类型
template <>
void print<int>(int value) {
    std::cout << "Specialized template for int: " << value << std::endl;
}

int main() {
    print(3.14); // 输出: Generic template: 3.14
    print(42);   // 输出: Specialized template for int: 42

    return 0;
}
```

---

### 4. **模板特化的应用场景**
- **性能优化**：为特定类型提供更高效的实现。
- **类型约束**：为特定类型提供特定的行为。
- **兼容性**：为某些特殊类型（如指针、引用）提供定制化处理。

---

### 5. **模板特化的注意事项**
- **全特化必须定义在通用模板之后**：编译器需要先看到通用模板，才能理解特化的意义。
- **偏特化仅适用于类模板**：函数模板不支持偏特化，但可以通过重载实现类似功能。
- **避免过度特化**：过多的特化会导致代码复杂度增加，维护困难。

---

### 6. **模板特化 vs 函数重载**

- **模板特化**：适用于类模板和函数模板，提供针对特定类型的定制实现。
- **函数重载**：适用于函数，通过参数类型或数量的不同提供多种实现。

#### 示例对比：
```cpp
// 函数模板特化
template <typename T>
void foo(T value) { /* 通用实现 */ }

template <>
void foo<int>(int value) { /* 针对 int 的特化实现 */ }

// 函数重载
void foo(int value) { /* 针对 int 的重载实现 */ }
```

## sp iso对模板特化的例子`std::vector<bool>` 的特化及其问题

### 1. **`std::vector<bool>` 的特化**
`std::vector<bool>` 是 `std::vector` 的一个显式特化版本，专门用于存储布尔值。它的内部实现使用了一种称为“位压缩”的技术，将多个布尔值存储在一个字节中，从而节省内存。

#### 示例：
```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<bool> vec = {true, false, true, false};
    for (bool b : vec) {
        std::cout << b << " ";
    }
    return 0;
}
```

#### 输出：
```
1 0 1 0
```

---

### 2. **`std::vector<bool>` 的优点**
- **节省内存**：每个布尔值只占用一个比特，而不是一个字节。
- **空间效率高**：对于存储大量布尔值的场景，可以显著减少内存占用。

---

### 3. **`std::vector<bool>` 的问题与弊端**
尽管 `std::vector<bool>` 在内存使用上有优势，但它也带来了一些严重的问题，导致它在实际使用中备受争议。

#### 3.1 **行为不符合标准容器**
`std::vector<bool>` 的行为与其他 `std::vector` 类型不一致。具体表现为：
- **元素类型不是 `bool`**：`std::vector<bool>` 的元素类型是一个代理类（proxy class），而不是直接的 `bool` 类型。
- **不能获取元素的地址**：由于元素是位压缩存储的，无法直接获取某个布尔值的地址。

#### 示例：
```cpp
#include <vector>

int main() {
    std::vector<bool> vec = {true, false, true};
    bool* ptr = &vec[0]; // 错误：无法获取 std::vector<bool> 元素的地址
    return 0;
}
```

#### 3.2 **与标准算法的兼容性问题**
由于 `std::vector<bool>` 的元素类型是代理类，许多标准算法无法直接使用。例如：
- **迭代器问题**：`std::vector<bool>::iterator` 的行为与其他容器的迭代器不同。
- **无法直接使用 `auto`**：`auto` 推导出的类型可能是代理类，而不是 `bool`。

#### 示例：
```cpp
#include <vector>
#include <algorithm>

int main() {
    std::vector<bool> vec = {true, false, true};
    auto it = vec.begin(); // it 的类型是 std::vector<bool>::iterator，不是 bool*
    *it = false;           // 可以修改值，但行为与其他容器不同
    return 0;
}
```

#### 3.3 **性能问题**
- **访问速度慢**：由于需要解压缩位，访问 `std::vector<bool>` 的元素比访问普通 `std::vector` 的元素更慢。
- **修改效率低**：修改某个布尔值可能需要读取和写入整个字节，而不是直接修改一个比特。

#### 3.4 **与其他容器的接口不一致**
`std::vector<bool>` 的接口与其他容器不一致，导致代码的可移植性和可维护性降低。例如：
- **无法直接与其他容器交互**：例如，无法直接将 `std::vector<bool>` 转换为 `std::vector<int>`。


## L23 类型别名

### 概念
类型别名是为已有的类型赋予一个新的名称，它在编程中起到简化代码、提高可读性和可维护性的作用。

### 定义方式
在 C++ 中，可以使用 `using` 关键字来定义类型别名。例如：
```cpp
using IntVector = std::vector<int>;
```
这样，`IntVector` 就成为了 `std::vector<int>` 的别名，后续在代码中可以直接使用 `IntVector` 来代替 `std::vector<int>`，使代码更加简洁。

### 应用场景
1. **简化复杂类型**：当需要频繁使用复杂的类型，如嵌套的模板类型时，类型别名可以大大简化代码。例如：
    ```cpp
    using MapOfVectors = std::map<std::string, std::vector<int>>;
    ```
    使用 `MapOfVectors` 比直接使用 `std::map<std::string, std::vector<int>>` 要清晰得多。
2. **提高代码可读性**：为具有特定含义的类型定义别名，可以让代码的意图更加明确。例如，在处理图形相关的数据时：
    ```cpp
    using Point = std::pair<int, int>;
    ```
    使用 `Point` 来表示二维坐标点，比直接使用 `std::pair<int, int>` 更能表达出数据的含义。

## L24 逆向迭代器

### 概念
逆向迭代器是一种特殊的迭代器，它允许我们从容器的末尾开始向前遍历容器中的元素，与正向迭代器的遍历方向相反。

### 获取逆向迭代器
在 C++ 标准库的容器中，可以通过 `rbegin()` 和 `rend()` 成员函数来获取逆向迭代器。其中，`rbegin()` 返回指向容器最后一个元素的逆向迭代器，`rend()` 返回一个指向“过去末尾”（past-the-end）的逆向迭代器，用于表示逆向迭代的结束位置。

### 使用示例
```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};
for (auto it = vec.rbegin(); it != vec.rend(); ++it) {
    std::cout << *it << " ";
}
// 输出：5 4 3 2 1
```
在这个例子中，通过逆向迭代器从 `vec` 的最后一个元素开始向前遍历，依次输出每个元素的值。

### 与正向迭代器的关系
逆向迭代器和正向迭代器之间存在一定的转换关系。例如，可以通过 `base()` 成员函数将逆向迭代器转换为对应的正向迭代器。需要注意的是，转换后的正向迭代器会指向逆向迭代器所指元素的下一个位置。例如：
```cpp
auto it = vec.rbegin();
auto normalIt = it.base();
// 此时 normalIt 指向 vec 的倒数第二个元素
```



## L25 `std::map`

#### 概述
- `std::map` 是一个基于红黑树实现的关联容器，用于存储键值对。
- 键值对中的键是唯一的，且按键的顺序自动排序。
- 位于 `<map>` 头文件中。

#### 特点
- **键值对存储**：存储键值对，键是唯一的。
- **自动排序**：按键的顺序自动排序，可以自定义排序规则。
- **高效查找**：查找、插入和删除操作的时间复杂度为 O(log n)。
- **支持迭代器**：可以使用标准库的算法和范围 for 循环。

#### 示例代码
```cpp
#include <iostream>
#include <map>

int main() {
    // 创建一个 map，键为 int，值为 string
    std::map<int, std::string> m;

    // 插入键值对
    m[1] = "one";
    m[2] = "two";
    m[3] = "three";

    // 查找键值对
    auto it = m.find(2);
    if (it != m.end()) {
        std::cout << "Found: " << it->first << " -> " << it->second << std::endl; // 输出: Found: 2 -> two
    }

    // 遍历 map
    for (const auto& pair : m) {
        std::cout << pair.first << " -> " << pair.second << std::endl;
    }

    // 删除键值对
    m.erase(2);

    // 检查键是否存在
    if (m.count(2) == 0) {
        std::cout << "Key 2 is not found." << std::endl; // 输出: Key 2 is not found.
    }

    return 0;
}
```

### `std::map` 常用方法

#### 1. 构造函数
- **默认构造**：创建一个空的 `std::map`。
  ```cpp
  std::map<int, std::string> m;
  ```
- **自定义排序**：使用自定义的比较函数。
  ```cpp
  struct Compare {
      bool operator()(int a, int b) const {
          return a > b; // 降序排序
      }
  };
  std::map<int, std::string, Compare> m;
  ```

#### 2. 插入
- **使用 `[]` 操作符**：插入键值对，如果键已存在，则更新值。
  ```cpp
  m[1] = "one";
  ```
- **使用 `insert` 方法**：插入键值对，如果键已存在，则不插入。
  ```cpp
  m.insert({2, "two"});
  m.insert(std::make_pair(3, "three"));
  ```

#### 3. 查找
- **使用 `find` 方法**：查找键值对，返回迭代器。
  ```cpp
  auto it = m.find(2);
  if (it != m.end()) {
      std::cout << "Found: " << it->first << " -> " << it->second << std::endl;
  }
  ```
- **使用 `count` 方法**：检查键是否存在，返回 0 或 1。
  ```cpp
  if (m.count(2) == 1) {
      std::cout << "Key 2 is found." << std::endl;
  }
  ```

#### 4. 删除
- **使用 `erase` 方法**：删除键值对。
  ```cpp
  m.erase(2); // 删除键为 2 的键值对
  ```

#### 5. 遍历
- **使用范围 for 循环**：遍历 `std::map`。
  ```cpp
  for (const auto& pair : m) {
      std::cout << pair.first << " -> " << pair.second << std::endl;
  }
  ```
- **使用迭代器**：遍历 `std::map`。
  ```cpp
  for (auto it = m.begin(); it != m.end(); ++it) {
      std::cout << it->first << " -> " << it->second << std::endl;
  }
  ```

#### 6. 其他方法
- **获取大小**：使用 `size()` 方法获取 `std::map` 的大小。
  ```cpp
  size_t size = m.size(); // 获取 map 的大小
  ```
- **清空**：使用 `clear()` 方法清空 `std::map`。
  ```cpp
  m.clear(); // 清空 map
  ```
- **检查是否为空**：使用 `empty()` 方法检查 `std::map` 是否为空。
  ```cpp
  bool isEmpty = m.empty(); // 检查 map 是否为空
  ```

### 示例代码
```cpp
#include <iostream>
#include <map>

int main() {
    // 创建一个 map，键为 int，值为 string
    std::map<int, std::string> m;

    // 插入键值对
    m[1] = "one";
    m[2] = "two";
    m[3] = "three";

    // 查找键值对
    auto it = m.find(2);
    if (it != m.end()) {
        std::cout << "Found: " << it->first << " -> " << it->second << std::endl; // 输出: Found: 2 -> two
    }

    // 遍历 map
    for (const auto& pair : m) {
        std::cout << pair.first << " -> " << pair.second << std::endl;
    }

    // 删除键值对
    m.erase(2);

    // 检查键是否存在
    if (m.count(2) == 0) {
        std::cout << "Key 2 is not found." << std::endl; // 输出: Key 2 is not found.
    }

    return 0;
}
```

### 输出
```
Found: 2 -> two
1 -> one
2 -> two
3 -> three
Key 2 is not found.
```

## L26 智能指针
智能指针是一种包装了原始指针的类，能够自动管理内存的生命周期。它们可以帮助我们避免内存泄漏和悬空指针等常见的内存管理问题。C++ 标准库提供了几种智能指针类型，最常用的有：

- `std::unique_ptr`
- `std::shared_ptr`
- `std::weak_ptr`

### 1. `std::unique_ptr`

`std::unique_ptr` 是最基础的智能指针，它保证指向的对象唯一且不可共享。当 `unique_ptr` 被销毁时，所指向的对象也会自动销毁。

#### 特点：
- **独占所有权**：每个 `unique_ptr` 只能有一个所有者，不能共享。
- **移动语义**：可以通过 `std::move()` 转移所有权，但不能复制 `unique_ptr`。
- **自动释放资源**：当 `unique_ptr` 超出作用域时，它所管理的资源会自动释放。

#### 使用场景：
- 当你希望拥有对象的独占所有权，并且不需要共享资源时，使用 `std::unique_ptr`。

#### 示例代码：
```cpp
#include <memory>
#include <iostream>

class Resource {
public:
    Resource() { std::cout << "Resource acquired!" << std::endl; }
    ~Resource() { std::cout << "Resource destroyed!" << std::endl; }
};

int main() {
    // 创建 unique_ptr，自动管理资源
    std::unique_ptr<Resource> ptr1 = std::make_unique<Resource>();

    // 资源自动释放，无需手动 delete
    return 0;
}
```

### 2. `std::shared_ptr`

`std::shared_ptr` 允许多个指针共享同一块资源。当最后一个指向该资源的 `shared_ptr` 被销毁时，资源才会被释放。

#### 特点：
- **共享所有权**：多个 `shared_ptr` 可以共享同一个资源。
- **引用计数**：`shared_ptr` 内部使用引用计数来跟踪有多少个指针共享资源。当引用计数归零时，资源被销毁。
- **线程安全**：引用计数的增加和减少是线程安全的，但对资源本身的访问不保证线程安全。

#### 使用场景：
- 当你希望多个对象共享同一资源，并且你不确定谁会在最后释放资源时，使用 `std::shared_ptr`。

#### 示例代码：
```cpp
#include <memory>
#include <iostream>

class Resource {
public:
    Resource() { std::cout << "Resource acquired!" << std::endl; }
    ~Resource() { std::cout << "Resource destroyed!" << std::endl; }
};

int main() {
    // 创建 shared_ptr，多个指针可以共享资源
    std::shared_ptr<Resource> ptr1 = std::make_shared<Resource>();
    std::shared_ptr<Resource> ptr2 = ptr1;  // 引用计数增加

    std::cout << "ptr1 use_count: " << ptr1.use_count() << std::endl;
    std::cout << "ptr2 use_count: " << ptr2.use_count() << std::endl;

    return 0;
}
```

### 3. `std::weak_ptr`

`std::weak_ptr` 是一种不控制资源生命周期的智能指针。它用来解决 `shared_ptr` 引用循环的问题。`weak_ptr` 不增加引用计数，因此不会影响资源的释放。

#### 特点：
- **弱引用**：`weak_ptr` 不能直接访问所指向的对象。它需要通过 `shared_ptr` 转换来访问。
- **解决循环引用问题**：`weak_ptr` 用来打破 `shared_ptr` 之间的循环引用。
- **不管理资源**：`weak_ptr` 不拥有资源，它只是观察资源。

#### 使用场景：
- 用于观察对象而不干预其生命周期，特别是在避免循环引用的场景中使用。

#### 示例代码：
```cpp
#include <memory>
#include <iostream>

class Resource {
public:
    Resource() { std::cout << "Resource acquired!" << std::endl; }
    ~Resource() { std::cout << "Resource destroyed!" << std::endl; }
};

int main() {
    std::shared_ptr<Resource> ptr1 = std::make_shared<Resource>();
    std::weak_ptr<Resource> weakPtr = ptr1;  // 创建 weak_ptr

    std::cout << "ptr1 use_count: " << ptr1.use_count() << std::endl;

    // weak_ptr 转换为 shared_ptr 使用
    if (auto spt = weakPtr.lock()) {
        std::cout << "Resource is still available." << std::endl;
    } else {
        std::cout << "Resource is no longer available." << std::endl;
    }

    return 0;
}
```

### 4. 总结

- **`std::unique_ptr`**：用于唯一拥有资源，自动释放资源，不能共享。
- **`std::shared_ptr`**：用于共享资源，引用计数确保资源不会过早释放。
- **`std::weak_ptr`**：不增加引用计数，用于打破循环引用，避免不必要的资源保持。

通过使用智能指针，C++ 程序可以更安全、简洁地管理资源，避免手动内存管理中的错误。

## L27 std::transform
`std::transform` 是 C++ 标准库中用于将容器中的元素转换（映射）到新值的算法。它常常与容器（如 `std::vector`）一起使用，用于执行元素级的操作，如转换、修改、映射等。

#### 函数签名：
```cpp
template <class InputIt, class OutputIt, class UnaryOperation>
OutputIt transform(InputIt first, InputIt last, OutputIt d_first, UnaryOperation op);
```

#### 参数：
- **InputIt first, InputIt last**: 输入序列的范围，`first` 和 `last` 是输入容器的迭代器，表示处理元素的范围。
- **OutputIt d_first**: 输出序列的起始位置，指定转换结果的存储位置。
- **UnaryOperation op**: 一个一元操作，作用于输入序列中的每个元素，并生成转换后的结果。

#### 返回值：
- 返回指向目标容器（输出序列）末尾的迭代器。

#### 例子：
1. **基本用法：** 将容器中的每个元素乘以 2。
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v{1, 2, 3, 4, 5};
    std::vector<int> result(v.size());

    // 使用 transform 将每个元素乘以 2
    std::transform(v.begin(), v.end(), result.begin(), [](int x) {
        return x * 2;
    });

    for (int num : result) {
        std::cout << num << " "; // 输出：2 4 6 8 10
    }

    return 0;
}
```

2. **转换到不同数据类型：** 使用 `std::to_string` 转换整数为字符串。
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>

int main() {
    std::vector<int> v{8, 13, 21, 34, 55};
    std::vector<std::string> result(v.size());

    // 使用 transform 转换为字符串
    std::transform(v.begin(), v.end(), result.begin(), [](int x) {
        return std::to_string(x * 2); // 乘以 2 并转为字符串
    });

    for (const std::string& str : result) {
        std::cout << str << " "; // 输出：16 26 42 68 110
    }

    return 0;
}
```

3. **修改原容器：** 可以将结果直接存储回原容器，避免额外创建目标容器。
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v{1, 2, 3, 4, 5};

    // 使用 transform 修改原容器
    std::transform(v.begin(), v.end(), v.begin(), [](int x) {
        return x * 2;
    });

    for (int num : v) {
        std::cout << num << " "; // 输出：2 4 6 8 10
    }

    return 0;
}
```

#### 变种：
1. **带有两个输入序列的版本：** `std::transform` 也可以接受两个输入序列，执行元素级的操作。
```cpp
std::transform(first1, last1, first2, d_first, binary_op);
```
- **first1, last1**: 第一个输入序列的范围。
- **first2**: 第二个输入序列的起始位置。
- **d_first**: 输出序列的起始位置。
- **binary_op**: 一个二元操作，接受两个输入序列的元素进行操作。

#### 例子：
将两个容器中的元素相加：
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v1{1, 2, 3, 4, 5};
    std::vector<int> v2{5, 4, 3, 2, 1};
    std::vector<int> result(v1.size());

    // 使用 transform 对应元素相加
    std::transform(v1.begin(), v1.end(), v2.begin(), result.begin(), std::plus<int>());

    for (int num : result) {
        std::cout << num << " "; // 输出：6 6 6 6 6
    }

    return 0;
}
```

#### 总结：
- `std::transform` 是一个非常强大的算法，适用于多种场景，尤其是当需要修改、转换或组合容器中的数据时。
- 它支持一元或二元操作，可以处理不同数据类型和容器类型。
- 通过与 Lambda 表达式配合使用，`std::transform` 可以简洁地实现复杂的元素级处理。



