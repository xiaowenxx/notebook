# Factors

1. virtual function itself

    needs to look up the virtual function's address at runtime

    - performance: 
        - there is a noticeable overhead for small functions (18%)
        - for the large function, the overhead is negligible

2. vector of pointers

    to activate virtual function mechanism, need to access the object through a pointer or a referenece. accessing objects on the heap can be very slow

    - reason: data cache misses, no guarantee that neighboring pointers will point to neighboring objects in memory
    - alternatives:
        - std::variant with std::visitor
        - polymorphic_vector
        - boost::base_collection

3. compiler optimizations

    virtual functions inhibit compiler optimizations

    - reason: not inlinable
    - solution: **type based processing**
        - don't mix the types, each type has its own container and its own loop
        - boost::base_collection
    - case dependent, smaller functions in principle benefit more

4. jump destination guessing

    the CPU guesses which virtual function will get called, if the guess is wrong, it cost time

    - reason: speculative execution
    - solution: **type based processing**, not always usable
    - most pronounced with short virtual functions

5. instruction cache evictions

    the code that has already been excuted is hot code

    - reason:
        - its instructions are in the instruction cache
        - its branch predictors know the outcome of the branch (true/false)
        - its jump predictors know the target of the jump
    - most likely to occur with large virtual funcitons on mixed-type unsorted vectors with many different derived types

# Conclusion

- virtual functions do not incur too much additional cost by themselves
- it is the environment where they run which determines their speed
- the hardware craves predictablilty: same type, same function, neighboring virtual address

# Improvement

- arrangement of objects in memory is very important
- try to make make small functions non-virtual
- try to keep objects in the vector sorted by type

