# template type deduction

```C++
int x = 27; f(x);
const int cx = x;
const int& rx = x;
const int* px = &x;
const char arr[] = "J. P. Briggs";
void fun(int, double);

// Reference or Pointer, Not Universial Reference
template<typename T> void f(T& params);
f(x);   // T is int, param's type is int&
f(cx);  // T is const int, param's type is const int&
f(rx);  // T is const int, param's type is const int&
f(arr); // T is const char[13], param's type is const char (&)[13]
f(fun); // param's type is void (&)(int, double)

template<typename T> void f(const T& params);
f(x);  // T is int, param's type is const int&
f(cx); // T is int, param's type is const int&
f(rx); // T is int, param's type is const int&

template<typename T> void f(T* params);
f(&x); // T is int, param's type is int*
f(px); // T is const int, param's type is const int*

// Universial Reference
template<typename T> void f(T&& params);
f(x);  // lvalue, T is int&, param's type is int&
f(cx); // lvalue, T is const int&, param's type is const int&
f(rx); // lvalue, T is const int&, param's type is const int&
f(27); // rvalue, T is int, param's type is int&&

// Neither Reference nor Pointer
template<typename T> void f(T params);
f(x);   // T is int, param's type is int
f(cx);  // T is int, param's type is int
f(rx);  // T is int, param's type is int
f(arr); // T is const char*
f(fun); // param's type is void (*)(int, double)

```

# auto type deduction

```C++
auto x = 27; // int
const auto cx = x;  // const int
const auto& rx = x; // const int&
auto&& uref_x = x;   // lvalue, int&
auto&& uref_cx = cx; // lvalue, const int&
auto&& uref_27 = 27; // rvalue, int&&

const char arr[] = "R. N. Briggs";
auto arr1 = arr; // const char*
auto& arr2 = arr; // const char (&)[13]

void func(int, double);
auto func1 = func;  // void (*)(int, double)
auto& func2 = func; // void (&)(int, double)

auto x1 = 27; // int
auto x2(27); // int
auto x3 = { 27 }; // std::initializer_list<int>
auto x4{ 27 }; // std::initializer_list<int>
auto x5 = { 1, 2, 3.0 } // error! can't deduce T for std::initializer_list<T>

template<typename T> void f1(T param);
f1({ 1, 2, 3 }); // error! can't deduce type for T
template<typename T> void f2(std::initializer_list<T> param);
f2({ 1, 2, 3 }); // T is int, param's type is std::initializer_list<int>

auto f1() { return { 1, 2, 3 }; } // error! can't deduce type for {1,2,3}
auto f2 = [](const auto& param) = {};
f2({ 1, 2, 3 });  // error! can't deduce type for {1,2,3}
```

# decltype

```C++
template<typename C>
auto f1(C& c) -> decltype(c[0]) {
  return c[0]; // return type T&
}
template<typename C>
auto f2(C& c) {
  return c[0]; // return type is T
}
template<typename C>
decltype(auto) f3(C& c) {
  return c[0]; // return type is T&
}

int x = 27; f(x);
const int& rx = x;
auto ax = rx; // int
decltype(auto) dx = rx; // const int&

decltype(auto) f1() {
  int x = 0;
  return x; // return type int
}
decltype(auto) f2() {
  int x = 0;
  return (x); // return type is int&
}
```
