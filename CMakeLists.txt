################################################################################
##
## Copyright (c) 2018-2022 Advanced Micro Devices, Inc. All rights reserved.
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
project ("rocm-validation-suite-app")

#including GNUInstall Dir definitions.
include( GNUInstallDirs )

# Prerequisite - Check if rvs was already installed
find_package (rvs)
if(rvs_FOUND)
  message(STATUS "rvs found")

  set(RVS_INCLUDE_DIR "${ROCM_VALIDATION_SUITE_INCLUDE_DIR}")
  set(RVS_LIBRARY_DIR "${ROCM_VALIDATION_SUITE_LIB_DIR}")

  message(STATUS "${RVS_INCLUDE_DIR}")
  message(STATUS "${RVS_LIBRARY_DIR}")

endif(rvs_FOUND)

#execute_process(COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/rvs_os_helper.sh
#  RESULT_VARIABLE result
#  OUTPUT_VARIABLE RVS_OS_TYPE
#  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
#  OUTPUT_STRIP_TRAILING_WHITESPACE
#)
#if (result)
#   message(FATAL_ERROR "Could not obtain OS type: ${result}")
#endif()
#message(STATUS "RVS_OS_TYPE: ${RVS_OS_TYPE}")


# Making ROCM_PATH, CMAKE_INSTALL_PREFIX, CMAKE_PACKAGING_INSTALL_PREFIX as CACHE
# variables since we will pass them as cmake params appropriately, and 
# all find_packages relevant to this build will be in ROCM path hence appending it to CMAKE_PREFIX_PATH 
set(ROCM_PATH "/opt/rocm" CACHE PATH "ROCM install path")
set(CMAKE_INSTALL_PREFIX "/opt/rocm" CACHE PATH "CMAKE installation directory")
set(CMAKE_PACKAGING_INSTALL_PREFIX "/opt/rocm" CACHE PATH "Prefix used in built packages")
list(APPEND CMAKE_PREFIX_PATH "${ROCM_PATH}")
set(ROCR_INC_DIR "${ROCM_PATH}/include" CACHE PATH "Contains header files exported by ROC Runtime" FORCE)
set(ROCR_LIB_DIR "${ROCM_PATH}/lib" CACHE PATH "Contains library files exported by ROC Runtime" FORCE)
set(HIP_INC_DIR "${ROCM_PATH}" CACHE PATH "Contains header files exported by ROC Runtime")
set(ROCT_INC_DIR "${ROCM_PATH}/include" CACHE PATH "Contains header files exported by ROC Trunk" FORCE)

## Include common cmake modules
#include ( utils )

set(HCC_CXX_FLAGS  "-fno-gpu-rdc  --amdgpu-target=gfx90a --amdgpu-target=gfx1030 --amdgpu-target=gfx803 --amdgpu-target=gfx900 --amdgpu-target=gfx906 --amdgpu-target=gfx908 -g")

################################################################################
# check if yaml-cpp available at configure time
find_package(yaml-cpp)
if (yaml-cpp_FOUND)
  message("yaml-cpp found")
else()
  message(FATAL_ERROR "yaml-cpp not found !!! Install  to proceed ...")
endif(yaml-cpp_FOUND)

if (0)
# Download and unpack yaml-cpp at configure time
configure_file(CMakeYamlDownload.cmake yaml-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/yaml-download )
if(result)
  message(FATAL_ERROR "CMake step for yaml-download failed: ${result}")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/yaml-download )
if(result)
  message(FATAL_ERROR "Build step for yaml-download failed: ${result}")
endif()

execute_process(COMMAND ${CMAKE_COMMAND} ${CMAKE_BINARY_DIR}/yaml-src -B${CMAKE_BINARY_DIR}/yaml-build
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/yaml-src )
if(result)
  message(FATAL_ERROR "Config step for yaml-src failed: ${result}")
endif()

add_custom_target(rvs_yaml_target
  DEPENDS ${CMAKE_BINARY_DIR}/yaml-build/libyaml-cpp.a
)

add_custom_command(OUTPUT ${CMAKE_BINARY_DIR}/yaml-build/libyaml-cpp.a
  COMMAND make -C ${CMAKE_BINARY_DIR}/yaml-build
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/yaml-src
  COMMENT "Generating yaml-cpp targets"
  VERBATIM)

endif()
################################################################################

add_subdirectory(src)

##add_dependencies(rvs rvs_yaml_target)
##add_dependencies(rvs)


