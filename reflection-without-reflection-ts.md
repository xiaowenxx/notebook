# Reflection Technique

1. template/function specialization
2. template argument deduction
3. Substitution Failure Is Not An Error
4. std::source_location::function_name
5. structured bindings
6. clang's __builtin_dump_struct

# Reflection Implementation

1. check if a type is a pointer

```C++
template <typename T>
struct is_pointer { static constexpr auto value = false; }
template <typename T>
struct is_pointer<T*> { static constexpr auto value = true; }
template <typename T>
struct is_pointer<T* const> { static constexpr auto value = true; }
template <typename T>
struct is_pointer<T* volatile> { static constexpr auto value = true; }
template <typename T>
struct is_pointer<T* const volatile> { static constexpr auto value = true; }

template <typename T>
struct is_array { static constexpr auto value = false; }
template <typename T, std::size_t N>
struct is_array<T[N]> { static constexpr auto value = true; }
```

2. get the raw array length

```C++
template <std::size_t N>
constexpr std:integral_constant<std::size, N> array_length_helper(auto (&&)[N]);

template <typename T>
static constexpr auto array_length =
    decltype( array_length_helper( std::declval<T>()) )::value;
```

3. check if a type is polymorphic

```C++
template <typename T>
std::true_type check_expression(decltype(sizeof(dynamic_cast<void*>( std::declval<T*>() ))));
template <typename T>
std::false_type check_expression(...);

template <typename T>
static constexpr auto is_polymorphic = decltype(check_expression<T>(0))::value != 0);
```

4. check if a type is class or struct

```C++
// no way, std::is_class<T> is implemented with compiler magic source
```
5. get the name of an enum value as string

```C++
template <auto enumValue>
constexpr std:string_view enum_name() {
  // when enum_name<Color::red>()
  // thisFunctionName = "std::string_view enum_name() [enumValue = Color::red]"
  std::string_view thisFunctionName(std::source_location::current().function_name());
  auto enum_start = thisFunctionName.substr(thisFunctionName.find('=') + 2);
  auto enum_value = enum_start.substr(enum_start.find("::") + 2);
  return enum_value.substr(0, enum_value.size() - 1);
}

// depend on compiler implementation
// use magic_enum library by Daniil Goncharov
```

6. count the number of members

```C++
struct Any {
  template <typename T> operator T() const;
}

template <typename T, std::size_t... Is>
std::true_type check_expression(std::index_sequence<Is...>,
  decltype(sizeof( std::declval<T>() = {(Is, Any())...} )));
template <typename T, std::size_t... Is>  
std::false_type check_expression(std::index_sequence<Is...>, ...);

template <typename T, std::std::size_t N>
static constexpr auto hasAtLeastNMembers =
  decltype(check_expression<T>(std::make_index_sequence<N>(), 0))::value;

template <typename T, std::size_t I =1>
consteval std::size_t countMembers() {
  if constexpr (hasAtLeastNMembers<T, I>)
    return countMembers<T, I + 1>();
  return I - 1;
}
```

7. get the types of a struct's members

```C++
template <typename T>
auto struct_to_tuple_helper(T const& s, std::integral_constant<std::size_t, 1>) {
  auto [a] = s;
  return std::make_tuple(a);
}
//...
template <typename T>
auto struct_to_tuple_helper(T const& s, std::integral_constant<std::size_t, 5>) {
  auto [a, b, c, d, e] = s;
  return std::make_tuple(a, b, c, d, e);
}

template <typename T>
auto struct_to_tuple(T const& s) {
  return struct_to_tuple_helper<T>(s,
    std::integral_constant<std::size_t, countMembers<T>()>());
}
```

```C++
template <typename T, typename Lambda, std::size_t I, typename... U>
struct Any {
  Any(Lambda && l) : lambda(std::move(l)) {}
  
  template <typename V>
  operator V() {
    T t = { U()..., V(), Any<T, Lambda, I-1, U..., V>(std::move(lambda))) };
    return V();
  }
  Lambda lambda;
};

template <typename T, typename Lambda, typename... U>
struct Any<T, Lambda, 1, U...> {
  Any(Lambda && l) : lambda(std::move(l)) {}
  
  template <typename V>
  operator V() {
    lambda(std::type_identity<std::tuple<U..., V>>());
    return V();
  }
  Lambda lambda;
};

template <typename T, typename Lambda>
void reflect_struct(Lambda && lambda) {
  T t = { Any<T, Lambda, countMembers<T>()>(std::move(lambda)) };
}
```

8. get the struct's member names

```C++
//clang only
MyStruct mystruct;
__builtin_dump_struct(&mystruct, [](const char* fmt, auto... args) {
  char buffer[1024];
  std::snprintf(buffer, 1024, fmt, args...);
  // buffer would be like:
  // MyStruct {
  //   int a = 4067
  //   std::string b = *0xffffff
  // }
});
```
