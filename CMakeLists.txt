# MIT License
#
# Copyright (c) 2018-2023 The ViaDuck Project
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
#

cmake_minimum_required(VERSION 3.2...3.27)
project(icu-cmake)

# Dependency management

if (PROJECT_IS_TOP_LEVEL)
    if(NOT DEFINED CPM_SOURCE_CACHE)
        set(CPM_SOURCE_CACHE ${PROJECT_SOURCE_DIR}/3rdparty/cpm_cache CACHE STRING "")
    endif()

    set(CPM_DOWNLOAD_ALL                ON CACHE BOOL "")
    set(CPM_USE_NAMED_CACHE_DIRECTORIES ON CACHE BOOL "")
endif(PROJECT_IS_TOP_LEVEL)

include(cmake/CPM.cmake)

# cache variables
set(BUILD_ICU         OFF    CACHE BOOL   "Enable compilation of ICU")
set(SYSTEM_ICU        OFF    CACHE BOOL   "Use system-provided icu libraries (instead of prebuilts or building")
set(ICU_STATIC        OFF    CACHE BOOL   "Use static instead of shared ICU (does not work for system)")
set(ICU_SOURCE_DIR    ""     CACHE PATH   "User defined ICU sources path")
set(ICU_CFG_OPTIONS   ""     CACHE STRING "Additional ICU configuration options")
set(ICU_CROSS_ARCH    ""     CACHE STRING "Target triplet for ICU cross compile")

# allow including our modules
list(INSERT CMAKE_MODULE_PATH 0 "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
list(INSERT CMAKE_MODULE_PATH 0 "${CMAKE_CURRENT_SOURCE_DIR}/cmake-modules")

# declare interface target for ICU
add_library(icu INTERFACE)

if (SYSTEM_ICU)
    # try to find icu
    find_package(ICU COMPONENTS data uc i18n io REQUIRED)
else()
    # build own or or use prebuilts
    
    # set up target
    if (BUILD_ICU)
        include(BuildICU)
    else()
        include(PrebuiltICU)
    endif()

    if (NOT ICU_STATIC)
        include(GNUInstallDirs)
        install(FILES ${ICU_LIBRARY_FILES} DESTINATION ${CMAKE_INSTALL_LIBDIR})
    endif()
endif()

# link system, cross or host libraries to interface target
target_link_libraries(icu INTERFACE ${ICU_LIBRARIES})
target_include_directories(icu INTERFACE ${ICU_INCLUDE_DIRS})

add_library(ICU::data ALIAS icu)
add_library(ICU::i18n ALIAS icu)
add_library(ICU::uc ALIAS icu)
