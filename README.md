# EpicMovie.github.io
Chapter 01: Towards Modern C++

1.1 Deprecated Features ( If you don't know, don't even try to understand about that )

• The string literal constant is no longer allowed to be assigned to a char *. If you need to assign and initialize a char * with a string literal constant, you should use const char * or auto.

char *str = "hello world" -> string = "hello world" or const char *str = "hello world"

• C++98 exception description, unexpected_handler, set_unexpected() and other related features are deprecated and should use noexcept.

• auto_ptr is deprecated and unique_ptr should be used.

• register keyword is deprecated and can be used but no longer has any practical meaning.

• The ++ operation of the bool type is deprecated.

• If a class has a destructor, the properties for which it generates copy constructors and copy assignment operators are deprecated

• C language style type conversion is deprecated 

1.2 Compatibilities with C

You should have the idea that "C++ is not a superset of C" in your mind

When you have to use C, you should pay attention to the use of extern "C", separate the C language code from the C++ code
```
// foo.h
#ifdef __cplusplus
    extern "C" {
#endif

int add(int x, int y);

#ifdef __cplusplus
}
#endif
```

```
// foo.c
int add(int x, int y) 
{
    return x+y;
}
```
```
// 1.1.cpp
#include "foo.h"
#include <iostream>
#include <functional>
int main() 
{
    [out = std::ref(std::cout << "Result from C code: " << add(1, 2))]()
    {
        out.get() << ".\n";
    }();

    return 0;
}
```
Chapter 02: Language Usability Enhancements

2.1 Constants
- nullptr

  The purpose of nullptr appears to replace NULL

  C++ without the void * implicit conversion has to define NULL as 0. 

  Defining NULL to 0 will cause the overloading feature in C++ to be confusing.

// Source 
#include <iostream>
#include <type_traits>

void foo(char *);
void foo(int);

int main() 
{
    if (std::is_same<decltype(NULL), decltype(0)>::value)
    {
        std::cout << "NULL == 0" << std::endl;
    }

    if (std::is_same<decltype(NULL), decltype((void*)0)>::value)
    {
        std::cout << "NULL == (void *)0" << std::endl;
    }

    if (std::is_same<decltype(NULL), std::nullptr_t>::value)
    {
        std::cout << "NULL == nullptr" << std::endl;
    }

    foo(0); // will call foo(int)
    // foo(NULL); // doesn't compile
    foo(nullptr); // will call foo(char*)

    return 0;
}

void foo(char *) 
{
    std::cout << "foo(char*) is called" << std::endl;
}

void foo(int i) 
{
    std::cout << "foo(int) is called" << std::endl;
}

  The outputs are:

  foo(int) is called
  foo(char*) is called

- constexpr

  C++ itself already has the concept of constant expressions, such as 1+2, 3*4. 
  
  Such expressions always produce the same result without any side effects. 

  If the compiler can directly optimize and embed these expressions into the program at compile-time, it will increase the performance of the program.

  In addition, the function of constexpr can use recursion

#include <iostream>

#define LEN 10

int len_foo() 
{
    int i = 2;
    return i;
}

constexpr int len_foo_constexpr() 
{
    return 5;
}

constexpr int fibonacci(const int n) 
{
    return n == 1 || n == 2 ? 1 : fibonacci(n-1) + fibonacci(n-2);
}

int main() 
{
    char arr_1[10]; // legal
    char arr_2[LEN]; // legal

    int len = 10;
    
    // char arr_3[len]; // illegal
    const int len_2 = len + 1;
    constexpr int len_2_constexpr = 1 + 2 + 3;

    // char arr_4[len_2]; // illegal, but ok for most of the compilers
    char arr_4[len_2_constexpr]; // legal

    // char arr_5[len_foo()+5]; // illegal
    char arr_6[len_foo_constexpr() + 1]; // legal
    
    // 1, 1, 2, 3, 5, 8, 13, 21, 34, 55
    std::cout << fibonacci(10) << std::endl;

    return 0;
}

2.2 Variables and initialization

- if-switch

#include <iostream>
#include <vector>
#include <algorithm>

int main() 
{
    std::vector<int> vec = {1, 2, 3, 4};
    
    // since c++17, can be simplified by using `auto`
    const std::vector<int>::iterator itr = std::find(vec.begin(), vec.end(), 2);

    if (itr != vec.end()) 
    {
        *itr = 3;
    }

    if (const std::vector<int>::iterator itr = std::find(vec.begin(), vec.end(), 3); itr != vec.end()) 
    {
        *itr = 4;
    }

    // should output: 1, 4, 3, 4. can be simplified using `auto`
    for (std::vector<int>::iterator element = vec.begin(); element != vec.end(); ++element)

    std::cout << *element << std::endl;
}

- Initializer list
  Binds the concept of the initialization list to the type and calls it std::initializer_list, 
  allowing the constructor or other function to use the initialization list like a parameter

#include <initializer_list>
#include <vector>
#include <iostream>

class MagicFoo 
{
    public:
        std::vector<int> vec;

        MagicFoo(std::initializer_list<int> list) 
        {
            for (std::initializer_list<int>::iterator it = list.begin(); it != list.end(); ++it)
            {
                vec.push_back(*it);
            }
        }
};

int main() 
{
    // after C++11
    MagicFoo magicFoo = {1, 2, 3, 4, 5};

    std::cout << "magicFoo: ";

    for (std::vector<int>::iterator it = magicFoo.vec.begin(); it != magicFoo.vec.end(); ++it)
    {
        std::cout << *it << std::endl;
    }
}

- Structured binding
  Structured bindings provide functionality similar to the multiple return values

#include <iostream>
#include <tuple>

std::tuple<int, double, std::string> f() 
{
    return std::make_tuple(1, 2.3, "456");
}

int main() 
{
    auto [x, y, z] = f();

    std::cout << x << ", " << y << ", " << z << std::endl;

    return 0;
}

2.3 Type inference

C++11 introduces the two keywords auto and decltype to implement type derivation

- auto

#include <initializer_list>
#include <vector>
#include <iostream>

class MagicFoo 
{
    public:
        std::vector<int> vec;

        MagicFoo(std::initializer_list<int> list) 
        {
            for (auto it = list.begin(); it != list.end(); ++it) 
            {
                vec.push_back(*it);
            }
        }
};

int main() 
{
    MagicFoo magicFoo = {1, 2, 3, 4, 5};

    std::cout << "magicFoo: ";

    for (auto it = magicFoo.vec.begin(); it != magicFoo.vec.end(); ++it) 
    {
        std::cout << *it << ", ";
    }
    
    std::cout << std::endl;

    return 0;
}

  Since C++ 20, auto can even be used as function arguments

int add(auto x, auto y) 
{
    return x+y;
}

auto i = 5; // type int
auto j = 6; // type int

std::cout << add(i, j) << std::endl;

- decltype
  Its usage is very similar to typeof:

auto x = 1;
auto y = 2;
decltype(x+y) z;

if (std::is_same<decltype(x), int>::value)
{
    std::cout << "type x == int" << std::endl;
}

if (std::is_same<decltype(x), float>::value)
{
    std::cout << "type x == float" << std::endl;
}

if (std::is_same<decltype(x), decltype(z)>::value)
{
    std::cout << "type z == type x" << std::endl;
}

output

type x == int
type z == type x

- tail type inference

  C++11 also introduces a trailing return type, which uses the auto keyword to post the return type:

template<typename T, typename U>
auto add2(T x, U y) -> decltype(x + y)
{
    return x + y;
}

  In C++14, it is possible to directly derive the return value of a normal function, so the following way becomes legal

template<typename T, typename U>
auto add3(T x, U y)
{
    return x + y;
}

// after c++11
auto w = add2<int, double>(1, 2.0);

if (std::is_same<decltype(w), double>::value) 
{
    std::cout << "w is double: ";
}

std::cout << w << std::endl;

// after c++14
auto q = add3<double, int>(1.0, 2);

std::cout << "q: " << q << std::endl;

