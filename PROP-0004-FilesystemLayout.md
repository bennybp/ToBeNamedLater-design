# PROP 0004 - Filesystem Layout

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | Where files go                            |
| Status:        | In development                            |


## Purpose
It is important that developers know where to look for files and where to put new ones.  This proposal lays out the basic filesystem that the core of the project uses.

## Layout
- The top level directory is `BPModule`
  - **bpmodule** where the core files reside
    - **basisset** - contains includes and sources relating to the basis set class
    - **datastore** - the cache, graph, options, and Wavefunction, etc
    - **exception** - where code related to our error handeling lives
    - **math** - general mathematical classes and functionality
    - **modulebase** - module interfaces. Modules are required to conform to these interfaces
    - **modulemanager** - code for selecting, initializing, and loading modules
    - **output** - code for standard printing as well as more complicated printing
    - **parallel** - parallelization infrastructure
    - **python** - wrapper and helpers for Python-C++ interoperability
    - **system** - the molecule and system of interest. Handles things like electric fields, point charges, etc. and not just atoms. Hence, *system* rather than *molecule*.
    - **tensor** - wrapper and helpers for vectors, matrices, and tensors.
    - **testing** - functions and classes for testing various parts of the code, as well as external modules.
    - **util** - miscellaneous useful structures and functions that don't really fit anywhere else
  - **external** - where git submodules go
     - **libelemental** - a distributed matrix library
     - **madness** - a high-level, task-based parallel library on which TiledArray and Ryan's parallel library are based
     - **memwatch** - a top-level project designed to ensure that a program never exceeds a given amount of memory
     - **pybind11** - Python-C++ interoperability package
     - **tensor** - Ryan's tensor wrapper project
  - **modules** - some example and testing modules
  - **scripts** - contains scripts that generate code for things like new modules, physical constants, etc.
  - **test** - contains tests that ensure everything is working correctly 
