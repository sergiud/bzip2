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

set (CPACK_COMPONENTS_ALL Runtime)
set (CPACK_COMPONENT_APPLICATIONS_GROUP Runtime)
set (CPACK_COMPONENT_LIBRARIES_GROUP Development)
set (CPACK_COMPONENT_HEADERS_GROUP Development)
set (CPACK_PACKAGE_NAME bzip2)
set (CPACK_PACKAGE_DESCRIPTION_SUMMARY
  "A high-quality data compression program")
set (CPACK_PACKAGE_VERSION_MAJOR ${MAJOR_VERSION})
set (CPACK_PACKAGE_VERSION_MINOR ${MINOR_VERSION})
set (CPACK_PACKAGE_VERSION_PATCH ${VERSION_PATCH})
set (CPACK_PACKAGE_VERSION ${VERSION})
set (CPACK_PACKAGE_CONTACT "julian@bzip.org")

include (CMakePackageConfigHelpers)
include (CPack)
include (CTest)
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

install (TARGETS libz2 bzip2 bzip2recover
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

install (FILES bzip2.1 bzgrep.1 bzmore.1 bzdiff.1
  DESTINATION ${CMAKE_INSTALL_MANDIR}/man1
  COMPONENT Runtime)

install (PROGRAMS $<TARGET_FILE:bzip2>
  DESTINATION ${CMAKE_INSTALL_BINDIR}
  COMPONENT Runtime
  RENAME bunzip2${CMAKE_EXECUTABLE_SUFFIX}
)

install (PROGRAMS $<TARGET_FILE:bzip2>
  DESTINATION ${CMAKE_INSTALL_BINDIR}
  COMPONENT Runtime
  RENAME bzcat${CMAKE_EXECUTABLE_SUFFIX}
)

install (PROGRAMS bzdiff bzgrep bzmore
  DESTINATION ${CMAKE_INSTALL_BINDIR}
  COMPONENT Runtime
)

install (FILES LICENSE DESTINATION
  ${CMAKE_INSTALL_DATAROOTDIR}/licenses/bzip2
  COMPONENT Runtime)

if (WIN32)
  set (_COMPARE fc)
else (WIN32)
  set (_COMPARE cmp)
endif (WIN32)

add_test (NAME sample1 COMMAND ${CMAKE_COMMAND}
  -DCOMPARE=${_COMPARE}
  -DEXECUTABLE=$<TARGET_FILE:bzip2> -DARGS=-1
  -DIN=${CMAKE_CURRENT_SOURCE_DIR}/sample1.ref
  -DOUT=${CMAKE_CURRENT_SOURCE_DIR}/sample1.bz2
  -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/runtest.cmake)

add_test (NAME sample2 COMMAND ${CMAKE_COMMAND}
  -DCOMPARE=${_COMPARE}
  -DEXECUTABLE=$<TARGET_FILE:bzip2> -DARGS=-2
  -DIN=${CMAKE_CURRENT_SOURCE_DIR}/sample2.ref
  -DOUT=${CMAKE_CURRENT_SOURCE_DIR}/sample2.bz2
  -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/runtest.cmake)

add_test (NAME sample3 COMMAND ${CMAKE_COMMAND}
  -DCOMPARE=${_COMPARE}
  -DEXECUTABLE=$<TARGET_FILE:bzip2> -DARGS=-3
  -DIN=${CMAKE_CURRENT_SOURCE_DIR}/sample3.ref
  -DOUT=${CMAKE_CURRENT_SOURCE_DIR}/sample3.bz2
  -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/runtest.cmake)
