# PROP 0016 - Data Cache System

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   |                                           |
| Status:        | In development                            |
 

## Purpose

The data caching system is there to cache large amounts of data that may be reused
in different parts of the calculation, but through the same interfaces. Essentially, the intent is that this will be used something like:
HF asks 1eints module for the one-electron integrals and gives the 1eints module the molecule, basis set, and options objects. The 1eints module looks at the passed in objects and says "I already computed integrals for this exact combination" and then immediatly returns the cached integrals.  It is the responsibility of the 1eints module to determine if the passed in objects are identical to a combination it has seen before.  This leads to code design where everytime you want 1eints, you call the 1eints module, which then returns the integrals to you, as fast as possible, which may involve returning a cached result.

Prime targets for the cache mechanism are quantities that are computationally expensive to compute, but small enough to fit in memory.  The cache system is technically a sort of global buffer, but the interface all modules must go through will make it impossible for it to be used for communiction, which prevents the normal global variable problems.  Basically, each particular module can only access its own data cache and it is guaranteed that is in the state you last left it in, modulo clean-up of any files you marked as deletable. Again this is a place to store data that is expensive to calculate and can possibly reused
later from another instance of the module.


## Structure
 * Stored in and owned by the ModuleLocator
 * When a module is loaded, it is given its cache object by the ModuleLocator
 * May be checkpointed, but this is optional for a given run
 * Should be thread safe
 * Data must be serializable!!!!!


## CacheData object
 * Map of key to generic data (similar to CalcData)
 * Key must be unique for a given:
   * Molecule
   * BasisSet
   * Options
   * and whatever other things may impact the result

 * Possibility for the key:
   * Equality of molecule, options, basis set, etc
   * Hash of the molecule, basis set, etc 
   * Unique ID for molecule, basis set, etc
     * One ID that is incremented on construction, one local ID that
       is incremented for every change

 * Should be thread safe
