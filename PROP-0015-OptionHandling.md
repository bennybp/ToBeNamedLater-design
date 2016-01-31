# PROP 0015 - Option Handling

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   |                                           |
| Status:        | In development                            |
 

## Purpose

## Requirements
 * Allow the module developer to use custom validators
 * OptionMap class will handle conversions between types
   when appropriate (between integers, between floating point, etc).
   This casting should be exact, with no loss of precision or
   rounding errors. 

## Remarks
 * Options should be handled only by the core library
 * By the time the module is created, the options should have
   been checked and validated
 
