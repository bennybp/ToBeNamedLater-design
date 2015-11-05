# PROP 0001 - Code Style

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   |                                           |
| Status:        | In development                            |


## General Style
 * C++11 has been out for a while. Use it.
 * Use `const` religiously
 * Use the `deprecated` attribute
   * (Has not been created yet)
 * Use compile-time checking whenever possible
   * static_assert 
 * Use templates, but use them responsibly
 * Reduce code duplication
   * Templates and template patterns
     * Variadic templates
     * Tag dispatch
   * Once and only once
   * Copy-and-swap idiom
 

## File management
 * `.cpp` and `.hpp` for C++ files
 * `.c` and `.h` for C files
 * `.f90` for 'recent' fortran
 * `.py` for python
 * General rule of thumb is one class per header/source, with the filename being the name of the class
 * Header include guards can be fixed to a standard form with `headerguard.py`
 * Include paths should begin with the base path of the project
 * Wrap C header files with 


    #ifdef __cplusplus
    extern "C" {
    #endif

    #ifdef __cplusplus
    }
    #endif


## Compilation
 * Compile with almost all warnings enabled
   * For GCC: `-Wall -Wextra -pedantic -Wfloat-equal -Wshadow -Wswitch-default -Wconversion`
   * For Intel: `-w3`
     * Some useless remarks can be disabled project-wide
 * Selectively disable warnings for sections of code (see `pragma.h`)


## Pointers

 * Use smart pointers whenever possible
 * Use smart pointers from STL rather than boost, unless you have a very good reason
   * Currently, boost::python does not support std::shared_ptr because it does not support move semantics
 * Use shared_ptr only if ownership is really shared between objects. Otherwise, use unique_ptr 
 * Use smart pointers with arrays as well ( std::unique_ptr<double []> ptr )
 * Raw pointers within a class are ok, as long as ownership is clear


## Containers

 * Use STL containers
   * std::vector for arrays whose length are known only at runtime
   * std::array if the length can be fixed at compile time
 * Use std::array over a fixed-length array (ie, std::array<3, double> vs double[3])
   * Copy and move semantics, etc



## Globals
  * Globals should only be used for constant data and lookup tables
  * Can use STL containers for lookup tables (particularly std::map) if performance is not crucial
  * Avoid creating `Init()` for populating lookup tables
    * Can use brace initializers for instantiating complicated global data (ie, std::map)
      * Yes, this works with dynamic module loading
        * Really, it does
    * Do not construct with other global data
    * Use scripts to generate lookup table source files


## Namespaces
  * Separate the project into broad namespaces
  * Use a `detail` namespace for functions and data that should not be exposed to most developers
  * Use a `lut` namespace for lookup tables and constant global data
  * Use an `export_python` namespace for functions related to boost::python exports.


## Classes and Members
  * Whenever possible, let the compiler generate constructors, destructors, and assignment for you
    * Use " = default" and " = delete" for constructors, destructors, assignment
  * If possible, create move constructors and assignment as well
  * Use the copy-and-swap idiom
    * There should only be one function that copies from another class (the copy constructor).
    * Reduces code duplication
  * Provide iterators if it makes sense
  * Member functions should begin with uppercase characters
    * Exception - `begin()`, `end()`, `cbegin()`, `cend()`, `rbegin()`, `rend()`, `crbegin()`, `crend()`.
      Making these lowercase allows for use range-based for loops ( `for(auto & atom : molecule)` is nice )
  * Member data should be private (and, rarely, protected) 
    * Move protected data to private, and make protected methods
  * Use `const` religiously
