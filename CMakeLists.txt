#*********************************************************************
# The Taichi Programming Language
#*********************************************************************

cmake_minimum_required(VERSION 3.12)

project(taichi)

SET(TI_VERSION_MAJOR 0)
SET(TI_VERSION_MINOR 7)
SET(TI_VERSION_PATCH 21)

set(CMAKE_CXX_STANDARD 17)

execute_process(
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  COMMAND git rev-parse --short HEAD
  RESULT_VARIABLE SHORT_HASH_RESULT
  OUTPUT_VARIABLE TI_COMMIT_SHORT_HASH)
execute_process(
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  COMMAND git rev-parse HEAD
  RESULT_VARIABLE SHORT_HASH_RESULT
  OUTPUT_VARIABLE TI_COMMIT_HASH)
string(STRIP ${TI_COMMIT_HASH} TI_COMMIT_HASH)
string(STRIP ${TI_COMMIT_SHORT_HASH} TI_COMMIT_SHORT_HASH)

message("Taichi Version ${TI_VERSION_MAJOR}.${TI_VERSION_MINOR}.${TI_VERSION_PATCH}")
message("       Commit ${TI_COMMIT_HASH}")

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING
            "Choose the type of build, options are: Debug Release
RelWithDebInfo MinSizeRel."
            FORCE)
endif(NOT CMAKE_BUILD_TYPE)

set(TAICHI_CMAKE_DIR "${CMAKE_CURRENT_LIST_DIR}/cmake/")

if (WIN32)
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake/")
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_ROOT}/Modules")
else ()
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_ROOT}/Modules")
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake/")
endif ()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/build")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/build")

include(cmake/PythonNumpyPybind11.cmake)
include(cmake/TaichiCXXFlags.cmake)
include(cmake/TaichiCore.cmake)
include(cmake/TaichiMain.cmake)

option(TI_BUILD_TESTS "Build the CPP tests" OFF)

if (TI_BUILD_TESTS)
  add_subdirectory(external/googletest EXCLUDE_FROM_ALL)
  include(cmake/TaichiTests.cmake)
endif()

message("C++ Flags: ${CMAKE_CXX_FLAGS}")
message("Build type: ${CMAKE_BUILD_TYPE}")

if (NOT TI_WITH_CUDA)
    set(CUDA_VERSION "0.0")
    set(CUDA_TOOLKIT_ROOT_DIR "")
endif()

message("python=${PYTHON_EXECUTABLE}")

add_custom_target(
    generate_commit_hash
    COMMAND ${PYTHON_EXECUTABLE} "${CMAKE_CURRENT_LIST_DIR}/misc/generate_commit_hash.py"
)
add_dependencies(${CORE_LIBRARY_NAME} generate_commit_hash)

FILE(WRITE ${CMAKE_CURRENT_LIST_DIR}/taichi/common/version.h
        "#pragma once\n"
        "#define TI_VERSION_MAJOR \"${TI_VERSION_MAJOR}\"\n"
        "#define TI_VERSION_MINOR \"${TI_VERSION_MINOR}\"\n"
        "#define TI_VERSION_PATCH \"${TI_VERSION_PATCH}\"\n"
        "#define TI_CUDAVERSION \"${CUDA_VERSION}\"\n"
        )

option(TI_EXPORT_CORE "export taichi core" OFF)

if (TI_EXPORT_CORE)
  include(cmake/TaichiExportCore.cmake)
endif()
