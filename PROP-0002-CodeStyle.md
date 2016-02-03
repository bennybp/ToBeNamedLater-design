# PROP 0002 - Code Style

|                |                                            |
|:---------------|:-------------------------------------------|
| Description:   | General coding guidelines and coding style |
| Status:        | In development                             |

## Purpose

To provide some general guidelines and conventions for use within the
project. Although they are not written in stone, following these can
lead to more readable and maintainable code.


## General Style

  * C++11 has been out for a while. Use it.
    * Do not be afraid to use advanced features as long as they are standard and
      supported by most compilers.
  * If using astyle, recommended flags (on command line or in `$HOME/.astylerc`): `--style=allman -s4 -xn -xk -C -w -xw -Y -f -c -S`
    * (above is up for debate)
  * Use `const` religiously
  * Use compile-time checking whenever possible
    * `static_assert`
  * Use templates, but use them responsibly
    * Manually instantiate templates if possible
    * Think about using `extern template`
  * Reduce code duplication
    * Templates and template patterns
      * Variadic templates
      * Tag dispatch
    * Copy-and-swap idiom to avoid duplicating code in copy constructor and assignment operator
  * Use the `deprecated` attribute if needed
    * (Part of C++ 14, will need a wrapper until then)
  * Think about exception safety. Prefer the strong exception safety guarantee
    * **None** - May leak memory. Avoid if at all possible
    * **Basic** - Will not leak memory
    * **Strong** - If exception is thrown, objects are in their original state (including `this`). This is preferred, but not always possible
    * **noexcept** - Will not throw an exception. When possible, do this, and mark the function `noexcept`
 

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
  * Include only the parts of libraries (such as boost) that you need
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
  * Consider replacing `std::map` with `std::unordered_map` and `std::multimap` with `std::unordered_multimap`
    if ordering is not important (is basically a drop-in replacement and can result is some decent speedup).



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
  * Use an `export_python` namespace for functions related to exporting of functionality to python


## Classes and Members

  * Whenever possible, let the compiler generate constructors, destructors, and assignment for you
    * Use ` = default` and ` = delete` for constructors, destructors, assignment
  * Do not create default constructors (or explicitly delete them) if using it leaves the class in an invalid state
    * Null pointers, etc
  * If possible, create move constructors and assignment as well. Or explicitly let the compiler do it with `= default`
  * Use the copy-and-swap idiom
    * There should only be one function that copies from another class (the copy constructor).
  * Prefer deep copy for copy constructor. Either way, document the behavior
    * Aliasing problems were fixed in C++11 with rvalue semantics
  * Provide iterators if it makes sense
    * Make them 'standard' (derive from STL?)
  * Member functions should begin with uppercase characters
    * Exception - `begin()`, `end()`, `cbegin()`, `cend()`, `rbegin()`, `rend()`, `crbegin()`, `crend()`.
      Making these lowercase allows for use range-based for loops ( `for(auto & atom : molecule)` is nice )
  * Member data should be private (and, rarely, protected) 
    * Move protected data to private, and make protected methods
  * Use `const` religiously
