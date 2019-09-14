# CMake version
cmake_minimum_required (VERSION 3.9)
MESSAGE(STATUS "CMAKE_ROOT: " ${CMAKE_ROOT})

# Project name
project(Cpp-Taskflow VERSION 2.2.0 LANGUAGES CXX)

# Turn on the verbose
set(CMAKE_VERBOSE_MAKEFILE ON)

# Compiler vendors
## g++
if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS "7.3")
    message(FATAL_ERROR "\nCpp-Taskflow requires g++ at least v7.3")
  endif()
## clang++
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS "6.0")
    message(FATAL_ERROR "\nCpp-Taskflow requires clang++ at least v6.0")
  endif() 
## microsoft visual c++
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
  if(NOT MSVC_VERSION GREATER_EQUAL 1914)
    message(FATAL_ERROR "\nCpp-Taskflow requires MSVC++ at least v14.14") 
  endif()
else()
  message(FATAL_ERROR "\n\
Cpp-Taskflow currently supports the following compilers:\n\
- g++ v7.3 or above\n\
- clang++ v6.0 or above\n\
- MSVC++ v19.14 or above\n\
")
endif()

# defult release build
set(TF_DEFAULT_BUILD_TYPE "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${TF_DEFAULT_BUILD_TYPE}'")
  set(
    CMAKE_BUILD_TYPE "${TF_DEFAULT_BUILD_TYPE}" 
    CACHE
    STRING "Choose the type of build." 
    FORCE
  )
  # Set the possible values of build type for cmake-gui
  set_property(
    CACHE 
    CMAKE_BUILD_TYPE 
    PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo"
  )
endif()


# error setting
add_library(error_settings INTERFACE)
add_library(tf::error_settings ALIAS error_settings)

target_compile_options(
  error_settings
  INTERFACE
  $<$<CXX_COMPILER_ID:Clang>:-Wall -Wextra -Wfatal-errors>
  $<$<CXX_COMPILER_ID:GNU>:-Wall -Wextra -Wfatal-errors>
  $<$<CXX_COMPILER_ID:MSVC>:/W3 /permissive->
)

# additional features
add_library(features_settings INTERFACE)
add_library(tf::features_settings ALIAS features_settings)
target_compile_definitions(
  features_settings 
  INTERFACE 
  $<$<CXX_COMPILER_ID:MSVC>:_ENABLE_EXTENDED_ALIGNED_STORAGE>
)

# optimization
#
##! Msvc flags info
# /Zi - Produces a program database (PDB) that contains type information 
#       and symbolic debugging information for use with the debugger.
# /FS - Allows multiple cl.exe processes to write to the same .pdb file
# /DEBUG - Enable debug during linking
# /Od - Disables optimization
# /Ox - Full optimization
# /Oy- do not suppress frame pointers (recommended for debugging)
#
add_library(optimize_settings INTERFACE)
add_library(tf::optimize_settings ALIAS optimize_settings)

target_compile_options(
  optimize_settings INTERFACE
  $<$<AND:$<CONFIG:Release>,$<CXX_COMPILER_ID:Clang>>:-O2 -march=native>
  $<$<AND:$<CONFIG:Release>,$<CXX_COMPILER_ID:GNU>>:-O2 -march=native>
  $<$<AND:$<CONFIG:Release>,$<CXX_COMPILER_ID:MSVC>>:/O2 -DNDEBUG /MP>
  $<$<AND:$<CONFIG:Debug>,$<CXX_COMPILER_ID:GNU>>:-O0 -g>
  $<$<AND:$<CONFIG:Debug>,$<CXX_COMPILER_ID:Clang>>:-O0 -g>
  $<$<AND:$<CONFIG:Debug>,$<CXX_COMPILER_ID:MSVC>>:/Zi /FS /DEBUG /Od /MP /MDd /Oy->
)

