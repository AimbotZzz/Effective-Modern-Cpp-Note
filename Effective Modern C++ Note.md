## Chapter 1: Deducing Types
### Item1: Type Deduction
#### Case 1: `ParamType` is a Reference or Pointer, but not a Universal Reference

```
template<typename T>
void f(T& param);
```
```
int x = 27;
const int cx = x;
const int& rx = x;

f(x);                   // T is int, param's type is int&

f(cx);                  // T is const int,
						// param's type is const int&

f(rx);                  // T is const int,
                        // param's type is const int&
```


```
template<typename T>
void f(T* param);        // param is now a pointer

int x = 27;              // as before
const int *px = &x;      // px is a ptr to x as a const int

f(&x);                   // T is int, param's type is int*

f(px);                   // T is const int,
                         // param's type is const int*
```

#### Case 2: `ParamType` is a Universal Reference
```
template<typename T>
void f(T&& param);       // param is now a universal reference

int x = 27;              // as before
const int cx = x;        // as before
const int& rx = x;       // as before

f(x);                    // x is lvalue, so T is int&,
                         // param's type is also int&

f(cx);                   // cx is lvalue, so T is const int&,
                         // param's type is also const int&

f(rx);                   // rx is lvalue, so T is const int&,
                         // param's type is also const int&

f(27);                   // 27 is rvalue, so T is int,
                         // param's type is therefore int&&
```

#### Case 3: `ParamType` is Neither a Pointer nor a Reference
```
template<typename T>
void f(T param);         // param is now passed by value

int x = 27;          // as before
const int cx = x;    // as before
const int& rx = x;   // as before

f(x);                // T's and param's types are both int

f(cx);               // T's and param's types are again both int

f(rx);               // T's and param's types are still both int
```

```
template<typename T>
void f(T param);         // param is still passed by value

const char* const ptr =  // ptr is const pointer to const object
  "Fun with pointers";

f(ptr);                  // pass arg of type const char * const
```

##### Array Arguments
```
const char name[] = "J. P. Briggs";  // name's type is
                                     // const char[13]

const char * ptrToName = name;       // array decays to pointer
```

If an array is passed to a template taking a by-value parameter:
```
template<typename T>
void f(T param);      // template with by-value parameter

f(name);             // name is array, but T deduced as const char*
```
If we modify the template `f` to take its argument by reference, and we pass an array to it,
```
template<typename T>
void f(T& param);      // template with by-reference parameter
f(name);               // pass array to f
```
the type deduced for `T` is the actual type of the array. In this example, `T` is deduced to be `const char [13]`, and the type of `f`'s parameter (a reference to this array) is `const char (&)[13]`.

##### Function Arguments
```
void someFunc(int, double);   // someFunc is a function;
                              // type is void(int, double)

template<typename T>
void f1(T param);             // in f1, param passed by value

template<typename T>
void f2(T& param);            // in f2, param passed by ref

f1(someFunc);                 // param deduced as ptr-to-func;
                              // type is void (*)(int, double)

f2(someFunc);                 // param deduced as ref-to-func;
                              // type is void (&)(int, double)
```

### Item 2: `auto` type deduction.
```
auto x = 27;
const auto cx = x;
const auto& rx = x;
```

In cases 1 and 3 in **Item 1**:
```
auto x = 27;          // case 3 (x is neither ptr nor reference)

const auto cx = x;    // case 3 (cx isn't either)

const auto& rx = x;   // case 1 (rx is a non-universal ref.)
```
and case 2:
```
auto&& uref1 = x;     // x is int and lvalue,
                      // so uref1's type is int&

auto&& uref2 = cx;    // cx is const int and lvalue,
                      // so uref2's type is const int&

auto&& uref3 = 27;    // 27 is int and rvalue,
                      // so uref3's type is int&&
```
Array and function names decay into pointers for non-reference type specifiers, which also happens in `auto` type deduction:
```
const char name[] =            // name's type is const char[13]
  "R. N. Briggs";

auto arr1 = name;              // arr1's type is const char*

auto& arr2 = name;             // arr2's type is
                               // const char (&)[13]

void someFunc(int, double);    // someFunc is a function;
                               // type is void(int, double)

auto func1 = someFunc;         // func1's type is
                               // void (*)(int, double)

auto& func2 = someFunc;        // func2's type is
                               // void (&)(int, double)
```

`int` vs `std::initializer_list<int>`:
```
auto x1 = 27;             // type is _int_, value is 27

auto x2(27);              // ditto

auto x3 = { 27 };         // type is _std::initializer_list<int>_,
                          // value is { 27 }

auto x4{ 27 };            // ditto

auto x5 = { 1, 2, 3.0 };  // error! can't deduce T for
                          // std::initializer_list<T>
```
If the corresponding template is passed the same initializer, type deduction fails, and the code is rejected:
```
auto x = { 11, 23, 9 };   // x's type is
                          // std::initializer_list<int>

template<typename T>      // template with parameter
void f(T param);          // declaration equivalent to
                          // x's declaration

f({ 11, 23, 9 });         // error! can't deduce type for T
```

However, if you specify in the template that `param` is a `std::initializer_list<T>` for some unknown `T`, template type deduction will deduce what `T` is:
```
template<typename T>
void f(std::initializer_list<T> initList);

f({ 11, 23, 9 });         // T deduced as int, and initList's
                          // type is std::initialize
```
A function with an `auto` return type that returns a braced initializer won't compile:
```
auto createInitList()
{
  return { 1, 2, 3 };         // error: can't deduce type
}                             // for { 1, 2, 3 }
```
The same is true when `auto` is used in a parameter type specification in a C++14 lambda:
```
std::vector<int> v;
…

auto resetV =
  [&v](const auto& newValue) { v = newValue; };     // C++14

…

resetV({ 1, 2, 3 });          // error! can't deduce type
                              // for { 1, 2, 3 }
```

