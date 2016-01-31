# PROP 0002 - Filesystem Layout

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | Where files go                            |
| Status:        | In development                            |


## Purpose
It is important that developers know where to look for files and where to put new ones.  This proposal lays out the basic filesystem that the core of "the module project" uses.

## Layout
- The top level directory is `BPModule`
- Within this directory are:
  - `external` where git submodules go. As modules (in our project sense) are formed, added to git hub, and accepted into this project they will be added here as submodules. For the time we only have some key external projects desribed below:
     - `libelemental` a distributed matrix library, needs to be encoporated into Ryan's tensor wrapper
     - `madness` a high-level, task-based parallel library on which TiledArray and Ryan's parallel library are based
     - `memwatch` a top-level project designed to ensure that a program never exceeds a given amount of memory
     - `pybind11` this is our alternative to boost::python.  It is much lighther and fully encorporates C++11
     - `tensor` Ryan's tensor wrapper project
  - `bpmodule` where the core files reside
    - Inside `bpmodule` we have:
      - `basisset` contains includes and sources relating to the basis set class
      - `datastore` the cache, graph, options, and Wavefunction live here 
         - Should the graph library be moved to math?
      - `exception` where code related to our error handeling lives
      - `math` mathematical classes (except tensors) live here as do mathematical routines and functions
      - `modulebase` code common to all modules resides here (basically ensures your module can play nicely with the rest of the          framework.  You will probably never touch this directory.
      - `modulelocator` code for selecting and intializing modules lives here
      - `output` code for standard printing as well as more complicated printing (e.g. printing tables of data) lives here
      - `parallel` likely to be replaced by Ryan's wrapper around Madness
      - `python` is this used now that we have committed to Pybind11 ?
      - `system` where routines related to our "molecule" class live.  The name is meant to draw attention to the fact that this also handles things like electric fields, point charges, etc. and not just atoms.  This is not for system in the computer sense, we are open to a less confusing name consistent with the previously mentioned intent.
      - `tensor` likely to be replaced by Ryan's generic tensor wrapper
      - `testing` you guessed it, files related to establishing tests
      - `util` sort of the grab bag directory, contains useful structures and functions that don't really fit anywhere else
   - `modules` is this for our core modules (?)
   - `scripts` contains scripts that generate code for things like new modules, physical constants, etc.
   - `test` contains tests that ensure everything is working correctly 
