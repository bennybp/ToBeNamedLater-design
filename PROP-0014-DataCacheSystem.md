# PROP 0014 - Data Cache System

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   |                                           |
| Status:        | In development                            |
 

## Purpose

The data caching system is there to cache large amounts of data that may be reused
in different parts of the calculation. These include integrals, grids, etc.
It is not to be used to share data across modules, and should be designed that
this is impossible.

Each particular module can only access its own data cache. That is, it provides
a place to store data that is expensive to calculate and can possibly reused
later from another instance of the module.


## Structure
 * Stored in and owned by the ModuleLocator
 * When a module is loaded, it is given its cache object by the ModuleLocator
 * May be checkpointed, but this is optional for a given run
 * Should be thread safe


## CacheData object
 * Map of key to generic data (similar to CalcData)
 * Key must be unique for a given:
   * Molecule
   * BasisSet
   * Options
   * Reference state (possible?)

 * Possibility for the key:
   * Equality of molecule, options, basis set, etc
   * Hash of the molecule, basis set, etc 
   * Unique ID for molecule, basis set, etc
     * One ID that is incremented on construction, one local ID that
       is incremented for every change

 * Should be thread safe
