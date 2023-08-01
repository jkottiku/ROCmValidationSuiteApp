################################################################################
##
## Copyright (c) 2018-2022 2018 Advanced Micro Devices, Inc. All rights reserved.
##
## MIT LICENSE:
## Permission is hereby granted, free of charge, to any person obtaining a copy of
## this software and associated documentation files (the "Software"), to deal in
## the Software without restriction, including without limitation the rights to
## use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
## of the Software, and to permit persons to whom the Software is furnished to do
## so, subject to the following conditions:
##
## The above copyright notice and this permission notice shall be included in all
## copies or substantial portions of the Software.
##
## THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
## IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
## FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL THE
## AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
## LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
## OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
## SOFTWARE.
##
################################################################################

cmake_minimum_required ( VERSION 3.5.0 )

if ( ${CMAKE_BINARY_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR})
  message(FATAL "In-source build is not allowed")
endif ()
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

set ( RVS "rvsapp" )
set ( RVS_PACKAGE "rvs-roct" )
set ( RVS_TARGET "${RVS}" )

project ( ${RVS_TARGET} )

message(STATUS "MODULE: ${RVS}")

#add_compile_options(-std=c++11)
add_compile_options(-pthread)
add_compile_options(-Wall)
add_compile_options(-DRVS_OS_TYPE_NUM=${RVS_OS_TYPE_NUM})

if (RVS_COVERAGE)
  add_compile_options(-o0 -fprofile-arcs -ftest-coverage)
  set(CMAKE_EXE_LINKER_FLAGS "--coverage")
  set(CMAKE_SHARED_LINKER_FLAGS "--coverage")
endif()

## Set default module path if not already set
if ( NOT DEFINED CMAKE_MODULE_PATH )
    set ( CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../cmake_modules/" )
endif ()

## Include common cmake modules
#include ( utils )
#include (CTest)


# Determine HSA_PATH
if(NOT DEFINED HIPCC_PATH)
  if(NOT DEFINED ENV{HIPCC_PATH})
    set(HIPCC_PATH "${ROCM_PATH}" CACHE PATH "Path to which hipcc runtime has been installed")
     else()
       set(HIPCC_PATH $ENV{HIPCC_PATH} CACHE PATH "Path to which hipcc runtime has been installed")
     endif()
endif()

# Add HIP_VERSION to CMAKE_<LANG>_FLAGS
set(HIP_HCC_BUILD_FLAGS "${HIP_HCC_BUILD_FLAGS} -DHIP_VERSION_MAJOR=${HIP_VERSION_MAJOR} -DHIP_VERSION_MINOR=${HIP_VERSION_MINOR} -DHIP_VERSION_PATCH=${HIP_VERSION_GITDATE}")

set(HIP_HCC_BUILD_FLAGS)
set(HIP_HCC_BUILD_FLAGS "${HIP_HCC_BUILD_FLAGS} -fPIC ${HCC_CXX_FLAGS} -I${HSA_PATH}/include ${ASAN_CXX_FLAGS}")

# Set compiler and compiler flags
set(CMAKE_CXX_COMPILER "${HIPCC_PATH}/bin/hipcc")
set(CMAKE_C_COMPILER   "${HIPCC_PATH}/bin/hipcc")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${HIP_HCC_BUILD_FLAGS}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${HIP_HCC_BUILD_FLAGS}")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${ASAN_LD_FLAGS}")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} ${ASAN_LD_FLAGS}")

if(BUILD_ADDRESS_SANITIZER)
  execute_process(COMMAND ${CMAKE_CXX_COMPILER} --print-file-name=libclang_rt.asan-x86_64.so
            OUTPUT_VARIABLE ASAN_LIB_FULL_PATH)
  get_filename_component(ASAN_LIB_PATH ${ASAN_LIB_FULL_PATH} DIRECTORY)
else()
  set(ASAN_LIB_PATH "$ENV{LD_LIBRARY_PATH}")
endif()

if (RVS_ROCMSMI EQUAL 1)
  set(ROCM_SMI_INC_DIR "${CMAKE_BINARY_DIR}/rvs_smi-src/include")
  set(ROCM_SMI_LIB_DIR "${CMAKE_BINARY_DIR}/rvs_smi-build")
else()
  #NOTE:Include folders need to be updated to latest include dir as per REORG Folder structure.
  #For backward compatibility we are currently using version details to determine the include path
  #Current Work Around to check ROCMSMI Version is using ROCBLAS REORG VERSION
  if( RVS_ROCBLAS_VERSION_FLAT LESS 2044000 )
    set(ROCM_SMI_INC_DIR "${ROCM_PATH}/rocm_smi/include")
    set(ROCM_SMI_LIB_DIR "${ROCM_PATH}/rocm_smi/lib")
  else()
    message( STATUS "ROCBLAS REORG Enabled Version: ${RVS_ROCBLAS_VERSION_FLAT}" )
    set(ROCM_SMI_INC_DIR "${ROCM_PATH}/include")
    set(ROCM_SMI_LIB_DIR "${ROCM_PATH}/lib")
  endif()
endif()
set(ROCM_SMI_LIB "rocm_smi64" CACHE STRING "rocm_smi library name")

## define include directories
## define lib directories
#link_directories(${CMAKE_CURRENT_BINARY_DIR} ${RVS_LIB_DIR} ${ROCT_LIB_DIR} ${ROCBLAS_LIB_DIR} ${ROCM_SMI_LIB_DIR})
include_directories(${RVS_INCLUDE_DIR})
link_directories(${RVS_LIBRARY_DIR}/rvs ${ROCBLAS_LIB_DIR} ${ROCM_SMI_LIB_DIR})

## additional libraries
set(ROCBLAS_LIB "rocblas")
set(ROC_THUNK_NAME "hsakmt")
set(CORE_RUNTIME_NAME "hsa-runtime")
set(CORE_RUNTIME_TARGET "${CORE_RUNTIME_NAME}64")
##set(PROJECT_LINK_LIBS libdl.so "${YAML_LIB_DIR}/libyaml-cpp.a" libpci.so)
set(PROJECT_LINK_LIBS libdl.so libpci.so libpthread.so ${YAML_CPP_LIBRARIES})
# libpthread.so libpci.so)

## define target
add_executable(${RVS_TARGET} rvs_app.c)
#target_link_libraries(${RVS_TARGET} rvshelper rvslib rvslibrt ${PROJECT_LINK_LIBS})
target_link_libraries(${RVS_TARGET} rvslib
  ${ROCBLAS_LIB} ${ROCM_SMI_LIB} ${ROC_THUNK_NAME} ${CORE_RUNTIME_TARGET} ${PROJECT_LINK_LIBS})
 
#add_dependencies(${RVS_TARGET} rvshelper)

