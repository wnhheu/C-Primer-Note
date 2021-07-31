# Chapter 2

##　Type

wchar_t/char16_t/char32_t

>wchar的大小可以包括所有的字符（我的理解是因为这个它的大小是不确定的
>
>char16_t和char32_t分别表示16个bit和32个bit的char类型，他们各自对应一个名称的标准（像汉语、日语之类的）  

char

> char并没有一个统一的标准，可能是signed char，也可能是unsigned char，视机器而定（可能的代码漏洞，使用时要明确）

long 和 int的大小可能是一样的，如果需要更大的表示范围，用long long

如果数不为负，用unsigned

## Conversion

others->bool: others = 0 ？ bool = false : bool = true;

float->integer: store the part before decimal part

integer->float: fractional part is zero, potential risk of losing accuracy

out-of-range->unsigned: out-of-range % range

out-of-range->signed: undefined

为了增加程序的可移植性，应该避免未定义的或者在执行中定义的行为（像int的size是编译器而定的）

literal--its value is self-evident

### prefix and suffix

#### Integer literals

`int x = 0x100u`

> 这里的u作为suffix表示0x100是一个unsigned类型
>
> l-long ll-long long

#### Character and Character String Literals

`char ch = u'p'`

> 这里的u作为prefix表示p是一个Unicode 16 character(char16_t)
>
> U - Unicode 32 character(char32_t)
>
> L - wide character(wchar_t)
>
> u8 - utf-8(char)

#### Floating-Point Literals

`double d = 3.14f`

>  这里的 f / F 作为suffix表示3.14是一个float类型
>
> l / L - long double

---

single quote -- char

double quotation -- string literal

```C++
// multiline string literal
std::cout << "a really, really long string literal"
    		"that spans two lines" << std::endl;
// this is considered to be a single string literal
```

### List Initialization

```c++
int a = {0}
struct A
{
    int x;
    int y;
}a = {1, 2};
// is_ok a.x = 1, a.y = 2
struct B
{
    int x;
    int y;
    B()
    {
        x = 0;
        y = 0;
    }
}b = {1, 2};
// error b.x = 0, b.y = 0
// https://blog.csdn.net/janeqi1987/article/details/78623691
// 还不是很清楚列表初始化的意义是什么
```

当列表初始化可能导致数据精度的丢失或异常时，编译器会报错

###　Default Initialization

定义在全局变量中的variable会被初始化为0

在函数中定义的变量如果没有初始化函数就没有初始化

### declaration or definition

- declaration
  - A declaration makes a name known to the program  
  - A variable declaration specifies the type and name of a variable  
- definition
  - A definition creates the associated entity  
  - it also allocates storage and may provide the variable with an initial value  

#### extern -- obtain a declaration that is not also a definition  

``` c++
extern int i;	// declares but does not define i
int j;			// declares and defines j
```

If your declaration includes an explicit initializer, it will overrides extern

``` c++
extern double pi = 3.14	// definition
```

Variables must be defined exactly once but can be declared many times.  

It is important in **separate compilation**

### Conventions for Variable Names

- An identifier should give some indication of its meaning.

- Variable names normally are lowercase—index, not Index or INDEX.

- Like Sales_item, classes we define usually begin with an uppercase letter.

- Identifiers with multiple words should visually distinguish each word, for
  example, student_loan or studentLoan, not studentloan.  

Advice: define variables where you first use them

### reference and pointer

**declarator -- & / ***

#### Reference

``` c++
int ival = 1024;
int &refVal = ival;	// refVal refers to(it another name for) val
int &refval2;	// error, a reference must be initialized
```

**Reference is bind to its initializer and cannot be rebound.**

#### Pointer

```c++
int *ip1, *ip2;		// ip1 and ip2 are pointers to int
double dp, *dp2;	// dp2 is a pointer to double, dp is a double
```

**A pointer holds the address of another object**

**We use address-of operator(& operator) to get the address of an object**

