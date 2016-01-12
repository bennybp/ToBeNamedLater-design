# PROP 0017 - Math Utilities

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   |                                           |
| Status:        | In development                            |
 

## Purpose
 * Most vector, matrix, and tensor math should be in the tensor class / namespace, not this one.

## Features
 * Casting between types
 * Exact cast - no loss of precision
   * Cast between integer types or between floating point types only
   * Will check for underflow, overflow, and loss of precision
 * Round cast - Loss of precision ok
   * Can cast integer to floating point and vice versa
 * Physical and mathematical constants
 * Unit conversions
 * Use of templates for efficient code
 * When possible, checks are done at compile time

## Components
 * Permutation generators
 * Combination generators
 * Power set generators
 * Finite difference
 * Factorials
 * Double factorials
 * Binomial coefficients
 * Graph library (?)
