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
  * **When in doubt, follow the Cpp Core Guidelines (https://github.com/isocpp/CppCoreGuidelines)**
  * Capitalization:
    * `snake_case` for functions (both free and member functions)
    * `snake_case` for public typedefs (value_type, etc)
    * `CamelCase` for class names
    * `CamelCase` or a single capital letter for template parameters
    * Inside a function, prefer `snake_case` for local variables, but it is much less important here
  * Private member functions and data should end with a single underscore
  * C++11 has been out for a while. Use it.
    * Do not be afraid to use advanced features as long as they are standard/idiomatic and
      supported by most compilers.
  * Use `const` religiously

  * Use compile-time checking (via `static_assert`) whenever possible
  * Templates are a core part of C++. Use them whenever it makes sense
    * Manually instantiate templates if possible (and use `extern template`)
  * Reduce code duplication
    * Variadic templates and template patterns are ok
    * Copy-and-swap idiom to avoid duplicating code in copy constructor and assignment operator
    * Class member functions that set data - consider copy and move.
  * Be slightly concerned about compile time, but don't excessively worry about it

  * Think about exception safety. Prefer the strong exception safety guarantee when possible
    * **None** - May leak memory. Avoid if at all possible
    * **Basic** - Will not leak memory
    * **Strong** - If exception is thrown, objects are in their original state (including `this`). This is preferred, but not always possible
    * **noexcept** - Will not throw an exception. When possible, do this, and mark the function `noexcept`
    
## Code formatting
  * If using astyle, recommended flags (on command line or in `$HOME/.astylerc`): `--style=allman -s4 -xn -xk -C -w -xw -Y -f -c -S`
    * (above is up for debate)

 
## File management

  * Use standard file extensions
    * `.cpp` and `.hpp` for C++ files
    * `.c` and `.h` for C files
    * `.f90` for modern fortran
    * `.py` for python
  * General rule of thumb is one class or concept per header/source, with the filename being the name of the class
  * No header include guards - use `#pragma once`
  * Wrap C header files with `extern "C"`

```C++
    #pragma once
    // includes go here
    
    #ifdef __cplusplus
    extern "C" {
    #endif

    // code goes here

    #ifdef __cplusplus
    }
    #endif
```


## Includes

  * Include paths should begin with the base path of the project
  * Include only the parts of libraries (if they are exposed by the project)
  * Try to keep the include tree clean - include only what you need, and periodically clean up.


## Compilation

  * Compile with almost all warnings enabled
    * Example for GCC: `-Wall -Wextra -pedantic -Wfloat-equal -Wshadow -Wswitch-default -Wconversion`
    * Example for Intel: `-w3`
    * Some useless remarks can be disabled project-wide
  * Selectively disable warnings for sections of code (see `pragma.h`)


## Pointers

  * Use assertions to check for null pointers (psr_assert_ptr)
  * Use smart pointers whenever possible
    * `std::unique_ptr` has very low or nonexistant overhead in most applications
  * Use `shared_ptr` only if ownership is really shared between objects. Otherwise, use `unique_ptr`
  * Use smart pointers with arrays as well ( `std::unique_ptr<double []>` )
    * (Or just use `std::vector`)
  * Raw pointers coming in as function arguments or being returned from a function should
    always be assumed to be owned by someone else - do not delete/free it
  * Raw pointers as private members of a class are ok, as long as ownership is clear


## Containers

  * Use STL containers
    * `std::vector` for arrays whose length are known only at runtime
    * `std::array` if the length is constant and known at compile time
  * Prefer `std::array` over a fixed-length array (ie, `std::array<3, double>` vs `double[3]`)
    * Very low overhead
    * Provides better copy and move semantics, as well as iterators
  * Consider replacing `std::map` with `std::unordered_map` and `std::multimap` with `std::unordered_multimap`
    if ordering is not important, but speed is
    * Careful - hashes not guarenteed to be the same across program incarnations
    * Otherwise, is basically a drop-in replacement and can result is some decent speedup



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

  * The publicly-accessible API should reside under the `pulsar` namespace
  * Use a `detail` namespace for functions and data that should not be exposed to most developers
  * Use a `lut` namespace for lookup tables and constant global data


## Classes and Members

  * Whenever possible, let the compiler generate constructors, destructors, and assignment for you (via ` = default`)
  * Do not create default constructors (or explicitly delete them) if using it leaves the class in an invalid state
    * Null pointers, etc
  * If possible, create move constructors and assignment as well. Or explicitly let the compiler do it with `= default`
  * There should only be one function that copies from another class (the copy constructor). Use copy-and-swap for the rest
  * Prefer deep copy for copy constructor. Either way, document the behavior
    * Aliasing problems were fixed in C++11 with rvalue semantics
  * Provide iterators if it makes sense
    * Make them 'standard' (derive from STL and use iterator tags?)
  * Protect classes with mutexes if it is expected to be used from multiple threads. Document thread
    safety for all classes and functions.
  * Avoid `protected` data members whenever possible (`protected` member functions are generally ok)
