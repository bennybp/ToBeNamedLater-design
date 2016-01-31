# PROP 0003 - Build system

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | Conventions for the build system          |
| Status:        | In development                            |

## Purpose
To lay out the functionality required from the build system and how it will be achieved.

## Requirements

1. For this type of project, the build system performs three tasks. 
  1. It must build all external dependencies of the project
  2. It must build the project itself
  3. It must allow for easy linking of externally-build modules to both the external dependencies and the core project

2. For external projects
  1. Sources are stored under the `external` subdirectory
  2. Sources are packaged with git submodules when possible.
  3. The build system must allow for using libraries already found on the system
    1. Must be able to specify paths to existing libraries
    2. Must check for version
    3. If specified by the user, but it is not suitable, it must abort
  4. If not found, the build system must build it automatically

3. Other requirements
  1. Nothing should be installed to the final install directory until `make install` is invoked.
     This is to prevent installation to places that require administrator privileges (and
     would therefore need to run `make` as administrator, which is a bad idea).

  
## Proposed Implementation

The proposed implementation uses CMake (http://www.cmake.org).

Overall, the build process should follow the *superbuild* pattern
commonly used for projects like this. In a superbuild, everything,
including the main library/program you are trying to build, is built
as an external project (via `ExternalProject_Add`).

A few details of this setup:

  * Common dependencies are found at the very top-level CMake, and passed to the external projects. This
    ensures that all projects use the same libraries
    * Common packages: Python, Boost, OpenMP, MPI
    * How they are passed to the external projects depends on the package
  * Dependencies can only be added between external projects.
    * Cannot mix external and "internal"
    * Therefore, the core must also be external
  * External projects (including the core) are configured only right before they are built
    * Important for the core project since the external dependencies might not be available
      at the top-level configure
  * External projects (including the core) are installed to `${CMAKE_INSTALL_PREFIX}`, but with `DESTDIR` set to
    a staging directory within the build directory (`${CMAKE_BINARY_DIR}/stage`). This is
    sometimes called a *staged install*.
    * This installs it to `${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_PREFIX}`, but the package
      thinks it is being installed to `${CMAKE_INSTALL_PREFIX}`.
  * Each external project is responsible for finding its own dependencies (usually through CMake `find_package`)
  * For these calls to `find_package` to work, either
    * The package provides a CMake `<package>Config.cmake` file
    * The package provides a `Find<package>.cmake` file (and we add the path to the appropriate CMake variable)
    * We write our own `Find<package>.cmake` file (and we add the path to the appropriate CMake variable)
    * **Note** - It is usually difficult for us to create the `<package>Config.cmake` file ourselves outside
      of the building of the external project, since we do not have direct access to the project's targets
      at that time

### Filesystem Structure

```
   <project root>
   |  CMakeLists.txt
   |
   |-- bpmodule
   |
   |-- external
      |  CMakeLists.txt
      |
      |-- <dependency>
          |  CMakeLists.txt
          |
          |-- <dependency>-source
```

`<project root>/CMakeLists.txt` will, directly or indirectly, find
the common packages needed by the external dependencies and the core.

`<project root>/bpmodule` contains the core project, with its own CMakeLists.txt.

`<project root>/external` basically adds all the subdirectories to the build (via `add_subdirectory`.

`<project root>/external/<dependency>` contains the CMakeLists.txt which
first checks if the package already exists on the system in the default
search paths and in any paths specified by the user. If a path was
specified by the user and the package was not found, it should abort.
If the package is not found (and no path was specified by the user), this
CMakeLists.txt is then responsible for building (via `ExternalProject_Add`)
the package. This directory may contain other files relating to the external
project (for example, any patches).

`<project root>/external/<dependency>/<dependency>-source` contains the
external source for the project. Typically, this should be a git submodule,
but may be just contain the source directly.


<!---

One of the most fundamental operations in a build system is finding and (possibly) building dependencies.  Pre-built dependencies are always to be preferred unless they fail to meet the necessary requirements (too old, missing component, etc.).  Presently we purpose, for an external dependency called "Dependency", this is handled in `BPModule/external/Dependency/CMakeLists.txt`:

```cmake
#Should this be moved up to external/CMakeLists.txt(?)
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../cmake") 
find_package(Dependency)

if(NOT Dependency_FOUND)
   message(STATUS "We will build Dependency for you.")
   ExternalProject_Add(Dependency_external
                       SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/dependency-source
                       CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}/external/Dependency
                                  -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
                                  -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
                                  -DCMAKE_Fortran_COMPILER=${CMAKE_Fortran_COMPILER}
                                  -DMPI_C_COMPILER=${MPI_C_COMPILER}
                                  -DMPI_CXX_COMPILER=${MPI_CXX_COMPILER}
                                  <other arguments go here>
                       INSTALL_COMMAND ${CMAKE_MAKE_PROGRAM} install
    )
  #Libraries should look something like: ${CMAKE_INSTALL_PREFIX}/external/Dependency/lib/libDepend.so
  set(Dependency_LIBRARIES <the libraries Dependency builds as well as depends on>)
  #Include directories should look something like: ${CMAKE_INSTALL_PREFIX}/external/Dependency/include
  set(Dependency_INCLUDE_DIRS <path to Dependency's include files as well as those it depends on>)
else()
  message(STATUS "Using existing dependency")
  add_library(Dependency_external INTERFACE)
endif()
```

There are several key points.  First, we always look for the package, even if it's something that is uber unlikely to be found.  If it is not found (and not a super fundamental package: BLAS, pthreads, etc.) we should attempt to build it as an external project.  By convention the source for the dependency should be in a folder called `dependency-source` contained in the same folder as the `CMAkeLists.txt` we just detailed.  Next, we should always pass certain fundamental CMake variables to the external project.  Any of these variables, which are unused by the subproject should be deleted to keep the configuration warning free.  Then the external project should be installed.  As written, it wil be installed in the same place as the module project, but in a subdirectory: `external/Dependency`.  Finally, you should set the `*_LIBRARIES` and `*_INCLUDE_DIRS` variables appropriately so that they can be passed into other external projects.   In the event that the external project is already built and found by `find_package` a dummy interface target should be made with the same name as the external project and all other projects should refer to this target as the dependency.

Questions.  Is the setting of the `*_LIBRARIES` and `*_INCLUDE_DIRS` the way to go?  Consider making a directory: `${CMAKE_INSTALL_PREFIX}/config` which holds all the `*-config.cmake` files (copy done at build time).  We could then set CMAKE_PREFIX_PATH to `${CMAKE_INSTALL_PREFIX}/config`.  Now when other external packages look for packages (at run-time) they should find the `*-config.cmake` files, which may contain more than just the library/include variables.

Also I vote that "Dependency" and "dependency" in the code example above is always "DEPENDENCY".  This seems to be somewhat normal as far as naming conventions go.

-->
