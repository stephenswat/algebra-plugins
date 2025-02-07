# Algebra plugins library, part of the ACTS project (R&D line)
#
# (c) 2021 CERN for the benefit of the ACTS project
#
# Mozilla Public License Version 2.0

# Set up the project.
cmake_minimum_required( VERSION 3.14 )
project( algebra-plugins VERSION 0.3 LANGUAGES CXX )

# Standard CMake include(s).
include( GNUInstallDirs )

# Explicitly set the output directory for the binaries. Such that if this
# project is included by another project, the main project's configuration would
# win out.
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY
   "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}" CACHE PATH
   "Directory for the built binaries" )
set( CMAKE_LIBRARY_OUTPUT_DIRECTORY
   "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}" CACHE PATH
   "Directory for the built libraries" )
set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY
   "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}" CACHE PATH
   "Directory for the built static libraries" )

# Include the Algebra Plugins CMake code.
list( APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake" )
include( algebra-plugins-compiler-options )
include( algebra-plugins-functions )

# Flags controlling how the Algebra Plugins code should behave.
set( ALGEBRA_PLUGIN_CUSTOM_SCALARTYPE "double" CACHE STRING
   "Scalar type to use in Algebra Plugins" )
option( ALGEBRA_PLUGIN_INCLUDE_EIGEN
   "Include Eigen types in Algebra Plugins" FALSE )
option( ALGEBRA_PLUGIN_INCLUDE_SMATRIX
   "Include Smatrix types in Algebra Plugins" FALSE )
option( ALGEBRA_PLUGIN_INCLUDE_VC
   "Include Vc types in Algebra Plugins" FALSE )
option( ALGEBRA_PLUGIN_INCLUDE_VECMEM
   "Include VecMem types in Algebra Plugins" FALSE )
option( ALGEBRA_PLUGIN_BUILD_TESTING "Build the unit tests of Algebra Plugins"
   TRUE )

# Suppress developer warnings for all of the externals by default.
# Unless the user explicitly requested otherwise.
if( NOT DEFINED CMAKE_SUPPRESS_DEVELOPER_WARNINGS )
   set( CMAKE_SUPPRESS_DEVELOPER_WARNINGS TRUE )
   set( _unsetDevWarningFlag TRUE )
endif()

# Set up VecMem.
option( ALGEBRA_PLUGIN_SETUP_VECMEM
   "Set up the VecMem target(s) explicitly" FALSE )
option( ALGEBRA_PLUGIN_USE_SYSTEM_VECMEM
   "Pick up an existing installation of VecMem from the build environment"
   TRUE )
if( ALGEBRA_PLUGIN_SETUP_VECMEM )
   if( ALGEBRA_PLUGIN_USE_SYSTEM_VECMEM )
      find_package( vecmem REQUIRED )
   else()
      add_subdirectory( extern/vecmem )
   endif()
endif()

# Set up GoogleTest.
option( ALGEBRA_PLUGIN_SETUP_GOOGLETEST
   "Set up the GoogleTest target(s) explicitly" TRUE )
option( ALGEBRA_PLUGIN_USE_SYSTEM_GOOGLETEST
   "Pick up an existing installation of GoogleTest from the build environment"
   FALSE )
if( ALGEBRA_PLUGIN_SETUP_GOOGLETEST )
   if( ALGEBRA_PLUGIN_USE_SYSTEM_GOOGLETEST )
      find_package( GTest REQUIRED )
   else()
      add_subdirectory( extern/googletest )
   endif()
endif()

# Set up Eigen3.
option( ALGEBRA_PLUGIN_SETUP_EIGEN3
   "Set up the Eigen3 target(s) explicitly" FALSE )
option( ALGEBRA_PLUGIN_USE_SYSTEM_EIGEN3
   "Pick up an existing installation of Eigen3 from the build environment"
   TRUE )
if( ALGEBRA_PLUGIN_SETUP_EIGEN3 )
   if( ALGEBRA_PLUGIN_USE_SYSTEM_EIGEN3 )
      find_package( Eigen3 REQUIRED )
   else()
      add_subdirectory( extern/eigen3 )
   endif()
endif()

# Set up Vc.
option( ALGEBRA_PLUGIN_SETUP_VC
   "Set up the Vc target(s) explicitly" FALSE )
option( ALGEBRA_PLUGIN_USE_SYSTEM_VC
   "Pick up an existing installation of Vc from the build environment"
   TRUE )
if( ALGEBRA_PLUGIN_SETUP_VC )
   if( ALGEBRA_PLUGIN_USE_SYSTEM_VC )
      find_package( Vc 1.4.2 REQUIRED )
   else()
      add_subdirectory( extern/vc )
   endif()
   # Use the preferred compiler flags from Vc for the entire project. Do not
   # set them on the libraries individually, as the clients of the libraries
   # may want to use a different optimisation.
   vc_set_preferred_compiler_flags()
   add_compile_definitions( ${Vc_DEFINITIONS} )
   add_compile_options( ${Vc_COMPILE_FLAGS} ${Vc_ARCHITECTURE_FLAGS} )
endif()

# Undo the developer flag suppression.
if( _unsetDevWarningFlag )
   unset( CMAKE_SUPPRESS_DEVELOPER_WARNINGS )
   unset( _unsetDevWarningFlag )
endif()

# Set up the Algebra Plugin libraries.
add_subdirectory( common )
add_subdirectory( storage )
add_subdirectory( math )
add_subdirectory( frontend )

# Set up the test(s).
include( CTest )
if( BUILD_TESTING AND ALGEBRA_PLUGIN_BUILD_TESTING )
  add_subdirectory( tests )
endif()

# Set up the packaging of the project.
include( algebra-plugins-packaging )
