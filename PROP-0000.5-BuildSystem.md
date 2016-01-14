# PROP 0000.5 - Build system

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | Conventions for the build system          |
| Status:        | In development                            |

## Purpose
It is intended for this project to interface and work with a large number of other projects.  In order to do this in a semi-automatic fashion we will employ the powerful CMake build system.  For better or for worse, the CMake documentation is not always clear on what are the recommended steps to do something.  The present proposal is to establish a set of conventions by which we will do various CMake tasks.

## Specifications
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
