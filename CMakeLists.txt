#
# This file is part of WinSparkle (http://winsparkle.org)
#
# Copyright (C) 2015 Alexander Lamaison
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.

cmake_minimum_required(VERSION 2.8.11)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

project(WinSparkle)

# Parse version

file(READ ${CMAKE_CURRENT_SOURCE_DIR}/include/winsparkle-version.h _HEADER_CONTENTS)
string(
  REGEX REPLACE ".*#define WIN_SPARKLE_VERSION_MAJOR[ \t]+([0-9]+).*" "\\1"
  WinSparkle_VERSION_MAJOR "${_HEADER_CONTENTS}")
string(
  REGEX REPLACE ".*#define WIN_SPARKLE_VERSION_MINOR[ \t]+([0-9]+).*" "\\1"
  WinSparkle_VERSION_MINOR "${_HEADER_CONTENTS}")
string(
  REGEX REPLACE ".*#define WIN_SPARKLE_VERSION_MICRO[ \t]+([0-9]+).*" "\\1"
  WinSparkle_VERSION_PATCH "${_HEADER_CONTENTS}")

if(NOT WinSparkle_VERSION_MAJOR MATCHES "^[0-9]+$" OR
   NOT WinSparkle_VERSION_MINOR MATCHES "^[0-9]+$" OR
   NOT WinSparkle_VERSION_PATCH MATCHES "^[0-9]+$")
  message(
    FATAL_ERROR
    "Unable to parse version from"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/winsparkle-version.h")
endif()

set(WinSparkle_VERSION "${WinSparkle_VERSION_MAJOR}.${WinSparkle_VERSION_MINOR}.${WinSparkle_VERSION_PATCH}")

option(BUILD_SHARED_LIBS "Build Shared Libraries" OFF)

include(GNUInstallDirs)

# Package management ###########################################################

include(HunterGate)

HunterGate(
    URL "https://github.com/alamaison/hunter/archive/7e6cef91a0537e6558739fb79508dfd167c4d45f.tar.gz"
    SHA1 "3ec7ae8c1d8d3d8fe8ba17d3276efd7f48da86d1"
)

hunter_add_package(Expat)
hunter_add_package(wxWidgets)

find_package(EXPAT REQUIRED)

find_package(wxWidgets CONFIG REQUIRED core base)
include(${wxWidgets_USE_FILE})

################################################################################


set(HEADERS
  include/winsparkle.h
  include/winsparkle-version.h)
set(SOURCES
  ${HEADERS}
  src/appcast.h
  src/appcontroller.h
  src/download.h
  src/error.h
  src/settings.h
  src/threads.h
  src/ui.h
  src/updatechecker.h
  src/updatedownloader.h
  src/utils.h
  src/appcast.cpp
  src/appcontroller.cpp
  src/dll_api.cpp
  src/dllmain.cpp
  src/download.cpp
  src/error.cpp
  src/settings.cpp
  src/threads.cpp
  src/ui.cpp
  src/updatechecker.cpp
  src/updatedownloader.cpp
  src/winsparkle.rc)

add_library(WinSparkle ${SOURCES})

target_link_libraries(WinSparkle
  PRIVATE ${EXPAT_LIBRARIES} ${wxWidgets_LIBRARIES} wininet version)

target_compile_definitions(WinSparkle
  PRIVATE
  _CRT_SECURE_NO_WARNINGS XML_STATIC BUILDING_WIN_SPARKLE _UNICODE UNICODE
  _USRDLL WINSPARKLE_EXPORTS)

target_include_directories(WinSparkle
  PRIVATE ${EXPAT_INCLUDE_DIRS}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>
    ${wxWidgets_INCLUDE_DIRS})

set(CMAKE_CONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/WinSparkle")

include(CMakePackageConfigHelpers)

write_basic_package_version_file(WinSparkleConfigVersion.cmake
  VERSION ${WinSparkle_VERSION}
  COMPATIBILITY SameMajorVersion)

configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/WinSparkleConfig.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/to-be-installed/WinSparkleConfig.cmake"
  INSTALL_DESTINATION ${CMAKE_CONFIG_INSTALL_DIR})

install(
  FILES "${CMAKE_CURRENT_BINARY_DIR}/WinSparkleConfigVersion.cmake"
  DESTINATION ${CMAKE_CONFIG_INSTALL_DIR})

install(
  FILES "${CMAKE_CURRENT_BINARY_DIR}/to-be-installed/WinSparkleConfig.cmake"
  DESTINATION ${CMAKE_CONFIG_INSTALL_DIR})

install(FILES ${HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(TARGETS WinSparkle
  EXPORT WinSparkleTargets
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

# generate and install WinSparkleTargets.cmake
install(EXPORT WinSparkleTargets
  NAMESPACE WinSparkle:: DESTINATION ${CMAKE_CONFIG_INSTALL_DIR})

## During build, register package directly from build tree
export(TARGETS WinSparkle NAMESPACE WinSparkle:: FILE WinSparkleConfig.cmake)
export(PACKAGE WinSparkle)
