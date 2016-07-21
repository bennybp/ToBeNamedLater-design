# PROP 0015 - Option Handling

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | How Pulsar handles options                |
| Status:        | In development                            |
 

## Purpose

## Terminology
 * Each option is accessed via an object known as the `option key`
   * Tentatively the option key is simply a string written in MACRO_CASE
 * The value returned by the option key is known as `the option`
 * A `validator` is a Python function that takes an option and returns `True` if the value is valid and `False` otherwise
 * An option is a `required` option if the user must specify a value for it in order for the module to run

## Requirements
 * Each module must provide a dictionary of options (it may be empty) in its `modinfo.py` file
   * The keys of this dictionary should be the option keys themselves
   * A module may only use option keys contained within this dictionary
   * Each key in the dictionary must map to a tuple with the form: `(Type,Default Value,Is Required, Validators, Description)`
     * `Type` must be one of the enumerations contained in the Pulsar Datastore OptionType enumeration
     * `Default` may be `None` or a value of type `Type`
     * `Is Required` is a boolean that is true if the option is a required option
     * `Validators` is a list(?) of validators or `None`
     * `Description` is a string describing what the option does in detail
 * Allow the module developer to use custom validators
 * OptionMap class will handle conversions between types when appropriate (between integers, between floating point, etc). This casting should be exact, with no loss of precision or rounding errors. 

## Remarks
 * Options should be handled only by the core library
 * By the time the module is created, the options should have been checked and validated
 * Each module instance with the same module key will have the same options
 * When a given module key is instantiated from the module manager for the first time no more calls to `change_option` are accepted.
   * Internally the module may alter options (e.g. make options consistent), but this must be done in a deterministic manner, i.e. the same set of input options must always result in the same set of final options
   * A module should only alter its incoming options as a last resort

## Common Nomenclature
For the moment we are using strings for option keys.  Consequentially we need to try to be as consistent as possible among modules.  Once the set of option keys is more stable we might want to switch to enums to enforce this nomenclature.  At the moment these are the option keys we are using:
 * `METHOD` : The module key that maps to the EnergyMethod we would like to run (used in methods that farm out other jobs like many-body expansion and counterpoise corrections)
 * `FRAGMENTIZER` : The key for the SystemFragmenter that should be used to generate fragments for methods that need fragments
 * `BASIS_SET` : The key to be given to the System for the basis set
 * `FITTING_BASIS` : The key for the basis set used for fitting (e.g. RI, JK, f12, etc.)
 * `MAX_DERIV` : Intended more as an internal flag for modules that generate derivatives of a quantity, specifies the maximum analytic derivative the module knows how to do.  Used as an option because it is sometimes nice to be able to force finite difference of modules.
 * `MAX_ITER` : The maximum number of iterations for an iterative method
 * `DENS_TOLERANCE` : The tolerance for iterative procedures depending on densities
 * `KEY_INITIAL_GUESS` : The key for the module that makes the initial guess
 * `DAMPING_FACTOR`:The percentage of the previous Fock matrix (?.  It's density in Psi4) to add in
 * `FROZEN_CORE` : True if we are freezing the core orbitals for correlated computations
 
*To Be Continued Later...* 
 
 