**We can access the object through dereference operator(\*operator)**

```C++
int ival = 42;
int *p = &ival;
std::cout << *p;	// prints 42
*p = 0;
std::cout << *p;	// ival = 0, prints 0
```

##### nullptr

```C++
int *p1 = nullptr;
int *p2 = 0;
// include cstdlib
int *p3 = NULL;		// a preprocessor variable
```

##### void *

a void* pointer holds an address, but the type of the object at that address is unknown

We use a void* pointer to deal with memory as memory, rather than using the pointer to access the object stored in that memory.  

---

**A reference is not an object. Hence, we may not have a pointer to a reference.
However, because a pointer is an object, we can define a reference to a pointer  **

```C++
int i = 42;
int *p;
int *&r = p;
r = &i;
*r = 0;			// i = 0;
```

### Const Qualifier

- **const -- value cannot be changed**
  - it must be initialized when defined
  - const variables are defined as local to the file
    - 因为在编译中，const通常会被直接替换成它所代表的值，所以要在这个文件中知道const的值是在哪里被初始化的
  - if you want to apply one const object to other files, you can use **extern**

#### reference to const

```c++
const int ci = 1024;
const int &r1 = ci; // ok: both reference and underlying object are const
r1 = 42; 			// error: r1 is a reference to const
int &r2 = ci; 		// error: non const reference to a const object
```

**There are no const references. Considering a reference is not an object, we cannot make it const**

```C++
// We can initialize a reference to const from any expression that can be converted to the type of the reference
int i = 42;
const int &r1 = i; 		// we can bind a const int& to a plain int object
const int &r2 = 42; 	// ok: r1 is a reference to const
const int &r3 = r1 * 2; // ok: r3 is a reference to const
int &r4 = r * 2; 		// error: r4 is a plain, non const reference
```

You can understand the rule through this illustration

``` c++
double dval = 3.14;
const int &ri = dval;
// ri refers to an int, while dval is a floating-point integer
// so the compiler transform this code into something like:
const int temp = dval;
const int &ri = temp;
// ri is bound to a temporary object, which is created by the conpiler when it needs a place to store a result from evaluating an expression
// if ri is not const, it will refer to temp rather than dval and it's not what the programmer intended
```

Two interesting examples by zty and c++ primer

```C++
// example 1
int main()
{
    double i = 42;
    double &r1 = i;
    const int &r2 = i;
    std::cout << r2 << ' ';
    r1 = 0;
    std::cout << r2;
    return 0;
}	// prints 42 42
```

```C++
// example 2
int main()
{
    int i = 42;
    int &r1 = i;
    const int &r2 = i;
    std::cout << r2 << '\n';
    r1 = 0;
    //r2 = 0	//error, r2 is a reference to const
    std::cout << r2;
    return 0;
}	// print 42 0
```

从example 2我们可以看到，执行完const int &r2 = i  这条指令，实际上r2是i的一个别名，但是对于r2来说，它的对象的值是不可以改变的（但是事实上i的值是可以改变的,就是下面这段话）

> Like a reference to const, a pointer to const says nothing about whether the
> object to which the pointer points is const. Defining a pointer as a pointer to const
> affects only what we can do with the pointer.  

从example 1我们可以看到，const int &r2 = i  这条指令，实际上r2是temp的一个别名，当i改变的时候，r2不变

####　Pointer and Const

``` C++
// a pointer to const
const double pi = 3.14; // pi is const; its value may not be changed
double *ptr = &pi; // error: ptr is a plain pointer
const double *cptr = &pi; // ok: cptr may point to a double that is const
*cptr = 42; // error: cannot assign to *cptr
```

```C++
// const pointer
int errNumb = 0;
int *const curErr = &errNumb; // curErr will always point to errNumb
const double pi = 3.14159;
const double *const pip = &pi; // pip is a const pointer to a const
object
```

##### top-level const && low-level const

it means the pointer itself is a const

when a pointer can point to a const object, we refer to the const as a low-level const