- declytype(auto)

  It is mainly used to derive the return type of a forwarding function or package, 
  which does not require us to explicitly specify the parameter expression of decltype.

std::string lookup1();
std::string& lookup2();

// In C++11:
std::string look_up_a_string_1() 
{
    return lookup1();
}

std::string& look_up_a_string_2() 
{
    return lookup2();
}

// In C++14
decltype(auto) look_up_a_string_1() 
{
    return lookup1();
}

decltype(auto) look_up_a_string_2() 
{
    return lookup2();
}

2.4 Control flow

- if constexpr

  C++17 introduces the constexpr keyword into the if statement,
  allowing you to declare the condition of a constant expression in your code

// In code
#include <iostream>

template<typename T>
auto print_type_info(const T& t) 
{
    if constexpr (std::is_integral<T>::value) 
    {
        return t + 1;
    }
    else
    {
        return t + 0.001;
    }
}

int main() 
{
    std::cout << print_type_info(5) << std::endl;
    std::cout << print_type_info(3.14) << std::endl;
}

// At compile time
int print_type_info(const int& t) 
{
    return t + 1;
}

double print_type_info(const double& t) 
{
    return t + 0.001;
}

- Range-based for loop
  
#include <iostream>
#include <vector>
#include <algorithm>

int main() 
{
    std::vector<int> vec = {1, 2, 3, 4};

    if (auto itr = std::find(vec.begin(), vec.end(), 3); itr != vec.end())
    {
        *itr = 4;
    }

    for (auto element : vec)
    {
        std::cout << element << std::endl; // read only
    }

    for (auto &element : vec) 
    {
        element += 1; // writeable
    }

    for (auto element : vec)
    {
        std::cout << element << std::endl; // read only
    }
}

2.5 Templates

The philosophy of the template is to throw all the problems that can be processed at compile time into the compile time, 
and only deal with those core dynamic services at runtime, to greatly optimize the performance of the runtime.

- Extern templates

  C++11 introduces an external template that extends the syntax of the original mandatory compiler to instantiate a template at a specific location, 
  allowing us to explicitly tell the compiler when to instantiate the template:

template class std::vector<bool>; // force instantiation
extern template class std::vector<double>; // should not instantiation in current file

- The">"

template<bool T>
class MagicType 
{
    bool magic = T;
};

int main()
{
    // legal, but not recommended
    std::vector<MagicType<(1>2)>> magic; 

    // ... 
}

- Type alias templates

  Templates are used to generate types.

  In traditional C++, typedef can define a new name for the type, but there is no way to define a new name for the template

  C++11 uses "using" to introduce the following form of writing, and at the same time supports the same effect as the traditional "typedef:"

typedef int (*process)(void *);
using NewProcess = int(*)(void *);

template<typename T>
using TrueDarkMagic = MagicType<std::vector<T>, std::string>;

int main() 
{
    TrueDarkMagic<bool> you;
}

- Variadic templates

  C++11 added a new representation, allowing any number, template parameters of any category, 
  and there is no need to fix the number of parameters when defining.

template<typename... Ts> class Magic;

class Magic<int, std::vector<int>, std::map<std::string, std::vector<int>>> darkMagic;

  a number of 0 is also possible

class Magic<>

  If you do not want to generate 0 template parameters, you can manually define at least one template parameter

template<typename Require, typename... Args> class Magic;

  The function parameters also use the same representation to represent the indefinite length parameters, 
  which provides a convenient means for us to simply write variable length parameter functions

  Also we can use sizeof... to calculate the number of arguments:

#include <iostream>

template<typename... Ts>
void magic(Ts... args) 
{
    std::cout << sizeof...(args) << std::endl;
}

  We can unpack the parameters with several methods

  1. Recursive template function

#include <iostream>

template<typename T0>
void printf1(T0 value) 
{
    std::cout << value << std::endl;
}

template<typename T, typename... Ts>
void printf1(T value, Ts... args) 
{
    std::cout << value << std::endl;
    printf1(args...);
}

int main() 
{
    printf1(1, 2, "123", 1.1);
    return 0;
}

  2. Variable parameter template expansion

template<typename T0, typename... T>
void printf2(T0 t0, T... t) 
{
    std::cout << t0 << std::endl;
    
    if constexpr (sizeof...(t) > 0)
    {
        printf2(t...);
    }
}

  3. Initialize list expansion

template<typename T, typename... Ts>
auto printf3(T value, Ts... args) 
{
    std::cout << value << std::endl;

    (void) std::initializer_list<T>{([&args] { std::cout << args << std::endl; }(), value)...};
}

- Fold expression

  In C++ 17, this feature of the variable length parameter is further brought to the expression

#include <iostream>

template<typename ... T>
auto sum(T ... t) 
{
    return (t + ...);
}

int main() 
{
    std::cout << sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10) << std::endl;
}

- Non-type template parameter deduction

template <typename T, int BufSize>
class buffer_t 
{
    public:
        T& alloc();
        void free(T& item);
    
    private:
        T data[BufSize];
}

buffer_t<int, 100> buf; // 100 as template parameter

  Passing with a specific literal, can the compiler assist us in type derivation.
  In C++17, there is no longer a need to explicitly specify the type by using the placeholder "auto".

template <auto value> 
void foo() 
{
    std::cout << value << std::endl;
    return;
}

int main() 
{
    foo<10>(); // value as int
}

2.6 Object-oriented

- Delegate constructor

C++11 introduces the concept of a delegate construct, which allows a constructor to call another constructor in a constructor in the same class

#include <iostream>

class Base 
{
    public:
        int value1;
        int value2;
    
        Base() 
        {
            value1 = 1;
        }

        Base(int value) : Base() 
        { 
            // delegate Base() constructor
            value2 = value;
        }
};

int main() 
{
    Base b(2);
    std::cout << b.value1 << std::endl;
    std::cout << b.value2 << std::endl;
}

- Inheritance constructor

   C++11 introduces the concept of inheritance constructors "using" the keyword using:

#include <iostream>

class Base 
{
    public:
        int value1;
        int value2;

        Base() 
        {
            value1 = 1;
        }

        Base(int value) : Base()
        { 
            // delegate Base() constructor
            value2 = value;
        }
};

class Subclass : public Base 
{
    public:
        using Base::Base; // inheritance constructor
};

int main() 
{
    Subclass s(3);
    std::cout << s.value1 << std::endl;
    std::cout << s.value2 << std::endl;
}

- Explicit virtual function overwrite

  In traditional C++, it is often prone to accidentally overloading virtual functions.

  C++11 introduces the two keywords "override" and "final" to prevent this from happening

    1. override

    When overriding a virtual function, introducing the override keyword will explicitly tell the compiler to overload, 
    and the compiler will check if the base function has such a virtual function with consistent function signature, otherwise it will not compile:

struct Base 
{
    virtual void foo(int);
};

struct SubClass: Base 
{
    virtual void foo(int) override; // legal
    virtual void foo(float) override; // illegal, no virtual function in super class
};

    2. final
    "final" is to prevent the class from being continued to inherit and to terminate the virtual function to continue to be overloaded.

struct Base 
{
    virtual void foo() final;
};

struct SubClass1 final: Base {}; // legal
struct SubClass2 : SubClass1 {}; // illegal, SubClass1 has final
struct SubClass3: Base 
{
    void foo(); // illegal, foo has final
};

- Explicit delete default function

  The ability to accurately control the generation of default functions cannot be controlled.
  Also, the default constructor generated by the compiler cannot exist at the same time as the user-defined constructor. 

  C++11 allow explicit declarations to take or reject functions that come with the compiler

class Magic 
{
    public:
        Magic() = default; // explicit let compiler use default constructor
        Magic& operator=(const Magic&) = delete; // explicit declare refuse constructor
        Magic(int magic_number);
}

- Strongly typed enumerations
  
  In traditional C++, enumerated types are not type-safe, and enumerated types are treated as integers.

  C++11 introduces an enumeration class and declares it using the syntax of "enum class":

