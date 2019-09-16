# Copyright (c) 2019 Tang, Wenyi
# Author: Wenyi Tang
# E-mail: wenyi.tang@intel.com
# Welcome to LoSealL's C++ middle-ware
###########################################
#  - scripts
#
#  - ll_codec
#
#  - ll_graphic
#
###########################################

cmake_minimum_required(VERSION 3.14.2)
file(STRINGS version version)
string(TIMESTAMP date %Y%m%d)
string(APPEND version ".${date}")
project(libll VERSION ${version})
message("-- libll version: ${version}")
# update submodule
macro(update_submodule dir)
  find_package(Git REQUIRED)
  if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
    # Update submodules as needed
    message(STATUS "updating submodule ${dir}")
    execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init ${dir}
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                    RESULT_VARIABLE GIT_SUBMOD_RESULT)
    if(NOT GIT_SUBMOD_RESULT EQUAL "0")
      message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
    endif()
  endif()  
endmacro(update_submodule)

# enable folder scope in ide
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
cmake_policy(SET CMP0074 NEW)
if(NOT EXISTS ${PROJECT_SOURCE_DIR}/scripts/.git)
  update_submodule(scripts)
endif()
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/scripts/cmake)
include(msvc_helper)
# Set default output dir
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib CACHE PATH "lib dir")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib CACHE PATH "lib dir")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin CACHE PATH "exe dir")
set(CMAKE_DEBUG_POSTFIX "d")
# By default we build nothing
option(LL_BUILD_CODEC "Building codec library" OFF)
option(LL_BUILD_GFX "Building graphic library" OFF)
option(LL_BUILD_TESTS "Building all unit test" OFF)

# OS-depedant setting
if(MSVC)
  enable_compile_flag_mt()
  enable_unicode_charactor()
  set_msvc_warning_level(4)
  disable_minmax_macro()
endif()

if(LL_BUILD_TESTS)
  include(gtest_helper)
endif()

if(LL_BUILD_CODEC)
  if(NOT EXISTS ${PROJECT_SOURCE_DIR}/ll_codec/.git)
    update_submodule(ll_codec)
  endif()
  add_subdirectory(ll_codec)
endif()

if(LL_BUILD_GFX)
  if(NOT EXISTS ${PROJECT_SOURCE_DIR}/ll_graphic/.git)
    update_submodule(ll_graphic)
  endif()
  add_subdirectory(ll_graphic)
endif()

install(FILES scripts/cmake/ll-config.cmake DESTINATION ${CMAKE_INSTALL_PREFIX})
