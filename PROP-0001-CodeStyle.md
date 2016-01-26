# PROP 0001 - Code Style

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   |                                           |
| Status:        | In development                            |


## General Style
 * C++11 has been out for a while. Use it.
 * Use `const` religiously
 * Use compile-time checking whenever possible
   * `static_assert`
 * Use templates, but use them responsibly
   * Manually instantiate templates if possible
   * Think about `extern template`
 * Reduce code duplication
   * Templates and template patterns
     * Variadic templates
     * Tag dispatch
   * Copy-and-swap idiom to avoid duplicating code in copy constructor and assignment operator
 * Use the `deprecated` attribute if needed
   * (Part of C++ 14, will need a wrapper until then)
 

## File management
 * Use standard file extensions
   * `.cpp` and `.hpp` for C++ files
   * `.c` and `.h` for C files
   * `.f90` for modern fortran
   * `.py` for python
 * General rule of thumb is one class or concept per header/source, with the filename being the name of the class
 * Header include guards can be fixed to a standard form with `headerguard.py`
 * Wrap C header files with 

```C++
    #ifdef __cplusplus
    extern "C" {
    #endif

    #ifdef __cplusplus
    }
    #endif
```

## Includes
 * Include paths should begin with the base path of the project
 * Include only the parts of boost that you need
   * Do not include `boost/python.hpp`.
     Instead, include only what you need
     * `<boost/python/list.hpp>`
     * `<boost/python/dict.hpp>`
     * `<boost/python/extract.hpp>`
     * etc...
  * Periodically clean up the include tree


## Compilation
 * Compile with almost all warnings enabled
   * Example for GCC: `-Wall -Wextra -pedantic -Wfloat-equal -Wshadow -Wswitch-default -Wconversion`
   * Example for Intel: `-w3`
     * Some useless remarks can be disabled project-wide
 * Selectively disable warnings for sections of code (see `pragma.h`)


## Pointers
 * Use smart pointers whenever possible
   * `std::unique_ptr` has very low or nonexistant overhead in most applications
 * Use smart pointers from STL rather than boost, unless you have a very good reason
   * Currently, boost::python does not support std::shared_ptr because it does not support move semantics
 * Use `shared_ptr` only if ownership is really shared between objects. Otherwise, use `unique_ptr`
 * Use smart pointers with arrays as well ( `std::unique_ptr<double []>` )
   * (Or just use `std::vector`)
 * Raw pointers within a class are ok, as long as ownership is clear


## Containers
 * Use STL containers
   * `std::vector` for arrays whose length are known only at runtime
   * `std::array` if the length is constant and known at compile time
 * Prefer `std::array` over a fixed-length array (ie, `std::array<3, double>` vs `double[3]`)
   * Very low overhead
   * Provides better copy and move semantics, as well as iterators



## Globals
  * Globals should only be used for constant data and lookup tables
  * Can use STL containers for lookup tables (particularly `std::map`) if performance is not crucial
  * Avoid creating `Init()` for populating lookup tables
    * Can use brace initializers for instantiating complicated global data (ie, `std::map` of structures)
      * Yes, this works with dynamic module loading
        * Really, it does
    * Use scripts to generate lookup table source files
    * Do not construct global data with other global data
      * Global data is constructed in an indeterminant order



## Namespaces
  * Separate the project into broad namespaces
  * Use a `detail` namespace for functions and data that should not be exposed to most developers
  * Use a `lut` namespace for lookup tables and constant global data
  * Use an `export_python` namespace for functions related to boost::python exports.


## Classes and Members
  * Whenever possible, let the compiler generate constructors, destructors, and assignment for you
    * Use ` = default` and ` = delete` for constructors, destructors, assignment
  * If possible, create move constructors and assignment as well. Or explicitly let the compiler do it with `= default`
  * Use the copy-and-swap idiom
    * There should only be one function that copies from another class (the copy constructor).
  * Provide iterators if it makes sense
    * Make them 'standard' (derive from STL?)
  * Member functions should begin with uppercase characters
    * Exception - `begin()`, `end()`, `cbegin()`, `cend()`, `rbegin()`, `rend()`, `crbegin()`, `crend()`.
      Making these lowercase allows for use range-based for loops ( `for(auto & atom : molecule)` is nice )
  * Member data should be private (and, rarely, protected) 
    * Move protected data to private, and make protected methods
  * Use `const` religiously
  * Think about exception safety. Prefer the strong exception safety guarantee
    * **None** - May leak memory. Avoid if at all possible
    * **Basic** - will not leak memory
    * **Strong** - If exception is thrown, object is in its original state. This is preferred
    * **noexcept** - Will not throw an exception. When possible, do this, and mark the function `noexcept`