enum class new_enum : unsigned int 
{
    value1,
    value2,
    value3 = 100,
    value4 = 100
};

    1. It cannot be implicitly converted to an integer
    2. It cannot be compared to integer numbers
    3. It is even less likely to compare enumerated values of different enumerated types

  you can compare

if (new_enum::value3 == new_enum::value4) 
{ 
    std::cout << "new_enum::value3 == new_enum::value4" << std::endl;
}

We want to get the value of the enumeration value, we will have to explicitly type conversion,
but we can overload the << operator to output, you can collect the following code snippet:

#include <iostream>

template<typename T>
std::ostream& operator<<(typename std::enable_if<std::is_enum<T>::value, std::ostream>::type& stream, const T& e)
{
        return stream << static_cast<typename std::underlying_type<T>::type>(e);
}

Chapter 03: Language Runtime Enhancements

3.1 Lambda Expression

Lambda expressions provide a feature like anonymous functions.

Anonymous functions are used when a function is needed, but you don’t want to use a name to call a function.

- Basics

[capture list] (parameter list) mutable(optional) exception attribute -> return type 
{
    // function body
}

  The capture list can serve to transfer external data and can be divided into several types

    1. Value capture

    Similar to parameter passing, the value capture is based on the fact that the variable can be copied, 
    except that the captured variable is copied when the lambda expression is created, not when it is called

void lambda_value_capture() 
{
    int value = 1;
    auto copy_value = [value] 
    {
        return value;
    };

    value = 100;

    auto stored_value = copy_value();

    // At this moment, stored_value == 1, and value == 100.
    // Because copy_value has copied when its was created.
    std::cout << "stored_value = " << stored_value << std::endl;
}

    2. Reference capture

    Similar to a reference pass, the reference capture saves the reference and the value changes.

void lambda_reference_capture() 
{
    int value = 1;
    auto copy_value = [&value] 
    {
        return value;
    };
    value = 100;

    auto stored_value = copy_value();

    // At this moment, stored_value == 100, value == 100.
    // Because copy_value stores reference
    std::cout << "stored_value = " << stored_value << std::endl;
}

    3. Implicit capture

    • [] empty capture list
    • [name1, name2, …] captures a series of variables
    • [&] reference capture, let the compiler deduce the reference list by itself
    • [=] value capture, let the compiler deduce the value list by itself

    4. Expression capture

    C++14 gives us the convenience of allowing the captured members to be initialized with arbitrary expressions, 
    which allows the capture of rvalues

#include <iostream>
#include <memory> // std::make_unique
#include <utility> // std::move

void lambda_expression_capture() 
{
    auto important = std::make_unique<int>(1);
    auto add = [v1 = 1, v2 = std::move(important)](int x, int y) -> int 
    {
        return x + y + v1 + (*v2);
    };
    
    std::cout << add(3,4) << std::endl;
}

    5. Generic Lambda

    In C++14, the formal parameters of the lambda function can use the auto keyword to utilize template generics.

void lambda_generic() 
{
    auto generic = [](auto x, auto y) 
    {
        return x+y;
    };

    std::cout << generic(1, 2) << std::endl;
    std::cout << generic(1.1, 2.2) << std::endl;
}
    
3.2 Function Object Wrapper

- std::function

  The essence of a Lambda expression is an object of a class type (called a closure type) that is similar to a function object type (called a closure object). 
  
  When the capture list of a Lambda expression is empty, the closure object can also be converted to a function pointer value for delivery.

#include <iostream>

using foo = void(int); // function pointer, Lambda as a function type

void functional(foo f) 
{
    f(1);
}

int main() 
{
    auto f = [](int value) 
    {
        std::cout << value << std::endl;
    };

    functional(f); // call by function pointer
    f(1);          // call by lambda expression

    return 0;
}

  In C++11, these concepts are unified. 
  The type of object that can be called is collectively called the callable type. 
  This type is introduced by "std::function".

  std::function is a generic, polymorphic function wrapper whose instances can store, copy, and call any target entity that can be called. 
  It is also an existing callable to C++.
  A type-safe package of entities (relatively, the call to a function pointer is not type-safe), in other words, a container of functions.

#include <functional>
#include <iostream>

int foo(int param) 
{
    return param;
}

int main() 
{
    // std::function wraps a function that take int paremeter and returns int value
    std::function<int(int)> func = foo;

    int important = 10;

    std::function<int(int)> func2 = [&](int value) -> int 
    {
        return 1 + value + important;
    };

    std::cout << func(10) << std::endl;
    std::cout << func2(10) << std::endl;
}

- std::bind and std::placeholder

  std::bind is used to bind the parameters of the function call.

int foo(int a, int b, int c) 
{
    return a + b + c;
}

int main() 
{
    // bind parameter 1, 2 on function foo,
    // and use std::placeholders::_1 as placeholder for the first parameter.
    auto bindFoo = std::bind(foo, std::placeholders::_1, 1,2);

    // when call bindFoo, we only need one param left
    bindFoo(1);
}

3.3 rvalue Reference

- lvalue, rvalue, prvalue, xvalue

    1. lvalue ( left value )
    The value to the left of the assignment symbol. To be precise, an lvalue is a persistent object that still exists after and expression.

    2. Rvalue ( right value )
    The value on the right refers to the temporary object that no longer exists after the expression ends.

    The concpet of rvalue is further divided into two : prvalue and xvalue

      i) prvalue ( pure rvalue )
        (1) Purely literal such as 10 or true
        (2) The result of the evaluation that is equivalent to a literal or anonymous temporary object like 1 + 2
        (3) Temporary variables returned by non-references
        (4) Temporary variables generated by operation expressions, original literals, and lambda expressions

      Note that a literal (except a string literal) is a prvalue. However, a string literal is an lvalue with type const char array.

#include <type_traits>

int main() 
{
    // Correct. The type of "01234" is const char [6], so it is an lvalue
    const char (&left)[6] = "01234";

    // Assert success. It is a const char [6] indeed. Note that decltype(expr)
    // yields lvalue reference if expr is an lvalue and neither an unparenthesized
    // id-expression nor an unparenthesized class member access expression.
    static_assert(std::is_same<decltype("01234"), const char(&)[6]>::value, "");
    
    // Error. "01234" is an lvalue, which cannot be referenced by an rvalue reference
    // const char (&&right)[6] = "01234";
}

const char* p = "01234";    // Correct. "01234" is implicitly converted to const char*
const char*&& pr = "01234"; // Correct. "01234" is implicitly converted to const char*, which is a
const char*& pl = "01234";  // Error. There is no type const char* lvalue

      i) xvalue ( expiring value )
      a value that is destroyed but can be moved.

std::vector<int> foo() 
{
    std::vector<int> temp = {1, 2, 3, 4};
    return temp;
}

int main()
{
    std::vector<int> v = foo(); // Entire temp is copied to v and destroied ( a large overhead )
    // ...
}

      The return value generated by foo() is used as a temporary value. 

      Once copied by v, it will be destroyed immediately, and cannot be obtained or modified.
    
      The xvalue defines behavior in which temporary values can be identified while being able to be moved.

      After C++11, the compiler did some work for us, where the lvalue temp is subjected to this implicit rvalue conversion, 
      equivalent to static_cast<std::vector<int> &&>(temp), where v here moves the value returned by foo locally.

- rvalue reference and lvalue reference

  To get a xvalue, you need to use the declaration of the rvalue reference: T &&, where T is the type.

  The statement of the rvalue reference extends the lifecycle of this temporary value, 
  and as long as the variable is alive,
  the xvalue will continue to survive.

  C++11 provides the std::move method to unconditionally convert lvalue parameters to rvalues.

#include <iostream>
#include <string>

void reference(std::string& str) 
{
    std::cout << "lvalue" << std::endl;
}

void reference(std::string&& str) 
{
    std::cout << "rvalue" << std::endl;
}

