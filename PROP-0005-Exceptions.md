# PROP 0005 - Exceptions

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | How we handle errors at runtime           |
| Status:        | In development                            |
 

## Purpose

It is important that code try to determine if input will lead to
undesirable results or requests/requires features and capabilities that
do not exist yet. Instead of just crashing, exceptions let the calling
code know this in a graceful manner and give it the chance to do something
about it.

## GeneralException

 * Stores a general error string
 * Stores additional information as pairs of strings
 * Additional information can be appended as pairs of strings
   * Compile-time checking that all strings are paired
 * Information can also be appended as a (string, data) pair
   * Data is converted with `std::to_string`
 * Formats an string for output with `what()` that contains all the
   string pair information and the general error string

## Derived Classes

Classes derived from `GeneralException` are used to give semantic meaning
to some exceptions. They are generally similar to `GeneralException`, although
constructors and helper functions may assist in handling common key strings. 


## Assert

This is a debug check.  When the project is compiled in release mode,
these checks will not be made. Code for this is included in the assert
class and should not be replicated by the user. The syntax is as follows:

```C++
exception::assert<T>(condition, message, ...) 
```

  - `T` is the type of exception to throw if the assertion fails
  - `condition` is what to check, if it is false the assertion fails
  - `message` will form the bulk of the `what()` of the thrown exception
  - `...` optional pairs of strings following the GeneralException convention