### Item 3: `decltype`.
```
template<typename Container, typename Index>    // C++14;
auto authAndAccess(Container& c, Index i)       // not quite
{                                               // correct
  authenticateUser();
  return c[i];                  // return type deduced from c[i]
}
```
Wrong:
```
std::deque<int> d;
…
authAndAccess(d, 5) = 10;  // authenticate user, return d[5],
                           // then assign 10 to it;
                           // this won't compile!
```
Correct but not perfect:
```
template<typename Container, typename Index>   // C++14; works,
decltype(auto)                                 // but still
authAndAccess(Container& c, Index i)           // requires
{                                              // refinement
  authenticateUser();
  return c[i];
}
```
Refinement (c++14):
A client might simply want to make a copy of an element in the temporary container, for example:
```
std::deque<std::string> makeStringDeque();   // factory function

// make copy of 5th element of deque returned
// from makeStringDeque
auto s = authAndAccess(makeStringDeque(), 5);
```
Supporting such use means we need to revise the declaration for `authAndAccess` to accept both lvalues and rvalues. Overloading would work, but then we'd have two functions to maintain. To avoid this, use universal references. Therefore,
```
template<typename Container, typename Index>    // c is now a
decltype(auto) authAndAccess(Container&& c,     // universal
                             Index i);          // reference
```
Then, we need to update the template's implementation to apply `std::forward` to universal reverences: (**final c++14 version**)
```
template<typename Container, typename Index>       // final
decltype(auto)                                     // C++14
authAndAccess(Container&& c, Index i)              // version
{
  authenticateUser();
  return std::forward<Container>(c)[i];
}
```
The use of `decltype(auto)` can also be convenient for declaring variables when you want to apply the `decltype` type deduction rules to the initializing expression:
```
Widget w;

const Widget& cw = w;

auto myWidget1 = cw;             // auto type deduction:
                                 // myWidget1's type is _Widget_

decltype(auto) myWidget2 = cw;   // decltype type deduction:
                                 // myWidget2's type is
                                 // const Widget&
```

If `int x = 0;`, then `decltype(x)` is `int`, while `decltype((x))` is `int&` since C++ defines `(x)` to be an lvalue. Then, together with `decltype(auto)`, we have:
```
decltype(auto) f1()
{
  int x = 0;
  …
  return x;        // decltype(x) is int, so f1 returns int
}

decltype(auto) f2()
{
  int x = 0;
  …
  return (x);      // decltype((x)) is int&, so f2 returns int&
}
```

### Item 4: View deduced types
#### Compiler Diagnostics
A generally effective way to get a compiler to show a type it has deduced is to use that type in a way that leads to compilation problems. The error message reporting the problem nearly always mentions the type that’s causing it.
```
template<typename T>       // declaration only for TD;
class TD;                  // TD == "Type Displayer"

const int theAnswer = 42;

auto x = theAnswer;
auto y = &theAnswer;

TD<decltype(x)> xType;     // elicit errors containing
TD<decltype(y)> yType;     // x's and y's types
```
Error message:
```
error: aggregate 'TD<int> xType' has incomplete type and
    cannot be defined
error: aggregate 'TD<const int *> yType' has incomplete type
    and cannot be defined
```
or:
```
error: 'xType' uses undefined class 'TD<int>'
error: 'yType' uses undefined class 'TD<const int *>'
```
both of which show the types.

#### Runtime Output
`std::type_info::name` does not always work.
If do need to rely on libraries, plz use the Boost TypeIndex library.
Example:
```
#include <boost/type_index.hpp>

template<typename T>
void f(const T& param)
{
  using std::cout;
  using boost::typeindex::type_id_with_cvr;

  // show T
  cout << "T =     "
       << type_id_with_cvr<T>().pretty_name()
       << '\n';

  // show param's type
  cout << "param = "
       << type_id_with_cvr<decltype(param)>().pretty_name()
       << '\n';
  …
}
```

## Chapter 2. `auto`
### Item 5: Prefer `auto` to explicit type declarations.

The `std::function` approach is generally bigger and slower than the `auto` approach, and it may yield out-of-memory exceptions, too, i.e., `auto` is more efficient and requires less space.

There is a common example:
```
std::vector<int> v;
…
unsigned sz = v.size();
```
However, the official return type of `v.size()` is `std::vector<int>::size_type`, but few developers are aware of that, which act different in 64-bit Windows. Using `auto` ensures that you don’t have to spend time dealing with the related issues:
```
auto sz = v.size();  // sz's type is std::vector<int>::size_type
```
Another example:
```
std::unordered_map<std::string, int> m;
…

for (const std::pair<std::string, int>& p : m)
{
  …                   // do something with p
}
```
Use `auto`: 
```
for (const auto& p : m)
{
  …                             // as before
}
```
This is not only more efficient, it’s also easier to type. Also, if you use `auto` as the type of the target variable, you need not worry about mismatches between the type of variable you’re declaring and the type of the expression used to initialize it.

**Note:** `auto` is an option, not a mandate. If, in your professional judgment, your code will be clearer or more maintainable or in some other way better by using explicit type declarations, you’re free to continue using them. Although some developers are disturbed by the fact that using `auto` eliminates the ability to determine an object’s type by a quick glance at the source code. However, IDEs’ ability to show object types often mitigates this problem, and, in many cases, a somewhat abstract view of an object’s type is just as useful as the exact type. It often suffices, for example, to know that an object is a container or a counter or a smart pointer, without knowing exactly what kind of container, counter, or smart pointer it is. Assuming well-chosen variable names, such abstract type information should almost always be at hand.