int main()
{
    std::string lv1 = "string,"; // lv1 is a lvalue
    // std::string&& r1 = lv1; // illegal, rvalue can't ref to lvalue
    std::string&& rv1 = std::move(lv1); // legal, std::move can convert lvalue to rvalue
    
    std::cout << rv1 << std::endl; // string,

    const std::string& lv2 = lv1 + lv1; // legal, const lvalue reference can extend temp variable's lifecycle

    // lv2 += "Test"; // illegal, const ref can't be modified
    std::cout << lv2 << std::endl; // string,string,
    std::string&& rv2 = lv1 + lv2; // legal, rvalue ref extend lifecycle

    rv2 += "string"; // legal, non-const reference can be modified

    std::cout << rv2 << std::endl; // string,string,string,string
    reference(rv2); // output: lvalue

    return 0;
}

  rv2 refers to an rvalue, but since it is a reference, rv2 is still an lvalue.

#include <iostream>
int main() 
{
    // int &a = std::move(1); // illegal, non-const lvalue reference cannot ref rvalue
    const int &b = std::move(1); // legal, const lvalue reference can
    std::cout << b << std::endl;
}

  The first question, why not allow non-constant references to bind to non-lvalues? 

  This is because there is a logic error in this approach:

void increase(int & v) 
{
    v++;
}

void foo() 
{
    double s = 1;
    increase(s);
}

  Since int& can’t reference a parameter of type double, you must generate a temporary value to hold the value of s.
  Thus, when increase() modifies this temporary value, s itself is not modified after the call is completed.

  The second question, why do constant references allow binding to non-lvalues? The reason is simple because Fortran needs it.

- Move semantics

  Traditional C++ does not distinguish between the concepts of “mobile” and “copy”, resulting in a large amount of data copying, wasting time and space.

  The appearance of rvalue references solves the confusion of these two concepts

#include <iostream>

class A 
{
    public:
        int *pointer;

        A() : pointer(new int(1)) 
        {
            std::cout << "construct" << pointer << std::endl;
        }

        A(A& a) : pointer(new int(*a.pointer)) 
        {
            std::cout << "copy" << pointer << std::endl;
        }

        // meaningless object copy
        A(A&& a) : pointer(a.pointer) 
        {
            a.pointer = nullptr;
            std::cout << "move" << pointer << std::endl;
        }
        
        ~A()
        {
            std::cout << "destruct" << pointer << std::endl;
            delete pointer;
        }
};

// avoid compiler optimization
A return_rvalue(bool test) 
{
    A a,b;
    
    if(test) 
    {
        return a; // equal to static_cast<A&&>(a);
    }
    else
    {
        return b; // equal to static_cast<A&&>(b);
    }
}

int main() 
{
    A obj = return_rvalue(false);

    std::cout << "obj:" << std::endl;
    std::cout << obj.pointer << std::endl;
    std::cout << *obj.pointer << std::endl;

    return 0;
}

    1. First construct two A objects inside return_rvalue, and get the output of the two constructors

    2. After the function returns, it will generate a xvalue, which is referenced by the moving structure of A (A(A&&)), 
    thus extending the life cycle, and taking the pointer in the rvalue and saving it to obj. 
    In the middle, the pointer to the xvalue is set to nullptr, which prevents the memory area from being destroyed

  This avoids meaningless copy constructs and enhances performance. 

#include <iostream> // std::cout
#include <utility> // std::move
#include <vector> // std::vector
#include <string> // std::string

int main() 
{
    std::string str = "Hello world.";
    std::vector<std::string> v;

    // use push_back(const T&), copy
    v.push_back(str);

    // "str: Hello world."
    std::cout << "str: " << str << std::endl;

    // use push_back(const T&&),
    // no copy the string will be moved to vector,
    // and therefore std::move can reduce copy cost
    v.push_back(std::move(str));

    // str is empty now
    std::cout << "str: " << str << std::endl;
    return 0;
}

- Perfect forwarding

  The rvalue reference of a declaration is actually an lvalue. 
  This creates problems for us to parameterize (pass)

#include <iostream>
#include <utility>

void reference(int& v) 
{
    std::cout << "lvalue reference" << std::endl;
}

void reference(int&& v) 
{
    std::cout << "rvalue reference" << std::endl;
}

template <typename T>
void pass(T&& v) 
{
    std::cout << " normal param passing: ";

    // Since v is a reference variable, it always be an lvalue.
    reference(v);
}

int main() 
{
    std::cout << "rvalue pass:" << std::endl;
    pass(1);    // Even though value ifself is rvalue, v is reference value. So "void reference(int& v)" will be called

    std::cout << "lvalue pass:" << std::endl;
    int l = 1;
    pass(l);    // Successfully passed to "void reference(int& v)"

    reference(1);  // Successfully passed to "void reference(int&& v)"

    return 0;
}

  In traditional C++, it is impossible to continue reference a reference type.

  Now it is possible to reference references. But follow the rule below

  Function parameter type | Argument parameter type | Post-derivation function parameter type
            T&            |       lvalue ref        |                   T&
            T&            |       rvalue ref        |                   T&
            T&&           |       lvalue ref        |                   T&
            T&&           |       rvalue ref        |                   T&&

  Therefore, the use of T&& in a template function may not be able to make an rvalue reference, 
  and when a lvalue is passed, a reference of this function will be derived as an lvalue.

  No matter what type of reference the template parameter is, 
  the template parameter can be derived as a right reference type if and only if the argument type is a right reference.

  Perfect forwarding let us pass the parameters, keep the original parameter type

#include <iostream>
#include <utility>

void reference(int& v)
{
    std::cout << "lvalue reference" << std::endl;
}

void reference(int&& v)
{
    std::cout << "rvalue reference" << std::endl;
}

template<typename T>
void pass(T&& v)
{
    std::cout << "Normal Passing : ";
    reference(v);

    std::cout << "Move Passing : ";
    reference(std::move(v));

    std::cout << "Forward Passing : ";
    reference(std::forward<T>(v));

    std::cout << "static_cast<T&&> param passing: ";
    reference(static_cast<T&&>(v));
}

int main() {
    std::cout << "rvalue pass:" << std::endl;
    pass(1);

    std::cout << "lvalue pass:" << std::endl;
    int l = 1;
    pass(l);

    return 0;
}

  std::forward is the same as std::move, and nothing is done. 

  std::move simply converts the lvalue to the rvalue. 

  std::forward is just a simple conversion of the parameters. 

  std::forward<T>(v) is the same as static_cast<T&&>(v).

template<typename _Tp>
constexpr _Tp&& forward(typename std::remove_reference<_Tp>::type& __t) noexcept
{
  return static_cast<_Tp&&>(__t); 
}

template<typename _Tp>
constexpr _Tp&& forward(typename std::remove_reference<_Tp>::type&& __t) noexcept
{
  static_assert(!std::is_lvalue_reference<_Tp>::value, "template argument"
  " substituting _Tp is an lvalue reference type");

  return static_cast<_Tp&&>(__t);
}

  The function of std::remove_reference is to eliminate references in the type.

  std::is_lvalue_reference is used to check if the type derivation is correct.

Chapter 04 Containers

4.1 Linear Container

  1. Why introduce std::array instead of std::vector directly?
 
    Unlike std::vector, the size of the std::array object is fixed.

    Also, since std::vector is automatically expanded, 
    when a large amount of data is stored, 
    and the container is deleted, 
    The container does not automatically return the corresponding memory of the deleted element.

    You need to manually run shrink_to_fit() to release this part of the memory.

