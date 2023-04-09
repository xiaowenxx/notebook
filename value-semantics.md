# Value/Reference Semantics
* With reference semantics, assignment is a pointer-copy (i.e., a reference).
> Pros of reference semantics: flexibility and dynamic binding (you get dynamic binding in C++ only when you pass by pointer or pass by reference, not when you pass by value).
* Value (or “copy”) semantics mean assignment copies the value, not just the pointer.
> Pros of value semantics: speed. “Speed” seems like an odd benefit for a feature that requires an object (vs. a pointer) to be copied, but the fact of the matter is that one usually accesses an object more than one copies the object, so the cost of the occasional copies is (usually) more than offset by the benefit of having an actual object rather than a pointer to an object.

# Prefer value semantics over reference semantics
* easiser to understand (less code)
* eariser to write
* more correct (avoid many common bugs)
* potentially faster

# Reference Semantics Bugs:
```c++
#include <algorithm>
#include <vector>

// template< class ForwardIt, class T >
// ForwardIt remove( ForwardIt first, ForwardIt last, const T& value );
    
int main()
{
    std::vector<int> vec{ 1, -3, 27, 42, 4, -8, 22, 42, 37, 4, 18, 9 };

    auto const pos = std::max_element(begin(vec), end(vec)); // Determining the maximum element
    vec.erase(std::remove(begin(vec), end(vec), *pos), end(vec)); // Removing all maximum elements
    
    // remove function accept value as reference parameter
    print(vec); // 1, -3, 27, 4, -8, 22, 42, 37, 18, 9; // error: 42 and 4 are removed
}
```
