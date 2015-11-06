# PROP 0003 - Exceptions

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   |                                           |
| Status:        | In development                            |
 

## Purpose


## GeneralException
 * Stores a general error string
 * Stores additional information as pairs of strings
 * Additional information can be appended as pairs of strings
   * Compile-time checking that all strings are paired
 * Information can also be appended as a (string, data) pair
   * Data is converted with std::to_string
 * Formats an string for output with `what()` that contains all the
   string pair information and the general error string