#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v;
    std::cout << "size:" << v.size() << std::endl; // output 0
    std::cout << "capacity:" << v.capacity() << std::endl; // output 0
    
    // As you can see, the storage of std::vector is automatically managed and
    // automatically expanded as needed.
    // But if there is not enough space, you need to redistribute more memory,
    // and reallocating memory is usually a performance-intensive operation.
    v.push_back(1);
    v.push_back(2);
    v.push_back(3);
    std::cout << "size:" << v.size() << std::endl; // output 3
    std::cout << "capacity:" << v.capacity() << std::endl; // output 4
    
    // The auto-expansion logic here is very similar to Golang's slice.
    v.push_back(4);
    v.push_back(5);
    
    std::cout << "size:" << v.size() << std::endl; // output 5
    std::cout << "capacity:" << v.capacity() << std::endl; // output 8
    
    // As can be seen below, although the container empties the element,
    // the memory of the emptied element is not returned.
    v.clear();
    std::cout << "size:" << v.size() << std::endl; // output 0
    std::cout << "capacity:" << v.capacity() << std::endl; // output 8
    
    // Additional memory can be returned to the system via the shrink_to_fit() call
    v.shrink_to_fit();
    std::cout << "size:" << v.size() << std::endl; // output 0
    std::cout << "capacity:" << v.capacity() << std::endl; // output 0

    return 0;
}

  2. Already have a traditional array, why use std::array?

    std::array encapsulates some manipulation functions, 
    such as getting the array size and checking if it is not empty, 
    and also using the standard friendly.


#include <iostream>
#include <array>
#include <algorithm>

using namespace std;

int main()
{
    std::array<int, 4> arr = {1, 2, 3, 4};
    
    std::cout << arr.size() << std::endl; // return the size of the container 
    std::cout << std::endl;
   
    // iterator support
    for (auto &i : arr)
    {
        std::cout << i << std::endl;
    }    
   
    std::cout << std::endl;
   
    std::sort(arr.begin(), arr.end(), [](int a, int b) 
    {
        return b < a;
    });    
   
    // iterator support
    for (auto &i : arr)
    {
        std::cout << i << std::endl;
    }    
    
    arr.empty(); // check if container is empty
   
    std::cout << std::endl;
    std::cout << arr.size() << std::endl; // return the size of the container 

    // array size must be constexpr
    constexpr int len = 4;
    std::array<int, len> arr_constexpr = {1, 2, 3, 4};
    
    return 0;
}

4.2 Unordered Container

These elements are internally implemented by red-black trees. The average complexity of inserts and searches
is O(log(size)). When inserting an element, the element size is compared according to the < operator and the element is determined to be the same. 
And select the appropriate location to insert into the container. 
When traversing the elements in this container, 
the output will be traversed one by one in the order of the < operator.

The elements in the unordered container are not sorted, and the internals is implemented by the Hash table. 
The average complexity of inserting and searching for elements is O(constant), 
Significant performance gains can be achieved without concern for the order of the elements inside the container.

C++11 introduces two unordered containers: 

std::unordered_map/std::unordered_multimap

and 

std::unordered_set/std::unordered_multiset.

Their usage is basically similar to the original std::map/std::multimap/std::set/set::multiset
Since these containers are already familiar to us, we will not compare them one by one.

#include <iostream>
#include <string>
#include <unordered_map>
#include <map>

int main() {
    // initialized in same order
    std::unordered_map<int, std::string> u = {
        {1, "1"},
        {3, "3"},
        {2, "2"}
    };

    std::map<int, std::string> v = {
        {1, "1"},
        {3, "3"},
        {2, "2"}
    };

    // iterates in the same way
    std::cout << "std::unordered_map" << std::endl;

    for( const auto & n : u)
        std::cout << "Key:[" << n.first << "] Value:[" << n.second << "]\n";

    std::cout << std::endl;
    std::cout << "std::map" << std::endl;

    for( const auto & n : v)
        std::cout << "Key:[" << n.first << "] Value:[" << n.second << "]\n";
}

4.3 Tuples

#include <tuple>
#include <iostream>

// Construct tuples 
auto get_student(int id) 
{
    if (id == 0)
        return std::make_tuple(3.8, 'A', "John");
    if (id == 1)
        return std::make_tuple(2.9, 'C', "Jack");
    if (id == 2)
        return std::make_tuple(1.7, 'D', "Ive");

    // it is not allowed to return 0 directly
    // return type is std::tuple<double, char, std::string>
    return std::make_tuple(0.0, 'D', "null");
}

int main() 
{
    auto student = get_student(0);
    std::cout << "ID: 0, " 
        << "GPA: " << std::get<0>(student) << ", "
        << "Grade: " << std::get<1>(student) << ", "
        << "Name: " << std::get<2>(student) << '\n';

    double gpa;
    char grade;
    std::string name;

    // unpack tuples
    std::tie(gpa, grade, name) = get_student(1);

    std::cout << "ID: 1, "
        << "GPA: " << gpa << ", "
        << "Grade: " << grade << ", "
        << "Name: " << name << '\n';
}

std::get In addition to using constants to get tuple objects, C++14 adds usage types to get objects in tuples:

std::tuple<std::string, double, double, int> t("123", 4.5, 6.7, 8);

std::cout << std::get<std::string>(t) << std::endl;
std::cout << std::get<double>(t) << std::endl; // illegal, runtime error
std::cout << std::get<3>(t) << std::endl;

Runtime Indexing

If you think about it, you might find the problem with the above code. 
std::get<> depends on a compile-time constant, so the following is not legal:

int index = 1;
std::get<index>(t);

So what do you do?
The answer is to use std::variant<> (introduced by C++ 17) to provide type template parameters for variant<> 
You can have a variant<> to accommodate several types of variables provided

#include <variant>
template <size_t n, typename... T>
constexpr std::variant<T...> _tuple_index(const std::tuple<T...>& tpl, size_t i) 
{
    if constexpr (n >= sizeof...(T))
        throw std::out_of_range("￿￿.");
    if (i == n)
        return std::variant<T...>{ std::in_place_index<n>, std::get<n>(tpl) };
    return _tuple_index<(n < sizeof...(T)-1 ? n+1 : 0)>(tpl, i);
}

template <typename... T>
constexpr std::variant<T...> tuple_index(const std::tuple<T...>& tpl, size_t i) 
{
    return _tuple_index<0>(tpl, i);
}

template <typename T0, typename ... Ts>
std::ostream & operator<< (std::ostream & s, std::variant<T0, Ts...> const & v) 
{
    std::visit([&](auto && x){ s << x;}, v);
    return s;
}

int i = 1;
std::cout << tuple_index(t, i) << std::endl;

Merge and Iteration

Another common requirement is to merge two tuples, which can be done with std::tuple_cat:

auto new_tuple = std::tuple_cat(get_student(1), std::move(t));

You can immediately see how quickly you can traverse a tuple? 
But we just introduced how to index a tuple by a very number at runtime, 
then the traversal becomes simpler. 

First, we need to know the length of a tuple, which can: 

template <typename T>
auto tuple_len(T &tpl) 
{
    return std::tuple_size<T>::value;
}

This will iterate over the tuple:

for(int i = 0; i != tuple_len(new_tuple); ++i)
{
    // runtime indexing
    std::cout << tuple_index(new_tuple, i) << std::endl;
}

Chapter 05 Smart Pointers and Memory Management
5.1 RAII and Reference Counting
The reference count is counted to prevent memory leaks. 

The basic idea is to count the number of dynamically allocated objects. 

Whenever you add a reference to the same object, 

the reference count of the referenced object is incremented once.

Each time a reference is deleted, the reference count is decremented by one. 

When the reference count of an object is reduced to zero, the pointed heap memory is automatically deleted.

In traditional C++, “remembering” to manually release resources is not always a best practice.

Because we are likely to forget to release resources and lead to leakage. 

So the usual practice is that for an object, 

we apply for space when constructor, and free space when the destructor (called when leaving the scope). 

That is, we often say that the RAII resource acquisition is the initialization technology.

There are exceptions to everything, we always need to allocate objects on free storage. 

In traditional C++, we have to use new and delete to “remember” to release resources. 

C++11 introduces the concept of smart pointers by using the idea of reference counting so that programmers no longer need to care about manually releasing memory.

These smart pointers include 

std::shared_ptr
std::unique_ptr
std::weak_ptr

which need to include the header file <memory>.

5.2 std::shared_ptr

std::shared_ptr is a smart pointer that records how many shared_ptr points to an object, 
eliminating to call delete, 
which automatically deletes the object when the reference count becomes zero.

