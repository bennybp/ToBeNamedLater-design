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
 * Consideration of nested scenarios such as:
   - User wants fragments made by bondizer and a VMFC computation
   - VMFC needs to add additional fragments to the set bondizer makes
   - VMFC then sets the fragmentizer for MIM to UserDefined with the updated list
   - MIM now runs with UserDefined, what union of options does MIM see?
     * RMR- I think MIM should see the options bondizer was set with except, when
       VMFC had to modify them (i.e. truncation_order is now 1 so we don't form dimers etc.)
 