add_library(default_settings INTERFACE)
add_library(tf::default_settings ALIAS default_settings)
target_link_libraries(
  default_settings 
  INTERFACE 
  tf::error_settings 
  tf::optimize_settings 
  tf::features_settings
)

# CXX target properties
#set(CMAKE_CXX_STANDARD 17)
#set(CMAKE_CXX_STANDARD_REQUIRED ON)
#set(CMAKE_CXX_EXTENSIONS OFF)

# build options
option(TF_BUILD_EXAMPLES "Enables build of examples" ON)
option(TF_BUILD_TESTS "Enables build of tests" ON)
option(TF_BUILD_BENCHMARKS "Enables build of benchmarks" OFF)

# installation path
set(TF_INC_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/include")
set(TF_LIB_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/lib")
set(TF_UTEST_DIR ${PROJECT_SOURCE_DIR}/unittest)
set(TF_EXAMPLE_DIR ${PROJECT_SOURCE_DIR}/example)
set(TF_BENCHMARK_DIR ${PROJECT_SOURCE_DIR}/benchmark)

message(STATUS "CMAKE_HOST_SYSTEM: ${CMAKE_HOST_SYSTEM}")
message(STATUS "CMAKE_BUILD_TYPE: " ${CMAKE_BUILD_TYPE})
message(STATUS "CMAKE_CXX_COMPILER: " ${CMAKE_CXX_COMPILER})
message(STATUS "CMAKE_CXX_COMPILER_VERSION: " ${CMAKE_CXX_COMPILER_VERSION})
message(STATUS "CMAKE_CXX_FLAGS: " ${CMAKE_CXX_FLAGS})
message(STATUS "CMAKE_MODULE_PATH: " ${CMAKE_MODULE_PATH})
message(STATUS "CMAKE_CURRENT_SOURCE_DIR: " ${CMAKE_CURRENT_SOURCE_DIR})
message(STATUS "CMAKE_CURRENT_BINARY_DIR: " ${CMAKE_CURRENT_BINARY_DIR})
message(STATUS "CMAKE_EXE_LINKER_FLAGS: " ${CMAKE_EXE_LINKER_FLAGS})
message(STATUS "CMAKE_INSTALL_PREFIX: " ${CMAKE_INSTALL_PREFIX})
message(STATUS "CMAKE_MODULE_PATH: " ${CMAKE_MODULE_PATH})
message(STATUS "CMAKE_PREFIX_PATH: " ${CMAKE_PREFIX_PATH})
message(STATUS "PROJECT_NAME: " ${PROJECT_NAME})
message(STATUS "TF_BUILD_EXAMPLES: " ${TF_BUILD_EXAMPLES})
message(STATUS "TF_BUILD_TESTS: " ${TF_BUILD_TESTS})
message(STATUS "TF_BUILD_BENCHMARKS: " ${TF_BUILD_BENCHMARKS})
message(STATUS "TF_INC_INSTALL_DIR: " ${TF_INC_INSTALL_DIR})
message(STATUS "TF_LIB_INSTALL_DIR: " ${TF_LIB_INSTALL_DIR})
message(STATUS "TF_UTEST_DIR: " ${TF_UTEST_DIR})
message(STATUS "TF_EXAMPLE_DIR: " ${TF_EXAMPLE_DIR})
message(STATUS "TF_BENCHMARK_DIR: " ${TF_BENCHMARK_DIR})

# add the binary tree to the search path for include files
include_directories(${PROJECT_SOURCE_DIR})

# -----------------------------------------------------------------------------
# must-have package include
# -----------------------------------------------------------------------------

# Find pthread package
find_package(Threads REQUIRED)

# -----------------------------------------------------------------------------
# Cpp-Taskflow library interface
# -----------------------------------------------------------------------------

add_library(${PROJECT_NAME} INTERFACE)

target_compile_features(${PROJECT_NAME} INTERFACE cxx_std_17)
target_include_directories(${PROJECT_NAME} INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)