But not enough, because using std::shared_ptr still needs to be called with new, which makes the code a certain degree of asymmetry.

std::make_shared can be used to eliminate the explicit use of new, 

so std::make_shared will allocate the objects in the generated parameters.

And return the std::shared_ptr pointer of this object type. 

For example:

#include <iostream>
#include <memory>

void foo(std::shared_ptr<int> i) 
{
    (*i)++;
}

int main() 
{
    // illegal, no direct assignment
    // auto pointer = new int(10); 

    // Constructed a std::shared_ptr
    auto pointer = std::make_shared<int>(10);

    foo(pointer);
    std::cout << *pointer << std::endl; // 11

    // The shared_ptr will be destructed before leaving the scope
    return 0;
}

std::shared_ptr can get the raw pointer through the get() method and reduce the reference count
by reset(). And see the reference count of an object by use_count().

auto pointer = std::make_shared<int>(10);
auto pointer2 = pointer; // reference count+1
auto pointer3 = pointer; // reference count+1

int *p = pointer.get(); // no increase of reference count

std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl;     // 3
std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl;   // 3
std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl;   // 3

pointer2.reset();

std::cout << "reset pointer2:" << std::endl;

std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl;     // 2
std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl;   // pointer2 has reset, 0
std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl;   // 2

pointer3.reset();

std::cout << "reset pointer3:" << std::endl;
std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl; // 1
std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl; // 0
std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl;

5.3 std::unique_ptr

std::unique_ptr is an exclusive smart pointer that prohibits other smart pointers from sharing the same object,

thus keeping the code safe:

std::unique_ptr<int> pointer = std::make_unique<int>(10); // make_unique, from C++14
std::unique_ptr<int> pointer2 = pointer; // illegal

make_unique is not complicated. C++11 does not provide std::make_unique, which can be implemented by itself:

template<typename T, typename ...Args>
std::unique_ptr<T> make_unique( Args&& ...args ) 
{
    return std::unique_ptr<T>( new T( std::forward<Args>(args)... ) );
}

Since it is monopolized, in other words, it cannot be copied. 

However, we can use std::move to transfer it to other unique_ptr, for example:

#include <iostream>
#include <memory>

struct Foo 
{
    Foo() { std::cout << "Foo::Foo" << std::endl; }
    ~Foo() { std::cout << "Foo::~Foo" << std::endl; }
    void foo() { std::cout << "Foo::foo" << std::endl; }
};

void f(const Foo &) 
{
    std::cout << "f(const Foo&)" << std::endl;
}

int main() 
{
    std::unique_ptr<Foo> p1(std::make_unique<Foo>());
    
    // p1 is not empty, prints
    if (p1)
        p1->foo();

    {
        std::unique_ptr<Foo> p2(std::move(p1));

        // p2 is not empty, prints
        f(*p2);

        // p2 is not empty, prints
        if(p2)
            p2->foo();

        // p1 is empty, no prints
        if(p1) 
            p1->foo();

        p1 = std::move(p2);

        // p2 is empty, no prints
        if(p2)
            p2->foo();

        std::cout << "p2 was destroyed" << std::endl;
    }

    // p1 is not empty, prints
    if (p1)
        p1->foo();

    // Foo instance will be destroyed when leaving the scope
}

5.4 std::weak_ptr
If you think about std::shared_ptr carefully, you will still find that there is still a problem that resources cannot be released.

Look at the following example:

#include <iostream>
#include <memory>

class A;
class B;

class A
{
    public:
        std::shared_ptr<B> pointer;

        ~A() 
        {
            std::cout << "A was destroyed" << std::endl;
        }
};

class B 
{
    public:
        std::shared_ptr<A> pointer;

        ~B() 
        {
            std::cout << "B was destroyed" << std::endl;
        }
};

int main() 
{
    std::shared_ptr<A> a = std::make_shared<A>();
    std::shared_ptr<B> b = std::make_shared<B>();

    a->pointer = b;
    b->pointer = a;

    return 0;
}

The result is that A and B will not be destroyed. This is because the pointer inside a, b also references a, b, which makes the reference count of a, b becomes 2, leaving the scope. 

When the a, b smart pointer is destructed, it can only cause the reference count of this area to be decremented by one. 

This causes the memory area reference count pointed to by the a, b object to be non-zero, but the external has no way to find this area, it also caused a memory leak

The solution to this problem is to use the weak reference pointer std::weak_ptr, 

which is a weak reference (compared to std::shared_ptr is a strong reference). 

A weak reference does not cause an increase in the reference count. 

std::weak_ptr has no implemented * and -> operators, therefore it cannot operate on resources.

std::weak_ptr allows us to check if a std::shared_ptr exists or not. 

The expired() method of a std::weak_ptr returns false when the resource is not released; 

Otherwise, it returns true. 

Furthermore, it can also be used for the purpose of obtaining std::shared_ptr, which points to the original object.

The lock() method returns a std::shared_ptr to the original object when the resource is not released, or nullptr otherwise.

Chapter 06 Regular Expression

// Not part of C++

Chapter 07 Parallelism and Concurrency

7.1 Basic of Parallelism

std::thread is used to create an execution thread instance, so it is the basis for all concurrent programming.

It needs to include the <thread> header file when using it. 

It provides a number of basic thread operations, such as get_id() to get the thread ID of the thread being created,

use join() to join a thread, etc.

#include <iostream>
#include <thread>

int main()
{
    std::thread t([]()
    {
        std::cout << "hello world." << std::endl;
    });

    t.join();
    return 0;
}

7.2 Mutex and Critical Section

We have already learned the basics of concurrency technology in the operating system, or the database, and mutex is one of the cores. 

C++11 introduces a class related to mutex, with all related functions in the <mutex> header file.

std::mutex is the most basic mutex class in C++11, and you can create a mutex by instantiating std::mutex. 

It can be locked by its member function lock(), and unlock() can be unlocked. 

But in the process of actually writing the code, it is best not to directly call the member function, Because calling member functions, 

you need to call unlock() at the exit of each critical section, and of course, exceptions. 

At this time, C++11 also provides a template class std::lock_guard for the RAII syntax for the mutex.

RAII guarantees the exceptional security of the code while keeping the simplicity of the code.

#include <iostream>
#include <mutex>
#include <thread>

int v = 1;

void critical_section(int change_v) 
{
    static std::mutex mtx;

    std::lock_guard<std::mutex> lock(mtx);

    // execute contention works
    v = change_v;

    // mtx will be released after leaving the scope
}

int main() 
{
    std::thread t1(critical_section, 2), t2(critical_section, 3);
    t1.join();
    t2.join();
    std::cout << v << std::endl;
    return 0;
}

Because C++ guarantees that all stack objects will be destroyed at the end of the declaration period, such code is also extremely safe. 

Whether critical_section() returns normally or if an exception is thrown in the middle, 

a stack rollback is thrown, and unlock() is automatically called.

std::unique_lock is more flexible than std::lock_guard. 

Objects of std::unique_lock manage the locking and unlocking operations on the mutex object with exclusive ownership 
(no other unique_lock objects owning the ownership of a mutex object). 

So in concurrent programming, it is recommended to use std::unique_lock. 

std::lock_guard cannot explicitly call lock and unlock, and std::unique_lock can be called anywhere after the declaration. 

It can reduce the scope of the lock and provide higher concurrency. 

If you use the condition variable std::condition_variable::wait you must use std::unique_lock as a parameter.


#include <iostream>
#include <mutex>
#include <thread>

int v = 1;

void critical_section(int change_v) 
{
    static std::mutex mtx;
    std::unique_lock<std::mutex> lock(mtx);

    // do contention operations
    v = change_v;

    std::cout << v << std::endl;
    
    // release the lock
    lock.unlock();

    // during this period,
    // others are allowed to acquire v
    // start another group of contention operations
    // lock again
    lock.lock();
    v += 1;
    std::cout << v << std::endl;
}

int main() 
{
    std::thread t1(critical_section, 2), t2(critical_section, 3);
    t1.join();
    t2.join();
    return 0;
}

