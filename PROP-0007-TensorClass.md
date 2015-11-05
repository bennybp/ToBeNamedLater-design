# PROP 0007 - Tensor Class

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   |                                           |
| Status:        | In development                            |


## Purpose

## Implementation Features
 * A lookup map that allows for obtaining a World object from an MPI Comm (?)

## General Features
 * Wrap existing tensor codes
 * Templated class
   * Data type
   * Tensor rank
     * Some tensor libraries specify rank as a template parameter
     * Allows for compile time checking for some operations (ie, diagonalization of a rank-2)
 * Local and distributed version
 * Data locality generally hidden from the developer, although they can specify
   a preferred storage mechanism.
 * Expose wrapped library mechanisms for operations (use their template expressions or operators)
 * Memory should be somewhat managed


## Rank 1 and 2
 * Plain multiplication. Ie, `A * B` rather than `A["ij"] * B["jk"]`
 * Vector operations
   * norm, cross, dot
