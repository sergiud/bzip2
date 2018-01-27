# Copyright (c) 2018 Sergiu Deitsch
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
# FITNESS FOR A PARTMETISLAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

cmake_minimum_required (VERSION 3.1)
project (bzip2 C)

set (MAJOR_VERSION 1)
set (MINOR_VERSION 0)
set (VERSION_PATCH 6)
set (VERSION 1.0.6)

include (CMakePackageConfigHelpers)
include (GNUInstallDirs)

set (CMAKE_DEBUG_POSTFIX d)

add_library (libz2
  blocksort.c
  bzlib.c
  bzlib.h
  bzlib_private.h
  compress.c
  crctable.c
  decompress.c
  huffman.c
  randtable.c
)

if (BUILD_SHARED_LIBS)
  if (WIN32)
    target_sources (libz2 PRIVATE libbz2.def)
  endif (WIN32)
endif (BUILD_SHARED_LIBS)

add_library (BZip2::BZip2 ALIAS libz2)

add_executable (bzip2
  bzip2.c
)

target_link_libraries (bzip2 PRIVATE BZip2::BZip2)

add_executable (bzip2recover
  bzip2recover.c
)

target_link_libraries (bzip2recover PRIVATE BZip2::BZip2)

set_target_properties (libz2 PROPERTIES EXPORT_NAME BZip2::BZip2)
set_target_properties (libz2 PROPERTIES OUTPUT_NAME bz2)
set_target_properties (libz2 PROPERTIES PUBLIC_HEADER bzlib.h)
set_target_properties (libz2 PROPERTIES SOVERSION ${VERSION})
set_target_properties (libz2 PROPERTIES VERSION ${MAJOR_VERSION})

install (TARGETS libz2
  EXPORT bzip2-targets
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} COMPONENT Development
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT Runtime
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Development
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Development)

configure_package_config_file (bzip2-config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/bzip2-config.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake)
write_basic_package_version_file (bzip2-config-version.cmake
  VERSION ${VERSION} COMPATIBILITY SameMajorVersion)

export (TARGETS libz2 NAMESPACE BZip2:: FILE bzip2-targets.cmake)
export (PACKAGE bzip2)

install (EXPORT bzip2-targets NAMESPACE BZip2::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake)