7.3 Future

The Future is represented by std::future, which provides a way to access the results of asynchronous operations.

This sentence is very difficult to understand. 

To understand this feature, we need to understand the multi-threaded behavior before C++11.

Imagine if our main thread A wants to open a new thread B to perform some of our expected tasks and return me a result. 

At this time, thread A may be busy with other things and have no time to take into account the results of B. 

So we naturally hope to get the result of thread B at a certain time.

Before the introduction of std::future in C++11, the usual practice is:

1. Create a thread A
2. Start task B in thread A and send an event when it is ready 
3. save the result in a global variable.
4. The main function thread A is doing other things. 
5. When the result is needed, a thread is called to wait for the function to get the result of the execution.

The std::future provided by C++11 simplifies this process and can be used to get the results of asynchronous tasks.

Naturally, we can easily imagine it as a simple means of thread synchronization, namely the barrier.

To see an example, we use extra std::packaged_task, which can be used to wrap any target that can be called for asynchronous calls.

#include <iostream>
#include <thread>
#include <future>

int main() 
{
    // pack a lambda expression that returns 7 into a std::packaged_task
    std::packaged_task<int()> task([]()
    {
        return 7;
    });

    // get the future of task
    std::future<int> result = task.get_future(); // run task in a thread
    std::thread(std::move(task)).detach();
    std::cout << "waiting...";

    // block until future has arrived
    result.wait(); 
    
    // output result
    std::cout << "done!" << std:: endl << "future result is " << result.get() << std::endl;
    return 0;
}

After encapsulating the target to be called, you can use get_future() to get a std::future object
to implement thread synchronization later.

7.4 Condition Variable

The condition variable std::condition_variable was born to solve the deadlock and was introduced when the mutex operation was not enough.

For example, a thread may need to wait for a condition to be true to continue execution. 

A dead wait loop can cause all other threads to fail to enter the critical section so that when the condition is true, a deadlock occurs.

Therefore, the condition_variable instance is created primarily to wake up the waiting thread and avoid deadlocks.

notify_one() of std::condition_variable is used to wake up a thread; notify_all() is to notify all threads. 

Below is an example of a producer and consumer model:

#include <queue>
#include <chrono>
#include <mutex>
#include <thread>
#include <iostream>
#include <condition_variable>

int main() 
{
    std::queue<int> produced_nums;
    std::mutex mtx;
    std::condition_variable cv;

    bool notified = false; // notification sign
    auto producer = [&]() 
    {
        for (int i = 0; i < 2; i++) 
        {
            std::this_thread::sleep_for(std::chrono::milliseconds(500));
            std::unique_lock<std::mutex> lock(mtx);
            std::cout << "producing " << i << std::endl;
            produced_nums.push(i);
            notified = true;
            cv.notify_all();
        }
    };

    auto consumer = [&]() 
    {
        while (true) 
        {
            std::unique_lock<std::mutex> lock(mtx);
            while (!notified) 
            { // avoid spurious wakeup
                cv.wait(lock);
            }
            
            // temporal unlock to allow producer produces more rather than
            // let consumer hold the lock until its consumed.
            lock.unlock();

            // consumer is slower
            std::this_thread::sleep_for(std::chrono::milliseconds(1000));
            lock.lock();
        
            if (!produced_nums.empty()) 
            {
                std::cout << "consuming " << produced_nums.front() << std::endl;
                produced_nums.pop();
            }

            notified = false;
        }
    };

    std::thread p(producer);
    std::thread cs[2];

    for (int i = 0; i < 2; ++i) 
    {
        cs[i] = std::thread(consumer);
    }

    p.join();
    for (int i = 0; i < 2; ++i) 
    {
        cs[i].join();
    }

    return 0;
}

It is worth mentioning that although we can use notify_one() in the producer, it is not recommended to use it here. 

Because in the case of multiple consumers, our consumer implementation simply gives up the lock holding, 

which makes it possible for other consumers to compete for this lock, 

to better utilize the concurrency between multiple consumers. 

Having said that, but in fact because of the exclusivity of std::mutex,

We simply can’t expect multiple consumers to be able to produce content in a parallel consumer queue, and we still need a more granular approach.

7.5 Atomic Operation and Memory Model

Careful readers may be tempted by the fact that the example of the producer-consumer model in the previous section may have compiler optimizations that cause program errors.

For example, the boolean notified is not modified by volatile, and the compiler may have optimizations for this variable,

such as the value of a register.

As a result, the consumer thread can never observe the change of this value. 

This is a good question. 

To explain this problem, we need to further discuss the concept of the memory model introduced from C++11. 

Let’s first look at a question.

What is the output of the following code?

#include <thread>
#include <iostream>

int main() 
{
    int a = 0;
    volatile int flag = 0;

    std::thread t1([&]() 
    {
        while (flag != 1);
        int b = a;
        std::cout << "b = " << b << std::endl;
    });

    std::thread t2([&]() 
    {
        a = 5;
        flag = 1;
    });

    t1.join();
    t2.join();

    return 0;
}

Intuitively, it seems that a = 5; in t2 always executes before flag = 1; and while (flag != 1) in t1.

It looks like there is a guarantee the line std ::cout << "b = " << b << std::endl; will not be executed before the mark is changed.

Logically, it seems that the value of b should be equal to 5. 

But the actual situation is much more complicated than this, or the code itself is undefined behavior because, for a and flag,

they are read and written in two parallel threads.

There has been competition. 

Also, even if we ignore competing for reading and writing, it is still possible to receive out-of-order execution of the CPU and the impact of the compiler on the rearrangement of instructions.

Cause a = 5 to occur after flag = 1.

Thus b may output 0.

Atomic Operation

std::mutex can solve the problem of concurrent read and write, but the mutex is an operating system-level function.

This is because the implementation of a mutex usually contains two basic principles:

1. Provide automatic state transition between threads, that is, “lock” state
2. Ensure that the memory of the manipulated variable is isolated from the critical section during the mutex operation

This is a very strong set of synchronization conditions, in other words when it is finally compiled into a CPU instruction,

it will behave like a lot of instructions (we will look at how to implement a simple mutex later). 

This seems too harsh for a variable that requires only atomic operations (no intermediate state).

The research on synchronization conditions has a very long history, and we will not go into details here.

Readers should understand that under the modern CPU architecture, atomic operations at the CPU instruction level are provided.

Therefore, in the C++11 multi-threaded shared variable reading and writing, 

the introduction of the std::atomic template, 

so that we instantiate an atomic type, will be an Atomic type read and write operations are minimized from a set of instructions to a single CPU instruction. E.g:

std::atomic<int> counter;

And provides basic numeric member functions for atomic types of integers or floating-point numbers,
for example, Including fetch_add, fetch_sub, etc., and the corresponding +, - version is provided by overload. 
For example, the following example:

#include <atomic>
#include <thread>
#include <iostream>

std::atomic<int> count = {0};

int main() 
{
    std::thread t1([]()
    {
        count.fetch_add(1);
    });

    std::thread t2([]()
    {
        count++; // identical to fetch_add
        count += 1; // identical to fetch_add
    });

    t1.join();
    t2.join();

    std::cout << count << std::endl;

    return 0;
}

Of course, not all types provide atomic operations because the feasibility of atomic operations depends
on the architecture of the CPU and whether the type structure being instantiated satisfies the memory
alignment requirements of the architecture, so we can always pass std::atomic<T>::is_lock_free
to check if the atom type needs to support atomic operations, for example:

#include <atomic>
#include <iostream>
struct A {
float x;
int y;
long long z;
};
int main() {
std::atomic<A> a;
std::cout << std::boolalpha << a.is_lock_free() << std::endl;
return 0;
}

Consistency Model
Multiple threads executing in parallel, discussed at some macro level, can be roughly considered a
distributed system. In a distributed system, any communication or even local operation takes a certain
amount of time, and even unreliable communication occurs.
If we force the operation of a variable v between multiple threads to be atomic, that is, any thread
78
7.5 Atomic Operation and Memory Model CHAPTER 07 PARALLELISM AND CONCURRENCY
after the operation of v Other threads can synchronize to perceive changes in v, for the variable v,
which appears as a sequential execution of the program, it does not have any efficiency gains due to the
introduction of multithreading. Is there any way to accelerate this properly? The answer is to weaken
the synchronization conditions between processes in atomic operations.
In principle, each thread can correspond to a cluster node, and communication between threads is
almost equivalent to communication between cluster nodes. Weakening the synchronization conditions
between processes, usually we will consider four different consistency models:
1. Linear consistency: Also known as strong consistency or atomic consistency. It requires that any
read operation can read the most recent write of a certain data, and the order of operation of all
threads is consistent with the order under the global clock.
x.store(1) x.load()
T1 ---------+----------------+------>
T2 -------------------+------------->
x.store(2)
In this case, thread T1, T2 is twice atomic to x, and x.store(1) is strictly before x.store(2).
x.store(2) strictly occurs before x.load(). It is worth mentioning that linear consistency requirements
for global clocks are difficult to achieve, which is why people continue to study other
consistent algorithms under this weaker consistency.
2. Sequential consistency: It is also required that any read operation can read the last data written
by the data, but it is not required to be consistent with the order of the global clock.
x.store(1) x.store(3) x.load()
T1 ---------+-----------+----------+----->
T2 ---------------+---------------------->
x.store(2)
or
x.store(1) x.store(3) x.load()
T1 ---------+-----------+----------+----->
T2 ------+------------------------------->
x.store(2)
79
7.5 Atomic Operation and Memory Model CHAPTER 07 PARALLELISM AND CONCURRENCY
Under the order consistency requirement, x.load() must read the last written data, so
x.store(2) and x.store(1) do not have any guarantees, as long as x.store(2) of T2 occurs
before x.store(3).
3. Causal consistency: its requirements are further reduced, only the sequence of causal operations is
guaranteed, and the order of non-causal operations is not required.
a = 1 b = 2
T1 ----+-----------+---------------------------->
T2 ------+--------------------+--------+-------->
x.store(3) c = a + b y.load()
or
a = 1 b = 2
T1 ----+-----------+---------------------------->
T2 ------+--------------------+--------+-------->
x.store(3) y.load() c = a + b
or
b = 2 a = 1
T1 ----+-----------+---------------------------->
T2 ------+--------------------+--------+-------->
y.load() c = a + b x.store(3)
The three examples given above are all causal consistent because, in the whole process, only c has
a dependency on a and b, and x and y are not related in this example. (But in actual situations
we need more detailed information to determine that x is not related to y)
4. Final Consistency: It is the weakest consistency requirement. It only guarantees that an operation
will be observed at a certain point in the future, but does not require the observed time. So we
can even strengthen this condition a bit, for example, to specify that the time observed for an
operation is always bounded. Of course, this is no longer within our discussion.
x.store(3) x.store(4)
T1 ----+-----------+-------------------------------------------->
80
7.5 Atomic Operation and Memory Model CHAPTER 07 PARALLELISM AND CONCURRENCY
T2 ---------+------------+--------------------+--------+-------->
x.read() x.read() x.read() x.read()
In the above case, if we assume that the initial value of x is 0, then the four times ‘x.read() in T2
may be but not limited to the following:
3 4 4 4 // The write operation of x was quickly observed
0 3 3 4 // There is a delay in the observed time of the x write operation
0 0 0 4 // The last read read the final value of x,
// but the previous changes were not observed.
0 0 0 0 // The write operation of x is not observed in the current time period,
// but the situation that x is 4 can be observed
// at some point in the future.
Memory Orders
To achieve the ultimate performance and achieve consistency of various strength requirements,
C++11 defines six different memory sequences for atomic operations. The option std::memory_order
expresses four synchronization models between multiple threads:
1. Relaxed model: Under this model, atomic operations within a single thread are executed sequentially,
and instruction reordering is not allowed, but the order of atomic operations between different
threads is arbitrary. The type is specified by std::memory_order_relaxed. Let’s look at an
example:
std::atomic<int> counter = {0};
std::vector<std::thread> vt;
for (int i = 0; i < 100; ++i) {
vt.emplace_back([&](){
counter.fetch_add(1, std::memory_order_relaxed);
});
}
for (auto& t : vt) {
t.join();
}
std::cout << "current counter:" << counter << std::endl;
2. Release/consumption model: In this model, we begin to limit the order of operations between processes.
If a thread needs to modify a value, but another thread will have a dependency on that operation
of the value, that is, the latter depends on the former. Specifically, thread A has completed
three writes to x, and thread B relies only on the third x write operation, regardless of the first two
81
7.5 Atomic Operation and Memory Model CHAPTER 07 PARALLELISM AND CONCURRENCY
write behaviors of x, then A When active x.release() (ie using std::memory_order_release),
the option std::memory_order_consume ensures that B observes A when calling x.load() Three
writes to x. Let’s look at an example:
// initialize as nullptr to prevent consumer load a dangling pointer
std::atomic<int*> ptr(nullptr);
int v;
std::thread producer([&]() {
int* p = new int(42);
v = 1024;
ptr.store(p, std::memory_order_release);
});
std::thread consumer([&]() {
int* p;
while(!(p = ptr.load(std::memory_order_consume)));
std::cout << "p: " << *p << std::endl;
std::cout << "v: " << v << std::endl;
});
producer.join();
consumer.join();
3. Release/Acquire model: Under this model, we can further tighten the order of atomic operations
between different threads, specifying the timing between releasing std::memory_order_release
and getting std::memory_order_acquire. All write operations before the release operation is
visible to any other thread, i.e., happens before.
As you can see, std::memory_order_release ensures that a write before a release does not occur
after the release operation, which is a backward barrier, and std::memory_order_acquire ensures
that a subsequent read or write after a acquire does not occur before the acquire operation,
which is a forward barrier. For the std::memory_order_acq_rel option, combines the characteristics
of the two barriers and determines a unique memory barrier, such that reads and writes
of the current thread will not be rearranged across the barrier.
Let’s check an example:
std::vector<int> v;
std::atomic<int> flag = {0};
std::thread release([&]() {
v.push_back(42);
flag.store(1, std::memory_order_release);
});
std::thread acqrel([&]() {
int expected = 1; // must before compare_exchange_strong
while(!flag.compare_exchange_strong(expected, 2, std::memory_order_acq_rel))
82
7.5 Atomic Operation and Memory Model CHAPTER 07 PARALLELISM AND CONCURRENCY
expected = 1; // must after compare_exchange_strong
// flag has changed to 2
});
std::thread acquire([&]() {
while(flag.load(std::memory_order_acquire) < 2);
std::cout << v.at(0) << std::endl; // must be 42
});
release.join();
acqrel.join();
acquire.join();
In this case we used compare_exchange_strong, which is the Compare-and-swap primitive, which
has a weaker version, compare_exchange_weak, which allows a failure to be returned even if the
exchange is successful. The reason is due to a false failure on some platforms, specifically when
the CPU performs a context switch, another thread loads the same address to produce an inconsistency.
In addition, the performance of compare_exchange_strong may be slightly worse than
compare_exchange_weak. However, in most cases, compare_exchange_weak is discouraged due to
the complexity of its usage.
4. Sequential Consistent Model: Under this model, atomic operations satisfy sequence consistency,
which in turn can cause performance loss. It can be specified explicitly by
std::memory_order_seq_cst. Let’s look at a final example:
std::atomic<int> counter = {0};
std::vector<std::thread> vt;
for (int i = 0; i < 100; ++i) {
vt.emplace_back([&](){
counter.fetch_add(1, std::memory_order_seq_cst);
});
}
for (auto& t : vt) {
t.join();
}
std::cout << "current counter:" << counter << std::endl;
This example is essentially the same as the first loose model example. Just change the memory
order of the atomic operation to memory_order_seq_cst. Interested readers can write their own
programs to measure the performance difference caused by these two different memory sequences.

